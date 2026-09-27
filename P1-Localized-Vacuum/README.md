# Introduccion
This repository shows the development behind the design of a robotic vacuum which counts with localization algorithms and sistematic swipes
# Crear Transformacion de coordenadas a pixeles 
Para este paso se necesita recabar 10 puntos con sus pixeles aproximados obtenidos del png del mapa, para obtener esos valores he creado  un script de python que muestra el mapa en pantalla y va imprimiendo por la terminal las coordenadas de el pixel señalado en la imagen, como se muestra en esta demostracion

https://github.com/user-attachments/assets/e158e50a-f584-44ee-932c-d5da1bcbb27e


El siguiente paso es obtener la matriz de transformacion para relacionar la posicion en coordenadas xy del mapa con los pixeles uv de la imagen, de nuevo, usamos un script de python independiente para obtener la transformada afin de los 10 puntos seleccionados, en el enunciado se recomienda usar una matriz de trnsformacion homogenea distinta, pero como es una transformacion 2D (no necesitamos z) con la matriz afin basta. Nos ayudamos de la biblioteca de CV2 para esto, que cuenta con una funcion *cv2.estimateAffine2D* para esto. Mi matriz de transformacion resultante es la siguiente:

<img width="331" height="80" alt="image" src="https://github.com/user-attachments/assets/1ecb8dde-a6f1-4e9a-b46e-37defcf9b288" />
Que se traduce en:

<img width="313" height="84" alt="image" src="https://github.com/user-attachments/assets/5ad87759-9b5e-4fa5-8da7-eff1ddde9ff4" />


Con estos datos ya se puede relacionar la posición en pixeles 2D a las coordenadas xy del robot resolviendo el sistema de ecuaciones.

# Creación de las casillas del mapa
Primero, el mapa se tiene que dividir en obstaculo (fisico o virtual) y espacio navegable, que se representan como negro y blanco respectivamente. 
En esta etapa me entretuve porque al leer que habia que añadir cierta erosión preventiva al mapa (para no chocar con las paredes o pasar muy cerca etc) decidi añadirlo de golpe a toda la imagen usando OpenCV usando un kernel que se aplicaba a todos los pixeles y "engordaba" las paredes. No resulto muy bien porque dificultaba mucho el cuadrar despues las rejillas, que dejaban pequeños huecos o zonas muy estrechas donde el robot no podria pasar. Tras descartar este metodo, pase con el que me quede finalmente que es: Recorrer toda la imagen (1012x1012) en regiones de 35x35 pixeles que es el tamaño exacto del robot proporcionado por el enunciado. En cada region se hace recuento de los pixeles blancos y los negros y si hay mas de cierto umbral de negros se rellena la casilla entera. Asumiendo que el robot no podria pasar comodamente por el centro de la casilla. 
Una vez recorridas todas las regiones se representan las casillas visualmente con lineas negras, de nuevo con la libreria OpenCV, como se pintan encima de las regiones, las casillas acaban siendo un poco mas pequeñas que la aspiradora, que seria lo ideal.
<img width="1012" height="1012" alt="image" src="https://github.com/user-attachments/assets/c7ddadc3-78b1-4315-a5fe-ae286d447fb4" />

Una vez hecho esto, hay que decidir como se van a representar las casillas dependiendo de su estado durante la ejecucion, en mi caso elegi:

- Naranja: No visitada
- Verde: Visitada (esta no se implementa hasta implementar el movimiento)
- Azul claro: Puntos de retorno
- Rojo: Puntos Criticos
- Azul Oscuro: Casilla de inicio

# Planificacion
Como bien hemos dado en clase para la planificacion se va a usar el Backtracking Spiral Algorithm (BSA), este algoritmo es un algoritmo de cobertura completa, usa barridos sistematicos en forma de espiral siguiendo una prioridad establecida, en mi caso ESWN (Este,Sur,Oeste,Norte). El funcionamiento basico de este algoritmo (usando mi orden de prioridad como ejemplo) es el siguiente:

El robot empieza el movimiento hacia E, mientras no se tope con nada (obstaculo o casilla visitada) sigue en esta dirección. Si se topa con algo cambia la direccion hacia S y hace lo mismo, la diferencia es, segun q direccion lleve, va a comprobar primero una direccion u otra. Es decir, cuando se "choca" en E, primero comprueba S,luego W y por ultimo N, pero si se "choca" en S, primero comprueba W,luego N y por ultimo E. Este procedimiento lo ejecuta hasta que no queden casillas libres esto se denomina punto critico. 
Esta situacion tiene una facil solucion, durante el recorrido de la espiral va anotando que casillas vecinas fuera de la trayectoria estan libres, para por si se queda atascado volver a una de ellas, estos son los puntos de retorno

La función planificacionse ha planteado como una funcion recursiva por cada espiral, en cada llamada se pasa a traves de un bucle for de 4 iteraciones par buscar a que direccion moverse, siguiendo la prioridad, si encuentra una casilla libre, se mueve y vuelve a llamar a la funcion, si no encuentra ninguna casilla libre pasa a la siguiente direccion y pasa a la siguiente iteracion. Cuando termina la espirar la recursividad termina. Para evitar que se pare la planificacion, la llamada a spiral() comienza desde otra plan(), no para de llamar a spiral() hasta que se acaben los puntos de retorno. Durante este proceso se van pintando las casillas. A continuacion se puede ver como queda el plan finalmente.

https://github.com/user-attachments/assets/969b8a74-390e-4e60-94d4-d1580c064e77

El plan se genera mucho mas rapido, pero para su visualizacion he puesto un time.sleep()

Durante el desarrollo de esta parte he tenido algunos problemas con versiones anteriores donde no implementaba correctamente del todo el BSA, en un principio el algoritmo no era recursivo sino que se basaba en una unica llamada que aunque hacia un barrido completo y eficiente no era BSA. En esta versión la prioridad se implentaba de forma absoluta, con esto me refiero a que buscaba siempre la opcion de mayor prioridad basandose siempre en el orden ESWN, lo que reslutaba en barridos. Esta opcion no estaba tan lejos de lo que tengo actualmente si hubiera que la prioridad se buscase según la direcciona cual del robot, como he explicado antes. El cambio a una funcion recursiva ha surgido porque como no me funcinaba esta version, me puse a investigar y me salio como ejemplo de explicacion una version recursiva, que decidi implementar a la practica porque me gusto la idea. A continuacion se pueden ver algunos de los planes que generaban estas versiones pasadas.


https://github.com/user-attachments/assets/765c7420-08b6-4bad-b54e-7b7aec0c27b6



https://github.com/user-attachments/assets/0012d44f-de52-4ffc-ae76-57e80cc94be7

En la verion del barrido, hay una funcionalidad que no he implementado en el planificador final que es que haya un maximo de puntos de retorno, en un principio solo se iba a quedar con los mas recientes para evitar un exceso de ellos. En esa version que se recorria toda la sala poco a poco no habia problema, pero con la version final el robot suele irse a distintas partes del mapa dejando algunas zonas sin pasar por lo que si hubiese un maximo de puntos de retorno estos se perderian y esas zonas se quedarian sin limpiar.

## Busqueda de puntos de retorno
Para la busqueda de puntos de retorno uso 2 metodos principales, el primero es comprobar que se pueda ir directamente en linea recta, se traza una linea hasta los 3 puntos  de retorno mas cercano ( medido por la distancia euclidia )  y se comprueba si se puede llegar de manera directa alguno de ellos. Si no duese posible, se aplica el algoritmo de busqueda Breadth-First-Search, para encontrar una ruta hasta el punto de retorno mas cercano. Este algoritmo es basicamente ir comprobando recursivamente desde la casilla de inicio (punto critico donde se encuentra el robot) las casillas vecinas hasta encontrar un punto de retorno registrado, y guardar las casillas y el orden en el que se han comprobado apra establecer la ruta. 

<img width="612" height="510" alt="bfs_return_point" src="https://github.com/user-attachments/assets/d2c121bc-9c34-4bb6-8e6d-1dff13f08aac" />

# Movimiento
Esta fase ha sido la que mas se me ha complicado con diferencia, principalmente por el desfase que tiene el mapa de Unibotics respecto a la simulacion que hacia  que se chocase con paredes mientras que en el mapa superior debia estar en una puerta. EL movimiento de mi practica ha pasado por varias fases que han ido de mas complejo hasta una muy simple pero que era capaz de hacer el recorrido. En un principio trate de usar 2 PID distintos para controlar la velocidad angular y la velocidad rectilinea. Obviando lo tedioso que fue calibrar los controladores, esta aproximacion funcionaba bastante bien pero el desfase provocava que se quedase atascado y se acumulase el error. Mas adelante incorpore un mecanismo con el bumper para tratar de desatascar a la aspiradora, pero el problema era que si se atascaba en una pared mientra hacia un trazado a una casilla lejana, y hacia la maniobra para recolocarse, al terminar mantenia la velocidad y se salia de la trayectoria propuesta. Ademas al chocarse se acumulaba el  error de la parte integral

Una vez descartada esa opcion, opte por un control mas sencillo que era un control puramente proporcional, regulado con el error para que no se pasase del destino. En esta etapa tenia una funcionalidad que acabe dejando de lado porque provocaba muchas colisiones. En un principio contaba con una funcion para simplificar el camino haciendo que si la siguiente casilla estaba alineada con la direccion del robot, podia saltar a la siguiente. EL problema era que estaba pensada para un robot que seguia trayectorias ortogonales, sin necesidad de evitar obstaculos inesperados debidos al desfase del mapa.  

La version final es: Un control proporcional donde primero se oriente hacia el siguiente waypoint, y luego 2 controles proporcionales para controlar la velocidad rectilinea y la velocidad angular para que no se desvie, todos segun el error del momento. SI durante el proceso se choca, se activa el bumper, retrocede, gira hacia el lado contrario (o hacia donde estaba girando si es de frente) y avanza un poco, y empieza el ciclo de control desde 0. Si a la hora de avanzar recto tiene una casilla libre mas adelante, la velocidad maxima sube para tratar de ahorrar tiempo.

Respecto al mapa, se va a notar en las musetras como es diferente la erosion de el planning mostrado anteriormente respecto al video de movimiento,esto se debe a que hize una modificacion en la funcion de erosion para reducir el impacto del desfase, haciendola mas restrictiva. Primero aplico una traslacion al mapa para tratar de alinearlo con el simulador, y luego sigo el mismo procedimiento que se comenta antes, el resultado es que se reduzan las casillas visitables por ejemplo, la parte norte de la mesa. 
