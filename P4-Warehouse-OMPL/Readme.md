# Planificacion con OMPL
En esta practica se pide diseñar un robot para transportar estanterias en un almacen, se dan 2 robots distintos, uno holonomico y otro que funciona con Ackermann. El robot debe planificar la ruta para situarse debajo de la estanteria con OMPL y despues depositarla en las zonas designadas usando tambien OMPL para la ruta de vuelta

## Registro del mapa
Para poder relacionar el mapa con las coordenadas de gazebo necesitamos hacer un par de funciones que nos permita pasar de coordenadas a pixeles y viceversa. PAra ello se puede seguir el procedimiento de la P1, pero si nos fijamos, el punto 0,0 es exactamente el medio de la imagen, por lo que sabiendo las dimensiones del del almacen y los pixeles de la imagen es facil crear una relacion entre ambos. Y esto es lo que hago, tengo 2 funciones que se dedican a pasar de pixel a coordeanda y viceversa.
## Wharehouse 1 - Robot Holonómico - Comprobacion de la geometria  del robot
En este escenario se comprueba durante la planificacion si el robot cabe en determinados espacios dada una "hitbox" indicada. Para permitir giros y mejor maniobrabilidad para evitar posibles fallos, se le asigna un area circular alrededor del robot, cuyo radio depende de si carga una estanteria o no.
### Gestión del mapa
 Antes de empezar a montar el plan con OMPL es importante ver como se va a manejar el mapa tanto para la planificacion como la visualizacion,hay que tener en cuenta que el mapa es dinamico, tambien he definido 2 funciones que se usan en las acciones de Soltar Objeto y Coger Objeto, que pintan en el mapa directamente, para borrar las patas del mapa al coger la estanteria pintando por encima de blanco y volver a pintarlas al soltar la estanteria, teniendo en cuenta la orientacion con la que se deja la estanteria. Esto es muy necesario para bloquear posibles rutas que interfeiran con los cambios del mapa y permitir otras que se desbloquean al mover las estanterias. Por lo demás en este caso no se manipula mucho el mapa.

[Screencast from 2026-11-05 18-09-34.webm](https://github.com/user-attachments/assets/663b41a8-eb47-4cd8-8245-0e72fd21b3be)

Por la parte de visualización se muestran los waypoints de la ruta actual con puntos verdes, tambien se muestra la "hitbox" del robot, que cambia dependiendo si lleva una estanteria encima o no.

### Planificación
Para la planificacion se nos pide usar OMPL, para ello lo primero es definir los parametros que dictan las caracteristicas del plan.

#### Configuracion del Espacio de Estado
Como el robot se va a mover unicamente en 2D necesitamos un espacio de estados vectorial con 2 dimensiones, que son las coordenadas X e Y del robot. También tenemos que definir los limites del espacio, que es el tamaño del almacén que vien indicado en el enunciado. Realmente se podria hacer mas pequeño porque hay muchos obstaculos en las paredes  pero no es necesario.

#### Validación de estados
A la hora de validar un estado se comprueba si el area actual del robot (con o sin estanteria) choca con algun obstaculo del mapa. En un principio trataba de usar el area rectangular de la estanteria, pero era demasiado apurado, y el error de posicion que genera el robot al moverse hacia que se acabase chocando contra otros obstaculos, por eso acabé decidiendo que era mejor usar una área circular para asegurarme de que pueda hacer giros en cualquier estado. Aunque esto derivaba a otros problemas, el area circular contaba con un margen adicional introducido aposta para asegurarse de que el robot tiene espacio pero hacia que se bloqueasen algunos caminos al ser tan grande. Por eso he hecho que cuando el algoritmo no encuentra una solucion viable, lo vuelva a intentar reduciendo ese margen. En el video del ejemplo completo se puede ver como al recoger la primera estanteria, que se tiene que colocar girada al lado de un montón de cajas, el robot se queda más tiempo de lo normal parado, eso es que esta buscando una ruta  valida, en cada intento reduciendo el margen

#### Definicion del problema
En esta parte es necesario establecer el punto de inicio y el punto final, que en este caso seria la posicion actual del robot y el punto de recogida o entrega. Se definen como un estado, especificando las coordenadas x e y. En la parte de optimizacion se busca recortar la distancia total para buscar la ruta mas corta, y que el robot de menos vueltas.

Por último se tiene que definir el planificador, al principio, siguiendo el ejemplo del enunciado se usaba RRTconnect, que devuelve la primera ruta que encuentra, no necesariamente la mas óptima, lo que hacia que el robot hiciese trayectorias inncecesarias y en la mayoria de los casos problematicos. Por eso he decidido usar RRTstar  que encuentra caminos óptimos y aún bastante rápido.

## Seguimiento de la ruta
Para cada estantería se definen 2 rutas, una de ida y otra de vuelta. En la de ida, se calculan los estados teniendo en cuenta las dimensiones del robot, pero en la de vuelta, se tienen en cuenta las dimensiones de la estanteria que carga el robot. Se generan las rutas en demanda, es decir, justo antes de ejecutarla. En un principio intenté hacerlas todas del tirón al comienzo, para que el robot no obstaculizase el almacém, sin embargo es díficil hacerlo de esta manera porque el mapa cambia cuando mueves las estanterias. En un caso real esto puede ser un problema porque el robot se queda pensando en medio del almacén, pero no tan grave porque en todo caso se quedaria pensando al comienzo y debajo de las estanterias donde no deberia molestar a nadie. Las rutas no se ejecutan hasta el final, ya que generalmente esto implica entrar debajo de las estanterias que es la parte mas complicada del trayecto,por eso solo se ejecuta emplea el plan hasta el un punto de aproximación. Una vez en ese punto, se alinea hacia el destino y avanza mecanicamente. Esto es lo que mejores resltados ha dado, con el otro panteamiento muchas vecesrozaba y movia las estanterias por las patas.

Para el movimiento de la ruta, se usa una funcion que dirige el movimiento hasta el waypoint que toque, primero se orienta, y despues avanza en linea recta.

El control general del programa lo rige una máquina de estados:
 1. PLANIFICAR_IDA
 2. MOVER_IDA
 3. GIRAR_IDA
 4. ENTRAR_OBJETO
 5. COGER_OBJETO
 6. SALIR_CARGADO
 7. PLANIFICAR_VUELTA
 8. MOVER_VUELTA
 9. GIRAR_VUELTA
 10. ENTRAR_CARGADO
 11. SOLTAR_OBJETO
 12. SALIR_VACIO

Los estados ENTRAR son como se ha indicado antes, desde el punto de aproximacion, ya orientado, avazar recto hasta la parte inferior de la estanteria. Los estados SALIR son para, antes de cambiar la hitbox, volver a un punto anterior para que al cambiar la hitbox y actalizar el mapa no se quede el robot atrapado.


[Screencast from 2026-11-06 12-20-37.webm](https://github.com/user-attachments/assets/ab1b2381-a7fe-437b-8ba6-9c1fa952b804)


[Screencast from 2026-11-05 16-27-23.webm](https://github.com/user-attachments/assets/27e81c87-8e6a-4b24-b23d-efb33b11ba3d)
