# Introducción
En esta practica se explica el desarrollo de una aspiradora de gama alta en Unibotics, el desarrollo pasa por 3 fases importantes. La primera la relacion de el mapa en la imagen con el simulador gazebo y la creacion del fridma. La segunda el algoritmo de planificacion (BSA) y la distribucion de colores y por ultimo el movimiento del robot.
# Crear Transformación de coordenadas a píxeles 
Para este paso se necesita recabar 10 puntos con sus píxeles aproximados obtenidos del png del mapa, para obtener esos valores he creado  un script de Python que muestra el mapa en pantalla y va imprimiendo por la terminal las coordenadas del píxel señalado en la imagen, como se muestra en esta demostración

https://github.com/user-attachments/assets/e158e50a-f584-44ee-932c-d5da1bcbb27e


El siguiente paso es obtener la matriz de transformación para relacionar la posición en coordenadas xy del mapa con los píxeles uv de la imagen, de nuevo, usamos un script de Python independiente para obtener la transformada afín de los 10 puntos seleccionados, en el enunciado se recomienda usar una matriz de transformación homogénea distinta, pero como es una transformación 2D (no necesitamos z) con la matriz afín basta. Nos ayudamos de la biblioteca de CV2 para esto, que cuenta con una función *cv2.estimateAffine2D* para esto. Mi matriz de transformación resultante es la siguiente:

<img width="331" height="80" alt="image" src="https://github.com/user-attachments/assets/1ecb8dde-a6f1-4e9a-b46e-37defcf9b288" />
Que se traduce en:

<img width="313" height="84" alt="image" src="https://github.com/user-attachments/assets/5ad87759-9b5e-4fa5-8da7-eff1ddde9ff4" />


Con estos datos ya se puede relacionar la posición en píxeles 2D a las coordenadas xy del robot resolviendo el sistema de ecuaciones.

# Creación de las casillas del mapa
Primero, el mapa se tiene que dividir en obstáculo (físico o virtual) y espacio navegable, que se representan como negro y blanco respectivamente. 
En esta etapa me entretuve porque al leer que había que añadir cierta erosión preventiva al mapa (para no chocar con las paredes o pasar muy cerca etc) decidí añadirlo de golpe a toda la imagen usando OpenCV usando un kernel que se aplicaba a todos los píxeles y "engordaba" las paredes. No resultó muy bien porque dificultaba mucho el cuadrar después las rejillas, que dejaban pequeños huecos o zonas muy estrechas donde el robot no podría pasar. Tras descartar este método, pasé con el que me quedé finalmente que es: Recorrer toda la imagen (1012x1012) en regiones de 35x35 píxeles que es el tamaño exacto del robot proporcionado por el enunciado. En cada región se hace recuento de los píxeles blancos y los negros y si hay más de cierto umbral de negros se rellena la casilla entera. Asumiendo que el robot no podría pasar cómodamente por el centro de la casilla. 
Una vez recorridas todas las regiones se representan las casillas visualmente con líneas negras, de nuevo con la librería OpenCV, como se pintan encima de las regiones, las casillas acaban siendo un poco más pequeñas que la aspiradora, que sería lo ideal.
<img width="1012" height="1012" alt="image" src="https://github.com/user-attachments/assets/c7ddadc3-78b1-4315-a5fe-ae286d447fb4" />

Una vez hecho esto, hay que decidir cómo se van a representar las casillas dependiendo de su estado durante la ejecución, en mi caso elegí:

- Naranja: No visitada
- Verde: Visitada (esta no se implementa hasta implementar el movimiento)
- Azul claro: Puntos de retorno
- Rojo: Puntos Críticos
- Azul Oscuro: Casilla de inicio

# Planificación
Como bien hemos dado en clase para la planificación se va a usar el Backtracking Spiral Algorithm (BSA), este algoritmo es un algoritmo de cobertura completa, usa barridos sistemáticos en forma de espiral siguiendo una prioridad establecida, en mi caso ESWN (Este, Sur, Oeste, Norte). El funcionamiento básico de este algoritmo (usando mi orden de prioridad como ejemplo) es el siguiente:

El robot empieza el movimiento hacia E, mientras no se tope con nada (obstáculo o casilla visitada) sigue en esta dirección. Si se topa con algo cambia la dirección hacia S y hace lo mismo, la diferencia es, según qué dirección lleve, va a comprobar primero una dirección u otra. Es decir, cuando se "choca" en E, primero comprueba S, luego W y por último N, pero si se "choca" en S, primero comprueba W, luego N y por último E. Este procedimiento lo ejecuta hasta que no queden casillas libres esto se denomina punto crítico. 
Esta situación tiene una fácil solución, durante el recorrido de la espiral va anotando qué casillas vecinas fuera de la trayectoria están libres, para por si se queda atascado volver a una de ellas, estos son los puntos de retorno

La función planificación se ha planteado como una función recursiva por cada espiral, en cada llamada se pasa a través de un bucle for de 4 iteraciones para buscar a qué dirección moverse, siguiendo la prioridad, si encuentra una casilla libre, se mueve y vuelve a llamar a la función, si no encuentra ninguna casilla libre pasa a la siguiente dirección y pasa a la siguiente iteración. Cuando termina la espiral la recursividad termina. Para evitar que se pare la planificación, la llamada a spiral() comienza desde otra plan(), no para de llamar a spiral() hasta que se acaben los puntos de retorno. Durante este proceso se van pintando las casillas. A continuación se puede ver cómo queda el plan finalmente.

https://github.com/user-attachments/assets/969b8a74-390e-4e60-94d4-d1580c064e77

El plan se genera mucho más rápido, pero para su visualización he puesto un time.sleep()

Durante el desarrollo de esta parte he tenido algunos problemas con versiones anteriores donde no implementaba correctamente del todo el BSA, en un principio el algoritmo no era recursivo sino que se basaba en una única llamada que aunque hacía un barrido completo y eficiente no era BSA. En esta versión la prioridad se implementaba de forma absoluta, con esto me refiero a que buscaba siempre la opción de mayor prioridad basándose siempre en el orden ESWN, lo que resultaba en barridos. Esta opción no estaba tan lejos de lo que tengo actualmente si hubiera que la prioridad se buscase según la dirección actual del robot, como he explicado antes. El cambio a una función recursiva ha surgido porque como no me funcionaba esta versión, me puse a investigar y me salió como ejemplo de explicación una versión recursiva, que decidí implementar a la práctica porque me gustó la idea. A continuación se pueden ver algunos de los planes que generaban estas versiones pasadas.


https://github.com/user-attachments/assets/765c7420-08b6-4bad-b54e-7b7aec0c27b6



https://github.com/user-attachments/assets/0012d44f-de52-4ffc-ae76-57e80cc94be7

En la versión del barrido, hay una funcionalidad que no he implementado en el planificador final que es que haya un máximo de puntos de retorno, en un principio solo se iba a quedar con los más recientes para evitar un exceso de ellos. En esa versión que se recorría toda la sala poco a poco no había problema, pero con la versión final el robot suele irse a distintas partes del mapa dejando algunas zonas sin pasar por lo que si hubiese un máximo de puntos de retorno estos se perderían y esas zonas se quedarían sin limpiar.

## Búsqueda de puntos de retorno
Para la búsqueda de puntos de retorno uso 2 métodos principales, el primero es comprobar que se pueda ir directamente en línea recta, se traza una línea hasta los n puntos de retorno más cercano (medido por la distancia euclídea) y se comprueba si se puede llegar de manera directa a alguno de ellos. Si no fuese posible, se aplica el algoritmo de búsqueda Breadth-First-Search, para encontrar una ruta hasta el punto de retorno más cercano. Este algoritmo es básicamente ir comprobando recursivamente desde la casilla de inicio (punto crítico donde se encuentra el robot) las casillas vecinas hasta encontrar un punto de retorno registrado, y guardar las casillas y el orden en el que se han comprobado para establecer la ruta. 

<img width="612" height="510" alt="bfs_return_point" src="https://github.com/user-attachments/assets/d2c121bc-9c34-4bb6-8e6d-1dff13f08aac" />

# Movimiento
Esta fase ha sido la que más se me ha complicado con diferencia, principalmente por el desfase que tiene el mapa de Unibotics respecto a la simulación que hacía que se chocase con paredes mientras que en el mapa superior debía estar en una puerta. El movimiento de mi práctica ha pasado por varias fases que han ido de más complejo hasta una muy simple pero que era capaz de hacer el recorrido. En un principio traté de usar 2 PID distintos para controlar la velocidad angular y la velocidad rectilínea. Obviando lo tedioso que fue calibrar los controladores, esta aproximación funcionaba bastante bien pero el desfase provocaba que se quedase atascado y se acumulase el error. Más adelante incorporé un mecanismo con el bumper para tratar de desatascar a la aspiradora, pero el problema era que si se atascaba en una pared mientras hacía un trazado a una casilla lejana, y hacía la maniobra para recolocarse, al terminar mantenía la velocidad y se salía de la trayectoria propuesta. Además al chocarse se acumulaba el error de la parte integral

Una vez descartada esa opción, opté por un control más sencillo que era un control puramente proporcional, regulado con el error para que no se pasase del destino. En esta etapa tenía una funcionalidad que acabé dejando de lado porque provocaba muchas colisiones. En un principio contaba con una función para simplificar el camino haciendo que si la siguiente casilla estaba alineada con la dirección del robot, podía saltar a la siguiente. El problema era que estaba pensada para un robot que seguía trayectorias ortogonales, sin necesidad de evitar obstáculos inesperados debidos al desfase del mapa.  

La versión final es: Un control proporcional donde primero se oriente hacia el siguiente waypoint, y luego 2 controles proporcionales para controlar la velocidad rectilínea y la velocidad angular para que no se desvíe, todos según el error del momento. Si durante el proceso se choca, se activa el bumper, retrocede, gira hacia el lado contrario (o hacia donde estaba girando si es de frente) y avanza un poco, y empieza el ciclo de control desde 0. Si a la hora de avanzar recto tiene una casilla libre más adelante, la velocidad máxima sube para tratar de ahorrar tiempo.

Respecto al mapa, se va a notar en las muestras cómo es diferente la erosión del planning mostrado anteriormente respecto al video de movimiento, esto se debe a que hice una modificación en la función de erosión para reducir el impacto del desfase, haciéndola más restrictiva. Primero aplico una traslación al mapa para tratar de alinearlo con el simulador, y luego sigo el mismo procedimiento que se comenta antes, el resultado es que se reducen las casillas visitables por ejemplo, la parte norte de la mesa.
