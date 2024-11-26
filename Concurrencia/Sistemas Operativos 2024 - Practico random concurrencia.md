Considere los procesos P0 y P1 a continuacion

```c
				Pre: n=0 & m=0

P0: while (n<100) {            P1: while (n<100) {
	n = n*2;                       n = n+1
	m = n;                         m = n;
}                              }

				Post: n=? & m=?

Veamos un poco como puede darse la ejecucion. Supongamos que se ejecuta P0, entra al while pero sucede un context switch a P1 el cual se ejecuta 99 veces, por lo que n=99 y m=99. Luego se ejecuta otra vez pero solo la primer linea haciendo que n=100 y sucede un context y volvemos de nuevo a P0 estando dentro del while por lo que ahora n=200 y m=200. Por ultimo en P0 vuelve a verificar la condicion del while pero no la pasa por lo que ahora volvemos con P1 que ejecuta m=n o sea que m=200 y se verifica tambien la condicion la cual no la pasa y listo.

a) ¿A cuanto pueden diferir como maximo m y n durante la ejecucion?

b)¿En cuantas iteraciones termina? Indicar maximo y minimo

c)¿Que valores pueden tomar n y m en la Post? Justificar
Los valores que pueden tomar n y m estan entre [100, 200]
```

```c
El siguiente programa asegura exclusion mutua en las regiones criticas:

						t=0 & !c0 & !c1 (c0 y c1 negados)

	P0:                                  P1:
	1:    while(1) {                       A:    while(1){
	2:        {Region no critica}          B:        {Region no critica}
	3:        (c0,t) = (true,1)            C:        (c1,t) = (true,0)
	4:        while(t!=0 && c1);           D:        while(t!=1 && c0);
	5:        {Region critica}             E:        {Region critica}
	6:        c0 = false                   F:        c1 = false
	7:    }                                G:    }

Es el algoritmo de Peterson, tiene tres variables, dos booleanos (c0 y c1) y el turno(t). Los booleanos indican la intencion de entrar por cada proceso y el t indica de quien es el turno para entrar.
Recordemos que este algoritmo solo sirve para 2 nucleos.
La logica de la linea 4 o D es: mientras el turno no sea el mio y el otro quiera entrar segui dando vueltas.

```

```c
Dados estos 2 procesos en paralelo

					Pre: x=0
	P0: while(1) {              P1: while(1) {
	    x = x+1;                    x = x+1;
	    x = x-1;                    x = x-1;
	}                           }

a)¿El multiprograma termina?
No, si cualquiera de las componente no termina entonces el multiprograma no termina. En este caso ninguna de las dos componentes terminan.

b)¿Que valores puede tomar x? Pensando en las operaciones son atomicas
0, 1, 2

0: porque arranca en 0
1: Ejecuta P0 y se ejecuta la primer linea
2: IDEM que al anterior pero sucede un context switch hacia P1 y ejecuta la primer linea

No puede nunca valer negativo porque para ejecutar las restas antes deben suceder las sumas.
```

```c

```
