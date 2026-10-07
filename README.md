# maarcoss06.github.io

# Practica 1: Navegación pseudoaleatoria con FSM en una aspiradora de gama baja

# Desarrollo

<img width="611" height="803" alt="Captura desde 2026-10-06 20-30-13" src="https://github.com/user-attachments/assets/8ff5a312-8fc4-43a8-81f5-1630f6113dfa" />

El software de la aspiradora está compuesto de cuatro estados. El primer estado es el SPIRAL, sirve para limpiar el máximo posible al comienzo de la simulación. Cuando el robot detecta con el laser un obstáculo a menos de 0,4 metros se detiene y entra el segundo estado: BACKWARD. El robot retrocede para separarse del obstáculo hasta que está a más de 0,7 metros del obstáculo. Cuando llega a ese punto se cambia al tercer estado: SPINNING. El robot gira sobre si mismo durante un tiempo aleatorio entre 1 y 2 segundos para dar que sea una navegación pseudoaleatoria tal y como pide la práctica. El último estado es FORWARD. El robot avanza en linea recta hasta que el laser lee un obstáculo y vuelve al segundo estado. De esta manera que sea un bucle infinito como pide la práctica.

# Problemas encontrados
Durante el desarrollo de la práctica me he encontrado con algunos problemas. Al hacer el estado SPIRAL no era capaz de que el robot hiciese la espiral. Al principio el radio de la circunferencia era muy grande por lo que la aspiradora intentaba hacer un círculo y se chocaba debido al valor del radio. La solución que implementé fue disminuir la velocidad lineal para disminuir el radio de la circunferencia. Sin embargo, apareció otro problema. Tenía que aumentar el radio poco a poco para que la aspiradora hiciese espirales y no círculos ya que el objetivo era limpiar el máximo espacio posible antes de detectar un obstáculo. Finalmente la solución que implementé fue aumentar ligeramente la velocidad lineal en cada iteración. De esta forma el radio crecía progresivamente lo que permitía al robot hacer espirales.

Por último, al hacer simulaciones el robot podía recorrer sitios con muchos obstáculos a su alrededor y que fuese muy difícil salir de esos sitios de manera que no podía limpiar más zonas quedándose bloqueado. Este problema no tiene solución ya que es parte de la práctica y al no tener un algoritmo sólido el robot recorre la casa de forma aleatoria.

# Vídeo

https://youtu.be/D-n8tlO7qMo

