Traducción de direcciones
```js
0: movl $128, %ebx
5: movl (%ebx), %eax
8: shll $1, %ebx
10: movl (%ebx), %eax
13: retq
```
**Primero siempre debo ver si se lee de izq a der o de der a izq**
Hacen lo siguiente:
- Carga el registro ebx con el nro 128 (*en este caso se lee de izq a der xq no puedo cargar un 128 con ebx*)
- Lee la dirección de memoria 128 (*por ser ebx*) y carga eso en el registro eax. (*notar que al tener paréntesis indica que accedo a un puntero*)
- Multiplica ebx x 2 (xq muevo 1 lugar hacia la izq)
- Leer la dirección de memoria 256 y cargar eso en el registro eax
Luego la secuencia de acceso a memoria física seria:
- 4096 (*xq es la base*)
- 4096+5 = 4101 (base+dirección virtual que accedo); 4096+128 = 4224 (base+llamada del código)(notemos que una instrucción provoco dos accesos a memoria)
- 4096+8 = 4104
- 4096+10 = 4106; 4096+256 = 4352(xq accedo al doble de 128 que es ebx) Pero este ultimo esta fuera del limite => Segmentation fault (notar que 4096+255 no da segfault)

### Arreglos
En C los puntero y arreglos son lo mismo, cuando me refieron por ejemplo a:
`a[N]` hago referencia a la posicion de memoria en donde empieza a
Los arreglos son un puntero al inicio del arreglo, podriamos hacer lo siguiente:
```c
int main(void) {
	int a[16];
	a[5] = 4;
	*(a+5) = 6; //Es lo mismo que la linea enterior
	*(5+a) = 8; //Lo mismo que la linea anterior
	5[a] = 10;  //Lo mismo que la linea anterior
}
```
Si pongo una variable como registro no esta en memoria sino que esta en el procesador.
El define lo que hace es sustituir el texto por el valor que le dimos
Si hago una variable global y la inicializo en 0 por mas grande que sea va a ocupar poco espacio y se almacena en el segmento BSS. Si la inicializo en 1 se guarda en el segmento text y va a ocupar el espacio correspondiente.

### TLB
¿Que pasa si en vez de usar un esquema (4,4) uso un esquema (2,6)? O sea si uso 2bits para marco de pagina y 6bits para offset.

En ese caso, tendriamos menos TLB miss, ya que en una pagina podriamos tener mas elementos.

Por ejemplo:

Si tenemos paginas de 4k, con 512 entradas en la TLB ¿Cual es el encubrimiento de la TLB?

```js
512*4KiB = 2048KiB = 2MiB
```

¿Y si las paginas son de 4MiB?

```js
512*4MiB = 2048MiB = 2GiB
```

***AutoHugePage*** permite 

### Paginacion lineal o tablas de un nivel
Tenemos lo siguiente:

```js
CR3 = 0x1FAFA //Tiene 20bits

entrada, marco fisico, P, RWX 
//marco fisico dice si esta presente, read, write, execute
0: 0xFC0CA, 1, 101
1: 0xAC01A, 1, 000
2: 0xFC0CA, 0, 111
3: 0xA05AD, 0, 000
4: 0xFDEAD, 1, 111
```

Queremos traducir de memoria virtual a memoria física.

```js
0x00001FFF
0x00000AAA
0x00040EF0
0x0000DEAF
```

Primero notemos que estas direcciones tienen 32bits. Entonces primero tenemos que dividir en 2:

```c++
//Por ejemplo la siguiente direccion quedaria asi:
direccion => entrada en la tabla de pagina, desplazamiento
0x00001FFF => 00001, FFF
```

Entonces me dice que debo mirar la entrada numero 1 de la tabla de paginas. Pero ahora nos podríamos preguntar ¿Donde esta la tabla de paginas? ***Esta donde apunta CR3*** 

En este caso nuestro **CR3 = 0x1FAFA** y su entrada numero 1 es **0xAC01A** (lo vemos en la tabla del comienzo)

Entonces ahora debemos traducir de virtual a fisica. Para ello debo agarrar la tabla mirar el marco numero 1 (en este caso) y tomar su marco fisico (o sea los 5hexa) y pegarlo en la direccion fisica y luego seguido pegar el desplazamiento

Entonces traduzcamos de ***virtual a fisica***

```js
0x00001FFF => 0xAC01AFFF //Notar que aunque esta presente no se puede acceder porque RWX=000
0x00000AAA => 0xFC0CAAAA
0x00000AAB => 0xFC0CAAAB
0x00000000 => 0xFC0CA000
0x00040EF0 => No sabemos porque no tenemos la entrada 64 en la tabla
0x0000DEAF => IDEM que anterior con 13
0x00003FFF => No es valida => Page Fault (PF)
0x00002000 => 0xFC0CA000 
0x00000000 => 0xFC0CA000
//Comparto la misma direccion fisica entre varias direcciones virtuales, es util por ejemplo con las librerias
```


***Traducir de fisica a virtual:***

```js
0xA05AD444 => 0x00003444
0xFC0CA111 => 0x00000111 && 0x00002111
0xFC0CA222 => 0x00000222 && 0x00002222
0xC0C0C0C0 => No sabemos
```


### Paginacion de dos niveles, la realidad en i386
***Esquema (10, 10, 12)***
CR3 = 0x01000, tiene 20bits

| 0x01000                                                                                     | 0x00001                                                                | 0x00002                                                               |
| ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 0: 0x00001, 1, 111<br>1: 0x00002, 1, 111<br>2: 0x00001, 1, 111<br>3: 0x00000, 0, 111<br>... | 0: 0xFAFAF, 1, 111<br>1:  0xC0CC0, 1, 111<br>2: 0xF0F0F, 1, 111<br>... | 0: 0x11111, 1, 111<br>1: 0x11111, 1, 111<br>2: 0x11111, 1, 111<br>... |
| 3FE:<br>3FF: 0x01000, 1, 111                                                                | 3FE:<br>3FF: 0xE0E0E, 1, 111                                           | 3FE: 0x11111, 1, 111<br>3FF: 0x11111, 1, 111<br>                      |

Notemos que son 1024 entradas.
***Traducir de virtual a fisica:***

Primero vemos que la columna izquierda es page directory porque corresponde con el CR3.

Lo importante del esquema (10, 10, 12) es que en vez de decirnos la primera columna cual es el marco de pagina fisico final nos dice cual es el marco de pagina fisico donde esta la page table. Entonces la tabla del CR3 nos dice donde esta la page table que voy a terminar leyendo.

Entonces los primeros 10bits forman el idxpgdir, los siguientes 10 el idxpgtable y los ultimos 12 el offset.

En el primer caso, veo que el idxpgdir es 0, el idxpgtab es 0 y el offset es 0. Entonces primero me fijo el valor 0 (porque el idxpgtab es 0) en la tabla 0x010000 y veo que ese valor es el 0x00001. Entones me voy a esa tabla y me fijo el primer valor (porque el idxpagdir es 0) el cual es 0xFAFAF. Entonces pongo eso + el offset.

```js
0x00000000 => idxpgdir=0, idxpgtab=0, offset=0 => 0xFAFAF000
0x00000FFF => idxpgdir=0, idxpgtab=0, offset=FFF => 0xFAFAFFFF
0x00002FFF => idxpgdir=0, idxpgtab=2, offset=FFF => 0xF0F0FFFF
0x00001FAA => idxpgdir=0, idxpgtab=1, offset=FAA => 0xC0CC0FAA
0x00802AAA => (0000 0000 10, 00 0000 0010, 1010 1010 1010)
			  idxpgdir=2, idxpgtab=2, offset=AAA => 0xF0F0FAAA
0xFF000000 => (1111 1111 00, 00 0000 0000, 0000 0000 0000)
			  idxpgdir=3FF, idxpgtab=0, offset=0 => PAGE FAULT (mapea al 3FE del 0x00001 que no esta indicada)
```