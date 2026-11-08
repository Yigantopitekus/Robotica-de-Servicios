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
Como el robot se va a mover unicamente en 2D necesitamos un espacio de estados vectorial con 2 dimensiones, que son las coordenadas X e Y del robot. También tenemos que definir los limites del espacio, que es el tamaño del almacén que vien indicado en el enunciado. Realmente se podria hacer mas pequeño porque hay muchos obstaculos en las paredes  pero no es necesario. Tambien se debe indicar un yaw para asegurarse de hacer trayectorias mas limpias y seguras.

#### Validación de estados
A la hora de validar un estado se comprueba si el area actual del robot (con o sin estanteria) choca con algun obstaculo del mapa. En un principio trataba de usar el area rectangular de la estanteria, pero era demasiado apurado, y el error de posicion que genera el robot al moverse hacia que se acabase chocando contra otros obstaculos, por eso acabé decidiendo que era mejor usar una área circular para asegurarme de que pueda hacer giros en cualquier estado. Aunque esto derivaba a otros problemas, el area circular contaba con un margen adicional introducido aposta para asegurarse de que el robot tiene espacio pero hacia que se bloqueasen algunos caminos al ser tan grande. Por eso he hecho que cuando el algoritmo no encuentra una solucion viable, lo vuelva a intentar reduciendo ese margen. En el video del ejemplo completo se puede ver como al recoger la primera estanteria, que se tiene que colocar girada al lado de un montón de cajas, el robot se queda más tiempo de lo normal parado, eso es que esta buscando una ruta  valida, en cada intento reduciendo el margen

#### Definicion del problema
En esta parte es necesario establecer el punto de inicio y el punto final, que en este caso seria la posicion actual del robot y el punto de recogida o entrega. Se definen como un estado, especificando las coordenadas x e y, y la orientación. La orientación es muy importante en la meta, para asegurarse de que el robot accede adecuadamente a la estantería En la parte de optimizacion se busca recortar la distancia total para buscar la ruta mas corta, y que el robot de menos vueltas.

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
## Ejemplo

[Screencast from 2026-11-05 16-27-23.webm](https://github.com/user-attachments/assets/27e81c87-8e6a-4b24-b23d-efb33b11ba3d)

## Wharehose 1 - Robot Holonómico - Representado como un punto
En este escenario se engrosa todo el mapa asumiendo una forma circular del robot, y se comprueba un estado valido si la coordenada del robot se encuentra en un obstaculo, el engrosamiento varía en funcion de si el robot carga o no una estantería. Este escenario es una version mas primeriza de la otra para este robot.

### Gestión del mapa
En este otro caso, la diferencia es que el mpa se engorda en funcion de la htbox actual del robot, si esta cargando una estanteria se engordan todos los obstaculos en funcion de las dimensiones de la estantería y sino, en funcion de las dimensiones del robot. En ambas se asume que la geometria del robot es redonda. Esto permite representar el robot com un punto 2D, ya que las medids de precaucion se han tomado de manera relativa a los obstáculos.

### Planificación y seguimiento de ruta
EL seguimien to de la ruta es esencialmente el mismo procedimiento, en cuanto a la planificacion lo único que cambia es la validación de estados. Ya que al haber engrosado todos los obstaculos, no hace falta ocomprobar los alrededores del robot, sino únicamente su posicion actual.
## Ejemplo

[Screencast from 2026-11-06 12-20-37.webm](https://github.com/user-attachments/assets/316604b9-1cd4-4542-934e-1410e4dbafc5)

## Warehose 1 - Robot Ackermann - Comprobando la geometría del robot
Este caso se asemeja al Robot holonomico - comprobando su geometria , las diferencias residen en la geometria dle robot y en su conduccion. Este robot es rectangular, y usa una geometria ackerman, es decir que no puede girar sobre si mismo como el robot anterior.

### Gestión del mapa
El mapa se maneja de igual manera que el caso uno, la principal diferencia es la representacion d ela "hitbox" del robot, que en este caso tiene que ser rectangular y no circular. EN el otro caso se hacia circular para permitir giro sobre si mismo, ahora, con la geometria ackerman solo se representa un area rectangular, mas grande o mas pequeña si el robot esta cargado o no.

### Planificacion y seguimiento de la ruta
En este ambito hay bastantes cambios mas considerables, en el espacio de estados, validación de estados, y seguimiento de la ruta.

#### Cambios en el espacio de estados
En el caso anterior el espacio de estados era `RealVectorStateSpace(2)`, que solo contiene las coordenadas x e y. Yaw no se tenía en cuenta porque al usar puntos de aproximacion, esto se abordaba fuera de la planificación.

Un robot Ackermann no puede girar sobre sí mismo y su orientación afecta hacia dónde puede moverse. Por eso el yaw tiene que formar parte del estado, y además hay que considerar que el robot tiene una capacidad de giro límitada. Para ello he usado los espacios de estados de OMPL para vehículos con curvatura limitada:

- **`DubinsStateSpace`**: conecta dos posiciones (x, y, yaw) con lineas que pueden ser curvas (respetando el radio minio de giro,lo obtuve con un script aparte) o rectas, solo hacia delante, es decir sin girar sobre sí mismo.
- **`ReedsSheppStateSpace`**: igual, pero puede usar marcha atrás que es muy necesario para algunas posiciones.
  
Se intenta planificar con **DubinsStateSpace**, pero si da error o la ruta es muy costosa, lo intenta con **`ReedsSheppStateSpace`**.Ambos siguen usando RRTstar, pero se le añade la configuracion del bias para que en vez de explorar aleatoriamente exploren mas hacia la meta.

#### Validación de estados
El estado se valida con un rectángulo orientado según el yaw actual del estado, en lugar de un círculo. El tamaño depende de si el robot esta cargando la estanteria o va vacío. AL igual que el caso del robot holonómico, se comprueban los pixeles del interior del rectangulo para ver si colisiona. Y el mapa sigue siendo dinámico, y si no se encuentra ninguna solución se va reduciendo el área poco a poco.


#### Seguimiento de la ruta
A la hora de generar la ruta, se usa la opcion _interpolate_ para que la ruta sean muchos waypoints muy cercanos, para poder hacer el seguimiento de la ruta mas preciso y controlado. Y el movimiento se gestiona por segmentos de trozos de la ruta donde se va hacia adelante o hacia atras.

Para mover al robot a través de cada segmento, se usa un controlador de pure pursuit, donde a partir de un punto cercano a robot, se mira una distanca hacia adelante y se elige un punto, para estimar la posible trayectoria para llegar a dicho punto.

Uno de los problemas que tenía con este controlador era que oscilaba mucho la trayectoria, y se salia de la ruta proupesta, esto se arreglaba cambiando los parametros del controlador. Aumentando el lookahead, reduciendo velocidades en  las curvas y evitando giros muy bruscos (con el robot ackerman era complicado)

Si el robot se desvía mucho de la ruta, o pasa 8 s sin avanzar, se da por perdido y se replanifica desde la pose actual.

Al terminar la ruta se comprueba que la pose final esté alineada con la estantería, tanto en yaw como en desviación lateral. Si no lo está se replanifica, con una tolerancia algo mayor en cada reintento. Tras dos reintentos se da por buena, para evitar replanificar todo el rato.

#### Máquina de estados
A diferencia del holonomico, aqui si se añade el yaw en el goal, asique no hace falta añadir los estados de GIRAR que se usaban en el otro caso:

 1. PLANIFICAR_IDA
 2. MOVER_IDA
 3. ENTRAR_OBJETO
 4. COGER_OBJETO
 5. SALIR_CARGADO
 6. PLANIFICAR_VUELTA
 7. MOVER_VUELTA
 8. ENTRAR_ENTREGA
 9. SOLTAR_OBJETO
 10. SALIR_VACIO

## Ejemplo

[screencast_comprimido.webm](https://github.com/user-attachments/assets/bd2a4bd6-9c76-4c71-b397-afec70361f30)


