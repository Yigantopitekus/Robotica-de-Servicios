# Planificacion con OMPL
En esta practica se pide diseñar un robot para transportar estanterias en un almacen, se dan 2 robots distintos, uno holonomico y otro que funciona con Ackermann. El robot debe planificar la ruta para situarse debajo de la estanteria con OMPL y despues depositarla en las zonas designadas usando tambien OMPL para la ruta de vuelta

## Wharehouse 1 - Robot Holonómico
### Registro del mapa
Para poder relacionar el mapa con las coordenadas de gazebo necesitamos hacer un par de funciones que nos permita pasar de coordenadas a pixeles y viceversa. PAra ello se puede seguir el procedimiento de la P1, pero si nos fijamos, el punto 0,0 es exactamente el medio de la imagen, por lo que sabiendo las dimensiones del del almacen y los pixeles de la imagen es facil crear una relacion entre ambos. Y esto es lo que hago, tengo 2 funciones que se dedican a pasar de pixel a coordeanda y viceversa.

### Gestión del mapa
 Antes de empezar a montar el plan con OMPL es importante ver como se va a manejar el mapa tanto para la planificacion como la visualizacion, En etapas tempranas, lo que hize es representar al robot como un punto y engordar el resto del mapa. Para eso se usaba una funcion similar a la de la P1, y daba algunos resultados, pero en muchos casos pasaba rozando las patas de las estanterias, lo que no se levantasen bien y por ende al dejarlas tampoco se quedaban bien.
No tengo video de ejemplo, pero la idea era engordar todos los obstaculos del mapa el valor de la diagonal del robot mas un margen por si acaso, esto hacia que la ruta variase, habia veces que intentaba entrar por el lado estrecho de la estanteria porque la ruta eram mas corta pero en ocasiones rozaba las patas y movia las estanterias, ya que a la hora de buscar el plan prueba al azar.

Tambien hay que tener en cuenta que el mapa es dinamico, tambien he definido 2 funciones que se usan en las acciones de Soltar Objeto y Coger Objeto, que pintan en el mapa directamente, para borrar las patas del mapa al coger la estanteria pintando por encima de blanco y volver a pintarlas al soltar la estanteria, teniendo en cuenta la orientacion con la que se deja la estanteria. Esto es muy necesario para bloquear posibles rutas que interfeiran con los cambios del mapa y permitir otras que se desbloquean al mover las estanterias.

[Screencast from 2026-11-05 18-09-34.webm](https://github.com/user-attachments/assets/663b41a8-eb47-4cd8-8245-0e72fd21b3be)

Por la parte de visualización se muestran los waypoints de la ruta actual con puntos verdes, tambien se muestra la "hitbox" del robot, que cambia dependiendo si lleva una estanteria encima o no.

### Planificación
Para la planificacion se nos pide usar OMPL, para ello lo primero es definir los parametros que dictan las caracteristicas del plan.
