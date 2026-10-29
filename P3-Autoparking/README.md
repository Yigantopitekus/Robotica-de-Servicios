# Autoparking
En esta practica se pide desarrollar un algoritmo donde un coche con un LIDAR instalado debe encontrar un hueco y conseguir aparcar en distintas situaciones.
## Cambio de carril
Al comienzo de la simulación, el coche se encuentra en el centro de la carretera, lo que no es adecuado para buscar sitio para aparcar. Asi que lo primero que hay que hacer es colocarlo en el carril que le corresponde. Para esto se diseña una fucnión sencilla que mueve el coche un poco hacia la derecha y despues se reorienta para circular recto. Durante este proceso, no se trata de encontrar hueco.

## LIDAR
En mi caso he decidido abordar directamente el mundo del LIDAR en lugar de probar con el mundo de los láseres, ya que tenia una idea clara de como debía hacerlo. La tarea que se debe realizar con el lidar es encontrar un hueco suficientemente grande para el coche e iniciar la maniobra de aparcamiento. Algunas dificultades que se encuentran en esta `parte es filtrar la nube de puntos, el LIDAR devuelve una nube de puntos que envuelve el alrededor del coche, pero a nosostros solo nos interesa el lado derecho que es donde queremos aparacar, para ello es necesario hacer un filtro que solo deje los puntos a la derecha del coche en una franja un poco mayor que las dimensiones del coche (Estas se pueden obtener imprimiento su posicion por la terminal y moviendo el coche en el simulador, usando de referencia los coches blancos que son más o menos del mismo tamaño). Algo a tener en cuenta sobre esta franja, es que las coordenadas que devuelve el LIDAR son locales y las del coche son globales, entonces no se pueden comparar directamente; para solucionarlo he creado una funcion _parse_lidar_ que toma como parametros las coordenadas de un punto de la nube de puntos y devuelve unas coordenadas que sí se pueden comparar con las del coche. Esta funcion se puede deducir mirando los ejes de coordenadas del coche y del LIDAR. Por otro lado, el lidar esta en la parte posterior del coche por lo que si miramos los puntos  justo a la derecha , cuando se detecte el hueco, el coche ya habrá adelantado unos metros antes de iniciar la maniobra, esto se puede ignorar  y sigue funcionando porque el hueco en el mundo es muy grande pero es preferible ajustar las mediciones para hacerlo más realista.

Respecto al filtro, el hueco se da por válido si la longitud de la nube de puntos filtrada por distancias es menor que cierto umbral, pero hay que tener en cuenta que hay fluctuaciones importantes a tener en cuenta, la nube de puntos aumenta de tamaño cuando hay un objeto en la acera (como una farola en nuestro caso) o  una pared al lado del hueco y  baja bastante en al pasar cerca de los huecos entre los coches (aunque sean pequeños). Por lo que hay que tener cuidado al establecer el umbral.

### Encontrando hueco con las farolas
En este ejemplo el aparcamiento esta desactivado, en la terminal se va indicando la cantidad de puntos que detecta en la franja, y si lo considera hueco o no.

[Deteccion_con_farolas](https://github.com/user-attachments/assets/77ad8cb5-27b7-4323-9af1-e9416d692bcd)

### Encontrando hueco con "pared" u obstáculo en la acera
En este ejemplo el aparcamiento esta desactivado, en la terminal se va indicando la cantidad de puntos que detecta en la franja, y si lo considera hueco o no.

[Detección_con_pared](https://github.com/user-attachments/assets/898cc809-d6be-4b56-8baf-3aeb35d41e31)

El truco esta en  ajustar la dsitancia del filtro del LIDAR para que solo cuente los puntos hasta un maximo de distancia , evitando asi analizar los puntos detectados mas alla de la calzada
## Maniobra de aparcamiento
La maniobra para aparcar es relativametne sencilla ya que es muy mecanica, lo mas tedioso ha sido encontrar las velocidades y distancias adecuadas para cuadrarlo bien en todos los escenarios. Uno de los principales problemas que he encontrado ha sido el limitado rango de giro del coche, lo que me ha obligado a diseñar una maniobra con movimientos muy amplios, que en un escenario real podria obstruir el tráfico. La maniobra es la siguiente:

  1. Avanzar 5 m desde donde se detecta el hueco, para dar espacio para la marcha atras
  2. Girar hacia el lado contrario de donde se encuentra el hueco 30 grados para poder hacer una marcha atras mas adelante mas limpia, para una maniobra mas corta y con menos trazas
  3. Girando hacia el otro lado, iniciamos la marcha atras hasta que el coche este aproximadamente a la distancia a la que encontro el hueco
  4. Aun marcha atras, giramos hacia el otro lado hasta conseguir alinear el coche al angulo de la carretera (obtenido mientras se circulaba buscando hueco)

En versiones anteriores he intentado hacer la maniobra usando la libreria **time**, pero no ha dado muy buenos resultados, porque hacer un pequeño cambio en una fase implicaba cambiar todas las demas a base de prueba y error para que vuelva a cuadrar todo. Ademas, daba la sensación de que en ejecuciones seguidas sin cambios no daba resultados iguales en algunos casos. Sobre todo, cuando intentaba grabar los resultados de la ejecución.

A continuación algunos ejemplos de ejecución:

[Aparcamiento_ref_detras.webm](https://github.com/user-attachments/assets/5000d53f-8a44-4ddf-8620-05d5332070d3)

[Aparcamiento_ref_delante.webm](https://github.com/user-attachments/assets/e09ce49a-5e7c-4510-8b75-363f094184ea)

[Aparcamiento_2ref.webm](https://github.com/user-attachments/assets/ec3eb247-af88-494f-9f09-5c604e25a1aa)


Notesé que se usan velocidades muy bajas, esto se hace aposta para intentar hacer movimientos mas precisos y evitar colisiones con la acera u otros coches.

## Futuras posibles implementaciones
Algunas caracteristicas que se podian implementar como por ejemplo; una maniobra hibrida entre los laseres y el lidar, usando el lidar par detectar el hueco teniendo en cuenta posibles obstaculos (como escombros con poca altura que el laser pasaria por alto) y el laser para ajustar la maniobra dependiendo del tipo de hueco que se encuentre. Por otro lado, optimizar la maniobra para que sea mas rápida y no obstaculice el carril ni invada el carril contrario.
