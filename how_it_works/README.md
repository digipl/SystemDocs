# Como funciona

Cuando un usuario decarga e instala el cliente Maidsafe, un vault se configura en su ordenador..

Despues de unirse y firmar en la red SAFE, el usuarío verá un nuevo disco virtual montado en su sistema. Seleccionando este disco le mostrará los datos estáticos que han sido encriptados y distribuidos por otros vault.

Un vault de usuario tambien podra manejar datos dinámicos como, por ejemplo, una comunicación a través a una aplicación VOIP.

Antes que los datos se guarden en la red SAFE son automaticamnet encriptados. El proceso de autoencriptación implica dividir los datos en pequeños pedazos llamados chunks y encriptarlos usando tanto los datos del inicio de sesión (clave, pin, password) del usuario como los propios datos en sí mismo. Esto hace que para ver los datos se necesitan tanto los correspondientes al inicio de sesión como los de los datos y ninguna de esta información se guarda en en ningún servidor u otro tercer lugar.

Típicamente el usuario se conecta a la red a traves de un router. En la red SAFE el router usa el protocol RUDP (Reliable UDP) para dicha conexion. RUDP es un protocolo más robusto que UDP ya que los paquetes perdidos son retrasmitidos y es capaz de atravesar el NAT del router no como el TCP. El uso del RUDP para la red SAFE permite que los datos pasen a traves de los router sin poder ser corrompidos o interceptados.

The user's Vault connects to other Vaults as part of the storage and management of data. The Vaults are constantly checked and ranked (by the Data holder managers personna) using the following criteria:

* **Availability** - how often the Vault is on or off
* **Storage** - how much storage space is in the Vault
* **CPU** - how much CPU resource of the Vault
* **Bandwidth** - how fast or slow the access is to the Vault

As demand and resources on the SAFE Network change, the Vaults adapt and continually balance the load of the network. This adjustment process is done automatically by the Vaults themselves. As the SAFE Network is completely autonomous, it can react quickly and without the need for any human intervention.

When a user provides more storage space than the amount they use on the SAFE Network, they are awarded safecoins at random by the network. Safecoins are required to access services on the network. The user can see how many safecoins they have by looking at their wallet. The wallet is automatically set up and configured as part of the MaidSafe client installation and sign up process.

App Builders earn safecoins when they create apps and programs for the SAFE Network. As the apps are used, the App Builder earns safecoins.

Core Developers can also be awarded safecoins by contributing to the SAFE Network codebase.
