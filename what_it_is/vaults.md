# Vaults
La red SAFE consiste en diferentes procesadores de software (nodos) que se conocen como Vault. Cada Vault reside en el disco duro del usuario y realiza multiples funciones necesarias para el funcionamiento de la red. Una de las funciones principales es el guardado de datos encriptados que otros Vault se encargan de hacerselo llegar.

Otra función, o "persona", de un vault es gestionar otros Vault. En la medida que diferentes chunk de datos se envian de un vault a otro, tambien se gestiona y monitoriza la integridad de dichos datos.

Un Vaults tambien puede ser gestor de transcciones o validador. Estos se encargan de gestionar las transferencia de safecoin.

Los Vaults no saben nada de los datos que gestionan o que guardan (estos solo son desencriptables por cada cliente) y, en tanto en cuanto cada dato solo es un pequeño trozo del original no es posible descifrar cual es su fuente, ya sea, por ejemplo, un documento o una comunicación.

Por cada Vault que guarda un dato, hay otros 31 que tambien se encargan de guardar los mismos datos. Eso hace que si un Vault se apaga o sus datos se corrompen, ningún dato se pierda. Inmediatamente, y de forma automática, se encuentra otro Vault que se encarga de guardar dichos datos.

Los Vaults no son visibles para los usuarios de la red SAFE. Lo que un usuario ve es un disco virtual montado en su ordenador. Mientras tanto los diferentes Vaults se comunican y monitorizan unos a otros para que el acceso a los datos de dicho disco virtua sea instantaneo.
