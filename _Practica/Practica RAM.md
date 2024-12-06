# API de memoria
![ScreenShot](Imagenes_practico_ram/ej01_ram.png)

| Variable | Lugar                                         |
| -------- | --------------------------------------------- |
| N        | program code 1024                             |
| a        | bss (esta entre pc y el heap, por ser global) |
| argc     | stack (variable preformada)                   |
| argv     | stack (variable preformada)                   |
| i        | stack                                         |
| s        | cpu (por ser register)                        |
| b        | stack (direccion de memoria)                  |
| *b       | heap (contenido de la variable)               |
| return s | stack                                         |

Si iniciamos un arreglo `int a[N] = 0` se inicia en el bss ya que aca se almacenan las variable globales sin inicializar (o sea con 0)

![ScreenShot](Imagenes_practico_ram/ej02_ram.png)

```c
char *s = malloc(512);
gets(s);
```
Falta el sizeof. Ademas el gets no se debe usar segun la man


```c
char *s = "Hello Waldo";
char *d = malloc(strlen(s));
strcpy(d,s);
```
Falta el +1 en el strlen

```c
char *s = "Hello Waldo";
char *d = malloc(strlen(s));
d = strdup(s);
```
No hay que hacer malloc, con el strdup basta


```c
int * a = malloc(16);
a[15] = 42
```
Memoria mal pedida

![ScreenShot](Imagenes_practico_ram/ej03_ram.png)

a)Falso, es una funcion de una libreria de C
b)Falso, puede reutilizar memoria que ya pidio
c)Verdadero, hace brk
d)Verdadero
e)No, siempre es el mismo


![ScreenShot](Imagenes_practico_ram/ej04_ram.png)

```js
0: movl $128, %ebx
5: movl (%ebx), %eax
8: shll $1, %ebx
10: movl (%ebx), %eax
13: retq
```
Notemos que se lee de izq a derecha (xq por ej con la primer linea no puedo cargar el valor ebx a un registro que se llame 128)

El programa hace lo siguiente:
- Primero carga el valor 128 en el registro ebx (indico con un $ que es un nro y con un % que es un registro)
- Segundo leo la direccion de memoria ebx y lo que hay ahi lo copio hacia eax
- Tercero multiplico por dos el contenido del registro de ebx. O sea hago 128x2 = 256
- Cuarto leo la dirrecion de memoria ebx y guardo eso en eax (o sea ahora ebx=256)
- Quinto hago return

Notemos que el registro base es 4096 y el bounds es 256
Luego la secuencia de accesos a memoria es la siguiente (hacemos memoria fisica + memoria virtual):

- 4096 + 0 = 4096(por ser registro base + 1ra direccion virtual)
- 4096 + 5 = 4101 (base + dv) ; 4096 + 128 = 4224 (base+llamada del registro)
- 4096 + 8 = 4104 (base + dv) 
- 4096 + 10 = 4106 (base + dv) ; 4096 + 256 = 4352 (base+llamada del registro) pero como este esta fuera del limite => SEGMENTATION FAULT

# Manejo de espacio libre

![ScreenShot](Imagenes_practico_ram/ej09_ram.png)

Primero recordemos como son los ajustes:

- First fit: Devuelve el primer chunk con memoria disponible que pueda satisfacer la solicitud.
- Best fit: Primero busca todos los chunks de memoria con tamaño mayor al de la peticion y devuelve el de menor tamaño entre ellos
- Worst fit: Devuelve el chunk de memoria mas grande posible con el objetivo de dejar solo chunks grandes de memoria
- Next fit: Deja un puntero en el ultimo chunk de memoria que se busco funcionando como un first fit pero usando el puntero como partida.

Entonces tenemos los siguientes chunks de memoria ordenados de la siguiente manera:
```js
10KiB, 4KiB, 20KiB, 18KiB, 7KiB, 9KiB, 12KiB, 15KiB
  c1    c2    c3     c4     c5    c6    c7     c8 //Solo es para nombrarlos
```
Y tenemos que atender la siguiente solicitud:
```js
12KiB, 10KiB, 9KiB
```

Veamos como se comportan las distintas politicas ante estas peticiones:

***First fit***: 
```js
//12KiB => Uso el chunk3
10KiB, 4KiB, 8KiB, 18KiB, 7KiB, 9KiB, 12KiB, 15KiB
//10KiB => Uso el chunk1
0KiB, 4KiB, 8KiB, 18KiB, 7KiB, 9KiB, 12KiB, 15KiB
//9KiB => Uso el chunk4
0KiB, 4KiB, 8KiB, 9KiB, 7KiB, 9KiB, 12KiB, 15KiB
```

***best fit***: 
```js
//12KiB => Uso el chunk7
10KiB, 4KiB, 20KiB, 18KiB, 7KiB, 9KiB, 0KiB, 15KiB
//10KiB => Uso el chunk1
0KiB, 4KiB, 20KiB, 18KiB, 7KiB, 9KiB, 0KiB, 15KiB
//9KiB => Uso el chunk6
0KiB, 4KiB, 20KiB, 18KiB, 7KiB, 0KiB, 0KiB, 15KiB
```
***worst fit***: 
```js
//12KiB => Uso el chunk3
10KiB, 4KiB, 8KiB, 18KiB, 7KiB, 9KiB, 20KiB, 15KiB
//10KiB => Uso el chunk7
10KiB, 4KiB, 8KiB, 18KiB, 7KiB, 9KiB, 10KiB, 15KiB
//9KiB => Uso el chunk4
10KiB, 4KiB, 8KiB, 9KiB, 7KiB, 9KiB, 10KiB, 15KiB
```
# Paginacion

![ScreenShot](Imagenes_practico_ram/ej10_ram.png)



![ScreenShot](Imagenes_practico_ram/ej11_ram.png)

Tenemos paginas de 4KiB y un TLB con 64 entradas.

```c
int x[N];                   //Declaro arreglo de tamaño N
int step = M;               //Declaro step = M
for (int i=0; i<N; i+=step) //Itero haciendo i= i+setp
	x[i] = x[i]+1;          //x[i] es el siguiente que deberia
```

Con un M>1024 no funciona porque son 64 entradas y como son int son 4bytes, entoces haria 4x1025>4096 Segmantation fault. N no influye en nada

Si pasamos las 64 iteraciones no sabemos donde vamos a escribir

![ScreenShot](Imagenes_practico_ram/ej12_ram.png)

![ScreenShot](Imagenes_practico_ram/ej12tabla_ram.png)

Tenemos paginas de 4KiB => 4xKiB = 2² x 2¹⁰ = 2¹²

Entonces como tenemos en total 16 bits vemos que estos son de offset (el final) ya que la tabal esta formada por los primero 4bits de los 16totales (16 por la cantidad que hay).
Entonces

0000 valido
0001 valido
0010 no
0011 valido
0100 valido
0101 valido
0110 no
0111 no
1000 valido
1001 valido
1010 valido
1011 no
1100 no
1101 no
1110 no
1111 valido

No hay nignun patron.

a)bits de direccionamiento virtual
VPN + offset = 4 + 12 = 16

bits de direccionamiento fisico 
FPN + offset = 3 + 12 = 15

b)

```python
#Direccion virtual 39424
39424 / 4096 = 9 + 0.625*1024*4 = 9 + 2560B
pagina virtual = 9 me fijo en la tabla a que valor se refiere el 9
pagina fisica = 6

direccion fisica = pagina fisica * tamaño de pagina + offset
direccion fisica = 6 * 4096 + 2560
direccion fisica = 27136
```


![ScreenShot](Imagenes_practico_ram/ej13_ram.png)

```js
0x003FF666
0000 0000 0011 1111 1111 0110 0110 0110

idxpage= 0000 0000 00 => 0
idxpagetable = 11 1111 1111 => 1023
offset = 0110 0110 0110 => 0x666

Entonces la direccion fisica es: 0xCAEEA666

```

![ScreenShot](Imagenes_practico_ram/ej14_ram.png)



![ScreenShot](Imagenes_practico_ram/ej15_ram.png)



![ScreenShot](Imagenes_practico_ram/ej16_ram.png)



![ScreenShot](Imagenes_practico_ram/ej17_ram.png)



![ScreenShot](Imagenes_practico_ram/ej18_ram.png)


