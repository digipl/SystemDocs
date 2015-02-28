# Conexiones subida-bajada en los routers
la red SAFE permite la conexión entre diferentes Vaults.

La mayoria de las conexiones en los hogares se realizan usando routes que proveen de conexiones privadas que no pueden retrasmitirse al exterior. Una manera estandar es el uso de servidores [STUN](http://es.wikipedia.org/wiki/STUN).

This mechanism is not acceptable in a decentralised network. In the SAFE Network, RUDP is used in a mechanism that emulates a decentralised STUN server, with the distributed hash table (DHT) providing connection information to negotiating Vaults.

The SAFE Network can handle router connections without the user having to adjust their connection to the network through their router.
