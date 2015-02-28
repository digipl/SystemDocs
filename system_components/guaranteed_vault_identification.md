# Garantía de identificación del Vault
La identificación de los Vaults es esencial si estos se van a encargar y responsabilizarse, en gran medida, de la gestión de los recursos.

Sin identificación, cualquier Vault podria suplantar cualquier otro. Hay métodos conocidos de identificación digital, como los emitidos por una autoridad de certificación (como Verisign) o, en algunos casos una web de confianza. En tanto en cuanto la red SAFE no tiene servidores, no hay intervención humana y es completamente no confiable, estas opciones no son adecuadas.

The Vault identification process involves creating two key pairs. One key pair is a revocation key and is used only to create and invalidate a real key.

The real key pair is created and the public key is signed by the revocation private key and this packet (public key plus signature) is stored on the network as a Vault Identification key type. This Hash is then used as the Vault identity.

The SAFE Network can retrieve this identification packet on request from any Vault. The only way to alter this packet is by a message signed by the same ID as the one that signed the packet.

This process allows Vaults to identify themselves by advertising the hash of the identification packet that they created and stored.

As only the creator has the private key paired with the public key contained in the identification packet then we can assume this is the correct Vault. All messages are encrypted with this ID, so fraudulent Vaults would not be able to decrypt such messages, including connect requests which are required to join the network.
