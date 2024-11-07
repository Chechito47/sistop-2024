# ***CONCURRENCIA***

## ***Anotaciones sueltas***
### Volatile
Volatile, es un tipo de varibale, dice que nunca podemos suponer que si escribrimos un valor por ejemplo 4, luego nunca vamos a poder que vale 4
La variable a la cual estoy marcando como volatil puede ser modifica por otro hardware. No ncesariamente lo que escriba va a permanecer constante por mas que no la toque.

### Mythread
mythread es una funcion

### Hilos
Los hilos comparten variables globales, heap y codigo del programa del que son parte
Con concurrencia puede haber muchos hilos que concurrentemente esten tocando una variable.

### Race condition
Condicion de carrera, es una lucha entre los hilos para ver quien agarra primero un pedazo de codigo. 

### No deterministico
No determinismo es cuando puede dar cualquier valor, por ejemplo 1 o 2 y no se la probalbilidad de dicha cosa. Ausencia total del valor final que va tomar el programa

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

***Intercambiar de hilos es muy barato, solo hay que cambiar el CR3.*** Simplemente ahora el stack es otro.

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

### Creacion de threads
La primera cosa que hay que hacer para escribir programas multi-thread es crear nuevos threads, se hace de la siguiente manera:

```c
#include <pthread.h>
int
pthread_create(pthread_t    *thread,
        const pthread_attr_t *attr,
        void    *(*start_routine)(void*),
        void    *arg);
```

Vemos que hay 4 argumentos:

- ***thread***: puntero a una estructura de tipo **pthread_t**, estructura la cual usamos para interactuar con este thread por lo que necesitamos pasarlo a **pthread_create()** para inicializarlo.
- ***attr***: se usa para especificar cualquier atributo que el thread pueda tener *por ejemplo el tamaño del stack o informacion sobre prioridad del planificador en el thread.* Generalmente le pasamos el valor NULL.
- ***start_routine***: es la mas compleja, nos debemos preguntar: ¿Que funcion debe comenzar a correr el thread?. En C llamamos a esto funcion puntero (o function pointer). Le pasamos void como argumento y devuelve void. (Tener un puntero void como argumento de esta funcion nos permite pasarle cualquier tipo de argumento. Tenerlo como return nos permite que el thread haga return de cualquier valor)
- ***arg***: es el argumento exacto que va a ser pasado a la funcion cuando el thread comience.

#### Ejemplo
En este ejemplo creamos un thread, le pasamos dos argumentos empaquetados (*myarg_t*). Una vez creado el thread simplemene casteamos su argumento con el tipo que espera y asi desempaquetamos.

Cuando creamos un thread tenemos otra live execution entity con su propio stack, corriendo en el mismo espacio de direcciones con los otros threads.

```c
#include<pthread.h>
#include<stdio.h>

typedef struct {
    int a;
    int b;
} myarg_t;

void *mythread(void *arg) {
    myarg_t *args = (myarg_t *) arg;
    printf("%d %d\n", args->a, args->b);
    return NULL;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    myarg_t args = { 10, 20 };
    
    int rc = pthread_create(&p, NULL, mythread, &args);
    ...
}
```

### Esperar hasta que un thread termine
Si queremos esperar hasta que un thread termine debemos usar la rutina ***pthread_join()***:

```c
int pthread_join(pthread_t thread, void **value_ptr);
```

Esta rutina toma dos argumentos:

- ***phtread_t***: es usado para especificar por que thread esperar. Es inicializada por el thread creation routine (*pthread_create()*).
- El segundo argumento es un puntero hacia el valor de retorno que esperamos obtener. Como la rutina puede devolver cualquier cosa usamos void. Como *pthread_join()* routine cambia el valor del argumento pasado necesitamos pasar otro en un puntero hacia ese valor, no el valor en si y por eso es que usamos otro void.

#### Ejemplo 2
Veamos otro ejemplo: un solo thread es creado y se le pasa un par de argumentos via la estructura **myarg_t**. Para los valores de retorno el tipo **myret_t**. Una vez que el thread termine de correr el main thread que estaba esperando dentro de **pthread_join()** hace return y podemos acceder a los valores del thread que esta nombrado en **myret_t**.

```c
typedef struct { int a; int b; } myarg_t;
typedef struct { int x; int y; } myret_t;

void *mythread(void *arg) {
    myret_t *rvals = Malloc(sizeof(myret_t));
    rvals->x = 1;
    rvals->y = 2;
    return (void *) rvals;
}

int main(int argc, char *argv[]) {
    pthread_t p;
    myret_t *rvals;
    myarg_t args = { 10, 20 };
    Pthread_create(&p, NULL, mythread, &args);
    Pthread_join(p, (void **) &rvals);
    printf("returned %d %d\n", rvals->x, rvals->y);
    free(rvals);
    return 0;
}
```

Notemos algunas cosas:

1. Primero que algunas veces no es necesario empaquetar y desempaquetar los argumentos. Podriamos crear el thread sin argumentos pasandole NULL.
2. Si estamos pasando solo un valor (por ejemplo un long int) no es necesario empaquetarlo como argumento. 
3. Tenemos que ser muy cuidados con como los valores retornan del thread, especialmente nunca retornar un puntero que refiera a algo alojado en el thread's call stack.

## Locks
Los locks son funciones caras en rendimiento que proveen exclusión mutua a una región. Las rutinas mas básicas son:

```c
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

Cuando tenemos una región de código que es una ***región critica*** y necesitamos protegerla para asegurar la operacion los locks son utiles. Por ejemplo podemos hacer lo siguiente:

```c
pthread_mutex_t lock;
pthread_mutex_lock(&lock);
x = x + 1; // or whatever your critical section is
pthread_mutex_unlock(&lock);
```

Si ningun otro thread tiene el lock cuando *pthread_mutex_lock()* es llamado el thread va a obtener el lock y entrar a la seccion critica. Si otro thread intenta hacer obtener el lock, el mismo no va a retornar hasta que lo obtenga lo cual implica que para ello el thread que tenia el lock debe haberlo dejado via **unlock**. Por lo tanto los thread pueden quedarse trabados esperando.

Aunque debemos notar algunas cosas, ya que el codigo anterior esta mal por dos importantes razones:

1. Falta de inicializacion, debemos iniciar el lock y se puede hacer de dos maneras:

	 - Usando **PTHREAD_MUTEX_INITILIZER**:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

setea el lock a los valores de defecto y lo hace usable.

- 
	- De manera dinamica haciendo una llamada a **pthread_mutex_init()**

```c
int rc = pthread_mutex_init(&lock, NULL);
assert(rc == 0); // always check success!
```

Donde el primer argumento de la rutina es la direccion del lock en si y el segundo es set de atributos el cual es opcional.

Aunque ambas formas funcionan ***normalmente usamos la forma dinamica***. Notemos tambien que es necesario realizar una llamada a **pthread_destroy()** cuando terminamos de hacer lo que queriamos con el lock.

2. El segundo problema con el codigo es que falla al chequear errores, habria que hacer algo asi:

```c
// Keeps code clean; only use if exit() OK upon failure
void Pthread_mutex_lock(pthread_mutex_t *mutex) {
    int rc = pthread_mutex_lock(mutex);
    assert(rc == 0);
}
```

### Condition variables
El otro componente mayor de la libreria threads son las ***condition variables*** las cuales son utiles cuando queremos señalizar algo entre threads, por ejemplo si un thread esta esperando que otro haga algo antes de continuar. Las dos rutinas para esto son:

```c
int pthread_cond_wait(pthread_cond_t *cond,
                      pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

La primera rutina **pthread_con_wait**() pone el thread llamado a dormir esperando que otro thread lo señalice lo cual usualmente se da cuando algo en el programa cambio y que el thread que se durmio debe tener en cuenta. 

#### Ejemplo
Un ejemplo de uso tipico es el siguiente:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

Pthread_mutex_lock(&lock);
while (ready == 0)
    Pthread_cond_wait(&cond, &lock);
Pthread_mutex_unlock(&lock);
```

En este codigo luego de inicializacion del lock y de la condicion, un thread verifica si la variable **ready** es distinto que 0. Si no lo es el thread llama a la rutina wait para dormir el trhead hasta que otro thread lo despierte.

#### Wakeup thrad
El codigo para despertar un thread es el siguiente:

```c
Pthread_mutex_lock(&lock);
ready = 1;
Pthread_cond_signal(&cond);
Pthread_mutex_unlock(&lock);
```

Notemos algunas cosas de esta secuencia de codigo:

1. Cuando señalizamos debemos asegurarnos de tener el lock para asi evitar una race condition.
2. Vemos que la wait call toma el lock como segundo parametro mientras que el signal call toma solo un parametro. El motivo de esto es que la wait call ademas de hacer que el thread se duerma libera el lock cuando lo pone a dormir. Luego cuando se despierta recupera el lock con **pthread_cond_wait()**.
3. El thread que queda en waiting re-chequea la condition con un loop while.

### Compilacion y como correr
Incluimos la libreria **pthread.h**. Luego en la link line debemos especificar el link con la pthread library agregando la flag **-pthread**. Por ultimo para correrlo hacemos:

```js
prompt> gcc -o main main.c -Wall -pthread
```

## ***Locks***
### Introducción
Un lock es una variable y por eso es necesario declararlo (por ejemplo con mutex). Estos guardan el estado de lock en todo momento los cuales son **available(unlocked o free)** o **acquired(locked o held)**. También pueden guardar otra información como que trhead tiene el lock o una lista para el orden en que se hará acquired pero esta información esta escondida del usuario.

### Como funcionan
El funcionamiento de **lock()** y **unlock()** es sencillo: Llamando a *lock()* tratamos de adquirir el lock, si ningún otro thread lo tiene (o sea si esta free) lo acquire (adquirimos) y entramos a la seccion critica (esto generalmente se conoce como **owner del lock**). Si otro thread hace lock() de la misma variable de lock no va a retornar mientras el lock lo tenga otro thread.

Una vez que el owner hace *unlock()* el lock esta disponible. Si ningún otro thread esta esperando por el lock se pone en estado free. En cambio si hay threads esperando (estancados en lock() esperando) eventualmente notaran o serán informados que el lock esta libre y lo tomaran.

Para usarlos agregamos un poco de código alrededor de la sección critica:

```c
lock_t mutex; // some globally-allocated lock ’mutex’
...
lock(&mutex);
balance = balance + 1;
unlock(&mutex);
```

### Pthreads
El nombre que se la da a los locks en POSIX es ***mutex***:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

Pthread_mutex_lock(&lock); // wrapper; exits on failure
balance = balance + 1;
Pthread_mutex_unlock(&lock);
```

En lugar de tener un bloque grande que es usado cada vez que accedemos a la sección critica es mejor tener distintos locks que protejan la información permitiéndonos que varios threads tengan lock a la vez.

### Contruir un lock
Para construir un lock es necesario ayuda del hardware y del SO. Veremos la ayuda del SO mas en detalle.

### Evaluar el lock
Antes de propiamente construir el lock es necesario evaluarlo para saber si sera útil o no. El criterio que usaremos consta de 3 puntos claves:

- **Mutual exclusión**: Tenemos que asegurarnos que el lock previene que múltiples threads accedan al lock asegurando la exclusión mutua.
- **Fairness**: Hay que asegurarse que todos los threads que quieren el lock eventualmente lo obtengan y ninguno sufra de **starvation**.
- **Performance**: Usar lock empeora el rendimiento pero hay que ver cuanto porque hay varios casos. Un caso es del **no contention** donde un solo thread esta corriendo adquiere el lock y luego lo suelta. Otro es el caso en que múltiples threads están compitiendo por el lock con un solo CPU. También esta el caso en que haya múltiples CPUS... 

### Controlar las interrupciones 
Una de las primeras y mas primitivas soluciones para asegurar la exclusion mutua era desactivar las interrupciones en las secciones criticas, fue inventado para procesadores con un solo CPU.

```c
void lock() {
    DisableInterrupts();
}
void unlock() {
    EnableInterrupts();
}
```

De esta manera nos aseguramos que el código dentro de la sección critica no sera interrumpido y que se ejecutara como si fuera atómico. Cuando se termine le ejecución de la sección critica activamos de nuevo las interrupciones.

Lo positivo de esta implementacion es su sencillez pero tiene muchos **puntos negativos**:

- La implementacion requiere una operación privilegiada por lo que es necesario confiar en el código.
- No funciona con múltiples CPUS
- Desactivar las interrupciones por largos periodos de tiempo puede provocar fallos o perdida de información en el sistema.

### Usar loads y stores (Fallido)
Veamos una implementacion fallida la cual nos dará una idea general. Para ello usaremos una sola variable y accederemos a ella via loads y stores. El código es el siguiente:

```c
typedef struct __lock_t { int flag; } lock_t;

void init(lock_t *mutex) {
    // 0 -> lock is available, 1 -> held
    mutex->flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex->flag == 1) // TEST the flag
         ; // spin-wait (do nothing)
    mutex->flag = 1;
 // now SET it!
}

void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```

La idea es sencilla, usar la variable flag para indicar si algún thread tiene el lock o no. El primer thread que entre a la sección critica llama a lock() el cual verifica si la flag es 1 o no, si no lo es la setea en 1 para indicar que ahora tiene el lock. Cuando termina llama a unlock y pone flag en 0. Si otro thread llama a lock cuando no esta libre entra en un **spin-wait** hasta que lo este.

El código tiene dos problemas, uno de **correctness** y otro de **performance**:

Imaginemos que el código se intercala de la siguiente manera cuando flag=0

![ScreenShot](Imagenes/Teorico_Concurrencia/locks_failed_attempt.png)

Vemos que se produce un caso en que ambos threads setean flag=1 y por ende ambos entran a la seccion critica.

En cuanto el problema de performance se da cuando un thread espera que otro libre el lock porque queda infinitamente chequeando el valor de flag, queda en lo que se conoce como un **spin-wait**.

### Construir spin locks con Test-And-Set
Porque las implementacion anteriores no fueron utiles fue necesario crear otra, esta se conoce como la instruccion **test-and-set(o atomic exchange)**, funciona con el siguiente codigo:

```c
int TestAndSet(int *old_ptr, int new) {
	int old = *old_ptr; // fetch old value at old_ptr
	*old_ptr = new;     // store ’new’ into old_ptr
	return old;         // return the old value
}
```

Devuelve el valor viejo apuntado por `old_ptr` y simultáneamente lo actualiza al nuevo valor `new`. Se llama test-and-set porque nos permite testear el valor antiguo a la vez que seteamos el lugar de memoria del nuevo valor por lo que es una instruccion lo suficientemente poderosa como para construir un **spin lock**. Entonces veamos un ejemplo de ello:

```c
typedef struct __lock_t {
	int flag;
} lock_t;

void init(lock_t *lock) {
	// 0: lock is available, 1: lock is held
	lock->flag = 0;
}

void lock(lock_t *lock) {
	while (TestAndSet(&lock->flag, 1) == 1)
		; // spin-wait (do nothing)
}

void unlock(lock_t *lock) {
	lock->flag = 0;
}
```

Imaginemos un caso en que llamamos a lock() y esta libre por lo que flag=0. Cuando el thread llame a TestAndSet(flag=1) la rutina devolvera el valor viejo de flag el cual es 0 haciendo que el thread que estaba llamando a lock el cual ahora esta testando el valor de flag no caiga en un spin-wait y adquiera el lock. Finalmente cambia el valor a 0 y hace sus cosas hasta que libera el lock y pone flag en 0.

El segundo caso que nos podemos imaginar es que un thread ya tiene el lock. En este caso el thread que quiere el lock llama a lock() y a TestAndSet(flag, 1) pero esta vez este ultimo devuelve el valor viejo de flag el cual es 1 porque el lock esta ocupado a la vez que pone flag en 1 de nuevo. Hasta que no se libere el lock, flag sera 1 cayendo en un spin hasta que se libere. Cuando se libera sucede todo con normalidad.

Entonces haciendo el test del valor viejo y actualizando al nuevo valor en la misma operacion nos aseguramos que sea atomico.

#### Spin lock
Se llama spin lock porque esta en spin consumiendo CPU hasta que el lock este free.

### Peterson's algorithm
Este es un algoritmo que solucino los problemas con los locks:

```c
intintflag[2];
turn;

void init() {
	// indicate you intend to hold the lock w/ ’flag’
	flag[0] = flag[1] = 0;
	// whose turn is it? (thread 0 or 1)
	turn = 0;
}
void lock() {
	// ’self’ is the thread ID of caller
	flag[self] = 1;
	// make it other thread’s turn
	turn = 1 - self;
	while ((flag[1-self] == 1) && (turn == 1 - self))
		; // spin-wait while it’s not your turn
}
void unlock() {
	// simply undo your intent
	flag[self] = 0;
}
```

Posteriormente cayo en deshuso.

### Evaluemos el spin lock
Evaluemos la implementacion del spin lock propuesta (no la de peterson, la anterior):

Lo mas importante es si cumple el correctness o sea si da exclusion mutua lo cual es verdadero.

El siguiente punto es fariness, o sea si podemos asegurar los threads no sufran de starvation lo cual falla ya que es posible que un thread quede en spin infinitamente.

El ultimo punto es la performance. Aca nos tenemos que imaginar distintos escenarios. Primero supongamos que tenemos solo una CPU por lo que el rendimiento sera malo ya que mientras que un thread tiene el lock y esta en la seccion critica y otros threds quieren el lock estos tendran que ejecutarse para decirles que no pueden obtener el lock consumiendo ciclos de CPU.

En cambio si hay varios CPUS el rendmiento mejora ya que a esas comprobaciones las puede hacer otro CPU.

### Compare and swap
Es otra implementacion primitiva, se conoce como compare-and-swap o compare-and-exchange. Es una ayuda del hardware. Su codigo en C es el siguiente:

```c
int CompareAndSwap(int *ptr, int expected, int new) {
	int original = *ptr;
	if (original == expected)
		*ptr = new;
	return original;
}
```

Es muy similar a Test-And-Set, lo unico que cambia es el lock():

```c
void lock(lock_t *lock) {
	while (CompareAndSwap(&lock->flag, 0, 1) == 1)
		; // spin
}
```

### Load-linked and Store-Conditional