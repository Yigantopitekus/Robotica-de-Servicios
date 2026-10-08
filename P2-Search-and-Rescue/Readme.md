# Dron en un Search and Rescue (SAR)
En esta practica el objetivo es programar un dron para hacer un reconocimiento en una zona donde se cree que hay personas naufriagadas.
## Traduccion de coordenadas
Lo primero de todo es traducir las coordenadas globales proporcionadaas a unos valores mas manejables. En este caso usamos un traductor online para pasar de GPS a UTM (Northing,y Westing). Sin embargo estas coordenadas son globales, y no son aplicables directamente en nuestro dron, para obtener las coordenadas locales de la zona de rescate debemos restar las coordenadas globales del origen a las globales de la zona de rescate.
## Movimiento
La primera fase del trabajo ha sido el iniciar el movimiento hacia la zona. Para esto lo primero es hacer despegar el dron una distancai minima para que la plataforma base no estorbe, despues la idea es acercarlo a la zona de rescate y una vez ahi empezar a patrullar. Al compienzo de la practica plantee usar el cmd_pos para hacerlo mas sencillo pero era muy lento,al final he optado por hacer una funcion que dado una posicion, aplica un control proporcional a la distancia restante.
