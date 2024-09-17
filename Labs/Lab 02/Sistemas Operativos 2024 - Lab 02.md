## Makefile
En el makefile hay que modificar el U y ponerle el nombre de nuestro programa
Recomendable empezar con con echo.c

## Que se tiene que aguantar
Nuestras system calls deben ser capaces de aguantar nro negativos, nros grandes, nros chiquitos, punteros vacíos, etc. Las llamadas al sistemas son el punto de entrada al kernel por lo que debemos sanitizar las entradas lo mejor que podamos.