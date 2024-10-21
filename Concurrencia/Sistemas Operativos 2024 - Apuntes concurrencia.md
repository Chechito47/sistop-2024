# ***CONCURRENCIA***
## ***Threads o hilos***
Ahora vamos a introducir una nueva abstracción llamada ***threads o hilos***.

Anteriormente teníamos una noción algo pobre, asumíamos que siempre trabajabamos en un entorno single-thread, ahora vamos a hacerlo en uno multi-thread lo cual hace que tengamos mas de un punto de ejecución.

Una forma de pensar esto es como si se tratasen de procesos separados pero con una importante diferencia, comporten el mismo espacio de direccionamiento y por ende pueden acceder a la mismas direcciones e información.

### Diferencias mas en detalle
Tenemos un program counter (PC) que trackea de donde el programa esta tomando las instrucciones.

Cada thread tiene su propio set de registros. Por lo tanto si hubiera por ejemplo dos threads corriendo en un solo procesador cuando cambiemos de uno a otro se hace un context switch. El context switch entre threads es similar al de procesos, debemos guardar el estado del primer thread y restaurar el del segundo thread antes de volver a correrlo. En los procesos se guardaban en la process control block (PCB), ahora en los threads esto se guarda en el ***thread control block (TCBs)***. Notar que vamos a necesitar generalmente varios de estos. Ademas notar que cuando hacemos context switch ahora el espacio de direcciones no cambia.

Otra gran diferencia entre procesos y threads es con el stack. Si tenemos un solo thread no cambia nada, pero generalmente hay mas de uno haciendo que tengamos un multi-thread donde cada thread corre independientemente de los otros por lo que debe haber un stack para cada thread.

![ScreenShot](Imagenes/Teorico_Concurrencia/threads_simple_and_multi.png)

Entonces todo lo que pongamos en el stack como por ejemplo variables, parametros, valores de retorno y demas va a guardarse en un ***thread-local storage***.

Notar que el tener varios stacks arruina la idea de que crezca hacia arriba de forma teorica, pero en la practica los stack son pequeños por lo que no hay problema con ellos.

### ¿Por que usar threads?
Hay dos motivos principales por los cuales usar threads
#### Primer motivo: Paralelismo
Supongamos que estamos haciendo un programa que hace operaciones con arrays muy largos, si lo corremos en un solo procesador corre hasta que este listo. Si lo corremos con varios procesadores podemos disminuir el walltime. Esto se llama paralelizacion y es usual dejar un thread para que haga esto.

#### Segundo motivo: Evitar bloquear programas por I/O lentos
Imaginemos que escribimos un programa que hace varios I/O. Mientras espera la respuesta queremos que nuestro programa haga algo, usar threads es una forma de solucionar esto, el CPU scheduler puede cambiar a otro thread que esten en ready. Se conoce esto como ***threading*** lo cual activa ***overlap*** (o sea superponer I/O con otras actividades).

Obviamente podriamos usar multiples programas en lugar de threads, pero como los threads comparten el espacio de direccionamiento es mas facil compartir memoria entre ellos.

### Ejemplo de creacion de un thread
Supongamos que queremos correr un programa que crea dos threads que hacen cosas independientes, en este caso uno imprime "A" y otro "B". El codigo es el siguiente:

```c
#include <stdio.h>
#include <assert.h>
#include <pthread.h>
#include "common.h"
#include "common_threads.h"

void *mythread(void *arg) {
	printf("%s\n", (char *) arg);
	return NULL;
}
int
main(int argc, char *argv[]) {
	pthread_t p1, p2;
	int rc
	printf("main: begin\n");
	Pthread_create(&p1, NULL, mythread, "A");
	Pthread_create(&p2, NULL, mythread, "B");
	// join waits for the threads to finish
	Pthread_join(p1, NULL);
	Pthread_join(p2, NULL);
	printf("main: end\n");
	return 0;
}
```

El main crea dos threads los cuales corren en mythread() y luego uno es ejecutado o puesto en ready segun el scheduler.

Luego de crear los dos threads, main llama phthread_join() que espera que un thread particular termine y lo hace dos veces para asegurarse de que los dos threads corran antes de permitir al main que corra otra vez. Cuando lo haga se printea "main: end" y se termina.

Entonces el orden de ejecucion que tenemos es el siguiente:

![ScreenShot](Imagenes/Teorico_Concurrencia/traza_de_ejecucion_1.png)

Pero notemos que no el unico caso posible, dependiendo del schduler se pueden dar otros como por ejemplo que cuando se cree el thread corra inmediatamente:

![ScreenShot](Imagenes/Teorico_Concurrencia/traza_de_ejecucion_2.png)

Tambien podria darse el caso en que imprima primero "B" y luego "A"

![ScreenShot](Imagenes/Teorico_Concurrencia/traza_de_ejecucion_3.png)

### Shared data
Veamos un ejemplo en el cual se comparte informacion: tenemos dos threads que quieren actualizar una variable global

```c
#include <stdio.h>
#include <pthread.h>
#include "common.h"
#include "common_threads.h"
static volatile int counter = 0;
// mythread()
//
// Simply adds 1 to counter repeatedly, in a loop
// No, this is not how you would add 10,000,000 to
// a counter, but it shows the problem nicely.
//
void *mythread(void *arg) {
	printf("%s: begin\n", (char *) arg);
	int i;
	for (i = 0; i < 1e7; i++) {
		counter = counter + 1;
	}
	printf("%s: done\n", (char *) arg);
	return NULL;
}
// main()
//
// Just launches two threads (pthread_create)
// and then waits for them (pthread_join)
//
int main(int argc, char *argv[]) {
	pthread_t p1, p2;
	printf("main: begin (counter = %d)\n", counter);
	Pthread_create(&p1, NULL, mythread, "A");
	Pthread_create(&p2, NULL, mythread, "B");
}
// join waits for the threads to finish
Pthread_join(p1, NULL);
Pthread_join(p2, NULL);
printf("main: done with both (counter = %d)\n",
counter);
return 0;
```

Luego si corremos este codigo generalmente nos debe dar 2millones el cual es el resultado esperado:

```js
prompt> gcc -o main main.c -Wall -pthread; ./main
main: begin (counter = 0)
A: begin
B: begin
A: done
B: done
main: done with both (counter = 20000000)
```

Pero puede darse el caso en que pase lo siguiente:

```js
prompt> ./main
main: begin (counter = 0)
A: begin
B: begin
A: done
B: done
main: done with both (counter = 19345221)
```

O también podría pasar esto:

```js
prompt> ./main
main: begin (counter = 0)
A: begin
B: begin
A: done
B: done
main: done with both (counter = 19221041)
```

Esto pasa por algo llamado uncontrolled scheduling.

### Uncontrolled scheduling
Para entender porque se dan estos casos debemos entender el segmento de código que actualiza a *"counter"*. En este caso simplemente queremos ir agregandole 1. El assembly seria el siguiente:

```js
mov 0x8049a1c, %eax
add $0x1, %eax
mov %eax, 0x8049a1c
```

En este código la instrucción **mov** obtiene el valor de la dirección **0x8049a1c** y lo pone en el registro **eax**. Luego agregamos 1 al registro **eax** y por ultimo guardamos el contenido de **eax** en la dirección de donde lo habíamos sacado originalmente actualizando así esta.

#### Ejemplo que podría pasar
Tenemos dos threads.

Supongamos que thread 1 entra a esta región de código, carga el valor del **counter** (supongamos que vale 50) en el registro **eax**, luego agrega 1 al **eax** pero sucede un timer interrupt por lo que el OS guarda el estado del thread en el threads control block (TCBs).

Ahora supongamos que pasa algo todavía peor, el scheduler elige al thread 2 para correr, entonces ejecuta la primera instrucción y como los threads tienen sus propios registros el valor que carga es 50, luego le suma 1 y lo termina guardando de nuevo en el registro de donde lo saco. Por lo que el valor de la dirección 0x8049A1C es 51.

Ahora supongamos que hay otro context switch y volvemos con el thread 1, por lo que resume en donde lo dejo y ejecuta la ultima instrucción que le faltaba "sobrescribiendo" el valor de 0x8049A1C con 51 pero este valor es el mismo que el anterior. Entonces vemos que en este caso no estaríamos sumando como querríamos ya que nos debería haber dado 52 bajo circunstancias normales. Ahora pensemos que como este programa hace millones de sumas esto podría pasar varias veces y por eso es posible que de valores distintos en cada ejecución.

![ScreenShot](Imagenes/Teorico_Concurrencia/uncontrolled_execution.png)

#### Race condition
Con lo anterior demostramos el fenómeno conocido como ***race condition*** (o mas específicamente para este caso ***data race***) ya que los resultados dependen del timing de la ejecución. 

#### Deterministic y indeterminate
Con un poco de mala suerte podemos obtener resultados incorrectos en lugar de uno ***deterministico*** como se esperaría de una computadora. Cuando sucede que obtenemos el resultado incorrecto decimos que tenemos un resultado ***indeterministico*** ya que no sabemos cual va a ser la salida y puede variar según lo corramos.

#### Critical section
Cuando multiples threads ejecutando una parte del código puede provocar una race condition llamamos a esa parte ***critical section***, una parte del código que accede a una variable global o recurso compartido la cual no debe ser ejecutada de manera concurrente por mas de un thread.

#### Mutual exclusion
Lo que queremos para esa parte del código es una ***mutual exclusion*** lo cual nos garantiza que si un thread esta ejecutando una critical section los otros threads no podrán hacerlo.

### Atomicidad
Con "atómico" nos referimos a "una unidad" o "todo o nada".

Una forma de solucionar ese problema seria tener una instrucción mas poderosa que en un solo paso haga exactamente lo que queremos evitando así las posibles interrupciones. Por ejemplo algo así (aunque no existe esta instrucción):

```js
memory-add 0x8049a1c, $0x1
```

Vemos que una instrucción así se ejecutaría de manera atómica y por ende no podría ser interrumpida en mitad de se ejecución porque es precisamente lo que nos garantiza el hardware: cuando una interrupción ocurra la instrucción ya fue ejecutada en su totalidad o no fue ejecutada, no hay un estado medio.

### Synchronization primitives
Volviendo a nuestro caso real, no tenemos una instrucción así de poderosa por lo que debemos ejecutarlas como son:

```js
mov 0x8049a1c, %eax
add $0x1, %eax
mov %eax, 0x8049a1c
```

Podemos construir un set de lo que llamaríamos ***synchronization primitives*** la cual es una ayuda del hardware. Usando esto y con un poco de ayuda del SO podríamos construir un multi-thread code que acceda a las secciones criticas de una manera controlada y sincronizada para producir así el resultado esperado.









Volatile, es un tipo de varibale, dice que nunca podemos suponer que si escribrimos un valor por ejemplo 4, luego nunca vamos a poder que vale 4
La variable a la cual estoy marcando como volatil puede ser modifica por otro hardware. No ncesariamente lo que escriba va a permanecer constante por mas que no la toque.


mythread es una funcion

Los hilos comparten variables globales, heap y codigo del programa del que son parte

Con concurrencia puede haber muchos hilos que concurrentemente esten tocando una variable.

Condicion de carrera, es una lucha entre los hilos para ver quien agarra primero un pedazo de codigo. 

No determinismo es cuando puede dar cualquier valor, por ejemplo 1 o 2 y no se la probalbilidad de dicha cosa. Ausencia total del valor final que va tomar el programa