# Seguridad - Autoencriptación (Self encryption)
La seguridad de los datos de usuarios es un aspecto crítico de la red SAFE y, en parte, está proporcionado por el sistema de autoencripción (Self encryption). La red SAFE requiere que los datos guardados sean irrenonocibles como dato y resistentes a la desencriptación, incluso en el caso que el algoritmo de encriptación este comprometido.

La autoencriptación (Self encryption) se usa para mezclar y encriptar los datos antes de enviarlos a la red SAFE. Este proceso se realiza de forma automática y ocurre instantaneamente.

Cuando los datos se guardan en el disco virtual SAFE, son divididos en un mínimo de tres trozos (chunk), Hasheados (http://es.wikipedia.org/wiki/Funci%C3%B3n_hash) y encriptados. Adicionalmente, y para aumentar la ofuscación, cada trozo (chunk) es tratado con la función XOR (http://es.wikipedia.org/wiki/Cifrado_XOR) usando el hash de otros trozos.
Cada trozo se rompe de nuevo en 32 piezas y un par de valores clave se añaden a una tabla en el ordenador del usuario llamada mapa de datos. Cada mapa de datos contiene la localización de cada trozo en que se ha dividido el fichero. Este mapa de datos, que contienen los hash previos y posteriores a la encriptación, se usan para recuperar y decodificar los datos ya que el proceso de encriptacion es no reversible.

Este proceso se realiza en el ordenador del cliente por lo que los datos siempre se encuentran encriptados dentro de la red SAFE y solo el usuario con las debidas credenciales puede desencriptar el fichero. Esto hace que el password no puede ser robado desde la red ya que nunca sale del ordenador del usuario.
Para aumentar la seguridad, el mapa de datos tambien pasa por su propio proceso de autoencriptación.

La red SAFE usa deduplicación de datos (http://en.wikipedia.org/wiki/Data_deduplication) para asegurar que dicho espacio se usa de manera eficiente cuando se intenta guardar multiples copias de los mismos datos. La red es capaz de saber si dos piezas son identicas gracias a poseer identico Hash. Tambien los diferentes Vault usan un Hash para identificarse a si mismo.

[Aqui se puede ver un viseo explicativo de la autoencriptación](https://www.youtube.com/watch?v=Jnvwv4z17b4)

Aquí debajo tenemos una visión de conjunto del proceso.

![Self encryption figure](./img/self-encryption.png)
