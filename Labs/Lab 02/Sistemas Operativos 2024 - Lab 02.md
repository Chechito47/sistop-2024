## Semaforos
Un semaforo es una variable o tipo de datos abstracto que constituye el metodo clasico para restringir o permitir el acceso a recursos compartidos en un entorno de multiprocesamiento (donde se ejecutan varios proceso a la vez).
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
cuando su valor es 0. El valor del semaforo nunca puede ser menor a 0

### Como implementarlas
Para implementar las syscalls deberán usar `acquire() release(), wakeup(),`
`sleep() y argint()`