
# Dron en un Search and Rescue (SAR)
En esta practica el objetivo es programar un dron para hacer un reconocimiento en una zona donde se cree que hay personas naufriagadas. El dron debe acercarse a una zona señalada, iniciar un patrullaje y reconocer caras usando un detector de caras de OpenCV, guardar la ubicacion de los supervivientes, evitando guardarlos multiples veces y mantener un control de la bateria.
## Traduccion de coordenadas
Lo primero de todo es traducir las coordenadas globales proporcionadaas a unos valores mas manejables. En este caso usamos un traductor online para pasar de GPS a UTM (Northing,y Westing). Sin embargo estas coordenadas son globales, y no son aplicables directamente en nuestro dron, para obtener las coordenadas locales de la zona de rescate debemos restar las coordenadas globales del origen a las globales de la zona de rescate. Lo que nos quedan unas coordenadas con el barco como origen.
## Movimiento
La primera fase del trabajo ha sido el iniciar el movimiento hacia la zona. Para esto lo primero es hacer despegar el dron una distancai minima para que la plataforma base no estorbe, despues la idea es acercarlo a la zona de rescate y una vez ahi empezar a patrullar. Al comienzo de la practica plantee usar el cmd_pos para hacerlo mas sencillo pero era muy lento,al final he optado por hacer una funcion que dado una posicion, aplica un control proporcional a la distancia restante usando _Hal.set_cmd_vel(vx,vy,vz,az)_, otra opcion muy viable era usar la version _mix_ ya que la altura (z) debe ser la misma todo el rato, pero he preferido usar la otra para manejar todas las coordenadas de igual manera, para facilitar la implementacion del control. De todas formas, durante la ultima traza de movimiento se cambia a la funcion _Hal.set_cmd_pos(x,y,z)_ para acercarte de forma mas precisa al punto indicado, porque si no se hiciese de esta manera, si el dron fuese muy rapido y se pasase del punto, daria una vuelta innecesaria (**Nose Forward Mode**). En este control tambien se tiene en cuenta la altura a la que debe estar el dron, ya que al moverse, este se inclinaba y segun se movia se acercaba mas al nivel del agua. 

[Screencast from 2026-10-13 19-29-38.webm](https://github.com/user-attachments/assets/84ce86d8-8a94-4550-af97-3725dc3887ac)

Respecto al patrullaje, en un principio se quedo como un cuadrado con lados de un tamaño indicado, por simplicidad y para poder trabajar en el detector de caras. Pero según avanzo la practica, se cambió por otro mas adecuado. En un principio trate de usar una espiral medianamente circular, pero resulto muy compleja para como habia diseñado el movimiento del robot, asique preferí optar por una espiral cuadrada, donde dada una longitud maxima de lado, un centro y un valor de crecimiento, desde el centro indicado va formando una espiral que crece en cada vuelta como el valor de crecimiento hasta llegar a un trazo mayor que el maximo. Como en un escenario real, no se sabe donde estan las victimas exactamente, por lo que la mayoria de el patrullaje es sobre agua vacia.

Todo el movimiento usa la misma funcion _go_to_point_ para usar el mismo control para x,y y z, asi como indicar si se esta patrullando, moviendo hacia la zona o volviendo a cargar la batería. Esto sirve para no estar usando todo el rato el detector de caras, y para regular la velocidad maxima, ya que en la zona de patrullaje deberia ser un movimiento mas lento para buscar a las personas.

[Screencast from 2026-10-13 19-33-30.webm](https://github.com/user-attachments/assets/055b894e-d768-4f70-a541-aa31fd4ea4b4)

## Reconocimiento de Caras
Para reconocer las caras de las victimas usamos el detector face cascade detector de OpenCV, donde usamos una funcion DetectAndDisplay que viene en el codigo de ejemplo. Para que funcione correctamente hay que girar las imágenes varias veces ya que solo detectas caras rectas. A esta funcion solo se le pasa la imagen ventral del dron, ya que la imagen frontal no nos sirve en este caso. Tambien se puede usar para depurar, para ver como funciona la deteccion, y que partes detecta. En OpenCv tambien existe un EyesDetector pero no es necesario su uso, aunque se podria usar para evitar falsos positivos.

Un problema que surge durante este proceso es la duplicacion de detecciones, ya que la deteccion de victimas se guarda segun la posición del dron, y no la de la victima, para amenizar este problema, según se guardan las posiciones de los naufragos, se comprueba que no se haya guardado una posicion en un radio de 4 metros. Si no se hiciese esto, cada vez que aparece un naufrago se reportarian cientos de ellos.

## Bateria
Para simular la bateria se sa una variable que comienza en 100, y baja en 2.5 puntos cada vez que llega a un destino, caundo ocurre eso vuelve a baja velocidad ( porque de otra manera era dificil acertar el aterrizaje en el barco y se caía al agua ), aterriza, recarga al 100% otra vez y vuelve a tomar la ruta desde donde lo dejo. Aqui hay un ejemplo con el gasto de la bateria exagerado para que no sea muy largo:

[Screencast from 2026-10-13 21-34-55.webm](https://github.com/user-attachments/assets/947e57cb-977a-4952-a0bb-896410c6a23e)

## Ejemplo de ejecucion

[screencast_recortado.webm](https://github.com/user-attachments/assets/0a0875ac-a404-43e8-bdfd-81d59ef2d5bb)

## Futuras posibles implementacions
Algunas mejoras que se podrian aplicar a esta practica son por ejemplo, la traza de espirales circulares en lugar de cuadradas ya que serian mas rapidas y fluidas. Lo que mejoraria mucho la eficiencia de rescate. Por otro lado tambien el realismo de la bateria, actualmente gasta lo mismo patrullando que moviendose del barco a la zona de rescate, ademas gasta lo mismo ir a un punto lejano que uno muy cercano. Aunque tiene facil arreglo, decidi dejarlo asi porque de esta manera era mas sencillo hacer testear los diferentes requerimientos por separado. Por último, aunque no procede para esta practica, en un escenario mas realista, el detector de caras de OpenCV no seria suficiente, y seria preferible usar un modelo entrenado para detectar caras especificamente.


