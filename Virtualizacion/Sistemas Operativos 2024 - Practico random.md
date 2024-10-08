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