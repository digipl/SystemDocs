#Safecoins

Los safecoins pueden ser ganados, negociados o comprados. El valor de los safecoins vendrá determinado por el mercado, a través de la presión de la oferta y la demanda. 

## Precio de mercado
En número de safecoin en circulación aumentará segun el uso de la red. Casi todos los poseedores iniciales de safecoin serán Farmers (granjeros) que con su suministro de recursos crearán tanto liquidez como distribución de riqueza. Se prevee que casi todos los usuarios poseerán al menos unos safecoins en su wallet.

Los usuarios pueden negociar sus safecoin por servicios en la red, o por liquidez (u otra moneda digital) usando un exchange. 
El ratio de los safecoin guardados en los wallet y aquellos gastados por los Farmers generarán un precio. Este precio será el valor de mercado de los safecoin.

## Recursos y monedas

Los safecoin se usan para acceder a servicios dentro de la red SAFE. Esto incentiva la reutilización constante cuyo resultado es el aumento de la demanda de un recurso finito. Esto hace que el valor de las safecoins aumente con el tiempo. Mientras que las monedas aumentan de valor, la cantidad de servicios en la red (recursos) que cada safecoin pueda comprar también aumenta. Esto se muestra en la fig. 2.

![](safecoin resources.png)

## Velocidad de farming 

Farming es un proceso en el cual los usuarios suministran recursos (espacio almacenamiento, CPU, ancho de banda) a la red.

Como muestra la figura 1, el algoritmo de ganancia de los safecoin es una curva sigmoide, en la que todos los Vault empiezan a ganar lentamente y donde su tasa aumenta a medida que los farmers alcanzan el promedio de la red. La tasa de ganancia tambien tiene en cuenta el rango de cada Vaul, un proceso en el cual la red puntua la utilidad de cada nodo entre 0 (el peor) y 1 (el mejor). La tasa del farming de safecoin es, en última instancia, el resultado de su media en la red, el balance entre oferta y demanda multiplicado por el rango del Vault. La tasa comenzará a descender para los que se situen en el 20% superior del total de la media. Esto evitará la existencia de grandes Vault que podria llevar a la centralización del proceso de Farming. Los safecoin se asignan por la red y se paga cuando un nodo entrega correctamente un dato (GETS) y no cuando se guarda (PUTS).


![](safecoin farming speed.png)

The network automatically increases farming
rewards as space is required and reduces them
as space becomes abundant. Data is evenly distributed on the network and therefore farmers
looking to maximise their earnings may do so
by running several average performance nodes
rather than one high specification node.

## Safecoin transfer mechanism
Unlike bitcoin, the SAFE Network does not use
a blockchain to manage ownership of coins. Conversely,
the SAFE Network’s Transaction Managers
are unchained, meaning that only the past
and current coin owner is known. It is helpful to
think of safecoin as digital cash in this respect. 

One of the major problems any virtual currency
or coin must overcome is the ability
to avoid double spending. Within the SAFE
Network, transfer of data, safecoin included,
is atomic, using a cryptographic signature to
transfer ownership.

Safecoin, the currency of the SAFE network, is generated in response to network use. As data is stored, or as apps are created and used, the network generates safecoins, each with their own  unique ID. As these coins are divisible, each new denomination is allocated a new and completely unique ID.

As the coins are allocated to users by the network, only that specific user can transfer ownership of that coin by cryptographic signature. For illustrative purposes, when Alice pays a coin to Bob via the client, she submits a payment request. The Transaction Managers check that Alice is the current owner of the coin by retrieving her public key and confirming that it has been signed by the correct and corresponding private key. The Transaction Managers will only accept a signed message from the existing owner. This proves beyond doubt that Alice is the owner of the coin and the ownership of that specific coin is then transferred to Bob and now only Bob is able to transfer that coin to another user.This process is highlighted in figure 3.

![](safecoin transfer mech.png)



