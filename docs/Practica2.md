---
id: practica-2
title: 'Práctica 2: Control Control Visual con PID de un Fórmula1'
sidebar_position: 2
---

# Practica 2: Control Visual con PID de un Fórmula1

# Desarrollo
El objetivo del control visual es que el robot vea donde está la línea roja respecto a su propio centro. 
El primer paso para conseguir este objetivo es hacer una conversión de color. HAL.getImage() nos da una imagen en formato BGR, la cual es muy delicada a los cambios de luz y sombras por lo que para que sea más sencillo convertimos la imagen a formato HSV que es mucho más resistente ante estas características. A continuación, creamos la máscara, con cv2 y los límites de color establecidos filtramos la imagen. De esta manera Todo lo que sea de color rojo se vuelve un píxel blanco y el resto se vuelve negro. Sin embargo, analizar toda la imagen no tiene sentido ya que la línea roja solo se ve de mitad para abajo por lo que recortamos la imagen por la mitad. Esto permite anticipar la curva sin que el horizonte o las líneas lejanas confundan al coche. Por último se calcula el centroide para poder determinar el error, es decir, la diferencia entre el centro ideal de la cámara y dónde está realmente la línea roja.

Una vez que sabemos cuántos píxeles de desviación tenemos, necesitamos calcular cuánto hay que girar el volante (velocidad angular w). Lo primero es calcular proporcional. Para ello se multiplica el error actual por la constante Kp. A mayor desvío, mayor volantazo. Es el encargado de meter el coche en la curva, pero si actúa solo, el coche oscilaría o se pasaría de largo. Después calculamos derivativo. Compara el error actual con el error del fotograma anterior para saber a qué velocidad te estás desviando. Multiplicado por Kd, actúa como un freno. Si el coche se está acercando rápidamente al centro de la línea, el derivativo contrarresta al proporcional para clavar el morro en el centro exacto sin pasarse al lado contrario evitando el oscilamiento que se había comentado antes. Por último, hallamos el integral. Suma todos los errores pasados. Multiplicado por Ki, ayuda en curvas muy largas y cerradas donde el Proporcional se queda sin fuerza. La suma de estas tres fuerzas da la velocidad angular final, es decir determina cuánto hay que girar el volante para no salirse de la línea blanca o volver a la línea blanca en caso de desvío.

Para hacer el coche más rápido en rectas sin que se saliera en las curvas, he implementado un control de velocidad. el robot evalúa qué tan brusco es el giro que ha calculado el PID. Si el giro es fuerte, se aplica un freno agresivo al instante. De esta forma el coche es rápido sin salirse de la trazada.

# Problemas encontrados
El primer problema que tuve fue el cálculo del centroide. Al principio el cálculo del centroide se hacía procesando la máscara binaria de toda la imagen capturada por la cámara. Esto generaba un comportamiento errático, ya que el horizonte de la imagen provocaba un desvío en el centroide calculado. Como consecuencia, el coche giraba antes de tiempo o calculaba mal la curvatura. Para solucionarlo, en lugar de analizar toda la imagen, se recortó la máscara para aislar exclusivamente una franja horizontal por delante del morro del vehículo. Al calcular los momentos sobre esta franja el robot siguió la línea de forma uniforme. 

El segundo problema surgió al implementar el controlador PID. Al principio el coche sufría fuertes oscilaciones al salir de las curvas, lo que provocaba que en las rectas hiciese una trayectoria de zig zag. Este problema fue provocado por Ki. Durante giros prolongados, la variable acumuladora de errores crecía hasta alcanzar valores muy grandes. Al terminar la curva, esa acumulación seguía forzando el giro en la dirección contraria, desestabilizando el control. Para solucionar este efecto, se diseñó un mecanismo de reseteo. Se añadió una condición lógica en el código que evalúa si el error cambia de signo respecto al fotograma anterior, lo que significa que el morro del coche acaba de cruzar el centro exacto de la línea. En ese instante, la suma acumulada de errores se reinicia a cero, acabando así con las oscilaciones al entrar en las rectas.

Por último, con el objetivo de mejorar el tiempo por vuelta, se aumentó la velocidad lineal en las rectas. Sin embargo, al ir tan rápido, el coche no tenía tiempo de frenar y se salía de la trazada. Para solucionarlo, en lugar de usar velocidades estáticas como tenía al principio una para las rectas y otra para las curvas, se relacionó la velocidad lineal con la velocidad angular. Dependiendo de lo mucho que haya que girar el volante la velocidad aumenta o disminuye. De esta manera en las rectas el coche consigue una velocidad máxima uniforme y en las curvas dependiendo de lo cerrada que sea disminuye más o menos la velocidad.

# Vídeos
# Circuito Simple

https://youtube.com/shorts/8FCI7DqttTU

# Circuito Montreal
https://youtube.com/shorts/gJ4dcnNUt9U
