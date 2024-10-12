Volatile, es un tipo de varibale, dice que nunca podemos suponer que si escribrimos un valor por ejemplo 4, luego nunca vamos a poder que vale 4
La variable a la cual estoy marcando como volatil puede ser modifica por otro hardware. No ncesariamente lo que escriba va a permanecer constante por mas que no la toque.


mythread es una funcion

Los hilos comparten variables globales, heap y codigo del programa del que son parte

Con concurrencia puede haber muchos hilos que concurrentemente esten tocando una variable.

Condicion de carrera, es una lucha entre los hilos para ver quien agarra primero un pedazo de codigo. 

No determinismo es cuando puede dar cualquier valor, por ejemplo 1 o 2 y no se la probalbilidad de dicha cosa. Ausencia total del valor final que va tomar el programa