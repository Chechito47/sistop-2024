# Mecanismos
![ScreenShot](Imagenes_practico_cpu/ej01_cpu.png)

Tenemos los resultados del tiempo en el formato (wall, user):

a) Al ver que los tiempos no cambian con 2 procesos a la vez pero si con 4 => tenemos 2 cpus

b) El CPUtime es todo el tiempo que corre en CPU, si no se ejecuta de manera parelela este siempre sera menor que el walltime. Este es nuestro caso, el proceso solo se ejecuta en 1 cpu



![ScreenShot](Imagenes_practico_cpu/ej02_cpu.png)

Porque se debe haber ejecutado en mas de un cpu

![ScreenShot](Imagenes_practico_cpu/ej03_cpu.png)

real > user => Cuando el proceso se ejecuta en un solo cpu, se puede dar el caso en que sea ligeramente mayor (no hay otros procesos compitiendo pero si hay context switch) o se puede dar el caso en que sea bastante mayor (hay otros procesos compitiendo)

real < user => Cuando el proceso se ejecuta en mas de una CPU

real = user => El proceso se ejecuta en un solo CPU sin ningun context switch

![ScreenShot](Imagenes_practico_cpu/ej04_cpu.png)

a) Vale 100 hace una copia exacata
b) Son independientes

![ScreenShot](Imagenes_practico_cpu/ej05_cpu.png)

a, aa, aaaa, aaaaaaaa => 8 + 4 +2 +1 = 15

n=1 => 2⁰ => 1
n=2 => 2¹ + 2⁰ => 3
n=3 => 2² + 2¹ + 2⁰ => 7
n=3 => 2³ + 2² + 2¹ + 2⁰ => 15

Sumatoria de (2^n) -1 con n desde el valor que querramos hasta 0 incluido 

![ScreenShot](Imagenes_practico_cpu/ej06_cpu.png)

0 porque el execv es correcto, caso contrario imprime 1. Lo unico que hace este programa es pasar como comando "/bin/date -R" que es un comando que nos muestra la hora y fecha un poco disntinto el cual no tendria porque fallar.

![ScreenShot](Imagenes_practico_cpu/ej07_cpu.png)

Toma argc y argv como argumentos. Chequea si 0 es menor que argc ya decrementado en 1. Si lo es lo pone como NULL y pasa eso como comando con execvp

El otro programa toma argc y argv como argumentos. Si argc es menor igual que 1 termina. Si no lo es hace fork y verifica que este bien hecho, si no lo esta termina tambien devolviendo error. Si esta bien hecho y es el hijo termina. Si no es niguno de los dos casoas anetrior pone el argc-1 en null y pasa eso como comando.

![ScreenShot](Imagenes_practico_cpu/ej08_cpu.png)

Es util el dup para poder duplicar file descriptors. en le primer caso funciona porque previamente cerre ese fd por lo que ya no se puede usar y sale por donde lo cerre pero si quisiera hacer algo similar con varios no podria ya que se emzclaria todo. Si uso dup si podria.

![ScreenShot](Imagenes_practico_cpu/ej09_cpu.png)

Es un programa que no termina nunca y hace fork hasta el infinito. Es posible mitigar sus efectos si creo un archivo que limite la cantidad de procesos que puede crear un shell.

![ScreenShot](Imagenes_practico_cpu/ej10_cpu.png)

![ScreenShot](Imagenes_practico_cpu/ej10b_cpu.png)

Quito la de running a ready: Un proceso nunca terminaria por context switch por lo que estos no existen, solo podrian terminar por un I/O.
Quito la de ready a running. Los procesos jamas podrian correr por lo que nada funciona
Quito la de running a blocked. No podria hacer esperaras por I/O
Quito la de blocked a ready: los procesos no podrian volver despues de un I/O

![ScreenShot](Imagenes_practico_cpu/ej11_cpu.png)

nose

![ScreenShot](Imagenes_practico_cpu/ej12_cpu.png)

a) cpu+kernel < wall. Supongamos que tengo muchos procesos en cola y que el que quiero es muy corto. Supongamos que comienza a ejecutarse pero hace un context siwth y se ejecuta otro y luego se vuelve a ejcuatr y termina. Entonces paso poco tiempo en CPU, poco en conext siwth porque solo fue 1 y mucho en wall porqeu tuve que que esparar a que terminen varios procesos.
b) Falso, incluso pueden usar la misma memoria virtual si comparten parte del codigo
c)Verdadero, ademas tengo que multiplexar en espacio, ram, heap y stack
d)Falso, no puede ejeuctar las de kernel
e)Verdadero, el planificador decide seguir ejeuctando el mismo proceso
f)verdadero
g)verdadero
h)Falso, si el padre muere se le asigna un nuevo padre
i)Falso, el hijo puede comuncarse con los return y el waitpid. el padre puede comunicarse con el hijo por argv y argc
j)Falso, se ejecuta si el execvp fallo
k)Verdadero, queda en estado zombie

# Politicas

![ScreenShot](Imagenes_practico_cpu/ej13_cpu.png)

En cada ciclo pasan 10 Tcpu => me haran falta 6 ciclos
FCFS
Llegan al mismo tiempo, se decide por PID

| 0   | 10  | 20  | 30  | 40  | 50  |
| --- | --- | --- | --- | --- | --- |
| A   | A   | A   | B   | B   | C   |

SJB

| 0   | 10  | 20  | 30  | 40  | 50  |
| --- | --- | --- | --- | --- | --- |
| C   | B   | B   | A   | A   | A   |

TurnAroundTime=tiempo que tarda desde que fue ejecuttado por primera vez hasta terminar
ResponseTime=tiempo que tarda desde que llega hasta ser atendido

Entocnes:
FCFS:
	TurnAroundTIme
		A=30
		B=20
		C=10
	ResponseTime
		A=0
		B=30
		C=50
SJB:
	TurnAroundTime
		A=30
		B=20
		C=10
	ResponseTime
		A=30
		B=10
		C=0

![ScreenShot](Imagenes_practico_cpu/ej14_cpu.png)

Tenemos dos politicas de planificaciones
STCF similar a SJB pero es segun el tiempo que le queda
Va a ser minimamente de 8 la tabla de procesos

| Tiempo  | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- |
| Running | B   | B   | B   | A   | C   | A   | A   | A   |
| Ready   |     |     | A   |     | A   |     |     |     |
| Arrival | B   |     | A   |     | C   |     |     |     |

| Proceso | Tarrival | Tcpu | Tfirstrun | Tcompletition | Tturnaround | Tresponse |
| ------- | -------- | ---- | --------- | ------------- | ----------- | --------- |
| A       | 2        | 4    | 3         | 7             | 5           | 1         |
| B       | 0        | 3    | 0         | 2             | 3           | 0         |
| C       | 4        | 1    | 4         | 4             | 1           | 0         |

RR
Decido que se va a ejecutar el de menor PID

| Tiempo  | 0   | 1   | 2   | 3   | 4    | 5   | 6   | 7   |
| ------- | --- | --- | --- | --- | ---- | --- | --- | --- |
| Running | B   | B   | A   | A   | B    | A   | A   | C   |
| Ready   |     |     | B   | B   | A, C | C   | C   |     |
| Arrival | B   |     | A   |     | C    |     |     |     |


| Proceso | Tarrival | Tcpu | Tfirstrun | Tcompletition | Tturnaround | Tresponse |
| ------- | -------- | ---- | --------- | ------------- | ----------- | --------- |
| A       | 2        | 4    | 2         | 6             | 5           | 0         |
| B       | 0        | 3    | 0         | 4             | 5           | 0         |
| C       | 4        | 1    | 7         | 7             | 1           | 3         |


![ScreenShot](Imagenes_practico_cpu/ej15_cpu.png)



![ScreenShot](Imagenes_practico_cpu/ej16_cpu.png)

Nos haran falta 26 tiempos de CPU
RR Q=2 
POLITCA: CORRE EL QUE LLEVA MAS TIEMPO SIN SER EJECUTADO, CASO CONTRARIO POR PID

| Tiempo  | 0   | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  | 12  | 13  | 14  | 15  | 16  | 17      | 18  | 19  | 20  | 21  | 22  | 23      | 24  | 25      |
| ------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | ------- | --- | --- | --- | --- | --- | ------- | --- | ------- |
| Running | A   | A   | C   | A   | B   | B   | B   | B   | C   | C   | A   | A   | B   | B   | B   | B   | C   | ***A*** | B   | B   | B   | B   | C   | ***C*** | B   | ***B*** |
| Ready   |     | C   | AB  | B   |     |     | C   | C   | AB  | AB  | B   | B   |     |     |     | C   | A   | B       |     |     |     | C   | B   | B       |     |         |
| Arrival | A   | C   | B   |     |     |     | C   |     | A   |     |     |     |     |     |     | C   | A   | B       |     |     |     | C   |     |         |     |         |
| I/O     |     |     |     | AC  | AC  | AC  | A   | A   |     |     | C   | C   | AC  | AC  | AC  | A   | B   | C       | C   | C   | C   |     |     |         |     |         |


![ScreenShot](Imagenes_practico_cpu/ej17_cpu.png)

DECIDO POR PID

| Tiempo | 0       | 1       | 2       | 3         | 4       | 5          | 6         | 7       | 8          | 9         | 10        | 11       | 12       | 13      | 14      | 15        | 16        | 17        | 18        | 19       | 20      |
| ------ | ------- | ------- | ------- | --------- | ------- | ---------- | --------- | ------- | ---------- | --------- | --------- | -------- | -------- | ------- | ------- | --------- | --------- | --------- | --------- | -------- | ------- |
| Q1     | ***A*** | ***B*** | ***C*** |           | ***D*** |            |           | ***E*** |            |           |           |          |          |         |         |           |           |           |           |          |         |
| Q2     |         | A       | AB      | ***A***BC | ABC     | ***A***BCD | ***B***CD | BCD     | ***B***CDE | ***C***DE | ***C***DE | ***D***E | ***D***E | ***E*** | ***E*** |           |           |           |           |          |         |
| Q4     |         |         |         |           |         |            | A         | A       | AB         | A         | A         | AC       | AC       | AC      | AC      | ***A***CE | ***A***CE | ***A***CE | ***A***CE | ***C***E | ***E*** |
| Q8     |         |         |         |           |         |            |           |         |            |           |           |          |          |         |         |           |           |           |           |          |         |


![ScreenShot](Imagenes_practico_cpu/ej18_cpu.png)

a)Falso, se puede devolver si el proceso termina antes del quanto
b)FIFO
c)verdadero
d)verdadero, por ejemplo un proceso podria tener menor pid o ejecutarse antes que otros y ser extremadamente largo y nunca terminar
e)verdadero, mlfq evita hacer tield y que otro se ejecute en su lugar dandole lugar a hacer cosas malas