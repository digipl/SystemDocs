## Descripción general libreria Vault

La Red MaidSafe consiste en procesadores de software (nodos), conocidos como Vaults. Estas Vaults realizan muchas funciones en la red y estos sus componentes funcionales se conocen como "personas". La red subyacente, cuando se une con [MaidSafe-Routing] (https://github.com/maidsafe/MaidSafe-Routing/wiki), es una red XOR y como un tal, un nodo puede expresar la cercanía o responsabilidad repecto a cualquier otro nodo o elemento en la red, si este nodo se encuentra lo suficientemente próximo. En este resumen, la frase ** ** NAE (Net Address Element) se utiliza para referenciar a cualquier cosa con una dirección de red incluyendo datos.

Los vaults cuentan con el [MaidSafe-Routing](https://github.com/maidsafe/MaidSafe-Routing/wiki) para calcular las responsabilidades sobre los NAE a través de [llamdas API](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/routing_api.h) como por ejemplo
```C++
GroupRangeStatus IsNodeIdInGroupRange(const NodeId& group_id, const NodeId& node_id) const;
GroupRangeStatus IsNodeIdInGroupRange(const NodeId& group_id) const;
bool EstimateInGroup(const NodeId& sender_id, const NodeId& info_id) const;
```
Estas llamadas nos permiten calcular la red desde el punto de vista de cualquier NAE del que podemos ser responsables. Hay que insistir en que la única manera de determinar la responsabilidad sobre un NAE es ver a la red desde el punto de vista del propio NAE. Si clasificamos el vector de nodos que conocemos y sus nodos cercanos (referido como la matriz del grupo) y no aparecemos en los primeros K nodos (contando la replicación) entonces no somos responsables del NAE. Este es un tema fundamental y la importancia de esto no puede enfatizarse lo suficiente.

En tanto en cuanto la red es muy fluida en términos de rotación, Los Vaults deben ser capaces de medir e informar sobre cada Vault individual y esto es importante para asegurarse que todos las "persona" de cualquier Vault están realizando sus tareas para el NAE del que son responsables. Para facilitar esto, usamos la característica de Routing [Matrix change (Churn Event)](https://github.com/maidsafe/MaidSafe-Routing/wiki/Documentation#matrix-change-churn-event). En el caso de una rotación, alrededor de un segmento de red determinado, se crea un objeto de cambio de matriz por el Routing y se transmite al Vault. Este objeto contiene la lista de los nodos viejos y nuevos de la matriz del grupo. Basándose en esta información, esto proporciona una función de ayuda para obtener cierta información relacionada con cualquier NAE dado.
Si el nodo que recibe el evento churn está entre los k nodos más cercanos al proporcionado por el NAE. En caso afirmativo, el nuevo nodo(s) necesitará información relacionada con el NAE proporcionado. Si no es así, se eliminará cualquier información almacenada relacionada con el NAE dado.

La rotación, la duplicación de datos y garantizar que todos los miembros de un grupo están de acuerdo es gestionado por una combinación de sincronización, el acumulador y los mensajes del grupo. Se trata de un complejo conjunto de normas que requiere considerable atención en los casos difíciles.

# Terminos y convenciones usados
**_Por favor, tener en cuenta que estos términos se usan ampliamente en este documento y su lectura será muy dificil sin que sean totalmente comprendidos._**

* Sy - [Sync](#sync), esta función sincroniza datos entre nodos del mismo grupo (conectados por proximidad).
* Sr - [Sync](#sync), esta función sincroniza los resultados de los mensajes entre nodos del mismo grupo (conectados por porximidad).
* So - Enviar On, esta función envía el mensaje a la siguiente persona.
* Ac - Accumulate, esta función acumula mensajes de personas previas (las personas que mandan el mensaje a las personas actuales).
* Fw - Firewall, esta función asegura que a los mensajes duplicados se les impida progresar y conteste a dichos mensajes con la una respuesta calculada (como success:error:synchronising etc. puede incluir un mensaje que contienen datos)
* Uf - Update Firewall, esta función actualizará el firewall con el código de retorno recién calculado y un mensaje opcional.
* **Bold** representa Indirect Network Addressable Entities INAE
* _Italic_ representa Direct Network Addressable Entities DNAE
* ->>>> representa un mensaje de grupo (sin retorno)
* -> representa un mensaje directo (sin retorno)
* =>>>> representa un mensaje de grupo (con retorno)
* => representa un mensaje directo (con retorno)

### Identidades Maidsafe
La red MaidSafe consiste en muchos tipos de datos así como muchos tipos de identidad. Estas identidades se describen en [MaidSafe-Passport](https://github.com/maidsafe/MaidSafe-Passport/wiki). Las "Personas" están particularmente enfocados en 4 de estas identidades y asegura que las entidades correspondientes cumplen los requisitos impuestos por estas identidades.

* MAID (MaidSafe Anonymous ID) - La identidad cliente para manipular datos no estructurados. Un cliente solo puede tener una.
* PMID (Proxy Maidsafe ID) - La identidad cliente para guardar de forma segura datos no estructurados. Un cliente puede tener varios.
* MPID (Maidsafe Public ID)- la identidad cliente que permite id públicas (nombre públicos como nombres de persona o alias) para comunicarse de forma segura. Un cliente puede tener varios de estas.
* MSID (Maidsafe Share ID) - la identidad cliente para gestionar grupos de MPID que comparten datos privados (structurados y no estructurados). Un cliente puede tener varios de estas. Este tipo de identidad no tiene soporte NAE por motivos de seguridad.

### Vault "Personas"
Las "personas" empleadas por un Vault de MaidSafe entran en dos categorias distintas, a saber, Data Management y Node Management. Estas categorias define las acciones, la agrupación de los nodos y su conocimiento de los alrededores y mensajes.
* Data Management nodes son responsables de datos NAE específicos, por lo general punteros a los datos.
* Node Management personas son responsables por una entidad "persona" (nodo) y gestiona las acciones y peticiones de dicha entidad.

Los mensajes entrantes son [demultiplexed](https://github.com/maidsafe/MaidSafe-Vault/blob/master/src/maidsafe/vault/demultiplexer.h) para identificar a la persona y el tipo de dato que se envía. Los mensajes son dirigidos a esa persona.

# Gestión de datos Generalizado
___
###Datos no versionados

Almacenar un trozo de datos en la red requiere de la coordinación y la colaboración de varias Personas. Un chunk de datos a partir de datos de usuario se crea en MaidNode y es administrado por DataManagers. DataManagers asegura que varias copias de los datos se almacenan y están disponible en diferentes PmidNodes todo el tiempo. El siguiente diagrama muestra un flujo muy básico del proceso de almacenamiento de datos en la red.

![UnversionedData](./img/UnversionedData.png)


| Data Management| Node Management   | Nodes     |
| ---------------|:-----------------:| ---------:|
| [**DataManager**](#datamanager)    | [**MaidManager**](#MaidManager)| [_MaidNode_](#maidnode)  |
| [**VersionManager**](#versionmanager) | [**PmidManager**](#PmidManager)| [_PmidNode_](#pmidnode)  |

####`Put<Data>`
|  _MaidNode_ [So]=>>>>|[Ac, Fw] **MaidManager** [So, Sy]->>>> | [Ac, Fw] **DataManager** [So,Sy]->>>> * 4|[Ac, Fw] **PmidManager** [So, Sy]=>| [Ac, Fw]_PmidNode_ |

####`Put<Unique>` _(a specialisation)_
|  _MaidNode_ [So]=>>>>|[Ac, Fw] **MaidManager** [So, Sy]=>>>> | [Ac, Fw] **DataManager** [So,Sy]->>>> * 4|[Ac, Fw] **PmidManager** [So, Sy]=>| [Ac, Fw]_PmidNode_ |

####`Delete<Data>`
|  _MaidNode_ [So]=>>>> |[Ac, Fw] **MaidManager** [So, Sy]->>>> | [Ac, Fw] **DataManager** [So, Sy]->>>> * 4| [Ac, Fw] **PmidManager** [So, Sy] =>|[Ac, Fw]_PmidNode_ |

####`Get<Data>`
|  _MaidNode_ [So]=>>>> | [Ac, Fw] **DataManager** [So]=> * (# of live PmidNodes)|[Ac, Fw] _PmidNode_ |

##Anatomía de un gestor "Persona" con datos no versionados
Todos las "Personas" siguen un patrón muy similar, acumulan, firewall, sujetan y envian los mensajes y sincronizan sus resultados, con lo cual el acuerdo que registran es el resultado de una operación en una base de datos. Los resultados o valores almacenados por operación difieren según las diferentes "Personas", pero poco más.

En la práctica, se supone, a priori, que los nodos de la red son indignos de confianza, sin embargo, la cooperación/colaboración de los nodos son esenciales para la estabilidad y la salud de la red. Para asegurar que se cumplan las dos condiciones, al recibir de un INAE, el grupo de nodos, bajo el disfraz de una "Persona" específica, que rodea el elemento objetivo se convierten en responsable de la validación de la solicitud. La sincronización entre el grupo permite a cada uno acumular la de otro dando como resultado un número predeterminado suficiente para satisfacer la solicitud. El proceso, inherentemente, detecta la manipulación del mensaje o los aspirantes a hackers y, con un mecanismo de clasificación, los delincuentes son forzados al aislamiento de la red.

Cuando se recibe desde un DNAE cada "Persona" se acumula en un solo mensaje. Esto es posible ya que existe una conexión entre los nodos, y tanto el enrutamiento como el RUDP han confirmado la identidad de otros nodos de una manera criptográficamente segura.

La variedad de mensajes de "no datos" se gestionan de manera diferente por cada "Persona".

___

###Datos versionados
Almacenar un trozo de datos versionado en la red, como una versión del directorio, es gestionada por diferentes "Personas" basados en la configuración de privacidad del directorio definido por el usuario. Las configuraciones de privacidad incluyen privadas, públicas y datos versionados compartidos. Diferentes identidades se emplean en base a la configuración de privacidad asociados a los directorios. Aunque se crea diferentes versiones de datos y se envía por diferentes "Personas", todo ello se gestiona y se almacena en el VersionManager.
El siguiente diagrama muestra un flujo muy básico del proceso de almacenamiento de datos versionados en la red, sobre la base de diferentes configuraciones de privacidad.

![VersionedData](./img/VersionedData.png)


####`PutVersion<Data>`
`Put <Data>`|  _MaidNode_ [So]=>>>>|[Ac, Fw] **MaidManager** [So, Sy]->>>> | [Ac, Fw] **DataManager** [So,Sy]->>>> * 4|[Ac, Fw] **PmidManager** [So, Sy]=>| [Ac, Fw]_PmidNode_ | +
* |  _MaidNode_ [So]=>>>>|[Ac, Fw] **MaidManager** [So, Sy]->>>> | [Ac, Fw, Sy, Uf] **VersionManager** |


####`GetVersion<Data>`
* |  _MaidNode_ [So]=>>>> |  [Ac, Fw] **VersionManager** | +
* `Get<Data>` |  _MaidNode_ [So]=>>>> | [Ac, Fw] **DataManager** [So]=> * (# of live PmidNodes)|[Ac, Fw] _PmidNode_ |


#####`DeleteVersionTillFork<Data>`
* `Delete<Data>` |  _MaidNode_ [So]=>>>> |[Ac, Fw] **MaidManager** [So, Sy]->>>> | [Ac, Fw] **DataManager** [So, Sy]->>>> * 4| [Ac, Fw] **PmidManager** [So, Sy] =>|[Ac, Fw]_PmidNode_ | +
* |  _MaidNode_ [So]=>>>> | [Ac, Fw, Sy, Uf] **VersionManager** |

##Anatomía de un gestor "Persona" con datos versionados

Todos estos VM "personas" deben estar de acuerdo sobre la consulta en el momento que se recibe. Para lograr esto, los VM "Personas" mantienen la acumulación de los mensajes recibidos de las "Personas" anteriores. Una vez que se alcanza el límite de espera, la petición se sincroniza con otros VM "Personas" en el grupo. Cualquier VM "Persona" que recibe el número esperado de mensajes de sincronización intenta realizar la solicitud. Al realizar la solicitud, continuará devolviendo el resultado de la acción al acumulador y llamando a los funtores de contestación asociados a las peticiones de los personajes anteriores. Una vez más, un `GET` simplemente intenta devolver la respuesta autocalculada de velocidad, pero todas las llamadas mutadas requieren de un acuerdo con todos los personajes VersionManager involucrados en la llamada. Esta es una diferencia fundamental con los administradores de datos no versionados. Esta diferencia se debe a estas "Personas" son la última llamada a la solicitud, y la sincronización se utiliza para el acuerdo en lugar de auto-calcular la respuesta y dejarlo al acuerdo del solicitante.


## Gestión de comunicación generalizada
| Node Management      | Node      |
| ----------------------|----------:|
| [**MpidManager**](#mpidmanager)    | [_MpidNode_](#mpidnode)  |



---
### "Personas" Data Management
Data management "Personas" son responsables de mantener vínculos entre claves NAE y las direcciones NAE que contiene el contenido de la clave en cuestión. Estos Vault "Personas" pueden ser considerados como responsables de los punteros a los datos, así como de la duración de los datos y la gestión de la replicación, con capacidad para resistir la rotación de la red.

###Data Manager<a id="datamanager"></a>
The **DataManager** "Persona" gestiona punteros a datos no versionados, como contenido de ficheros, claves de red como las definidas en [passport](https://github.com/maidsafe/MaidSafe-Passport/wiki) y cualquier otro elemento de datos estático.

### Estructura de Contenedor
Usa un único ManagerDb

* Key : data key + type
* Value : DataManagerValue object (serialised)

### Entrada de mensajes
* `Put<Data>` Este mensaje es recibido por un grupo de MaidManager (de K) y se acumula. Se comprueba la base de datos para ver si esa clave existe. Si el registro no existe, entonces cada uno de los Data Managers de este grupo, cercanos al NAE que se quiere guardar, selecciona un nodo conectado como PmidNode para guardarlo (si el mensaje tiene una pista del Data Holder entonces el más cercano de los gestores de datos de este NAE intenta almacenarlo allí). Para guardar un chunk, un mensaje `PUT<Data>`(key, content) se manda al PmidManagers responsable del PmidNode seleccionado.
* * | **MaidManager** ->>>> | [Ac, Fw]**DataManager**[So, Sy]->>>>|

* `Delete<Data>` este mensaje es recibido por un grupo de MaidManagers group (de K) y se acumula. Se comprueba la base de datos para ver si esa clave existe. Si existe, el contador decrece. Cuando el contador llega a cero (si alguna vez lo hace) entonces un Delete<data>(key) es enviado a todos los PmidManagers para borrar los datos.
* * | **MaidManager** ->>>> | [Ac, Fw] **DataManager** [So, Sy]->>>>|

* `Get<Data>` este mensake no se acumula, solo se envía. Este mensaje puede venir de cualquier "Persona", aunque no debe llegar a este personaje muy frecuentemente debido al almacenamiento en caché. Si este mensaje no llegue aquí entonces el DataManager envía un GET <Datos> directamente a un PmidNode y recupera los datos, enviando de nuevo a través de la reply_functor enviada por [routing](https://github.com/maidsafe/MaidSafe-Routing/wiki).
* * |  _MaidNode_ ->>>> | [Ac, Fw] **DataManager** | (firewall)
* `Node Status Change`
| **PmidManager** ->|[Ac, Fw] **DataManager** [So, Sy]->>>> |


### Salida de mensajes
* `Put<Data>` Cuando un elemento de datos se almacena por primera vez o una DataManager requiere añadir otro PmidNode para almacenar datos (debido a que el PmidNodes falla, se desconecta o pierde datos de otra manera) se realiza esta llamada NFS para almacenar los datos.
* * | **DataManager** [So, Sy, Uf]->>>> | **PmidManager** |

* `Delete<Data>` Cuando el recuento de suscripción se aproxima a cero o la PmidNode se considera que deja de ser responsable de un elemento de datos, se hace esta llamada NFS.
* * | **DataManager** [So, Sy, Uf]->>>> | **PmidManager** ->|

* `Get<Data>` En respuesta a recibir un Get<data> este nodo intentará recuperar los datos desde cualquier PmidNode vivo y enviará los datos al solicitante a través del Routing reply_functor proporcionada con la solicitud.
* * | **DataManager** => * live PmidNodes| _PmidNode_ |

### Data Integrity Checks
En respuesta a un evento de rotación, (detectado como un Node Status Change) el DataManager creará un trozo aleatorio de datos y lo enviará a cada PmidNode con una solicitud para añadir estos datos y hacer un hash de los nuevos datos y responder con el nuevo valor. Esto permite que el DataManager evalue que todos los PmidNodes tienen los mismos datos (no dañados o perdidos).
* * | **DataManager** [So, Sr]=>>>> * 4| **PmidManager** ->| _PmidNode_ |


### Version Manager<a id="versionmanager"></a>
The **VersionManager**, gestiona datos versionados. Esto, actualmente, incluye datos que puede ser definidos en [StructuredDataVersions](https://github.com/maidsafe/MaidSafe-Private/blob/master/include/maidsafe/data_types/structured_data_versions.h). Directorios de datos privados, directorios privados y directorios compartidos públicos se manejan actualmente en esta estructura. Esta "Persona" no se limita a las versiones requeridas y por consiguiente se puede utilizar para datos estructurados de muchos tipos con facilidad.

### estructura del contenedor

Usa un único ManagerDb

* Key : data key + type + Entity ID
* Value : StructuredDataVersions object (serialised)

### Messages In
* `PutVersion<Data>`
* * | **MaidManager** =>>>> | [Ac, Fw] **VersionManager** [Sy, Uf] |
* `DeleteBranchTillFork<Data>`
* * | **MaidManager** =>>>> | [Ac, Fw] **VersionManager** [Sy, Uf] |
* `Get<Data>`
* * |  _MaidNode_ =>>>> | [Ac, Fw] **VersionManager** |
* `GetBranch<Data>`
* * |  _MaidNode_ =>>>> |[Ac, Fw] **VersionManager** |

### Mensaje de salida
 [None] Todas las entradas son devoluciones de llamada

### Node Management "Personas"
Node management "Personas", gestiona entidades de "Personas" (nodos).

### Maid Manager<a id="MaidManager"></a>
La función **MaidManager** gestiona el MaidNode. Esto significa asegurarse que el cliente tiene una cuenta y al menos un PmidNode registrado.

### Estructura del contenedor
Usa una AccountDb

* Key : Hash of the data key (hashed to protect clients data)
* Value : Number of copies (int32) : total cost (int32)

### Estructura de cabecera

* Tamaño total de datos almacenados
* espacio disponible

### Entrada de mensajes
* `PUT<Data>` Un cliente puede almacenar un paquete MAID anónimo en la red que crea la cuenta. Entonces el MaidNode debe registrar un PmidNode y lo hace con un paquete de inscripción del Vault, que está firmado por el MAID con las claves privadas PMID (las cuales el cliente debe tener acceso). El MaidManager individual consultará el grupo PmidManager en el arranque, en un evento de rotación y cuando el cliente está bajo de recursos y no tiene suficiente espacio que garantizar al PmidNode. En caso de éxito, la MaidNode puede enviar los datos a la DataManager relevante y recuperar el costo a pagar. En cuanto se page el costo, la intención es que la entrada en la base de datos se sincronice con los otros MaidManagers. El MergePolicy en este caso se sumará la clave de la base de datos y/o incrementará el número de copias y el campo de coste total.

* * |  _MaidNode_ ->>>> | [Ac, Fw]**MaidManager** [So]|

* `Delete<Data>` El MaidManager buscará en el registro de cuentas del cliente MAID un PUT correspondiente de la clave de datos. En caso de éxito el MaidManager reducirá el número de trozos PUT de esa clave. Si varios trozos se almacenan en la misma clave el costo se reduce tomando un promedio del costo total pagado por esa clave. La eliminación siempre devuelve un mensaje de éxito por lo que el cliente sólo desperdicia su propio tiempo en tratar un ataque de esa naturaleza.
* * |  _MaidNode_ =>>>> | [Ac, Fw] **MaidManager** [So] |
* `PutVersion<Data>`
* * |  _MaidNode_ =>>>> | [Ac, Fw] **MaidManager** [So] |
* `DeleteBranchTillFork<Data>`
* * |  _MaidNode_ =>>>> | [Ac, Fw] **MaidManager** [So] |
* `Register/Unregister vault`
* * |  _MaidNode_ =>>>> | [Ac, Fw] **MaidManager** [So] |

### Messages Out
* `PUT<Data>`
* * | **MaidManager** [So] ->>>> | **DataManager** |
* `Delete<Data>`
* * | **MaidManager** [So] ->>>> | **DataManager** |
* `PutVersion<Data>`
* * | **MaidManager** [So] =>>>> | **VersionManager** |
* `DeleteBranchTillFork<Data>`
* * | **MaidManager** [So] =>>>> | **VersionManager** |
* `GetPMIDHealth`
* * | **MaidManager** [So] =>>>> | **PmidNode** |

### Pmid Manager<a id="PmidManager"></a>
La función PAH (Pmid Account Holder) gestiona al Pmid client. Esto significa asegurar que el cliente tiene una cuenta y un registro exacto de los elementos de datos realizada y ninguna perdida (versión inicial de rango).

### Estructura del contenedor

Una una AccountDb

* Key : Data key
* Value : Size (int32)

### Header Structure

* Tamaño total de datos almacenados
* Tamaño total de datos perdidos
* Espacio disponible


### Entrada de mensajes

* `Put<Data>` Guarda datos en PmidNode y actualiza el contador de almacenaje para ese PmidNode
* * | **DataManager** =>>>> | [Ac, Fw] **PmidManager** |
* `Delete<Data>` Manda un borrado a PmidNode and eleimina el contador de almacenaje de ese Pmidnode
* * | **DataManager** ->>>> | [Ac, Fw] **PmidManager** |
* `Get<Data>` Devuelve dato o error
* * | **DataManager** =>>>> | [Ac(1), Fw] **PmidManager** |
* `Lose<Data>` Enviar un borrado a PmidNode y añade uno al contador de perdida
* `GetPMIDHealth`
* * | **MaidManager** =>>>> | [ **PmidManager** | (no Fw or Ac, devuelve cada llamada)
* 
### Salida de mensajes

* `SendAccount` Se envía a _PmidNode_ cuando se une al grupo. Esto se detectará por el **PmidManager** a la recepción de un evento de rotación. 
La cuenta en este caso son todos los datos que _PmidNode_ debe haber guardado y no incluirá ningún dato borrado o perdido que ocurra mientras el _PmidNode_ esté off line.
* *   | **PmidManager** [So]-> | **PmidNode**


### Mpid Manager<a id="mpidmanager"></a>

The MPAH (Mpid Account Holder) function is to manage the Mpid client. This means ensuring the client has an account and a blacklist or whitelist if required by the client. This persona will also hold messages destined for an Mpid client who is currently off-line.

### Container Structure
Uses the AccountDb

* Key : MpidName + SenderMpidName + messageID
* Value : message contents

This database should only hold records while _MpidNode_ is off-line.

### Header Structure

* whitelist
* blacklist


### Messages In

* `Message`
* * | _MpidNode_ =>>>> | this**MpidManager** [So]|
* * | **MpidManager** =>>>> | this**MpidManager** [So]|
* `AddToWhiteList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `RemoveFromWhiteList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `WhiteList`
* * | _MpidNode_ =>>>> | this**MpidManager** [Sy]|(reply with whitelist)
* `AddToBlackList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `RemoveFromBlackList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `BlackList`
* * | _MpidNode_ =>>>> | this**MpidManager** [Sy]|(reply with blacklist)

### Messages Out
* `Message` On receipt of a message the lists are checked and the message is sent on. If the node is off-line then the message is stored locally, on receipt of a churn event the off-line list is checked for messages and these are sent to the client.
* * | this**MpidManager** [So] -> | _MpidNode_ | (may come from database)
* * | this**MpidManager** [So] =>>>> | **MpidManager** [Sy]| (live message from remote _MpidNode_)

## Entity personas
### Pmid Node<a id="pmidnode"></a>

### Messages In

* `Put<Data>`
* *   | **PmidManager** -> |[Ac, Fw] **PmidNode**
* `Delete<Data>`
* *   | **PmidManager** -> |[Ac, Fw] **PmidNode**
* `Get<Data>`
* *   | **DataManager** => |[Ac, Fw] **PmidNode**
* `SendAccount` This is sent to a _PmidNode_ when it rejoins the group. This will be detected by the **PmidManager** on reciept of a churn event. The account in this case is all data the _PmidNode_ should have stored and will not include any deleted or lost data that occurred whilst the _PmidNode_ was off line.
* *   | **PmidManager** -> |[Ac, Fw] **PmidNode**


### Messages Out

[none]

### Maid Node <a id="maidnode"></a>

### Messages In

[none]

### Messages Out

* `Put<Data>` Returns error_code and PmidHealth
* * |  _MaidNode_ [So]=>>>>| **MaidManager** |
* `Delete<Data>` Returns error_code
* * |  _MaidNode_ [So]=>>>>| **MaidManager** |
* `Get<Data>` Returns error_code and data (if success)
* * |  _MaidNode_ [So]=>>>>| **DataManager** |


### Mpid Node<a id="mpidnode"></a>

### Messages In

* `Message`
* * | _MpidNode_ =>>>> | this**MpidManager** [So]|
* * | **MpidManager** =>>>> | this**MpidManager** [So]|
* `AddToWhiteList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `RemoveFromWhiteList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `WhiteList`
* * | _MpidNode_ =>>>> | this**MpidManager** [Sy]|(reply with whitelist)
* `AddToBlackList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `RemoveFromBlackList`
* * | _MpidNode_ ->>>> | this**MpidManager** [Sy]|
* `BlackList`
* * | _MpidNode_ =>>>> | this**MpidManager** [Sy]|(reply with blacklist)

### Messages Out
[none]

# <a id="sync"></a>Synchronisation

The network is in a constant state of flux, with nodes appearing and disappearing continually. This forces a state change mechanism that can recognise which nodes are responsible for what data in real time and recognising churn events near a node. Using [MaidSafe-Routing](https://github.com/maidsafe/MaidSafe-Routing/wiki) the network can detect node changes in a very short time period. Routing also provides a mechanism that can guarantee with great precision which nodes are responsible for which NAE (including the node querying the network).

To resolve a data element, whether an `account transfer` or `unresolved action` then the node requires a majority of close nodes to agree on the element. As there can be multiple churn events during a transfer of such data, the node requires to handle this multiple churn event itself. To do this with each churn event the new node and old nodes are handled by checking if the old node had given a data element and replace this node_id with the new node id. If the data element did not contain the new node but should have (i.e. the new node would have been responsible for an unresolved element) then the new node_id is added as having sent the data. To make this a little simpler, all synchronise data has the current node added as having 'seen' the element, this helps with the majority size required. When a node adds itself to unresolved data in this manner it adds itself to the end of the container. This will prevent the node sending this element in a synchronise message.

When a node sends a synchronise message it sends a unique identifier and it's own id with the unresolved data. Each receiving node creates a container of the node_id and entry_id along with the unresolved data. When the size of this container is 0.5 * the close node size +1 then the data is resolved and written to the data store.

There are two types of data which require cumulative data (from a group) to resolve:

* `synchronise` This is data that has not yet been synchronised (unresolved), but this node has 'seen' itself. This data contains the action required to be taken with the attributes to take that action, this can be thought of as a function (message type) and parameters to be applied.
* `account transfer`, this type of data has been resolved and stored by the node. It is sent as the database value to be stored.

##Synchronise to Validate Actions

Synchronise is used to ensure all nodes in the same close group responsible for a NAE agree on the data related to that NAE prior to storing (or performing) that related data. Nodes achieve this by creating a container of data to be resolved. For such data an unresolved entry may be added to the container when
* the message is received from previous persona
* an unresolved entry is received from similar persona (in a group) for a target the node is responsible for

The unresolved entry associated with data becomes resolved once the number of unresolved entries received for that data (from previous and similar personas) reaches an expected number. A node having resolved an entry may take part in sending to other node(s) deemed responsible for the data, ONLY if the node has received the data from a previous persona.

Using routing to confirm responsibility a **manager node** will check all data held to ensure it is still responsible for the data (via the routing api). The node then checks which data the new node should have (if any). The data is sent to the new node as `unresolved data`.


This process continues and each time the data is sent a counter is incremented, when this counter reaches a parameter (initially 100) the data is deemed non-resolvable and removed. When data does resolve it is not removed from the list as there could be further nodes to be added to the peer container component (we resolve on a majority and expect a unanimous number of nodes to be added and do not wish to create a new unresolved entry). When there are a complete set of peers who have sent the data (unanimous agreement) then the unresolved entry is removed.

### Account Transfer

Manager personas storing the information for nodes/data usually deliver their functionality based on cumulative decision making, where a group of close nodes co-operate to realise the desired functionality. Obviously, the correct decisions to a large extent depend on how well the group members are synchronised. The dynamic nature of the network, where a node in a group may leave or join, necessitates the demand for implementing efficient mechanisms to ensure each member of any group is synchronised with other group members and hold the most up to date information for any node/data it is responsible for.

As a new node enters a group, it is sent account transfer messages, it's own address is added to an unresolved account transfer record (it is the close group -1 node) to ensure the group size calculations do not change although the group size cannot equal the system group size as this node is replacing a now missing node from that group.

The account transfer record is a database entry and requires agreement and then writing direct to the database.

In addition unresolved data is provided to this node and again this node add's it's own id to this data, however, it does not synchronise this data back to the close group as all nodes have taken care of this as they picked up this new node in the churn event. The unresolved data should now resolve as per the above process.


### Accumulator

When receiving messages from a group of nodes, the network requires authority for the message to be validated. This is done by accumulating messages as they arrive after checking they come from a group that is responsible for sending the data (validated via routing api) and accumulating the messages until we receive at least close group -1 messages if the message comes from a **manager node** group. Messages that come from DNAE nodes accumulate on that single message. These nodes are verifiable as they are connected and will have performed an action to validate they are authorised to send that message.

### Firewall

When a message has been accumulated it is added to the firewall. This firewall will check any incoming messages and reply with the current error_code (and message if required by that message type). The error_code may be `synchronising` in cases where the accumulating node has accumulated enough messages to ensure they are valid, but cannot make a decision yet on the 'answer' to the message (it may require to synchronise or even send a message to another persona to retrieve further info).

### Caching

Caching is a very important facet of distributed networks. Cache data is likely to be stored in memory as opposed to disk. Currently two caching mechanisms have been defined that vaults aimed to stick with. The following sections present a general discussion, for the detailed implementation description, please have a look at [Caching](https://github.com/maidsafe/MaidSafe-Vault/wiki/Caching) .

### Opportunistic Caching

Opportunistic caching is always a first in first out (FIFO) mechanism. In response to a `Get` each node in the chain will add any cacheable data (such as self checking or immutable) to it's FIFO. This has many advantages:

* Multiple requests for the same data will speed up automatically.
* Denial of service attacks become extremely difficult as the data simply surrounds the requester of that data if continually asked for.
* Important data is faster when it's important and slows downs otherwise, making good use of network resources.
* Close nodes can become temporary stores of data very easily, allowing many advantages to system designers.
* If for instance a web site were located in a public share then that site would increase in speed with more viewers, this is a more logical approach than today's web sites which can run the risk of overload and crash.

### Deterministic Caching

This type of caching is not yet implemented.

Deterministic caching allows the network to effectively expand the number of nodes holding data and pointers to data when the data is accessed by a large number of nodes or users. This allows the network to cope with extremely popular data sets that may represent a microblogging technology that has extremely popular accounts.


In addition to this type of caching the network allows subscription lists, where identities can register to be alerted when a change happens that they wish to track, such as a microblogging site or a popular web site for news etc.

### NFS Policies

The API to the vault network is via Nfs policies and these are found in the [nfs library](https://github.com/maidsafe/MaidSafe-Network-Filesystem).

### Ranking

Misbehaving nodes require to be recognised by the network and acted on, likewise very helpful and well behaved nodes should command greater respect from other nodes. To achieve this a ranking mechanism is employed. This ranking mechanism in the vault network currently handles nodes that lose or otherwise corrupt data. This can be for many reasons such as:

* Poor bandwidth capability
* Slow cpu
* Infrequent availability (machine mostly off line for any reason)
* Hard drive corruption
* Hard drive being tampered with (users deleting system data)
* And many more

Rather than measuring each element, which would be very expensive and complex, the network instead measures the amount of data a node can store and compares this with the data it loses. These figures together allow a calculation on the available space and it's worthiness, with a high rate of lost to stored making the vault itself less valuable.
