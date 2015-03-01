# Vaults
Los Vaults se crean en el ordenador del usuario cuando se instala el cliente Maidsafe y se une a la red SAFE.

El vault de un usuario no puede ser visto por este. En lugar de ello, un usuario ve un nuevo disco virtual montado en su ordenador que proporciona acceso a sus datos distribuidos.

Cuando un usuario crea o altera un fichero de su disco virtual, este fichero es sometido a diversos procesos para asegurar su seguridad y hacer el mejor uso de los recursos de la red SAFE.

## Roles en cada Vault
Un Vault puede tener diferentes roles (llamados "persona") que realizan diferentes gestiones de los datos dentro de la red SAFE.
* **Client managers (Gestor de cliente)**<br/>
Un Vault Client manager recibe los chunks de datos de los vault de usuarios.
* **Data managers (Gestor de datos)**<br/>
Estos Vaults gestionan los chunks de datos de los Gestores de cliente (Client manager Vaults). tambien monitorizan el estatus e la red SAFET.
* **Data holders (Depositario de datos)**<br/>
Los Vault Data holder se usan para guardar chunks de datos.
* **Data holder managers (Gestor de depositario de datos)**<br/>
Data holder managers monitorizan los Vaults de los Data holder. Informan a los Data Manager si alguno de los chunks está corrompido o ha sido cambiado. Tambien informan si algún Data Holders está fuera de linea.
* **Vault managers (Gestor de Vaults)**<br/>
Los Vault manager mantiene el software actualizado y el Vault ejecutandose; reiniciandolo en caso de error.
* **Transaction managers (gestor de tranferencia)**<br/>
Los Transaction manager ayudan a gestionar las transferencias de safecoin.

## Data on the SAFE Network

There are 2 mechanisms utilised by the network that authorise an End User to carry out certain actions via the Client. Authority is obtained by group consensus whenever a Client is putting (storing) new data. Alternatively, [cryptographic signatures](http://en.wikipedia.org/wiki/Digital_signature) are used if the Client is amending already stored data (a version) or sending safecoin, for example.

** Group Consensus**<br/>
When an End User attempts to put a new piece of data, the file is encrypted and broken up into chunks as part of the self encryption process, it is passed to a close group of Client managers. This close group are comprised of the closest vault IDs to the users vault ID in terms of [XOR](http://en.wikipedia.org/wiki/Exclusive_or) distance. This is distance measured in the mathematical sense as opposed to the geographical sense. At least twenty eight of the thirty two Client managers much reach consensus before any network operations are carried out.

The Client managers then pass the chunks to thirty two Data managers, chosen by the network as their IDs are closest to the IDs of the data chunk, so the chunk ID also determines it's location on the network.

The network utilises a Scatter/Gather approach, based on [Rabin’s Information Dispersal Algorithm](http://people.seas.harvard.edu/~salil/rabin2011-slides/rabin2011-mitzenmacher.pdf), enabling small data loss (up to 4 pieces) without the requirement to retransmit data

Once consensus is reached, the Data manager passes the chunks to thirty two Data holder managers, who in turn pass the chunks for storage with Data holders. If a Data holder manager reports that a Data holder has gone offline, the Data manager decides, based on rankings assigned to Vaults, into which other Vault to put the chunk of data.

This way the chunks of data from the original file are constantly being monitored and supported to ensure the original data can be accessed and decrypted by the original user.

Any movement of data chunks can only be made if there is a consensus (28 of 32) from the surrounding Vaults. The Vaults cannot act in isolation.

All communications on the SAFE Network are carried out through close groups of 32 nodes. This prevents a rogue node(s) from behaving maliciously. It is not possible for a user to choose their own node ID, or to decide where their data is stored, this is calculated by the network. Every time a node disconnects from the network and reconnects, it is assigned a totally new and random ID.

[Click here to see a short video on how Vaults work](https://www.youtube.com/watch?v=txvKSeCaEP0)

** Cryptographic Signatures**<br/>
When End Users are making changes to existing data, such as changing the content of a file, or sending another End User safecoin, the network does not use group consensus as this layer of complexity and increased network load is not required. 

Cryptographic signatures mathematically validate the owner of any piece of data and can prove this beyond any doubt, provided the End User has kept their private key safe. If the End User is the owner of any piece of data and can prove this, by digitally signing their request with their private key, the network permits them access to change the data. 

