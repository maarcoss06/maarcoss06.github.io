---
id: practica-3
title: 'Practica 3: Navegación local esquivando obstáculos con algoritmo VFF'
sidebar_position: 3
---

# Practica 3: Navegación local esquivando obstáculos con algoritmo VFF

# Objetivo
Implementar la lógica de navegación local con el algoritmo Virtual Force Field (VFF) para un coche equipado con un láser de 180 rayos. El destino genera una fuerza de atracción y los obstáculos una fuerza de repulsión, de modo que la dirección del robot se obtiene a partir de la combinación de ambas. El robot debe recorrer el circuito pasando por una serie de waypoints sin chocar.

# Desarrollo
El programa se basa en un bucle principal que, en cada iteración, calcula tres vectores en el sistema de coordenadas relativo al robot y los convierte en órdenes de velocidad.

El vector objetivo apunta desde el coche hasta el siguiente waypoint y representa hacia dónde se quiere ir. Se calcula obteniendo el siguiente waypoint y se pasa de coordenadas absolutas a relativas con la función absolute2relative, dada por el enunciado. De esta manera se resta la posición del coche y rota según su orientación. Así el vector apunta siempre hacia el waypoint y no hacia delante del coche.

El vector obstáculo es el resultado de sumar la repulsión que genera cada lectura del láser, con sentido opuesto al obstáculo, y representa de qué hay que alejarse. Se calcula convirtiendo los 180 rayos del laser a coordenadas polares con la función del enunciado parse_laser_data. Al leer el láser iteramos por todos los puntos detectados creando pequeños vectores en la dirección opuesta al obstáculo. La magnitud del vector debe ser inversamente proporcional al cuadrado de la distancia. La suma de todos ellos da el vector obstáculo. Para evitar errores, he ignorado aquellas lecturas no válidas y las que superen una distancia de 4 metros.

El vector dirección es el resultado de combinar los dos anteriores. Marca hacia dónde debe moverse el coche en cada instante y de él se obtienen las velocidades. Sin embargo, Como el láser solo cubre 180° por delante, la componente X del vector obstáculo es siempre negativa y tiende a anular o invertir la componente X del resultado. Por ello la componente X de la dirección se toma únicamente del vector objetivo, y el vector obstáculo actúa sobre la componente Y, que es la que controla el giro.

Una vez calculado el vector dirección calculamos laas velocidades a partir de este. La velocidad lineal es proporcional a la distancia mínima detectada en un cono frontal de ±30°. Si hay un obstáculo cerca delante, el coche frena, si el camino está libre, va a la velocidad máxima. En cuanto a la velocidad angular la calculamos haciendo la arcotangente del vector dirección y el resultado lo multiplicamos por una constante.

# Problemas encontrados
Al principio el coche avanzaba prácticamente recto, pero vibraba de forma continua. El efecto no aparecía al inicio, sino pasado un tiempo. En la primera versión la dirección se calculaba sumando directamente los vectores objetivo y obstáculo. Con ese valor, la arcotangente produce saltos bruscos de ángulo ante variaciones mínimas del vector de la coordenada y de la dirección, y por tanto una velocidad angular que cambia de signo en cada ciclo. Para solucionarlo, dejé de sumar la componente X del vector obstáculo y tomar la X de la dirección únicamente del vector objetivo, usando la componente Y para esquivar los obstáculos. Además se limitó el alcance del láser con una distancia máxima constante, para que las paredes lejanas no contaminen el vector obstáculo.

Otro problema que tuve fue que al encontrar un obstáculo el coche tardaba en girar y no llegaba a esquivarlo. Se debía a que la velocidad lineal era constante, por lo que no había tiempo de maniobrar, y el láser sin límite de alcance mezclaba obstáculos lejanos con los realmente peligrosos. Para resolverlo simplemente implementé un frenado en función de la distancia mínima frontal descartando las lecturas más allá de la distancia máxima. 

Por último el problema que más tiempo me llevó resolver fue que el coche no detectaba uno de los waypoints y por tanto se volvía loco, disminuía su velocidad y oscilaba tan rápido que parecía que temblaba. El waypoint seguía siendo el objetivo aunque ya estuviera a su espalda, de modo que el vector objetivo pasaba a apuntar hacia atrás. Para solucionarlo me di cuenta que al esquivar obstáculos, y a velocidad alta, la trayectoria se podía desviar de la línea ideal y el coche podía pasar a más de 1 metro del waypoint sin entrar en el radio de llegada. Por ello, amplié el radio de llegada a 1,5 metros y añadí una condición de waypoint superado. Para que un waypoint se considere superado debe estar detrás del coche y a una distancia menor de 4 metros. Esto evita que el coche se salte un waypoint solo porque el esté orientado en otra dirección lejos de él.

# Conclusión
El algoritmo VFF permite resolver la navegación local con muy pocos elementos: un vector de atracción, uno de repulsión y una ley de control sencilla. Sin embargo, su implementación directa tiene problemas prácticos. La suma literal de los vectores no es válida cuando el sensor solo cubre el frente del robot, y el comportamiento es muy sensible a la velocidad y a cómo se decide que un waypoint se ha alcanzado. Los cambios más determinantes fueron, adaptar la velocidad a la proximidad de los obstáculos y hacer más robusta la detección de waypoints. Con ellos el coche completa el circuito de forma estable a velocidad elevada.

# Vídeo
https://youtu.be/Sdx_d9jyvYw

