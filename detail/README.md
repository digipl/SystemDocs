# Librerías

La red SAFE comprende 9 librerías individuales. La siguiente sección, destinada al público más técnico, proporciona una explicación más detallada del papel que cada librería realiza y cómo funcionan.

El código de cada librería está disponible en el [repositorio GitHub de MaidSafe](https://github.com/maidsafe). Las librerías son:

1.  Common - Librería de utilidades
2.  Passport - Proporciona una autocertificación de infraestrucctura de clave pública, PKI, independiente de cualquier directorio central
3.  RUDP - Implementacion del Reliable UDP
4.  Routing - Routing es una librería de una tabla Hash distribuida (DHT) basada en una tabla de ruta similar a Kademlia
5.  Encrypt - MaidSafe Encrypt implementa las funciones relativas a la "autoencripción" de ficheros y directorios
6.  Drive - MaidSafe-Drive es una unidad virtual que ofrece servicios para almacenar y recuperar información de cualquier medio de almacenamiento incluyendo los sistemas de archivos de redes
7.  Network FileSystem - Trata la red SAFE como un sistema de ficheros. Expone una interfaz pseudo RESTful (GET PUT POST DELETE)
8.  Vault Manager - Almacenamiento de datos, Tipos de datos y el gestor LifeStuff.
9.  Vault - Autoreparación, auto gestión y red completamente distribuida


