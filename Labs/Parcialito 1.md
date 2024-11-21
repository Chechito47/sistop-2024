Posibles preguntas parcialito 1

TEMAS:

- TADS opacos y como implementarlos
- Posibles bugs
- Interpretacion de los resultados de los reportes de los tests
- Modulos del proyecto
- Parsing
- Representacion de los comandos en las estructuras de datos
- Comportamiento esperado de ciertos fragmentos de codigo
- Leer y entender el codigo fuente relacionado a la implementacion del lab0 y lab1

### Syscalls usadas en el lab01
- **fork()**: para crear un nuevo proceso (hijo)
- **pipe()**: para crear una tuberia
- **dup()**: para modificar un descriptor de archivo
- **wait ()**: para bloquear un proceso
- **execvp()**: para ejecutar un programa externo


| Entrada                | syscalls relacionadas                                     | Comentario                                                                      |
| ---------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------- |
| cd ../..               | chdir()                                                   | El comando es interno, solo hay que llamar a la syscall de cambio de directorio |
| gzip Lab1G49.tar       | fork(); execvp(); wait();                                 | Ejecutar el comando, el padre espera                                            |
| xeyes &                | fork(); execvp()                                          | Un comando simples sin redirectores ni espera                                   |
| ls -l ej1.c > out < in | fork(); open(); close(); dup(); execvp(); wait();         | Redirige tanto la entrada como la salida y el shell padre espera                |
| ls \| wc -l            | pipe(); fork(); open(); close(); dup(); execvp(); wait(); | Sin ejecucion en segundo plano,dos comandos simples conectados por un pipeline  |


## Ejercicios de parcialito

```c
Nombrar que codigo ejecuta el hijo cuando se hace lo siguiente:

if(fork == 0) {
	//Codigo A
}
//Codigo B
}

RTA: Ejecuta los dos. El padre ejecuta primero el codigo B pero luego el hijo ejecuta el codigo A y B.
```

```c
Que pasa cuando hacemos <ctrl-d> en mybash?

RTA: mybash se cierra exitosamente sin devolver ningun error
```

```c
Como tipa en haskell un:

ls > out.txt < out.txt

RTA: Primero notemos que un scommand porque es solo un comando con sus argumentos y sus archivos de redireccion de entrada y salida: 
([char*], char*, char*) porque el ls viene acompañado de un archivo.

Veamos otros ejemplos:

scommand   wc -l > out.txt < in.txt:
([char*],char*,char*) = (["wc","-l"],"out.txt","in.txt")

scommand   cd ../.. 
([char*], NULL, NULL) = (["cd", "../.."], NULL, NULL)
Lo contamos asi porque consideramos que hasta 3 instancias puede leer mybash.


Ahora veamos ejemplos de pipelines los cuales consisten de dos o mas comandos simples conectados por el operador pipe |

pipeline   ls | wc -l
([scommand], bool) = ([(["ls"], NULL, NULL), (["wc", "-l"], NULL, NULL)], true)

pipeline   xeyes &
([scommand], bool) = ([(["xeyes"], NULL, NULL)], false)


La estructura de los scommands es la siguiente: Tenemos un maximo de 3 entradas, una para el comando en si con sus argumentos y otras 2 para redireccionar salida y entrada. 

Vemos que la estructura de los pipelines es la siguiente: uno o mas scommands separados por un pipe, seguidos de un bool que indica con un true si se ejecuta en modo foregound o un false si se ejecuta en modo background (uso del &)
```

```c
Por que suecede un error de una linea que sucede al hacer make test-command

RTA: 
```

```c
Que imprime en pantalla lo siguiente:

int fd=open("algo.txt", O_create | O_wronly, I_srxiu);
fd=dup(fd);
fd=dup(fd);
fd=dup(fd);
Write(1, "0", 1);
Write(2, "1", 1);
Write(3, "a", 1);
Write(4, "b", 1);
Write(5, "c", 1);

RTA: Imprime "01" y guarda "abc" en el .txt
Esto es porque el hacer open equivale a un int=3, luego si hacemos dup un dup de eso indicamos que queremos que el fd3 salga por el archivo algo.txt.
Entonces como dup duplica el fd y lo pone en otro, pasamos de que fd=3 a que fd=4 y hacemos lo mismo con fd4 indicando que salga por algo.txt y de igual forma con el fd5. (vale hasta el fd=6). Por lo tanto como para los fd 1 y 2 no indicamos que salgan por ningun lado se imprimen en pantalla y luego los a, b y c se guardan en el archivo.

Si tuvieramos un write(6, "d", 1) se guardaria en el archivo y si tuvieramos un write(7, "e", 1) no se guardaria ni imprimiria 
```

```c
¿La funcion exit pertence al espacio de kernel?

RTA: No, pertence al user space
```

```c
Codigo con write que pregunta que imprime entre las siguientes opciones:
- hola/0
- /0hola
- hola
- nada

RTA: imprime hola/0
```

```c
Para el siguiente codigo indicar cuantas 'a' imprime el programa:

void main() {
	close(0);
	fork();
	putchar('a'); fflush(NULL);
	close(1);
	fork();
	putchar('a'); fflush(NULL);
	close(2);
	fork();
	putchar('a'); fflush(NULL);
}

RTA: Primero cerramos el stdin que no cambia nada, hacemos fork por lo que ahora tenemos un padre e hijo e imprimos dos 'a' al putchear. Luego cerramos el stdout por lo que a partir de ahora nada que imprimamos se va a mostrar
```

```c
Para el siguiente codigo indicar que imprime y que se guarda en el archivo:

void main() {
	dup(open("test.txt", O_CREAT | O_WRONLY | S_IRUSR));
	write(3, "a", 1);
	write(4, "a", 1);
}

RTA: Se guardan dos "a" en el archivo y no imprime nada
Como el open vale 3, con el dup estoy indicando que redirigo el fd3 hacia el archivo test.txt y como lo duplica lo pone en 4, por lo tanto los fd3 y 4 son los que salen por test.txt.

Su tuviera un write(5, "a", 1) no saldria por ningun lado
```

```c
Que proceso corre con un fork if(id==0)
```

```c
Donde imprime un 
close->dup->open->close->dup
```

```c
En una implementacion el obfuscated no se puede acceder al struct con la flechita "->"
```

```c
Codigo con fork y exec y tengo que decir si se ejecutaba un comando con ; o && o |
```

```c
Como se parseaba una entrada por linea de comandos al TAD que piden
ls "cualquier cosa" < hola.txt > hola.txt

Parsea bien, redirige la entrada del ls al hola.txt y luego toma eso y lo pone en hola.txt por lo que termina escribien hola.txt
```

```c
Una de valgrind y decir de donde venian los memory leaks

RTA: por el glib
```

```c
Como se interpretaban los CHECKS, si los failed tests eran aprobados o no
```