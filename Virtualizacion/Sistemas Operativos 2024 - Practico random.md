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

Entonces traduzcamos de virtual a fisica

```js
0x00001FFF => 0xAC01AFFF //Notar que aunque esta presente no se puede acceder porque RWX=000
0x00000AAA => 0xFC0CAAAA
0x00000AAB => 0xFC0CAAAB
0x00000000 => 0xFC0CA000
0x00040EF0 => No sabemos porque no tenemos la entrada 64 en la tabla
0x0000DEAF => IDEM que anterior con 13
0x00003FFF => No es valida => Page Fault (PF)
```

