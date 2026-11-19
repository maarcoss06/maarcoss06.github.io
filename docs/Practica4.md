---
id: practica-4
title: 'Practica 4: Navegación global con algoritmo GPP, plan como recurso'
sidebar_position: 4
---

# Practica 4: Navegación global con algoritmo GPP, plan como recurso

# Objetivo
El objetivo de la práctica es implementar la lógica de navegación global de un coche autónomo que se mueve por un mapa de ciudad. Cuando el usuario selecciona un destino haciendo clic en el mapa, el algoritmo de planificación debe encontrar el camino más corto hasta él, evitando todo lo que no sea carretera, y el robot debe seguir ese camino de forma autónoma hasta llegar. Para ello se utiliza el método Gradient Path Planning (GPP)

# Desarrollo
Al arrancar se carga el mapa con WebGUI.getMap, que devuelve un array de NumPy en escala de grises. A partir de él se construye una máscara booleana de obstáculos: las celdas con un valor menor o igual que un umbral se consideran no transitables. Esta máscara se calcula una sola vez, ya que el mapa no cambia durante la ejecución.

Una vez se tiene la lectura del mapa se implementa la penalización. Si el camino se calculara solo con la distancia, iría pegado a los bordes de la carretera y el coche chocaría en las curvas. Por eso se calcula una matriz de penalización que suma un coste extra a las celdas cercanas a un obstáculo, mayor cuanto más cerca están y nulo a partir de cierto radio. Se calcula desplazando la máscara de obstáculos sobre todos los desplazamientos posibles dentro del radio y quedándose, en cada celda, con la penalización del obstáculo más cercano. Se ejecuta una sola vez al inicio, porque solo depende del mapa.

A continuación, se implementa el algoritmo Wave Front. Cuando se selecciona un destino nuevo se genera el campo de costes con una expansión tipo Dijkstra desde el destino, usando una cola de prioridad. Cada celda se expande a sus 8 vecinos: el coste de un paso es 1 en los movimientos ortogonales y 1,4 en los diagonales, y a ello se suma la penalización de la celda a la que se entra. Las celdas que son obstáculo no se expanden. Como los costes no son todos iguales, una celda puede recibir primero un coste y después uno menor, por lo que se actualiza cuando se encuentra un valor mejor y se descartan las entradas obsoletas de la cola. El resultado es una matriz de costes de 400x400 en la que el destino vale 0 y el coste crece al alejarse de él, de modo que bajando por los valores decrecientes se llega al destino.

En cada iteración del bucle principal, el coche obtiene su posición con HAL.getPose3d y la convierte a una celda del mapa. Desde esa celda se desciende por el gradiente un número fijo de pasos, eligiendo siempre la celda vecina de menor coste, y la celda obtenida se convierte en el punto objetivo local. Como el enunciado recomienda, el punto local se toma a una distancia suficiente para no perder las curvas. El control es proporcional. Se calcula el error entre el ángulo hacia el punto local y la orientación del coche, la velocidad angular es proporcional a ese error y la velocidad lineal disminuye al aumentar el error de giro, con un mínimo para que el coche no se detenga. Cuando la distancia al destino es menor que 3 m, el coche se detiene y el programa queda esperando a que se seleccione otro punto, sin terminar la ejecución. Al hacer clic en un nuevo destino se recalcula el campo y el coche vuelve a moverse.

# Problemas encontrados
En una primera versión, la matriz de costes era de tipo uint8 para poder mostrarla directamente. Esto truncaba los pesos decimales, el 1,4 de las diagonales pasaba a ser 1, y obligaba a limitar la expansión a un coste de 250, con lo que las celdas más lejanas quedaban sin valor y el coche podía quedarse parado. Para solucionarlo se usó una matriz de números decimales con infinito como valor de celda sin asignar, y se convirtió a uint8 solo al mostrar la imagen, normalizando el coste.

Durante el desarrollo de la práctica hubo problemas con el uso de coordenadas al trabajar con la rejilla. La función worldToGrid devuelve las coordenadas como (columna, fila), mientras que NumPy forma los arrays como [fila, columna]. Un error en esto hace que el campo se expanda por zonas equivocadas. Para solucionarlo se implementaron dos funciones, world_to_cell y cell_to_world, de modo que internamente todo el código trabaja con (fila, columna), y mostrar por la terminal las coordenadas del destino en el mundo y en la rejilla para comprobarlas.

En las primeras simulaciones el coche se chocaba con las esquinas al girar. Esto se debía a que el punto local se elegía como la celda de menor coste dentro de una ventana alrededor del coche. En una curva, esa celda podía quedar al otro lado de la esquina, y la línea recta hacia ella atravesaba la pared. Por lo que para solucionarlo, en lugar de buscar en una ventana, el punto local se obtiene bajando un número fijo de celdas por el gradiente, de forma que siempre está sobre un camino que sigue la carretera.

Otro problema que se tuvo en las primeras simulaciones fue que al llegar al destino, el coche se ponía a dar vueltas sobre si mismo hasta que se eligiese un nuevo destino. Al detectar la llegada se enviaban velocidad y giro cero, pero a continuación el programa ejecutaba igualmente el bloque de control y volvía a enviar una velocidad y un giro. Estas órdenes contradictorias, junto con un punto local cada vez más cercano que el coche sobrepasaba, hacían que orbitara alrededor del destino. Para solucionarlo, al llegar se ponen a cero las velocidades y se salta el resto de la iteración, además se usa una variable para imprimir el aviso de llegada una sola vez. El coche queda quieto hasta que se elige un nuevo destino.

# Conclusión
La práctica permite comprobar cómo un algoritmo de planificación basado en un campo de potencial puede resolver la navegación global: el wavefront calcula en una sola pasada el coste de llegar al destino desde cualquier celda, y seguir el gradiente descendente produce un camino que evita los obstáculos. La penalización cerca de las paredes y un punto local situado unas celdas por delante resultaron clave para que el coche tomara bien las curvas.

La solución cumple el objetivo de llegar al destino siguiendo el camino más corto por la carretera, con una visualización progresiva de la onda hasta el coche. Entre sus limitaciones están el recálculo completo del campo en cada destino nuevo, que bloquea el bucle durante unos instantes, y el lookahead fijo, que obliga a un compromiso entre las rectas y las curvas. Como posibles mejoras podrían plantearse un lookahead adaptativo según la curvatura del camino, el frenado antes de las curvas y optimizaciones en la expansión para reducir el tiempo de cálculo.

