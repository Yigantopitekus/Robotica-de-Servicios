# Mobtecarlo Loc
En esta practica se nos pide diseñar un algoritmo de montecarlo para una aspiradora de gamma alta, que tiene que poder autolocalizarse.
## Registro
PAra poder relacionar las coordenadas del robot con los pixeles del mapa necesitamos 2 funciones para pasarlas de un tipo a otro. Podemos usar las mismas funciones que en la p1 que funcionaban con una matriz de transformación.
## Primer conjunto de particulas aleatorias
Para comenzar con el algoritmo, necesitamos generar un conjunto de particulas generadas al azar alrededor del mapa. Para ello usamos una funcion especifica para esto que usa _np.random.uniform_ para generar los pares de pixeles, que despues se pasan a coordenadas usando las funciones del registro.
Los pares se generan como pixeles, ya que conocemos las dimensiones del mapa (1012x1012), y facilita comprobar si la particula cae en un obstaculo. AL añadirlo a la lista, se pasa a coordenadas de gazebo, ya que lo requiere _WUI.showParticles()_
