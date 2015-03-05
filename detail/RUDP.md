## Descripción general librería RUDP

RUDP (Reliable UDP) implementa pseudo-conexiones que utilizan UDP para lograr muchos de los beneficios de un protocolo de conexión como TCP, pero permitiendo algo crucial como permitir [NAT traversal](https://en.wikipedia.org/wiki/Network_address_translation#Type_of_NAT_and_NAT_traversal.2C_role_of_port_preservation_for_TCP) algo que TCP no puede hacer. Además, todos los datos se cifran punto a punto utilizando un mecanismo seguro y verificable de intercambio de clave pública RSA. Esto forma parte del [PKI](http://en.wikipedia.org/wiki/Public-key_infrastructure) proporcionado por la red SAFE.

La interfaz de la librería está disponible en los siguientes archivos:

* [managed_connections.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/managed_connections.h) - es el API principal y se analiza en más detalle a continuación
* [parameters.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/parameters.h) - proporciona las variables de configuración de la librería
* [return_codes.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/return_codes.h) - En el futuro, el uso de códigos de retorno será reemplazado por los mecanismos de control de errores previstos en el [MaidSafe-Common](https://github.com/maidsafe/MaidSafe-Common/wiki).
* [nat_type.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/nat_type.h) - Una enumeración de tipos relevantes [NAT](https://en.wikipedia.org/wiki/Network_address_translation) que la librería necesita identificar

La librería hace un uso intensivo de [Boost.Asio](http://www.boost.org/doc/libs/1_55_0/doc/html/boost_asio.html), tanto para las operaciones relacionadas con la red como para las operaciones asincrónas. También depende de las librerías MaidSafe [Common](https://github.com/maidsafe/MaidSafe-Common/wiki), [Private](https://github.com/maidsafe/MaidSafe-Vault-Manager/wiki), y [Passport](https://github.com/maidsafe/MaidSafe-Passport/wiki).


### Antecedentes

Dos nodos RUDP mantienen una pseudo-conexión o "conexión administrada" mediante el envío de pequeños mensajes de control continuamente entre sí. La conexión puede ser finalizada por cualquiera de los pares mediante el envío de un mensaje de control de apagado, por lo tanto, en circunstancias normales los pares son conscientes rápidamente de una conexión apagada. En caso de que un número fijo de intentos de conexion queden sin respuesta, el nodo considera que la conexión está muerta. Este es un mecanismo más lento que usando un mensaje de apagado, sin embargo, debería ser mucho menos común, ya que suele ser causado por un cierre inesperado o por una caida de red, por ejemplo.

Para cada conexión, un nodo tiene un `EndpointPair` asociado a sí mismo y otr para el par. El `EndpointPair` contiene el punto final "local" (IP/Puerto tal y como se ve en el interior o detrás del router) y el punto final "externo" (IP/Puerto tal y como se ve fuera del router). Se prefieren los puntos finales externos, los internos se utilizan solo en caso de fallar el del exterior y en caso que ambos pares estén detras de un router que no permite [hairpinning] (http://en.wikipedia.org/wiki/Hairpinning).

No puede establecerse una comunicación entre dos pares si ambos están detras de routers que proporcionan [NAT simétrico](https://en.wikipedia.org/wiki/Network_address_translation#Methods_of_port_translation). La librería [Routing](https://github.com/maidsafe/MaidSafe-Routing/wiki) Maidsafe gestiona este caso proporcionando un nodo proxy para cada uno de esos nodos "ocultos", y, como tal, la biblioteca RUDP no trata de resolver este problema.


### Details

[managed_connections.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/managed_connections.h) defines three functors; `MessageReceivedFunctor`, `ConnectionLostFunctor`, and `MessageSentFunctor`.  The first two must be provided in the `ManagedConnections::Bootstrap` function, can be called many times by RUDP and are self-explanatory.

The `MessageSentFunctor` will be invoked exactly once per call to `ManagedConnections::Send`.  It indicates that the associated message has been received by the target peer, (not just enqueued for sending, but actually received), or else has failed.  It is anticipated (but not required) that a separate instance of `MessageSentFunctor` will be passed in each call to `Send`.

Internally, a `ManagedConnections` class maintains several (up to `Parameters::max_transports`) transport objects, each with its own actual network socket.  This socket is used to maintain several, (currently up to 50) (see [transport.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/src/maidsafe/rudp/transport.h)), `Transport::kMaxConnections`) psuedo-connections.  Therefore, it is likely that having very few `ManagedConnections` objects will provide optimal performance.

Bootstrapping a `ManagedConnections` object can be a slow process, as connections are attempted to the various candidate endpoints until one succeeds.

`ManagedConnections::kResiliencePort()` provides a network-wide known port which can be used in a disaster-recovery situation.  Every `ManagedConnections` instance attempts to open this port locally.  After network segmentation for example, nodes can try to rejoin by attempting to connect to other local nodes, or to direct-connected nodes using this known port.

Further details of the individual functions can be found in the [managed_connections.h](https://github.com/maidsafe/MaidSafe-RUDP/blob/master/include/maidsafe/rudp/managed_connections.h) file itself.




