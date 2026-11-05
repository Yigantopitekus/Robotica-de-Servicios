# Planificacion con OMPL
En esta practica se pide diseñar un robot para transportar estanterias en un almacen, se dan 2 robots distintos, uno holonomico y otro que funciona con Ackermann. El robot debe planificar la ruta para situarse debajo de la estanteria con OMPL y despues depositarla en las zonas designadas usando tambien OMPL para la ruta de vuelta

## Wharehouse 1 - Robot Holonómico
### Registro del mapa
Para poder relacionar el mapa con las coordenadas de gazebo necesitamos hacer un par de funciones que nos permita pasar de coordenadas a pixeles y viceversa. PAra ello se puede seguir el procedimiento de la P1, pero si nos fijamos, el punto 0,0 es exactamente el medio de la imagen, por lo que sabiendo las dimensiones del del almacen y los pixeles de la imagen es facil crear una relacion entre ambos. Y esto es lo que hago, tengo 2 funciones que se dedican a pasar de pixel a coordeanda y viceversa.

### Gestión del mapa
 Antes de empezar a montar el plan con OMPL es importante ver como se va a manejar el mapa tanto para la planificacion como la visualizacion, En etapas tempranas, lo que hize es representar al robot como un punto y engordar el resto del mapa. Para eso se usaba una funcion similar a la de la P1, y daba algunos resultados, pero en muchos casos pasaba rozando las patas de las estanterias, lo que no se levantasen bien y por ende al dejarlas tampoco se quedaban bien.
