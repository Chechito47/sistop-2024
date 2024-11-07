## Temas
- Spinlocks y zonas criticas
- Implementacion de syscalls
- Manejo de semaforos como los implementaron en el lab 2. Ejemplos de codigo como el de ping pong.
- Syscalls sleep, wakeup, uptime, exit
- Estrategias de scheduling en general y como se implementarian en xv6
- Conceptos de quantum, ticks, interrupciones y panic
- Preguntas donde les damos resultados similares a los que obtuvieron en el lab 3 y les pedimos que los analicen. (Nosotros les damos los resultados, NO tienen que memorizar su lab 3)

## Anotaciones

### Funciones
#### Uptime
La funcion uptime devuelve cuantas interrupciones por ticks sucedieron desde el inicio de xv6. Por ejemplo el start tick es un llamado a uptime().

#### Sleep
`sleep(void *chan, struct spinlock *lk)`
Duermo un proceso que se estaba ejecutando, generalmente por la espera de I/O. Nos sirve para aumentar la prioridad en MLFQ porque sucede cuando no se acaba el quanto por lo tanto es para procesos io-bound.

#### Wakeup
`wakeup(void *chan)`
Despierta a los procesos de "chan" que fueron dormidos por sleep

#### Exit
`exit(int status)`
Pide a un proceso que termine su ejecucion. Cuando lo hace queda en estado zombie hasta que el padre llame a wait()

#### Wait
`wait(uint64 addr)`
Espera que un proceso hijo termine y devuelve si PID. Si no hay hijos devuelve -1.

### Conceptos
#### Quantum
Es la cantidad de tiempo maximo que un proceso puede correr antes de que suceda un context switch. Esta definada por un intervalo (ciclos por segundo) que lo utilizamos desde 1m hasta 1k.

#### Ticks
Es la medida que utilizamos para el tiempo.
Los ticks suman 1 cada vez que hay una interrupcion, y hay una interrupcion obligada cada vez que se acaba el quanto o cuando esperamos por un I/O. Por eso es que los ticks varian segun el quanto, si es mas corto habra mas interrupciones y es por eso que varia segun el bench, si es cpu habra mas tiempo entre los ticks de inicio de los procesos en cambio si es cpu es mas corto.

#### Interrupciones
Ocurren cuando un proceso se queda sin quanto o cuando un proceso pasa a sleep debido a la espera de I/O. Cuando sucede una interrupcion puede o no suceder un context switch, esto dependera del planificador.

#### Panic
Es una funcion utilizada para manejar errores criticos, como por ejemplo errores con la memoria o punteros nulos.
Imprime un mensaje con el error y aborta la ejecucion.