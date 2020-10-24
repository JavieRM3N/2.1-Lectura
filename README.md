![](imagen/portadatcnm.png)

#    Tecnológico Nacional de México
#   Instituto Tecnológico de Tijuana
#        Subdirección Académica

# Departamento de Sistemas y Computación
# Ingeniería en Sistemas Computacionales
# Lenguajes de interfaz 

# Practica Bloque: 2.1 Lectura y ejercicios de ARM32 del ebook OpenSource


# Mendoza Ramos Christian Javier
# Nu. Control: 18212223
   

# Profesor:
# MC. René Solis Reyes
# Semestre sep - ene 2020

-----
1.1. Lectura previa
1.1.1. Características generales de la arquitectura ARM
ARM es una arquitectura RISC (Reduced Instruction Set Computer=Ordenado con Conjunto Reducido de Instrucciones) de 32 bits, salvo la versión del core ARMv8-A que es mixta 32/64 bits (bus de 32 bits con registros de 64 bits).

El chip en concreto que lleva la Raspberry Pi es el BCM2835, se trata de un SoC (System on a Chip=Sistema en un sólo chip) que contiene además de la CPU otros elementos como un núcleo GPU (hardware acelerado OpenGL ES/OpenVG/Open EGL/OpenMAX y decodificación H.264 por hardware) y un núcleo DSP (Digital signal processing=Procesamiento digital de señales) que es un procesador más pequeño y simple que el principal, pero especializado en el procesado y representación de señales analógicas. 

Las extensiones de la arquitectura ARMv6k frente a la básica ARMv6 son mínimas mas por lo que a efectos prácticos trabajaremos con la arquitectura ARMv6.

**Registros

La arquitectura ARMv6 presenta un conjunto de 17 registros (16 principales más uno de estado) de 32 bits cada uno.

   *Registros Generales. Su función es el almacenamiento temporal de datos. Son los 13 registros que van R0 hasta R12.
   *Registros Especiales. Son los últimos 3 registros principales: R13, R14 y R15.

Como son de propósito especial, tienen nombres alternativos.

-SP/R13. Stack Pointer ó Puntero de Pila. Sirve como puntero para almacenar variables locales y registros en llamadas a funciones.

-LR/R14. Link Register ó Registro de Enlace. Almacena la dirección de retorno cuando una instrucción BL ó BLX ejecuta una llamada a una rutina.

-PC/R15. Program Counter ó Contador de Programa. Es un registro que indica la posición donde está el procesador en su secuencia de instrucciones. Se incrementa de 4 en 4 cada vez que se ejecuta una instrucción, salvo que ésta provoque un salto.

-Registro CPSR. Almacena las banderas condicionales y los bits de control. Los bits de control definen la habilitación de interrupciones normales (I), interrupciones rápidas (F), modo Thumb 1(T) y el modo de operación de la CPU. Existen hasta 8 modos de operación, pero por ahora desde nuestra aplicación sólo vamos a trabajar en uno de ellos, el Modo Usuario. Los demás son modos privilegiados usados exclusivamente por el sistema operativo.

Desde el Modo Usuario sólo podemos acceder a las banderas condicionales, que contienen información sobre el estado de la última operación realizada por la ALU. A diferencia de otras arquitecturas en ARMv6 podemos elegir si queremos que una instrucción actualice o no las banderas condicionales, poniendo una “s” detrás del nemotécnico 2. Existen 4 banderas y son las siguientes:

*N. Se activa cuando el resultado es negativo.

*Z. Se activa cuando el resultado es cero o una comparación es cierta.

*C. Indica acarreo en las operaciones aritméticas.

*V. Desbordamiento aritmético.

**Esquema de almacenamiento

El procesador es Bi-Endian, quiere decir que es configurable entre Big Endian y Little Endian. Aunque nuestro sistema operativo nos lo limita a Little Endian. Por tanto la regla que sigue es “el byte menos significativo ocupa la posición más baja”. Cuando escribimos un dato en una posición de memoria, dependiendo de si es byte, half word o word,... se ubica en memoria.

La dirección de un dato es la de su byte menos significativo. La memoria siempre se referencia a nivel de byte, es decir si decimos la posición N nos estamos refiriendo al byte N-ésimo, aunque se escriba media palabra, una palabra,... 

**1.1.2. El lenguaje ensamblador

El ensamblador es un lenguaje de bajo nivel que permite un control directo de la CPU y todos los elementos asociados. Cada línea de un programa ensamblador consta de una instrucción del procesador y la posición que ocupan los datos de esa instrucción.

Desarrollar programas en lenguaje ensamblador es un proceso laborioso. El procedimiento es similar al de cualquier lenguaje compilado. Un conjunto de instrucciones y/o datos forman un módulo fuente. Este módulo es la entrada del compilador, que chequea la sintaxis y lo traduce a código máquina formando un módulo objeto. Finalmente, un enlazador (montador ó linker) traduce todas las referencias relativas a direcciones absolutas y termina generando el ejecutable.

El ensamblador presenta una serie de ventajas e inconvenientes con respecto a otros lenguajes de más alto nivel. Al ser un lenguaje de bajo nivel, presenta como principal característica la flexibilidad y la posibilidad de acceso directo a nivel de registro. En contrapartida, programar en ensamblador es laborioso puesto que los programas contienen un número elevado de líneas y la corrección y depuración de éstos se hace difícil.

Generalmente, y dado que crear programas un poco extensos es laborioso, el ensamblador se utiliza como apoyo a otros lenguajes de alto nivel para 3 tipos de situaciones:

- Operaciones que se repitan un número elevado de veces.

- Cuando se requiera una gran velocidad de proceso.

- Para utilización y aprovechamiento de dispositivos y recursos del sistema.

La principal característica de un módulo fuente en ensamblador es que existe una clara separación entre las instrucciones y los datos. La estructura más general de un módulo fuente es: 

* Sección de datos. Viene identificada por la directiva .data. En esta zona se definen todas las variables que utiliza el programa con el objeto de reservar memoria para contener los valores asignados. Hay que tener especial cuidado para que los datos estén alineados en palabras de 4 bytes, sobre todo después de las cadenas. Alinear significa rellenar con ceros el final de un dato para que el siguiente dato comience en una dirección múltiplo de 4 (con los dos bits menos significativos a cero). Los datos son modificables. 

* Sección de código. Se indica con la directiva .text, y sólo puede contener código o datos no modificables. Como todas las instrucciones son de 32 bits no hay que tener especial cuidado en que estén alineadas. Si tratamos de escribir en esta zona el ensamblador nos mostrará un mensaje de error. 
De estas dos secciones la única que obligatoriamente debe existir es la sección .text (o sección de código).  Un módulo fuente, como el del ejemplo, está formado por instrucciones, datos, símbolos y directivas. Las instrucciones son representaciones nemotécnicas del juego de instrucciones del procesador. Un dato es una entidad que aporta un valor numérico, que puede expresarse en distintas bases o incluso a través de una cadena. 

Los símbolos son representaciones abstractas que el ensamblador maneja en tiempo de ensamblado pero que en el código binario resultante tendrá un valor numérico concreto. Hay tres tipos de símbolos: las etiquetas, las macros y las constantes simbólicas. Por último tenemos las directivas, que sirven para indicarle ciertas cosas al ensamblador, como delimitar secciones, insertar datos, crear macros, constantes simbólicas, etc... Las instrucciones se aplican en tiempo de ejecución mientras que las directivas se aplican en tiempo de ensamblado. 

**Datos 

Los datos se pueden representar de distintas maneras. Para representar números tenemos 4 bases. La más habitual es en su forma decimal, la cual no lleva ningún delimitador especial. Luego tenemos otra muy útil que es la representación hexadecimal, que indicaremos con el prefijo 0x. Otra interesante es la binaria, que emplea el prefijo 0b antes del número en binario. La cuarta y última base es la octal, que usaremos en raras ocasiones y se especifica con el prefijo 0. Sí, un cero a la izquierda de cualquier valor convierte en octal dicho número. Por ejemplo 015 equivale a 13 en decimal. Todas estas bases pueden ir con un signo menos delante, codificando el valor negativo en complemento a dos. Para representar carácteres y cadenas emplearemos las comillas simples y las comillas dobles respectivamente.

**Símbolos.

Como las etiquetas se pueden ubicar tanto en la sección de datos como en la de código, la versatilidad que nos dan las mismas es enorme. En la zona de datos, las etiquetas pueden representar variables, constantes y cadenas. En la zona de código podemos usar etiquetas de salto, funciones y punteros a zona de datos. 
Las macros y las constantes simbólicas son símbolos cuyo ámbito pertenece al preprocesador, a diferencia de las etiquetas que pertenecen al del ensamblador. Se especifican con las directivas .macro y .equ respectivamente y permiten que el código sea más legible y menos repetitivo.

**2.1. Lectura previa 


**2.1.1. Modos de direccionamiento del ARM 

En la arquitectura ARM los accesos a memoria se hacen mediante instrucciones específicas ldr y str (luego veremos las variantes ldm, stm y las preprocesadas push y pop). El resto de instrucciones toman operandos desde registros o valores inmediatos, sin excepciones. En este caso la arquitectura nos fuerza a que trabajemos de un modo determinado: primero cargamos los registros desde memoria, luego procesamos el valor de estos registros con el amplio abanico de instrucciones del ARM, para finalmente volcar los resultados desde registros a memoria. 

Existen otras arquitecturas como la Intel x86, donde las instrucciones de procesado nos permiten leer o escribir directamente de memoria. Ningún método es mejor que otro, todo es cuestión de diseño. Normalmente se opta por direccionamiento a memoria en instrucciones de procesado en arquitecturas con un número reducido de registros, donde se emplea la memoria como almacén temporal. En nuestro caso disponemos de suficientes registros, por lo que podemos hacer el procesamiento sin necesidad de interactuar con la memoria, lo que por otro lado también es más rápido.

*Direccionamiento inmediato. El operando fuente es una constante, formando parte de la instrucción.

*Direccionamiento inmediato con desplazamiento o rotación. Es una variante del anterior en la cual se permiten operaciones intermedias sobre los registros. Mo

*Direccionamiento a memoria, sin actualizar registro puntero. Es la forma más sencilla y admite 4 variantes. Después del acceso a memoria ningún registro implicado en el cálculo de la dirección se modifica

*Direccionamiento a memoria, actualizando registro puntero. En este modo de direccionamiento, el registro que genera la dirección se actualiza con la propia dirección. De esta forma podemos recorrer un array con un sólo registro sin necesidad de hacer el incremento del puntero en una instrucción aparte. Hay dos métodos de actualizar dicho registro, antes de ejecutar la instrucción (preindexado) o después de la misma (postindexado). 

**2.1.2. Tipos de datos 

Tipos de datos básicos. 

En la siguiente tabla se recogen los diferentes tipos de datos básicos que podrán aparecer en los ejemplos, así como su tamaño y rango de representación

| ARM           | Tipo C                                                        | BITS        |
|---------------|---------------------------------------------------------------|-------------|
| .byte         | unsigned char (signed) char                                   | 8 8         |
| .hword .short | unsigned short int (signed) short int                         | 16 16       |
| .word .int    | unsigned int (signed) int unsigned long int (signed) long int | 32 32 32 32 |
| .quad         | unsigned long long (signed) long long                         | 64 64       |

Punteros. Un puntero siempre ocupa 32 bits y contiene una dirección de memoria. En ensamblador no tienen tanta utilidad como en C, ya que disponemos de registros de sobra y es más costoso acceder a las variables a través de los punteros que directamente.

Vectores. Todos los elementos de un vector se almacenan en un único bloque de memoria a partir de una dirección determinada. Los diferentes elementos se almacenan en posiciones consecutivas, de manera que el elemento i está entre los i-1 e i+1.

Matrices bidimensionales. Una matriz bidimensional de N×M elementos se almacena en un único bloque de memoria. Interpretaremos una matriz de N×M como una matriz con N filas de M elementos cada una.
