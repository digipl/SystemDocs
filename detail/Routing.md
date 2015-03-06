## Descripción general librería Routing

El Routing es una librería de Tabla de Hash Distribuida basada en tablas de enrutamiento similares a [Kademlia] (http://es.wikipedia.org/wiki/Kademlia). El Routing especifica la estructura de la red y determina el camino entre el par de nodos en la red mediante el uso de la información local en cada nodo intermedio.
Cada nodo de enrutamiento local almacena información acerca de los nodos a los que está directamente conectado. Por otra parte, cada nodo tiene un conocimiento parcial de la información local de sus nodos cercanos (ID de espacio vecino).
La información guardada en cada nodo, contribuye a la infraestructura de envío de mensajes de enrutamiento. Intercambiar información de ruta supone, habitualmente, atravesar un número indeterminado de nodos. La comunicación, entre cada par de nodos, se logra empleando [MaidSafe-RUDP](https://github.com/maidsafe/MaidSafe-RUDP/wiki). Un mecanismo de reconocimiento/retransmisión se proporciona, tambien, para asegurar la entrega fiable de mensajes.
Su capacidad para ofrecer una plataforma eficiente, para el intercambio de mensajes entre pares, lo hace ideal como componente de enrutamiento de cualquier sistema P2P.


El Routing ofrece las siguientes características:
* Unión a la red
* NAT transversal
* Modos de operación
* Mensajería fiable directa y de grupo
* Evaluación de proximidad
* Identificación de nodo a través de PKI proporcionada por MaidSafe-Passport
* Mecanismo de almacenamiento en caché
* Manejo Churn
* Tabla de enrutamiento y el grupo matriz

La interfaz de la librería está disponible en los siguientes archivos:
* [routing_api.h](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/routing_api.h) - esta es la API principal y se discute en más detalle más adelante
* [api_config.h](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/api_config.h) - 
proporciona declaración de tipos que se puede usarse al utilizar la librería
* [node_info.h](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/node_info.h) - proporciona información acerca de un nodo
* [parameters.h](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/parameters.h) - proporciona las variables de configuración de la librería
* [return_codes.h](https://github.com/maidsafe/MaidSafe-Routing/blob/master/include/maidsafe/routing/return_codes.h)

### Sinopsis

### Unirse a la red
Un nodo puede unirse a la red como
1. un nodo normal cuando la red ya exista,
2. como un nodo de Estado Cero, cuando el nodo es uno de los dos primeros nodos que crean la red.

####Unirse como un nodo normal

Para unirse a una red existente, un nodo debe estar provisto de una lista de puntos de conexión que son una serie de nodos que ya forman parte de la red. El nodo intenta conectarse a cualquier punto de conexión de la lista. Una vez que un intento de conexión tiene éxito, el nodo que se une localiza y se conecta a su nodo más cercano [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc). Conectarse a su nodo más cercano [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) es vital para el funcionamiento fiable de un nodo. El resto de la tabla de enrutamiento está poblado principalmente por nodos al azar para permitir el acceso uniforme a diferentes partes de la red.

####Unirse como un nodo de Estado Cero

El ajuste inicial de la red, que se denomina Estado Cero, implica dos nodos. Estos dos nodos se ofrecen con los puntos finales de cada uno y conectándose uno a otro forma la red. Los nodos de Estado Cero son idénticos a otros nodos de la red y pueden abandonar/unirse a la red de forma similar al resto de los nodos.

###NAT Traversal

Un objetivo del Routing ha sido permitir la comunicación entre cada par de nodos en la red, independientemente de sus ajustes de configuración de red. El Routing en combinación con [MaidSafe-RUDP](https://github.com/maidsafe/MaidSafe-RUDP/wiki) ejecuta [hole punching](http://www.brynosaurus.com/pub/net/p2pnat/) para permitir la comunicación directa entre cada par de nodos.
El hole punching se puede conseguir siempre y cuando los dos nodos no estén detrás de routers simétricos. Si ambos nodos están detrás de routers simétricos, el Routing logra la comunicación entre los dos nodos gracias a la selección de un tercer nodo que actúa como proxy entre ambos nodos situados detrás de routers simétricos.


###Modos de Operación
####No-cliente

Un nodo no-cliente es un nodo de enrutamiento completo que contribuye a la operación y mantenimiento de la red. Los nodos no-clientes son parte del DHT y son parte activa en la toma de decisiones de enrutamiento.
Un nodo no-cliente puede :
* Enviar solicitudes a cualquier los nodos que no son clientes
* Enviar solicitudes a nodos cliente conectados
* Recibir las solicitudes entrantes desde cualquier nodo de la red

####Cliente

En contraste con los nodos que son no-clientes, un nodo cliente no contribuye a la infraestructura de enrutamiento de red.
Los nodos cliente son nodos de enrutamiento ligeros que usan los mínimos recursos de la red.
Estos nodos tendrán acceso a toda la red mediante el establecimiento de la conexión  [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) con los nodos cercanos a su propia ID.
Un nodo cliente puede :
* Enviar solicitudes a cualquier nodo no-cliente
* Enviar solicitudes a los clientes con la misma ID
* Recibir las solicitudes entrantes sólo de nodos no-clientes conectados

###Mensajería fiable directa y de grupo

Routing ofrece dos tipos de comunicación; directa y de grupo.

####Mensajería directa

En la mensajería directa el destino es un nodo conocido en la red y es el destinatario final del mensaje.

####Messajería de grupo

Un grupo se compone de un número de nodos que están próximos a una cierta ID. El número de miembros del grupo
se define por [Parameters::node_group_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc).

En la comunicación de grupo, el destino del mensaje, es el grupo de nodos que más cerca está del destino. Un nodo que comparta ID con el destino no será parte de una comunicación en grupo.

###Routing Table and Group Matrix
Routing table is the main component of the routing library. Routing table stores information about a number of nodes in the network in order to perform routing decisions to deliver messages. Each entry of the routing table corresponds to a direct connection from the routing node to that node.
Each node in the network has an ID of 512 bits. The distance between each pair of nodes in the network is calculated by XORing the pair.
A reliable and efficient routing operation in the network requires that each node i) to be aware of the nodes in its close proximity and, ii) to have access to different parts of the network. Routing table stores information about:
* [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest nodes
* Randomly chosen nodes in the network
* Any other node which finds the current node as one of its [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest nodes

The above information stored in routing table is enough in the majority of cases to enable correct decision makings. However, it might be found insufficient to make accurate decisions when two nodes are having different views regarding closeness to each other or another node. To handle these situations, the routing table is equipped with a group matrix.
The idea behind group matrix is to provide nodes with more knowledge of the area of the network and where they reside. This is realised by making each node partially aware of nodes in the routing table of its closest neighbours. A group matrix of a node contains:
* (i) The node’s [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest nodes
* (ii) Other nodes considering the node as one of their [Parameters::closest_nodes_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest
* (iii) Closest nodes of each entry in (i) & (ii)

Up to date information in the routing table and group matrix are necessary to make correct decisions. Therefore, routing offers some services; to quickly reflect any updates in the routing table of a node, and  the routing table or group matrix of any nodes which should be made aware of the updates.


###Proximity Evaluation

Most operations in the network are performed on the group of nodes which are in closest proximity of a given ID. Routing table along with group matrix, facilitates nodes with sufficient knowledge of the part of network (where they belong) to accurately identify other peer nodes who share the same group IDs.
The node which is part of a group will always be aware of other nodes in the group. Based on this knowledge, routing api provides methods to work out:
* If a node is closest to a given ID
* If a node is part of a given group
* New and old group members nodes after a churn event

Based on average network distance/population, it also provides methods to estimate if a node is close enough to a given ID.

###Churn Handling

In a P2P network, joining and leaving are common events. Peer turnover, often referred to as churn rate, is efficiently handled by the routing library. For example, a node joining or leaving the network is reflected in the routing tables of its close nodes within a few seconds.

###Matrix Change (Churn Event)

In the routing network, data is usually stored at a logical group ID. This means that data is stored at the nodes which are among the first [Parameters::node_group_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest to a given group ID.
Any churn in the network may result in moving a node near or far from a group ID in that segment of the network. This means that many of the logical groups reconfigure by having new members in the group and losing some old members of the group.

In event of:

1. Node(s) disappearing from a logical group : New node(s) will become closer to the group ID and will become among [Parameters::node_group_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest to the group ID. After this, all group messages sent to the group ID, will be received by new node(s) as well. It is the responsibility of other remaining nodes of that group to quickly replicate data to the new node(s).
2. Node(s) appearing in the logical group : Some of the group member nodes will move far from the group ID and will not remain under [Parameters::node_group_size](https://github.com/maidsafe/MaidSafe-Routing/blob/master/src/maidsafe/routing/parameters.cc) closest to the group ID. Further to this, these Node(s) will not receive any group message destined to the group ID. These Node(s) should delete data which they are no longer responsible for.


To reliably keep data alive and accessible in this dynamic group, all responsible nodes should replicate data as soon as they become aware of the network churn. Routing provides MatrixChange class to detect such churn quickly and to reliably workout if data replication is required for any new node appearing in the group. Nodes close to a segment of network will notice churn event by observing a change in their group matrix. If group matrix changes on any churn, routing creates MatrixChange object, containing a list of new and old group matrix nodes. This class provides helper function to evaluate any given group ID to work out if node receiving the churn event:

* Is still part of the group and responsible for data stored at the group.
* Is not part of the group any more and needs to delete stored data related to the group.
* Needs to replicate data to a new node.

```C++
  CheckHoldersResult CheckHolders(const NodeId& target) const;
```

CheckHoldersResult provides list of the new and old holders of the group and proximity_status of the node receiving the churn event wrt the given group ID.


```C++
struct CheckHoldersResult {
  std::vector<NodeId> new_holders;  // New holders = All 4 New holders - All 4 Old holders
  std::vector<NodeId> old_holders;  // Old holders = All 4 Old holder ∩ All Lost nodes
  routing::GroupRangeStatus proximity_status;
};
```




###Caching Mechanism

To enable fast access to popular contents, routing offers some services to allow caching data on intermediate nodes or to read cached data from intermediate nodes if data is available. The functionality is simply achievable by setting a flag in the routing message. If the type of message is a request, the cache in each intermediate node to the final destination is checked for data. If data is available in cache, the intermediate node will serve the request and a reply is sent to the sender. If the type of message is a response, data is stored at each intermediate node from destination to source.


###Callbacks

Offering an efficient platform to exchange messages between peers makes routing an ideal communication component of any P2P system. To allow simple and loosely coupled utilisation of the routing components, routing provides a number of callback functions to the host components. Some of these callbacks are mandatory and be provided by the host application. Other callbacks are additional features and are optional for host application.

Routing's callback functions:

* **MessageReceivedFunctor** : Is called when a a node-level request message is received. Node-level request message is an incoming message destined for the application layer using the routing node.

* **NetworkStatusFunctor** : Is called when a new connection is established or a connection drop happens. It show the percentage health in terms of node's connection to other valid peer nodes in the network or alternatively can be referred to as routing table health.

* **MatrixChangedFunctor** : Any churn event, resulting in change to nodes's group matrix triggers a call to this functor.

* **RequestPublicKeyFunctor** : As a part of the connection process to a non-client peer, routing needs to be provided with the public key of the peer. This is achieved by PKI infrastructure. The callback provides another callback (GivePublicKeyFunctor) which should be called with the valid public key of the connecting non-client peer

* **HaveCacheDataFunctor** & **StoreCacheDataFunctor** : Are called to look for cached data or store data in cache of an intermediate node

* **NewBootstrapEndpointFunctor** : Is called whenever a routing node connects to a peer whose endpoints are capable of bootstrapping a node. Since the network is very dynamic, this is important information for reconnecting to the routing network. A list of these endpoints must be supplied to routing to reconnect to the network.

