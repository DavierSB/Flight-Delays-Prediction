Este paper, presenta una nueva forma de manipular los datos, para realizar predicciones continuas usando cualquier tipo de regresión sobre cuanto se van a retrasar los vuelos, tomando solo en cuenta parámetros relativos a los horarios y rutas planificados para los vuelos (o sea dejando de lado clima, cantidad de pasajeros, y demás).<br>
Para predecir un vuelo se centra en el itinerario del avión en cuestión en particular y más específicamente, en el retraso que tuvo la última operación (salida o llegada) del avión.<br>
Se agrupan los datos por avión y se ordenan de manera cronológica, para incorporar dos nuevos features a cada datapoint que están directamente relacionados con el datapoint anterior. Estos nuevos features permiten al modelo aprender la influencia de los retrasos en etapas anteriores del itinerario del avión sobre su siguiente vuelo. (mejor explicado mas abajo)<br>
Se basa en el hecho de que los retrasos de los vuelos suelen ocurrir en cascada (el retraso de unos influye sobre otros)

Usan el dataset provisto en https://www.kaggle.com/datasets/ioanagheorghiu/historical-flight-and-weather-data/code

En esencia:<br>
Para cada vuelo crean dos datapoints, uno referido a su partida y otro referido a su llegada<br>
Ordenan los datapoints por id_del_avion, y luego por fecha y hora. De esta manera todos los datapoints protagonizados por un mismo avión quedan agrupados<br>
Independientemente de que los datapoints se refieran a dos momentos diferentes, dígase la salida o la llegada, todos tienen el mismo conjunto de features, de los cuales los mencionados en el paper y de relevancia son:<br>
(Hora planificada, Orientacion(salida o llegada), origen, destino, FTD, PFD, demora)<br>
FTD y PFD son los nuevos features que consisten en:<br>
-FTD (flight time duration): el tiempo planificado que durara el avion en una determinada labor. Para los datapoints de llegada corresponde al tiempo planificado que dure el vuelo (Tiempo estimado de llegada - Tiempo estimado de salida). Para los datapoints de salida corresponde al tiempo estimado que se mantendra el avion en el suelo haciendo labores de mantenimiento o lo que sea, es decir (Tiempo estimado de salida - Tiempo estimado para la llegada anterior)<br>
-PFD (previous flight delay): corresponde a la demora que tuvo la ultima operacion realizada por el avion (en el caso de la primera operacion de cada avion el PFD es 0)<br>
O sea, de esta manera para cada avion tenemos dos datapoints por cada vuelo, uno correspondiente a los datos de su salida y uno correspondiente a los datos de su llegada. A su vez estos datapoints se encuentran separados para cada avion, de manera que seguimos la trayectoria del avio en particular, y vamos a traves del feature PFD acumulando los delays previos.<br><br>

La prediccion:<br>
Sea cual sea el modelo de regresion que se use, el objetivo es siempre predecir la demora de la siguiente partida o llegada de un avion en particular.<br>
El modelo es entrenado con los datos historicos, tratando de predecir las demoras y luego comparando el resultado contra el PFD ya calculado.<br>
Por supuesto, para los vuelos futuros que queremos predecir el PFD no se ha calculado, sino solo para el primero de ellos.<br>
Luego, para realizar un volumen grande de predicciones en cuanto a la demora de los vuelos, basta ir pasando uno a uno los datapoints de los futuors vuelos al modelo, y el modelo ira prediciendo la demora de cada datapoint (partida o llegada), que luego sera usada como PFD del siguiente datapoint en el itinerario planificado para el avion. O sea, para predecir el retraso de grandes volumenes de vuelos se limita a predecir el retraso de la siguiente operacion de cada avion.<br>
En fin, que el modelo tomara el ultimo datapoint relativo a un avion, consistente en Hora planificada, Orientacion, origen, destino, ftd y pfd y sera capza de predecir el delay. Luego ese delay sera empleado como PFD en el calculo del delay del proximo datapoint (n-veces) hasta predecir todos los delays en vuelos requeridos<br>
