## System call
Una system call es una funcion que me provee el SO para interactuar. Por ejemplo si tengo linux, las syscalls son las mismas en todas las distros.

---
## File descriptors
Los file descriptor, descriptor de archivos es una clave(indice, puntero) en una tabla siendo 0, 1, 2 los que están por defecto que son stdin, stdout y stderr y que estos apuntan a algo como el teclado, consola y consola (en el caso de los ejemplos anteriores).

---
## Dup2
Sirve para manejar punteros a file.
Duplica un file descriptor, crea una copia del file descriptor viejo en el lugar del nuevo, entonces en el 0, 1, 2 que tenia stin, stdout, sterr podria hacer un dup2 del 1 y luego el 4 seria stdout.
Si modifico 4 no se modifica 1 pero si modifico el archivo al que apuntan si se modifican los dos.

---
### Cerrarlo
Uso close para cerrarlo, con esto deja de apuntar a donde lo hacia.
Si el file descriptor estaba previamente abierto es necesario cerrarlo, caso contrario quedaría abierto a pesar de que el proceso se termine.
Por ejemplo, si cierro el 1 el 4 sigue abierto y apuntando al mismo lugar (siguiendo el ejemplo anterior)
Si digo hago fprintf (que dice que file descriptor quiero usar como primer argumento) y hago por por ejemplo: `fprintf(4, "Hola!\n")` uso el **fd 4** que seria lo mismo que `fprintf(1, "Hola!\n")`

---
## Pid, fork y procesos hijos
pid es una variable, es el id del proceso, este no cambia nunca.
pid_t es el tipo de la variable pid.
### Fork
Fork hace un clon exacto del ese proceso ejecutando de ese punto exacto donde se hizo el fork copiando absolutamente todo en otro proceso nuevo independiente.
El pid del fork devuelve 0 si estamos en el hijo y !0 si estamos en el padre.
Si no hago esto y divido la ejecución, los dos procesos van a seguir siendo iguales.
### Hijo
El proceso hijo va a tener la tabla de file descriptors igual que el padre. Entonces luego si tengo algo en el padre lo voy a tener que cerrar en el padre e hijo
Devuelve -1 si hay error
Si pido el pid antes de hacer el fork ambos procesos van a tener el mismo pid.

---
## Execvp
execvp, el filename es el nombre del archivo donde esta el ejecutable que quiero, luego toma un char pointer con los argumentos de ese comando.
Sirve para reemplazar lo que estaba sucediendo es en proceso entero por otro
Reemplaza todo menos la tabla de los file descriptors.
El programa apuntado debe ser un ejecutable.
*"imagen de proceso actual"* es lo que le permite al proceso vivir, reemplazar la imagen del proceso es copiar todas las cosas que necesita otro proceso y ponerlas encima de las de otro proceso.b
execvp no retorna en caso de éxito, entonces si hace el printf es porque hubo un error entonces debemos poner el printf en un if
Si se ejecuta un execvp se sustituye toda la imagen entonces no se llega a ese printf a menos que haya un error

---
## Wait
wait hace que le proceso padre se queda esperando hasta que no recibe una señal de que alguno de sus hijos haya cambiado de estado. Por ejemplo con un sleep 5 se hace un wait.

---
## Pipe
pipe toma un arreglo con dos enteros, lo que hace es agarrar y abrir un buffer en el espacio kernel y devolver en el arreglo devuelve los fila descriptors de lectura y escritura del buffer creado.
El pipe es una llamada a sistema, no las tenemos que implementar las tenes que llamar
Vamos a usar pipe porque si el fork copia vamos  atener referencias a la file descriptors table, entonces si el padre escribe en esa table el hijo la va a poder leer.

---
## Buffer
El buffer es un archivo, un espacio de memoria. Se usa para que un proceso lea por un lado y otro escriba por otro
### Tad
El tad, el tipo abstracto de datos, las tenemos que implementar.

---
## Idea de como encarar el execute.c
Primero leemos el con el parser, si es un builtin como help y eso y si no es un execute donde hacemos lo de redirigir entradas y salidas y los pipelines
Con execvp llamo a los comandos cat, ls y todos los otros
El unico que tengo que implementar es el help que es un printf
*Con tener 0 en todos los demas en make test consideramos que no hay memleaks*
Usemos la librería string si hace falta
Static es porque es privada, no se ve en el .h pero si en el .c

---
## Parcialito
Como se hace una syscall, que archivos hay que tocar para hacer una syscall
### Shell
El shell es una interfaz entre el SO y el usuario ya que permite al usuario acceder a los servicios que provee el SO como ejecutar comandos, redireccionar entradas y salidas, etc. Podemos pensar en cierta forma que espera un string que represente un comando y lo ejecuta.
### Foreground y background
Un comando en modo foreground es aquel que se ejecuta y no nos deja hacer nada con el bash hasta que se termine su ejecucion. Es como si se ejecutara en primer y unico plano. Por ejemplo:
`evince -f file.pdf`
Un comando en modo background es aquel que se ejecuta y nos permite seguir usando ese mismo bash, es como si se ejecutara en segundo plano. Por ejemplo
`evince -f file.pdf &`
### Redireccion de entradas y salidas
Podemos usar los <> para redireccionar las entradas y salidas de los comandos. **Por ejemplo:**
`echo "Hola" > archivo`
*Redirige la salida "Hola" al archivo*
**Otro ejemplo:**
`echo "Hola" >> archivo`
*Redirige la salida "Hola" al archivo pero en una nueva linea*
**Otro ejemplo:**
`cat < archivo`
*Redirijo la entrada de cat con el archivo, entonces lee lo que hay en el archivo.*
**Otro ejemplo:**
`wc -l > out.txt < in.txt`
*Cuento cuantas lineas tiene el archivo in.txt y redirijo la salida hacia out.txt*
### Comandos en secuencia
Haciendo uso del ***&&*** puedo ejecutar comandos en secuencia si solo si el anterior se ejecuto correctamente. Por ejemplo
`cd .. && ls`
### Comandos en pipeline
Conecto distintos comandos mediante sus entradas y salidas tomando la salida del primer comando y redirigiendola como entrada para el segundo.
`ls -l | wc -l`
*Cuenta la cantidad de 'lineas' que hay en el directorio raiz*
**Otro ejemplo:**
`cat /proc/cpuinfo | grep 'model name'`
Muestro el contenido de cpuinfo y filtro lo que tenga la palabra 'model name'
### Fork
Tengo un proceso A1
Luego hago:
`rc = fork()`
y ahora existe el proceso A1 padre y el proceso A1 hijo. Dentro del código de A1 puedo hacer cosas dependiendo si **rc>0 (es el padre)** o si **rc=0(es el hijo)**. Si rc<0 error en el fork.
Notemos que **viven en mundos distintos y no comparten información a menos que hagamos un pipe**. *Por ejemplo si tenemos un int a y le hago fork, y luego en el hijo modifico a, en el padre no se modificara.*
También notemos que aunque acceden a la misma memoria virtual, no lo hacen con la memoria física. IDEM con los punteros.
### Pipe
Un pipe crea dos file descriptors (fd) uno de lectura y otro de escritura. Haciendo uso de estos puedo compartir información entre procesos.
EJEMPLO:
Tengo el siguiente proceso A:
	P(A)
	fd0 -> stdin
	fd1 -> out.txt
	fd2 -> stderr
Luego hago fork del proceso A, entonces como al hacer un fork se copia prácticamente todo del proceso padre, la tabla de fd del proceso hijo sera:
	P(A')
	fd0 -> stdin
	fd1 ->out.txt
	fd2 -> stderr
### Como funciona mybash
1)Dado un comando (un string) individualiza (parsea) cada parte del mismo:
- Nombre del programa (wc)
- Sus argumentos (-l)
- Archivos de redireccion de entrada y salida "out.txt", "in.txt"
2)Una vez parseado el comando, ejecuta el comando en el SO
#### Comando simple
Un comando simple consta de un nombre, sua argumentos y sus archivos de redireccion de entrada y salida. Ejemplo:
`wc -l > out.txt < in.txt`
#### Pipeline
Un pipeline consta de dos o mas comandos simples conectados via operador |
`ls -l | wc -l`
### Ejecutar un comando
Para ejecutar un comando implementamos el modulo execute.c, este es el que invoca las llamadas al sistema (syscalls) necesarias para ejecutar los comandos en un ambiente aislado de nuestro bash.
Algunas de las syscalls que vamos a necesitamos son:
- fork() -> crea un nuevo proceso hijo
- pipe() -> crea una tuberia para comunicar procesos
- dup() -> sirve para modificar los fd
- wait() -> para bloquear un proceso
- execvp() -> ejecutar un programa externo
Veamos algunos ejemplos:

| Entrada                | Syscalls relacionadas                                             | Comentario                                                                      |
| ---------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| cd ../..               | chdir()                                                           | El comando es interno, solo hay que llamar a la syscall de cambio de directorio |
| gzip LabG04.tar        | fork(); execvp(); wait();                                         | Ejecutar el comando y el padre espera                                           |
| xesyes &               | fork(); execvp();                                                 | Un comando simple sin redirectores ni espera                                    |
| ls -l ej1.c > out < in | fork(); open(); close(); dup(); execvp(); wait()                  | Redirige tanto la entarda como la salida y el shell padre espera                |
| ls \| wc -l            | pipe(); fork(); open(); open(); close(); dup(); execvp(); wait(); | Sin ejecucion en 2do plano, dos comands simples conectados por un pipeline      |
