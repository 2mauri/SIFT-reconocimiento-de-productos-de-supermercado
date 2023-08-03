# Reconocimiento de Productos de Supermercado
El objetivo es reconocer automáticamente y encontrar la orientación en la imagen de productos de supermercado cuando éstos son mostrados a la cámara utilizando matching de puntos SIFT. 

**Imágenes base**  
Se obtienen las imágenes de productos en páginas web de supermercados, en este caso se utilizaron cuatro productos distintos.  

**Imágenes a reconocer**   
Se adquirirán imágenes de cada uno de los productos reales con variación de distancia, orientación y posición. Además, se toman imágenes donde hay varios productos en una misma foto y también imágenes donde no hay productos.  

## Breve explicación del funcionamiento del algorítmo SIFT 

Como resumen introductorio, **SIFT** es un algorítimo que se basa en el espacio de escalas haciendo detección de puntos relevantes usando técnicas como el laplaciano de gaussianas o la diferencia de gaussianas (siendo esta más eficiente). Luego, hace un refinamiento de los puntos de interés mediante el ajuste de un modelo continuo para hacer más precisa su posición. Para hacer la descripción de cada punto, el algorítmo encuentra una orientación principal basandose en el gradiente y se obtiene un vector de 128 dimensiones basado en el histograma de como cambia la intensidad alredor de ese punto (básicamente el histograma del gradiente).

El laplaciano de gaussianas es un operador con el cual se convolucionará la imagen, este operador puede ser costoso de computar, por lo tanto se utiliza también la diferencia de gaussianas. El procedimiento es el siguiente, primero se convoluciona con una gaussiana con un cierto $\sigma_A$, después se convoluciona con otra gaussiana de un $\sigma_B$ y se toma la diferencia. La diferencia de gaussianas es una técnica menos exigente computacionalmente para obtener una aproximación para la salida del operador laplaciano de gaussianas. Estos operadores se utilizan para encontrar la descripción de una imagen a una cierta escala.

Luego se cambia la escala, lo cual consiste en un submuestreo de la imagen dada una cierta cantidad de operaciones. Esto se hace debido a que convolucionar con una gaussiana es filtrar pasabajos, por lo tanto realizar los filtrados se va perdiendo información, lo cual abre camino a submuestrar sin perder nada.

Los puntos de interés se buscan como extremos locales en el espacio de diferencia de gaussianas, teniendo en cuenta múltiples escalas. El espacio de escalas se construye como una suceción de la aplicación del operador sobre una imagen de entrada.

$$G = (G_0, G_1, ..., G_{M-1}) : G_m = I \ast G_{\sigma_m}$$

La relación entre dos escalas consecutivas viene dada por la siguiente relación.

$$\Delta_{\sigma} = \frac{\sigma_{m+1}}{\sigma_{m}} = 2^{1/Q}$$

Cada $Q$ escalas se duplica la escala, por lo que se reduce el ancho de banda a la mitad y se puede submuestrar sin perder información.

Luego de que se construyó el espacio de escalas, se prosigue buscando extremos de forma independiente en cada escala. Para esto se ajusta una función que varía suavemente y se busca el extremo, obteniendose un refinamiento a nivel subpixel. Luego se pasa por una etapa de eliminación de puntos bordes y finalmente se obtiene la salida del algoritmo **SIFT**.

Ordenando todo lo presentado anteriormente, el algoritmo de **SIFT** se puede explicar en los siguientes pasos.

**1.** Se crea el espacio de escalas

**2.** Se localizan los keypoints y se hace el refinamiento. Lo que se quiere hacer es obtener un valor más preciso para la posición del extremo.

**3.** Se toma un vecindario de 16x16 alrededor de los puntos localizados y se divide en 16 regiones de 4x4. Para cada región, se obtiene la dirección principal y se crea un histograma de 8 bins obteniéndose un total de 128 valores disponibles para bin.

**4.** Finalmente está la etapa de matching, se hace un matching entre puntos de diferentes imágenes mediante la identificación de sus vecinos más cercanos. En algunos casos el segundo match más cercano puede estar muy cerca del primero, una posible razón para esto es ruido. Entonces, se toma la distancia (el radio) del match más cercano con el segundo más cercano y se realiza una comparación. Si es mayor que 0.8, son rechazados.
