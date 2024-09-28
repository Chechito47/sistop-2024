## Semáforos
Un semáforo es una variable o tipo de datos abstracto que constituye el método clásico para restringir o permitir el acceso a recursos compartidos en un entorno de multiprocesamiento (donde se ejecutan varios proceso a la vez).
Vamos a trabajar con semaforos POSIX los cuales permiten a los proceso e hilos sincronizar sus acciones. Permite hacer dos acciones: 
- Incrementar el valor en 1 (`sem_post`)
- Decrementar el valor en 1 (`sem_wait`)
Si el valor del semaforo es 0 se bloquea hasta que se haga maor que 0, por lo tanto nunca podra ser un valor negativo. => unsigned int
### Semaforos nombrados

Estan definidos por tener un nombre de la forma `/nombre`. Dos procesos pueden operar sobre el mismo semaforo llamandolo mediante `sem_open`
### sem_open
Crea o abre un semaforo nombrado, luego de que se abra se puede operar con el usando `sem_post` y `sem_wait`. Cuando un proceso termina de usar el semaforo debe usar `sem_close` para cerrar el semaforo. Cuando todos los procesos terminan de usarlo se lo puede sacar del sistema usando `sem_unlink`

## Makefile
En el makefile hay que modificar el U y ponerle el nombre de nuestro programa
Recomendable empezar con con echo.c

## Que se tiene que aguantar
Nuestras system calls deben ser capaces de aguantar nro negativos, nros grandes, nros chiquitos, punteros vacíos, etc. Las llamadas al sistemas son el punto de entrada al kernel por lo que debemos sanitizar las entradas lo mejor que podamos.

## Que se tiene que hacer
Tenemos que implementar lo siguente:
1. Implementar 4 syscalls: sem_open(), sem_up(), sem_down(),sem_close().
2. Implementar un programa de espacio de usuario “pingpong” que funcione de la “manera natural”
### Syscalls
Veamos que hace cada syscall:
`int sem_open(int sem, int value)` → Abre y/o inicializa el semáforo “sem” con un
valor arbitrario “value”.
`int sem_close(int sem)` →Libera el semáforo “sem”.
`int sem_up(int sem)` →Incrementa el semáforo ”sem” desbloqueando los procesos
cuando su valor es 0.
`int sem_down(int sem)` →Decrementa el semáforo ”sem” bloqueando los procesos
cuando su valor es 0. El valor del semaforo nunca puede ser menor a 0.
#### Como implementarlas
Para implementar las syscalls deberán usar `acquire() release(), wakeup(),`
`sleep() y argint()`. Notar que estas **son funciones del kernel xv6**, por ende tenemos que ver como funcionan y utilizarlas a nuestro favor.
#### Acquire
***Formalmente:***
Toma un lock haciendo busy waiting hasta que esté disponible, similar al down de un semáforo. Útil para crear zonas de exclusión mutua (mutex).
Acquire espera en un loop que la palabra del lock se convierta a 0(porque esta libre) y cuando lo haga la transforma a 1 bloquenadola y luego la vuelve a 0
##### Lock
*Un lock asegura exclusion mutual, haciendo que un solo procesador pueda ejecutar al mismo tiempo. Se dice que lock protege la "informacion" porque lo que protege son una serie de invariantes que se aplican a la informacion*
*El lock tiene una palabra que es **0** si el lock esta libre(unlocked, free, released) o **1** si esta ocupado(locked, held, acquired)*

***En criollo:***
Nos podemos de acuerdo para usar un recurso, si alguien quiere hacer un acquire del recurso que estoy usando no va a poder. Se usa para recursos compartidos.
Supongamos que dos queremos agarrar el mismo recurso, el que sea mas rápido lo agarra y el otro se queda esperando.
#### Release
Libero el recurso que estaba usando para que otro lo pueda usar
#### Wakeup
Hace que levanten los procesos que se fueron a dormir dependiendo de que lo que esperan para despertarse, despierto a todos los procesos que esperan x cosa. Cambia los estados de los procesos de sleep o block a ready.
Tenemos que espcificar que "channel" queremos depertar. **El channel es un nro.**
#### Sleep
No hago mas nada como proceso entonces cambio de estado a sleep. Cuando voy a dormir espero a que alguien me despierto cuando suceda un evento x
Transforma su estado a sleep o block y se va a dormir hasta que lo despierten
#### argint
Sirve para pasar los argumentos. Necesitamos usarla para poder implementar el camino de las syscalls e implementar las syscalls con argumentos. Tiene la forma:
```c
argint(0, &pid);
```
Donde el primer argumento es el process id y el segundo es la prioridad que se obtiene de la siguiente funcion:
```c
setpty(getpid(), priority);
```
argint toma el argumento que pongo pongo primero y lo coloca en la direccion del pid
## Como funcionan con ejemplos
### Acquire
Recordemos que queremos hacer siempre el acquire de lock para poder usar las otras funciones, para ello hacemos locked=1. Pero antes tenemos que ver si no esta libre. Esto lo hace en un loop hasta que locked sea 0 y lo pueda cambiar a 1.
Funciona de la siguiente manera:
```c
if (*locked == 0) {
	*locked = 1;
} else
	loop and try again
```
Pero hay un problema con esto ya que puede haber otros procesos ejecutandose al mismo tiempo, por ejemplo podria pasar que dos procesadores quieran acceder a la misma posicion de memoria o intentar atender el mismo proceso y creer que ambos tienen el lock.
#### AMOSWAP
Para evitar eso RISC-V tiene una intruccion llamada "AMOSWAP". Esta intruccion lo que hace es copiar un valor en locked y su vez va a "recuperar" el valor anterior. Lo hace sin ninguna interrupcion.
Entonces lo que vamos a hacer en nuestro caso es escribir 1 en locked y retornar el valor anterior y se va a chequear lo siguiente (mediante un acquire):
```c
acquire
if (previous value != 0) {
	loop
}
//Si el valor previo es 1 listo, sino hace loop hasta que sea 1
```
Luego se hace un release:
```c
release
*locked = 0; //Copiamos el valor de libre en la palabra
```
#### Codigo acquire
Veamos el codigol acquire (viene del ***spinlock.c***):
```c
//Mutual exclusion spin locks
 #include "types.h"
 #include "param.h"
 #include "memlayout.h"
 #include "spinlock.h"
 #include "riscv.h"
 #include "proc.h"
 #include "defs.h"

void
initlock (struct spinlock *lk, char *name); { //lk es un puntero a la instruccion
	lk->name = name; //Luego no lo podemos cambiar
	lk->lock = 0; //iniciamos en 0
	lk->cpu = 0; //Porque el lock no esta en held
}
//Acquire the lock
//Lopps (spins) until the lock is acquire
void
acquire (struct spinlock *lk) {
	push_off(); //disable interrupts to avoid deadlock
	if (holding(lk))
		panic("acquire"); //error message

	//On RISC-V, sync_lock_test_and_set turns into atomic swap:
		//a5=1
		//s1=&lk->locked
		//amoswap.w.aq a5, a5, (s1)
	while (__sync_lock_test_and_set(&lk->locked, 1) != 0);
	//pasamos el puntero &lk->locked que es el que queremos actualizar y pasamos el nuevo valor que es 1, y esta funcion devuelve el viejo valor y luego chequeamos si es disitnto que 0, si no es 0 se repite el loop. Si es 0 ya hicimos el acquire del locked y listo.

	//tell the C compiler adn the processor to not move loads or stores
	//past this point, to ensure that the critical section's memory
	//references happen strictly after the locked is acquired.
//Lo que queremos es evitar que el compilador haga optimizaciones(podria por ejemplo reordenar cosas)
	//On RISC-V, this emits a fence instruction.
	__sync_synchronize(); //Fuerza al compilador a emitir codigo y que le procesador lo ejecute para verificar si todo lo que pasa antes de este punto ya fue completado y que todo lo que tenga que pasar deespues no haya pasado

	//Record info about lock acquisition for holding() and deb
	lk->cpu = mycpu(); //Guardamos en la variable de cpu todos los procesadores con sus estructuras. La funcion devuelve un puntero con la direccion del procesador que esta ejecutando
}
```
### Release
Libera lo que le hicimos acquire
```c
void
release (struct spinlock *lk) {
	if (!holding(lk)) {
		panic("release");
	}
lk->cpu = 0;

	//tell the C compiler adn the processor to not move loads or stores
	//past this point, to ensure that all the stores  in the critical section's
	//are visible to other CPUs before the lock is released
	//and that loads in the critical section occur strictly before
	//the lock is released.
	//On RISC-V, this emits a fence instruction.
	__sync_synchronize();

	//Release the lock, equivalent to lk->locked = 0.
	//This code doesn't use a C assigment, since the C standard
	//implies that an assigment might be implemented with
	//multiple store instructions.
	//On RISC-V, sync_lock_release turns into an atomic swap:
		//s1 = &sk->locked
		//amoswap.w zero, zero, (s1)
	__sync_lock_release (&lk->locked);
	//Aca copiamos el 0 en &lk->locked (se usa el amoswap de arriba en __sync_lock_release para hacerlo)

	pop_off();
	//Es como un push, activa las interrupciones
}

//Check whether this cpu is holding the lock.
//Interrupts must be off
int
holding (struct spinlock *lk) {
	int r;
	r = (lk->locked && lk->cpu == mycpu());
	return r;
}
```

#### push off y pop off
Hagamos un parentesis y veamos push off y pop off
Push off:
```c
void
push_off (void) {
	int old = intr_get(); //guardamos el estado previo de las interrupciones
	
	intr_off(); //disable interrupts
	if(mycpu()->noff == 0) { //accedemos a la estructura de la cpu y si la primera llamada al acquire es 0 guardamos el estado viejo
		mycpu()->intena = old;	//guardamos el estado viejo
	}
	mycpu()->noff += 1; //en cualquier caso incrementamos el contador
}
```
Pop_off:
```c
void
pop_off (void) {
	struct cpu *c = mycpu(); //puntero a la estructura del cpu

	if (intr_get()) { //caso raro, si las interrupciones estan activadas, es un error
		panic ("pop_off - interruptible");
	}
	if (c->noff < 1) { //chequeo que el contador no sea 0
		panic("pop_off);
	}
	c->noff -= 1; //decrementamos el puntero
	if (c->noff == 0 && c->intena) { //si el puntero es 0 y las interrupciones estaban previamente activadas, activamos las interrupciones
		intr_on(); //activamos las interrupciones
	}
}
```
### Sleep
```c
while(condition is false) {
	sleep(until condition is updated);
}
```
Pero hay un problema con esta implementacion:
	Podria darse el caso en que la condicion se vuelva true luego del while y antes del sleep (ya que hay otros procesos corriendo)
Veamos como funciona:
```c
n = argument 1 => (sleep argument in ticks)
starttime = ticks (global variable, share the tick time, the current time)
while (ticks - starttime < n) {
	if (killed)... aborte waiting
	sleep (until ticks is updated);
}
```
#### A tener en cuenta
Los ticks son una variable global la cual esta protegida por ***ticklock***, por lo que si queremos acceder, modificar o ver los ticks tenemos que hacer un **acquire de tickslock**. Por ejemplo para chequear la condicion de ticks-starttime < n tenemos que hacer eso.
No queremos olvidarnos de hacer wakeup
No queremos mantener el lock mientras duerme
#### Solucion
Entonces la solucion es hacer algo como lo siguiente:
```c
acquire lock
while (condition is false) {
	sleep (ptr to lock); //Atomic interruptions not allowed
	//lock is released
	//go to sleep
	//...
	//wakeup
	//re-acquire the lock
}
do something with shared data
release lock
```
#### Codigo del sleep
Tambien veamos como es la syscall del sleep (viene del ***sys_sleep***):
```c
unit64
sys_sleep (void) {
	int n;
	unit ticks0;
	if (argint(0, &n) < n) {
		return -1;
	}
	acquire (&tickslock);
	ticks0 = ticks;
	while (ticks - ticks0 < n) {
		if (myproc()->killed) {
			release(&tickslock);
			return -1;
		}
		sleep(&tciks, &tickslock);
	}
	release(&tickslock);
	return 0;
}
```
### Wakeup
Toma un channel y hace wakeup de todos los proceso durmiendo en ese channel.
Debe ser llamado sin ningun p->lock.
Tiene la siguiente estructura:
```c
void
wakeup (void *chan) { //channel
	struct proc *p;
	for (p=proc; p<&proc[NPROC]; p++) {
		if (p != myproc()) {
			acquire (&p->lock);
			if (p->state == SLEEPING && p->chan == chan) {
				p->state = RUNNABLE;
			}
		relese (&p->lock);
		}	
	}
}
//Notar que state y chan estan protegiso por lock tambien, por lo que hay que hacerle si o si el acquire a lock para poder usarlos
```

## Cosas a notar
Notemos que los "spinlocks" consumen recursos por lo que es importante hacer uso de sleep y wakeup.
Notar que tambien el lock esta protegido por lo que al hacer acquire() y release() hay una seccion critica en medio.
### Deadlock
Si hacemos un acquire (trap), llega una interrupcion de la consola y luego tratamos de acquire el lock(handler) estariamos ante un caso de deadlock.
La solucion es desactivar las interrupciones en el acquire() y volver a activarlas en el release() que es lo que estaba hecho antes en uno de los codigos.
#### Problema
Pero que pasaria si las interrupciones ya estan desactivadas?
Para solucionarlo tendriamos que usar un contador:
- acquire:
	- cnt++
	- disable interrupts
- release:
	- cnt--
	- if (cnt == 0) enable interrupts
*Esta ultima linea es:*
`if (intena=="previously enable") re-enable`
Notar que cada procesador tiene su propio contador (**noff**)
Tambien debemos recordar el estado de las interrupciones antes de que comenzaramos (**intena**)

