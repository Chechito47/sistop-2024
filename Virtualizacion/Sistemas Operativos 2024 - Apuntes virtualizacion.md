# VIRTUALIZACION
## **Curiosidades - anotaciones libres**
### Basurero de linux
La direccion /dev/null es el inodoro de linux.
### Watchdog
Toda computadora debe tener un watchdog (computadoras caras)
Es una pequeña computadora aparte de la computadora principal que esta todo el tiempo mirando si la computadora principal esta funcionando. Cuando detecta que no esta funcionando la reseta. Por ejemplo las computadoras que están en los satélites tienen un watchdog.
El reset es una interrupción por tiempo que se produce de vez en cuando, funciona de igual forma que un timer.
### Sitios webs
En los sitios webs puedo poner parámetros en la url para hacer cosas malas, se hace con
?a=10&b=11 por ejemplo o oculto=10 y así. Con esto podes obtener información para que esto no pase deben estar protegidos los sitios webs.

---

## **Multiplexación**
Veamos el uso del comando para ejecutar un programa, en este caso tengo un programa llamado .**/cpu** que hace ciertas cosas:
`./cpu` me devuelve `usage: cpu<string>` *(significa que debo pasarle un string para que funcione)*
Entonces si ejecuto `./cpu Hola` vemos que imprime **Hola** indefinidamente, luego con ***ctrl+c*** lo termino o si no también puedo hacer ***ctrl+alt+t***(abrir otro terminal)*, escribir `htop`, posicionarme en el proceso y apretar ***f9*** y luego ***enter*** para matarlo.
Con `bg`*(background)* hago que siga corriendo, y de esta forma puedo ejecutar dos ./cpu al mismo tiempo. Luego para volver a hacer lo mismo debo poner primero `fg`*(foreground, vuelve arriba)* y luego ***ctrl+z***  
Con esto podemos hacer creer al usuario que estamos ejecutando muchas cosas a la vez cuando en realidad no lo estamos ejecutando a la vez. Se llama ***MULTIPLEXACIÓN*** en tiempo.
### Técnicas de multiplexacion
A su vez, también existen distintos tipos o mejor dicho técnicas de multiplexacion
- Multiplexacion por tiempo: Por ejemplo un solo mozo tiene que atender varias mesas, entonces atiende un rato a cada una. El claro ejemplo es si tuviera un solo núcleo y varios procesos corriendo, el procesador se multiplexaria solo por tiempo
- Multiplexacion por espacio: Por ejemplo tengo un terreno y lo divido en distintas partes para su uso. Si tuviera 128 núcleos y 30 procesos solo se multiplexaria por espacio ya que podría dedicarle un núcleo a cada proceso
Pero en la realidad son muy raros los casos en los que solo se utiliza una técnica, el la gran mayoría de situaciones ambas técnicas trabajan en conjunto, por ejemplo la pantalla es un claro caso ya que la multiplexo en espacio al dividir el espacio para distintos programas y la multiplexo en tiempo al dejar de usar un programa y usar otro
Por lo tanto vemos que ***el sistema operativo es el encargado de sacarle provecho a una maquina***.

---
## **Procesos**
***Proceso/=programa***. Un programa puede tener varios procesos distintos.  
Los procesos puede estar en 4 estados (se ve en el htop)  
- **R**   *Corriendo*  
- **I**    *Listo, esperando, ready*
- **S**   *Bloqueado, esta esperando por algo, por ejemplo que hagamos algo*
- **Z**   *Este es mas raro, es el zombie*
El proceso zombie(se marca con Z), es el mas raro. Esta muerto pero todavía no se pueden liberar los recursos porque hay otros procesos que dependen de el que todavía no han devuelto información.
Podríamos tener 0 procesos corriendo, que es lo que busca el sistema operativo  
Podemos tener 0 procesos en ready, cada proceso que tenga acá al menos va a ocupar 256 bytes(en 64bits), entonces **la cantidad máxima de procesos va a estar dada por la cantidad de memoria RAM**. El proceso pasa de ready a running cuando haya espacio en la CPU, mientras se guarda en la RAM. 
### Como se pasan de un estado a otro
Cuando aprieto una tecla hace un system call, lo pone en ready y eventualmente pasa a running y se ejecutan las instrucciones, donde la cantidad de tiempo que esta en running es ínfima, son nanosegundos.
Los procesos también pasan de running a ready, cada 100ms el procesador ve quien esta corriendo y si uno esta corriendo hace 100ms pasa a ready para no monopolizar el uso de la cpu, luego el **planificador** elige otro proceso que estaba en ready y lo pasa a running. Caso contrario no tendría control de la maquina  
*Por ejemplo si hago un programa que lea mi teclado el proceso va a estar en estado S hasta que toque una tecla pero cuando la toque va a ser tan rápido que no voy a poder llegar a ver que cambie el estado del proceso en el htop. Entonces si el teclado no tiene nada para dar el sistema operativo pone el programa a dormir*.
### Archivos del proceso
Todo proceso cuando se inicia tiene 3 archivos abiertos:  
- ***stdin***(file descriptor 0)(se lo conecta al teclado) (es de entrada)  
- ***stdout***(file descriptor 1)(se lo conecta a la pantalla) (es de salida)  
- ***sterr***(file descriptor 2)(se lo conecta a la pantalla) (es de salida)
### API
API es Aplication program Interface, o sea son las funciones que provee un sistemas operativos para trabajar con procesos. Vamos a ver solamente la API de UNIX.
### Bomba Fork
Si la cantidad de procesos que tengo es demasiado alta en algun momento mi sistema linux no va a poder con todos ellos y voy a tener que apagar la maquina. A eso se lo conoce como bomba fork. Para evitar esto debería crear un archivo para que cada que se abra un shell limite la cantidad de procesos y así evito la bomba fork.

---
## **Argumentos**
- ***argc*** (arguments count)
- ***argv*** (vector de los argumentos, es char_*argv)
Entonces por ejemplo si ejecuto el programa ./p1 1 2 3 4 la cantidad de argc es 1 2 3 4 5, mientras que el argv es un vector que contiene al string 1, al string 2, al string 3 y al string 4. Ademas los argumentos recibidos no son solo 1 2 3 4, son esos y el nombre del programa, en este caso ./p1
Hay comandos que son distintos en su llamda pero hacen lo mismo. El argc es siempre >=0, por eso puedo nombrar distinto a varios comandos y que hagan lo mismo, por ejemplo podría hacer un grep que se llame papa
### Como modificar un archivo redirigiendo las entradas y salidas
Si yo cierro el **STDOUT**, lo puedo hacer con *close(STDOUT_FILENO)* y luego abro un archivo, se va a abrir en donde estaba el **STDOUT** ya que el open busca el primer espacio libre y al cerrar STDOUT **este** es el primer espacio **vació**. Por lo tanto redirige la **salida estándar** al archivo que abrí. Luego si vemos ese archivo va a tener el contenido de la salida
Entonces simplemente **redirigiendo las entradas, salidas y errores puedo modificar un archivo**. Por lo tanto en lugar de hacer algo como `wc -l p4.c > salida.txt` simplemente antes de hacer el **execpv** tengo que desconectar el **STDOUT***(que es el 1)* y en ese lugar hacer un open con el archivo **salida.tx**.

---
## **Procesos Hijos**
### Fork
En cierto sentido, usamos **fork** para "*crear*" un proceso (*ya que se usa para duplicar un proceso y asi crear un hijo)* y **exit** para destruirlo (*el exit destruye y devuelve la memoria junto a los recursos que se hayan pedido para ese proceso*). Poner varios fork bien seguidos es como hacer potencias. El fork siempre inicia en 0.
Cuando los padres mueren los hijos también. Aunque se puede romper esa relación si al ejecutar el hijo uso el **&.** En ese caso el padre muere y el hijo queda vivo y ahora pasaría a ser hijo del proceso superior al padre. *Se puede hacer un ejemplo así ejecutando un programa con la terminal ./programa & y luego esa misma y terminal la cierro con exit y el programa seguiría funcionando siendo hijo de systemd*
**systemd es el proceso padre de todo el sistema.**
### Posible situación
Si tenemos un programa que crea un hijo y que imprima primero el padre y luego el hijo, al ejecutarlo muchas veces, como por ejemplo usando un for, se puede dar el caso en que se ejecute primero el hijo y luego el padre debido a que si no se especifica en el código para que primero se ejecute el padre y luego el hijo el planificador podría darle mas prioridad al hijo. Mientras mayor sea la carga de la maquina mas probable es que pase ya que los **sistemas bajo carga se comportan distinto a los sistemas normal**. Para evitar eso podría usar un wait(NULL) y así hago que se ejecute primero uno y luego el otro, el wait espera hasta que el proceso anterior termine, solemos usar rc(return code) como variable para el fork. int rc = fork(); Básicamente wait sincroniza entre procesos.
### Wait
El wait es una llamada entre el sistema operativo y el programa, lleva **NULL** para que no me interese el estatus del proceso, solemos usar como variable el wc (wait code) int wc = wait(NULL);
Un ejemplo claro de wait es cuando ejecutamos algún programa con el terminal y mientras ese programa se ejecuta no podemos hacer nada con el terminal, ya que esta en wait. Con el **&** hago que no se le ponga el wait y así sigo pudiendo usar el terminal. *Esto era útil hace mucho ya que solo podíamos tener una consola, no existía la multiplexacion*
### Sleep
El sleep(1) por ejemplo hace el sleep y salta al return 0. El return 0 es lo mismo que exit(0)
### Sturup
Strdup retorna un puntero a un nuevo string que es el duplicado del string que es, obtiene la memoria con malloc y luego duplica.

---
## Execvp
execvp, esta pensado para funcionar con fork. Agarra el primero de los argumentos que es una cadena de caracteres y busca ese programa dentro del sistema de archivos, lo carga en memoria con los argumentos y empieza a ejecutar ese programa desde cero, o sea desde main. Entonces lo que pasa es que la memoria de todo el programa que estaba ejecutando desaparece porque se reemplaza por otro, se pisa. Entonces podemos decir que fork es copiar otro y execvp lo pisa. El execpv lo que hace es cambiar todo el mapa de memoria, entonces el hijo no muere si no que se transforma. Entonces con esta función o comando no debemos chequear errores ya que si hay algún error la función se transforma. Por ejemplo podemos hacer que de un error pasandole como argumento un comando no valido.

---
## User space y kernel space
***Hay procesadores que aceptan programas que otras no debido que manejan otras instrucciones mas modernas.***
Hay programas que tienen permisos especiales para poder hacer mas cosas sensibles como leer un puerto mientras que otros programas no tienen dichos permisos. Uno es el **user space** y el otro el **kernel space**, donde en el **espacio de usuario puedo hacer algunas cosas restringidas** para que en el espacio de kernel tenga todo el control y programas que se comporten mal se puedan agarrar y devolver todos los recursos que tenían
El modo usuario es un modo en el que no podemos hacer cosas malas, **al modo kernel no podemos acceder pero este puede hacer todo** y debemos confiar en el. El usuario cuando quiere acceder al hardware tiene que pasar por el kernel.
### System call y trap
system call es una función del sistema operativo.
Entonces el usuario le pide permiso al **kernel** para hacer cosas como leer en la pantalla a través de las **system calls**, el usuario le pide permiso al kernel para hacer cosas. **Cuando sucede esto se abre una puerta, se llama** **trap**, **es un momento donde paso de espacio usuario a espacio kernel** *ya sea por algún programa o que paso algún evento*, es ir abajo, ir al kernel (Es como si fuera un branch, debe preservar todos los registros) Luego cuando la llamada al sistema termina hay que hacer un **untrap**, o sea volver desde el fondo del sistema operativo, entonces cuando vuelve como yo tuve que guardar todos los registros anteriores seria posible cambiar del programa que estaba ejecutando a uno nuevo.
El hardware tiene que trabajar en conjunto con el sistema operativo, toda **I/O** pasa por el kernel, **por ejemplo un printf genera un syscall.**
### Formas de entrar al kernel
Entrar al kernel se puede hacer por un programa que lo necesite (trap por software) o también se puede hacer una instrucción ilegal, o acceder a una dirección de memoria prohibida o dividir por 0, o dividir un puerto que no podemos. También se puede entrar por hardware, por ejemplo cada vez que aprieto una tecla genera una **interrupción**, cuando muevo el mouse genera otra. Cada 100ms se genera una interrupción que genera un trap y entra al kernel, cada vez que la cámara hace el barrido genera una interrupción
### Interrupción
Una interrupción es un trap generado por hardware en algún momento que yo no se.
Una computadora tiene muchísimas interrupciones aunque no este haciendo nada, por ejemplo tocar el touchpad, apretar teclas, recibir paquetes de red son cosas que generan interrupciones. Leer el disco duro genera muchas interrupciones.
Las interrupciones en cierta medida están limitadas por la velocidad de las demás partes de la computadora, por ejemplo de la velocidad del disco, de la velocidad a la que usemos el touchpad, de la velocidad la que tecleamos, etc.
### Relación entre el SO, kernel y el programa
Veamos un poco como es que se relacionan el sistema operativo(kernel), el hardware y el programa. Para ello analicemos el siguiente diagrama:

![ScreenShot](Imagenes/relacion_kernel_usuario_programa.png)
- Primero el sistema operativo inicializa la **tabla de trap** *(en donde están todos los números de direcciones de memoria)*
- El hardware recuerda las direcciones necesarias *(le decimos el nro de la base de la trap table)*
- El sistema operativo quiere crear un proceso, para ello agarra el primer lugar del **process table**, pide memoria y adecua la pila para que apunte al **argc y argv**, luego como el programa esta iniciando manda una señal al kernel como si lo hubieran interrumpido antes
- El hardware restablece los registros, cambia a modo de usuario y salta al punto de inicio del programa que es el main
- Luego el programa corre el main y luego hace un system call provocando una trap al sistema operativo
- El hardware guarda los registros en el stack de kernel, se cambia a modo kernel y salta a donde dice la tabla de traps para este trap en particular *(tabla la cual el sistema operativo definió al comienzo)*
- Vuelve al sistema operativo y ejecuta la rutina en **código de maquina** que se corresponde con el systemcall, hace lo que tenga que hacer y vuelve del trap
- Luego se restablecen los registros de nuevo *(desde el kernel stack)*, se cambia a modo usuario y vuelve a ejecutar el programa en el punto que lo estaba ejecutando
- Luego el programa hace sus cosas hasta que el programa llega a una llamada del sistema exit que trapea al kernel
- Por ultimo el sistema operativo libera la memoria que estaba usando el y borra el proceso de la lista de procesos
***Por la tanto vemos el que hardware es fundamental.*** 
Notemos que se pierde tiempo entrando y saliendo del kernel. También notemos que todo programa termina en modo kernel.
Los discos tienen latencia (por ejemplo desde que abro la canilla tarda en salir agua) entonces desde que pido un archivo tarda cierto tiempo en devolverlo
### Relación entre el espacio de usuario y espacio de kernel
Veamos como se relacionan entre si el espacio de usuario y espacio de kernel, para ello vamos a analizar la siguiente imagen:
![ScreenShot](Imagenes/modos_kernel_y_usuario.png)
#### Primer programa (arriba)
- Los picos de las lineas de colores representan como se ejecuta o como avanza el **program counter(PC)** dentro de la memoria *(suponiendo que solo tenemos un procesador)*. En este caso como es un fork deberían ser iguales los picos aunque no lo son por estar mal dibujado
- Al final de los procesos deberían bajar a modo kernel para terminar ya que siempre al terminar se lo hace por medio de un exit
- En este caso el syscall es un fork entonces por eso se crea el proceso B
- Cuando se crea el nuevo proceso salta despues de la systemcall, salta al mismo punto en donde estaba el proceso A *(por eso esta en el mismo nivel de altura)* porque es proceso hijo del A
- Las lineas rosas indican la misma altura cuando se realiza la primer syscall en A y cuando se realiza la ultima syscall en B *(aunque en B esta mas abajo pero deberían tener la misma altura)* ya que indica la posición donde se esta ejecutando el programa en este momento *(ya que al hacer un syscall trapeo en el kernel y luego el entry point para entrar al kernel de nuevo es el mismo)*
- El punto para treapear y destrapear debería ser el mismo
- Durante la linea verde el hardware guarda todos los registros en el kernel stack, ademas pasa de modo usuario a modo kernel y empieza a ejecutar en un lugar especificado que le consulta a la trap table y luego comienza a ejecutar en modo kernel las rutinas necesarias. Cuando termina vuelve los registros a como estaban, salta para el proceso A y vuelvo a poner el stack en como estaba porque quiero que el proceso trabaje todos los datos en modo usuario siguiendo ejecutandose como si nada hubiera pasado. Entonces por eso me es posible pausar una ejecucion como en el programa B
- Cuando se produce un trap se switchea del user stack al kernel stack, el paso se hace por harware y es automático
#### Segundo programa (abajo)
- Tenemos una **timer trap** *(o sea que ocurre cada 100ms)* ***OJO NO NECESARIAMENTE SI HAY UN TIMER TRAP SE CAMBIA DE CONTEXTO***
- Arranca el proceso A, se hace la trap pasa a modo kernel, luego el kernel ejecuta la syscall de A durante un tiempo y decide cambiar de contexto y ejecutar en modo kernel para el proceso B, luego hace un return from trap, el proceso B hace sus cosas y de nuevo se ejecuta la syscall del proceso B, entra a modo kernel hace sus cosas y decide cambiar de contexto otra vez ahora el proceso A y luego hace un return from trap y vuelve al programa. Ahi deberia volver otra vez al kernel para finalizar por el exit
- Cuando en el modo kernel cambia de contexto, el proceso A pasa de running a ready *(los cambios de contexto solo se dan en modo kernel)*. Cuando se hace el cambio de contexto también se guardan los registros y hace un salto en el program counter
### Ejemplo de proceso modo usuario unicamente
Un ejemplo de un proceso que nunca cambia modo kernel es un `while(true)`, ya que esta linea de codigo no genera ninguna syscall entonces queda en un bucle infinito y constante en modo usuario.
### El kernel
El kernel rápidamente debe hacer sus cosas y volver a modo usuario, el SO quiere que estemos en modo kernel la menor cantidad de tiempo posible.
### Multitasking cooperativo
El multitasking cooperativo es utilizado en dispositivos muy lentos en donde yo no cambio a modo kernel, si no que le doy el control de forma voluntaria.
### Cache
La cache es la que nos permite tener mas cambios de contexto, mientras mas cache tenga mejor.

---
## **Planificación**
Es algo político, tomar decisiones, administrar los recursos lo cual es hago bastante difícil de hacer.
### Mecanismos
Con mecanismos nos referimos a entrar al kernel por alguna interrupción ya sea de software o de hardware, o sea que yo pido entrar o porque trapeo por algún necesidad. Este mecanismo permanece inalterable ya que no va a cambiar.
### Política
Tengo muchos procesos que quieren correr los cuales están en ready pero a su vez tengo pocas cpu capaces de correr esos procesos. Existe un algoritmo que decide cual de todos esos procesos va a pasar de ready a running.
### Mecanismos vs políticas
Existe una clara diferencia entre mecanismos y políticas. La parte de mecanismo es la que permanece igual a lo largo del tiempo mientras que la parte política es la que cambia, por ejemplo mediante un archivo de configuración, a través de una selección de un tiempo de ejecución. Hay una división entre el código que va a cambiar y el código que no va a cambiar.
### Tipos de planificadores
Vamos a suponer que tenemos una sola CPU. Recordemos que sigue existiendo la interrupción cada 100ms.

| POLITICA                 | ¿Apropiativa? | INTERACTIVA | ¿TENER EL TIEMPO? | ¿JUSTA?    | PROS                                | CONTRAS                                        |
| ------------------------ | ------------- | ----------- | ----------------- | ---------- | ----------------------------------- | ---------------------------------------------- |
| FIFO(first in first out) | No            | Lotes       | No                | Si         | Es muy sencilla de implementar      | Un proceso puede monopolizar                   |
| RR (round robin)         | Si            | Interactiva | No                | Si         | Protege ante monopolizacion del CPU | Pierde tiempo al tener que fijarse en la tabla |
| SJF                      | No            | Lotes       | SI                | Starvation | Throughput (rendimiento)            | Monopoliza                                     |
| STCF                     | Si            | Interactiva | Si                | Starvation | Throughput (rendimiento)            |                                                |
| Selfish                  |               |             |                   |            |                                     |                                                |
| MLFQ                     |               |             |                   |            |                                     |                                                |
*Aclaraciones:* 
- *Una política apropiativa(preintrive) es la que es capaz de agarrar un proceso que se esta ejecutando y cortarlo por alguna razón. Utiliza el cambio de contexto en medio de la ejecución y guarda todo el estado de proceso, registros y como estaba la memoria en la tabla de procesos.*
- *El tiempo t es el tiempo total de ejecución*
- *Que sea interactivo es que rápidamente los procesos pueden pasar a running*
- Justa significa que eventualmente atienda todos los procesos
Hablemos un poco mas en detalle de cada una:
#### FIFO (first in first out)
Es una politica basada en que quien llega primero se ejecuta primero. Es sencilla de implementar
Algunos ejemplos de diagramas de tiempo para FIFO son:

![ScreenShot](Imagenes/FIFO_1.png)
![ScreenShot](Imagenes/FIFO_2.png)

Entonces vemos procesos que se enumeran (en este caso por letras) ya que llegaron al mismo tiempo, vemos como se multiplexan por tiempo
Es de tipo por lote(Batch), primero todo A, luego todo B y luego todo C. Ademas como rápidamente permite pasar entre procesos es interactivo.
Es justa porque atiende a todos los procesos ya que no voy a ejecutar procesos infinitos
Se suele usar para super computadoras. En estas computadoras el planificador tiene distintas prioridades.
#### RR (round robin) - quanto (cantidad de tiempo que un proceso monopoliza la CPU)
Se basa en que un proceso no puede monopolizar la cpu tiene cierto tiempo para poder ejecutarse y pasado el tiempo vuelve al final de la cola.
El problema del round robin es que cuando ocurre una syscall perdemos mucho rendimiento. Ver impacto de las syscalls.
Por lo tanto depende mucho del **quanto**, uno muy pequeño daría lugar a que atendemos rápido y soltamos pero vamos a tener que hacer muchos cambios de contexto los cuales no son gratis. Si es uno muy grande estaríamos dedicándole tiempo extra a procesos que no lo necesitan.
Si aprieto una tecla despierto al proceso shell y este proceso no se va a ejecutar hasta que le toque el turno ni se va a adelantar. Por lo que por ejemplo si tengo muchos procesos voy a notar que al escribir va a tardar en mostrarse los caracteres en pantalla.
En cierta forma es similar a la FIFO, pero con la diferencia del quanto. Veamos las siguientes imágenes:

![ScreenShot](Imagenes/FIFO_RR.png)

En esta vemos que un FIFO se puede transformar en un RR haciendo uso del quanto, en este caso el quanto es de 1. En el primer gráfico A es quien sale beneficiado, B depende de lo que haga y C sale perjudicado, ya que el tiempo para que se ejecute A es 5, para B es 10 y para C es 15
Pero a su vez por la gran cantidad de cambios de contexto que se dan en el segundo gráfico, el tiempo real va a ser distinto entonces podemos decir que el gráfico es incorrecto porque debería tardar un poquito mas
Es justa porque atiende a todos
Se suele usar para computadoras de escritorio o celulares debido a que necesito una gran interactividad
#### SJF (shortest job first)
Elijo al proceso mas corto y lo ejecuto por lotes. No es apropiativa.
Necesito tener el tiempo de los procesos.
No es justa porque al ejecutar al mas corto primero puede darse la situación que un proceso largo nunca se llegue a ejecutar debido a infinitos procesos cortos que llegan y se ejecutan primero.

![ScreenShot](Imagenes/SJF_1.png)

En este gráfico vemos como A se esta ejecutando y llegan B y C
#### STCF (shortest time to completion first)
Primero van los que tengan menor tiempo en completarse el trabajo, es distinto a SJF ya que ahora compara por tiempo que restante no tiempo total.
Es interactiva.
Lo que pasaría con la figura del SJF si fuera STCF es lo siguiente:

![ScreenShot](Imagenes/STCF.png)

El A estaba ejecutándose y llegan B y C. Hay un context switch porque llegaron B y C entonces sea cual sea la forma por la que llegaron hubo un context switch o sea una interrupción.
Entonces como B y C le restan menos tiempo para terminar su ejecución los ejecuta primero y luego sigue con A.
Esta decisión se va tomando momento a momento.
Para esto es necesario que los procesos cuando lleguen anuncien su tiempo de ejecución.
También es starvation porque si llegan infinitos procesos muy rápidos de ejecutar el proceso largo nunca a se ejecutara.
#### Selfish (egoista)
Ejecuta lo que estaba en ejecución
#### MLFQ
Es mas complejo por eso esta detallado mas abajo
#### CFS
Es el que usa linux
#### Uso de recursos
Entonces el gran truco de las computadoras es hacer un optimo uso de los recursos que dispongo

![ScreenShot](Imagenes/uso_de_recursos.png)
- El primer diagrama es un I/O-bound, un tipo de proceso limitado por entrada/salida
- El otro diagrama es un CPU-bound, un tipo de proceso limitado por CPU
Del buen uso de los recursos surgen los TSS (time sharing system), los sistemas de tiempo compartido. Antes de esto si yo tenia un programa A corriendo y tenia que ir al disco que es lento y durante todo ese tiempo A estaba esperando a que el disco responda y durante todo ese tiempo la CPU no hacia nada.
Entonces lo que se los ocurrió fue aprovechar ese tiempo donde esta operando el disco y meter parte de otros procesos. Entonces vemos que las políticas de planificación cambian porque están activos un tiempo haciendo procesamiento en CPU y están inactivos cierto tiempo porque están haciendo entrada y salida.
#### Interactividad
Entonces vemos que prácticamente todos los procesos son interactivos. Por ejemplo la consola es altamente interactiva porque esta todo el tiempo esperando que yo tipee algo y cuando lo hago lo ejecuta. Por lo tanto **los procesos nunca son puramente de I/O ni puramente de procesamiento de CPU**.
Entonces aprovechando el sistema de cambio de contexto se logra encajar pedacitos de otros programas mientras el CPU no hace nada.

---
## **Impacto de las syscall**
Veamos el siguiente diagrama:

![ScreenShot](Imagenes/syscall_impact.png)

*IPC es Instrucciones por ciclo*
Sabemos que por cada ciclo de reloj se ejecuta una instrucción, o sea que el  **Instruction per cicle = 1**. Ahora los procesadores son superescalares, pueden ejecutar mas de una ejecución por ciclo.
El gráfico muestra cual es el IPC de la instrucciones cuando ocurre en medio de una syscall. Venia ejecutando 1.5 instrucciones por ciclo (promedio) y de golpe llega la syscall y ejecuto muchísimo menos y a medida que pasa el tiempo vuelvo a retomar el ritmo a los 16000 ciclos vuelvo al mismo ritmo que tenia antes. *Es como si estuviera haciendo algo, me desconcentro y vuelvo a concentrarme al cabo de unos segundos.*
Entonces vemos que el **problema del round robin** es que cuando hay una **syscall perdemos rendimiento**

---
## **MLFQ**
### Superposicion de computo
Es hacer algo mientras estamos esperando una entrada.
### ¿¿¿ CTSS (compatible time sharing system) ???
La idea es una cosa que se adapte automáticamente entre los procesos que toman muy poco tiempo y devuelven rápidamente el control y los procesos que son pesados en computo y por lo tanto CPU-bound.
Por lo tanto vemos que intenta ser todos los otros tipos de planificadores a la vez.
**En lugar de tener una cola donde se hace el RR vamos a tener muchas colas donde se hace:**

![ScreenShot](Imagenes/MLFQ_Example.png)

Las primeras colas son de alta prioridad mientras que las ultimas son de baja prioridad. **Las de alta prioridad tienen que quanto muy pequeño y las de baja prioridad tienen el quanto grande**. *(La cola ordena procesos listos para correr, o sea procesos ready)*
Lo que se trata de hacer es poner los procesos I/O-bound como alta prioridad y los CPU-bound como baja prioridad. Pero hay un problema, sabemos que no es posible a ciencia cierta calcular exactamente cuanto va tardar un proceso en ejecutarse. Ademas tampoco puedo saber si un proceso va a ser CPU-bound o I/O-bound, o si va a cambiar durante el tiempo. 
Entonces la idea es hacer un proceso adaptivo que todo el tiempo este evaluando si tiene que bajar o subir de prioridad
### Procesos dinámicos
Por ejemplo pi 1000000 tiene una fase CPU-bound que es cuando los calcula y una parte I/O-bound que es cuando muestra el resultado, por ende todos los procesos son dinámicos
### Regla
La regla que se plantea es: la cola superior que tenga algo va a ejecutar ese. Por lo tanto la cola Q1 va a ejecutar si solo si todas las colas que tiene por encima están vacías (Notar que los procesos A, B, C y D están listos para correr)
Por lo tanto si hay algo en Q8 se ejecuta, si no se mira Q7 y se ejecuta, si no Q6 y se ejecuta y asi sucesivamente...
En cierta forma clasifica los procesos según su calibre, ya que no puedo saber como van a ser los procesos por eso tengo que averiguarlo yo y por eso arranco desde Q1 y a medida que lo necesite muevo el proceso a una cola inferior.
Notar que **hasta ahora** tenemos regla de degradación pero no de promoción, entonces vemos que es una especie de RR (quédate acá mientras no se venza el quanto, cuando se venza baja, entonces eventualmente todos los procesos terminarían en Q1)
### Trampa a la regla
Hagamos un ejemplo con los procesos A y B tal cual en la imagen anterior.
Si se el tamaño de Q8 podría hacer un timer  (por ejemplo de 10ms) y voy a computar hasta 9.99ms y en ese momento hago *¿¿sketkill, schetgill??* (proceso para hacer CPU dependiente), entonces en ese momento el proceso se va hasta el fondo de la cola y empieza a correr B. Luego corre B que cae a Q7, Q6... por lo tanto el proceso A se quedaría siempre en Q8
### Regla parte 2
La regla para subir los procesos es algo rara, se denomina S y cada cierto tiempo sube todos los procesos hacia arriba. Si no hacemos esto, podria darse el caso de que aparezcan muchos procesos chiquitos y que un proceso en Q1 sufra de starvation (que nunca se ejecute)
### Solución a la trampa
Para que no hagan uso de la syscall y hacer caso omiso a las reglas, el planificador decide no tener en cuanta lo que uso actualmente el proceso si no que va sumando todo lo que fue usando desde su creación, entonces por mas que sea un proceso I/O-bound va a acumular CPU y por lo tanto eventualmente bajara 

---
## **Ciclos**
Un ciclo es el tiempo mas largo de una instrucción. Los procesadores super escalares ejecutan mas de una instrucción por ciclo, se llama paralelismo a nivel de instrucciones.
### Tiempos de ejecución
**El tiempo de ejecución en el CPU puede ser mayor que el real**, por ejemplo la ejecución de programa que se ejecuta en 4 nucleos y si tarda 1seg en cada nucleo el tiempo de ejecución es de 1 segundo pero el tiempo de CPU es de 4 segundos. **El tiempo de CPU es la suma de todo el tiempo que estuvo en cualquiera de los nucleos.**
### CPU Migrations
**Por default esta activado el CPU migration** que hace que un proceso se pueda ir a otro nucleo. Las migraciones se producen para tratar de evitar de poner un proceso en un nucleo que esta ocupado con otras cosas
### Sistemas operativos
**Sistemas operativos tiene como misión tratar de mantener la maquina funcional lo mejor posible**
### Fork
Fork hace un clon exacto del ese proceso ejecutando de ese punto exacto donde se hizo el fork copiando absolutamente todo en otro proceso nuevo independiente

---
## **RAM**
Cada proceso que uno forkea cree que tiene toda la RAM disponible, no hay forma de hacer que sepan que hay RAM ocupada.
El hilo es una forma de tener ejecución múltiple dentro de un proceso compartiendo toda la ram que tiene. El hilo esta adentro de un proceso y todos los procesos adentro de ese hilo ven la misma RAM.
Es imposible que un proceso pueda ver la memoria de otro ya que todos los procesos mapean en la misma memoria, por ejemplo dos procesos pueden escribir en la misma memoria. Cada proceso cree que vive en un espacio de memoria para el solo.
### Heap y Stack
Stack es donde se ponen los argumentos de función y la dirección de retorno de los argumentos de función. Si tenemos un programa recursion el stack crece mucho
Heap es donde se hacen los malloc y free. Si tenemos un programa que pide mucha memoria crece mucho el heap.
El stack crece para abajo y el heap para arriba enfrentados de forma tal que si no utilizamos mucho uno el otro si se pueda usar mucho. La suma de los espacios esta fija.

![ScreenShot](Imagenes/heap_and_stack.jpg)

Lo único que no se comparte es el stack. También hay un mecanismo de protección para que no se toquen el stack y el heap
High address es el punto de quiebre de la memoria.
Text es el programa.
*Si el stack creciera indefinidamente podría tocar el programa, por eso es que la memoria inicilizada y el text no lo puedo tocar y esta protegida, así no puedo hacer moco.*
Todo lo que ocupe la memoria del kernel no se puede leer, escribir, acceder ni ejecutar
