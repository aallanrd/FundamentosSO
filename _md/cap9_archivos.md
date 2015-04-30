# Sistemas de archivos

*Recopilado por:*  Juan Guevara Jiménez y Marco Navarro Navarro.

El objetivo principal de una computadora es crear, manipular, almacenar y retornar datos. El sistema de archivos es uno de los actores que provee la funcionalidad necesaria para dar soporte a esas tareas. Desde un punto de vista general un sistema de archivos se encarga de organizar, almacenar, retornar y manejar información en un dispositivo de almacenamiento permanente como un disco. Los sistemas de archivos forman parte integral de cualquier sistema operativo.

## Diseño de sistemas de almacenamiento

Existen varias estrategias para realizar la gestión de almacenamiento permanente. Por un lado están los sistemas de archivos que imponen restricciones suficientes para incomodar a los usuarios, provocando que su uso sea difícil. En el otro extremo están los sistemas de almacenamiento de objetos persistentes y bases de datos orientadas a objetos, que abstraen la noción completa de almacenamiento permanente de manera que ni el usuario ni el programador necesiten percatarse de ello.

El problema de almacenar, retornar y manipular información en una computadora es de una naturaleza tan general que hay muchas maneras de resolverlo. No hay una forma "correcta" de escribir un sistema de archivos. Al decidir qué tipo de sistema de archivo es apropiado para un sistema operativo en particular, debemos sopesar las necesidades del problema con las otras restricciones del proyecto. Por ejemplo, una tarjeta flash-ROM tal como se usa en algunas consolas de videojuegos tiene poca necesidad de una interfaz avanzada de búsqueda o soporte para atributos. La confiabilidad de que los datos sean escritos al dispositivo, no obstante, es crítica, entonces un sistema de archivos que soporte "journaling" (registro por diario) puede ser un requisito. De igual forma, un sistema de archivos para una computadora de alta gama (mainframe) requiere de un rendimiento extremadamente rápido en muchas áreas, pero poco en características amigables con el usuario, entonces técnicas que habilitan mas transacciones por segundo ganarían peso sobre aquella que hacen mas fácil para el usuario localizar archivos obscuros.

Es importante tener en mente la meta abstracta de lo que un sistema de archivos debe lograr: _almacenar, retornar, localizar y manipular_ información. Mantener la meta declarada en términos generales nos libera para pensar en implementaciones alternativas y posibilidades que de otro modo, no surgirían si tuviéramos que pensar en un sistema de archivos como una estructura típica, estrictamente jerárquica, basada en disco.

## Tipos de sistemas de archivo

En la actualidad podemos encontrar muchos ejemplos de sistemas de archivos. Sin embargo estos sistemas podrían potencialmente ubicarse en alguno de los siguientes tipos, dependiendo de la estrategia de implementación de los desarrolladores.

### Sistemas de archivos estructurados por registro

Los avances en el aspecto tecnológico están ejerciendo presión sobre los sistemas de archivos actuales. La aparición de procesadores más veloces, discos más grandes y de menor costo (pero no mucho mas rápidos) y las memorias aumentando su tamaño de manera exponencial; con estos factores en juego, el único parámetro que no está teniendo grandes avances es el tiempo de búsqueda en disco. Un cuello de botella de funcionamiento está creciendo en muchos sistemas de archivos. Los investigadores en Berkeley trataron de aliviar este problema al diseñar un tipo de sistema de archivos completamente nuevo llamado LFS (Log-structured File System, sistema de archivos estructurado por registro). Dicho sistema es una re-implementación de UNIX.

La idea que dio lugar al diseño de este sistema de archivo, es que a medida que los procesadores y las memorias RAM aumentan su capacidad, las cachés de disco crecen también. Producto de ello, ahora es posible satisfacer una fracción considerable de todas las peticiones de lectura desde la caché, sin requerir acceso directo al disco. Se toma el disco como una unidad completa, estructurándole como un registro.

Este sistema coloca todas las escrituras en un búffer en memoria y periódicamente se escriben en el disco en un solo segmento al final del registro. Dado que un disco es de tamaño limitado puede darse el problema de que este se llene, para lidiar con ello, este tipo de sistema de archivos tiene implementado un hilo limpiador que explora el registro para compactarlo. Su función es buscar segmentos en el registro de archivos que hayan sido borrados o sobre-escritos de manera que no estén siendo apuntados por nodos-i (nodos índice).

### Sistemas de archivos por bitácora

La idea básica de tener una bitácora es llevar un registro de lo que va a realizar el sistema de archivos antes de hacerlo, de manera que si ocurre una falla antes de que pueda realizar las tareas planeadas, posteriormente al momento de reiniciar el sistema pueda buscar en el registro para ver lo que estaba ocurriendo al momento de la falla y proceder a ejecutar las tareas pendientes. Ejemplos típicos son el sistema de archivos NTFS de Microsoft, así como los sistemas ext3 y ReiserFs de Linux.

Al utilizar una bitácora, la recuperación de errores puede ser rápida y segura. Este sistema está fundamentado en el concepto de bases de datos conocido como transacción atómica. Así, se pueden agrupar varias operaciones entre un principio y fin de transacción. De esta forma el sistema de archivos sabe que debe completar todas las operaciones agrupadas o ninguna de ellas.

### Sistemas de archivos virtuales

El fundamento básico es abstraer la parte del sistema de archivos que es común para todos los sistemas de archivos y poner ese código en una capa separada que sirva de interfaz entre el sistema operativo y el sistema de archivos para administrar los datos. Todas las llamadas al sistema operativo relacionadas con archivos pasan primero por el sistema de archivos virtual para ser procesados. Estas llamadas, son las llamadas de POSIX estándar, tales como open, read, write, lseek, etc. Posteriormente una vez procesadas, se llama a la rutina correspondiente en el sistema operativo.

La implementación de este tipo de sistema de archivo, permite que existan varios sistemas de archivos integrados como una sola estructura, es decir, que desde la perspectiva del usuario solo existe una jerarquía de sistemas de archivos.

## Localización de bloques

Un sistema de archivos debe dar seguimiento de cuáles bloques pertenecen a cada archivo; deben además dar seguimiento también a los bloques que están disponibles. Cuando se crea un nuevo archivo, el sistema de archivos localiza un bloque disponible y lo asigna. Cuando un archivo es borrado el sistema de archivos pone los bloques como disponibles para asignarlos posteriormente.

Las metas para lograr un buen sistema de asignación son:

* **Velocidad:** Asignar y liberar bloques debería ser una tarea rápida.
* **Uso mínimo del espacio:** Las estructuras de datos usadas por el asignador deberían ser pequeñas, dejando tanto espacio como sea posible para guardar datos.
* **Fragmentación mínima:** Si algunos bloques son dejados sin usar, o algunos son solamente utilizados de manera parcial, al espacio que no se utiliza se le llama *fragmentación*. Si se diera el caso el sistema asignador debiera ser capaz de usar dichos espacios o reducirlos al mínimo posible.
* **Contiguidad Máxima:** Aquellos datos que son utilizados al mismo tiempo, deberían estar ubicados físicamente contiguos en disco de ser posible para mejorar el rendimiento.

Diseñar un sistema de archivos que cumpla estos objetivos es una tarea difícil, pues el rendimiento del mismo está ligado a las características de carga de trabajo que vaya a tener, por ejemplo el tamaño de archivo, patrones de acceso, etc. Un sistema que ha sido diseñado para cumplir con los requerimientos de una carga de trabajo podría no desempeñarse bien para un carga diferente.

## Métodos de localización

La naturaleza del acceso directo a los discos nos da flexibilidad en la implementación de archivos. En la mayoría de los casos los archivos son almacenados en el mismo disco. El problema que queda por solventar es como asignar el espacio a estos archivos de manera que el espacio del disco sea aprovechado eficientemente y los archivos puedan ser accesados rápidamente. A continuación se enumeran las tres estrategias para localización de bloques.

### Asignación Contigua

Este método consiste en asignarle a un archivo un conjunto de bloques contiguos en el disco. Las direcciones (identificador inequívoco de cada bloque en disco) definen un orden lineal en el disco, es decir las direcciones están en orden ascendente. De esta forma, asumiendo que solamente una tarea está utilizando el disco, accesar  el bloque __b + 1__ después de un bloque __b__ no requiere movimiento de la cabeza lectora. Cuando se requiere movimiento de la cabeza (esto es pasar del último sector de un cilindro al primer sector del siguiente cilindro), la cabeza necesita solamente moverse una posición al siguiente cilindro. Así, el número de búsquedas en el disco para acceder a los archivos asignados contiguamente es mínimo, así como el tiempo para su lectura.

El problema inherente a la asignación contigua es la dificultad para encontrar espacio para ubicar un nuevo archivo. Si bien no ocurre cuando el disco está vacío, cuando ya hay datos suficientes será difícil encontrar un espacio contiguo completo para ubicar archivos de gran tamaño. Como respuesta a este problema, se han utilizado las técnicas _best fit_ y _first fit_ para asignar el espacio. Otros sistemas utilizan un sistema de asignación contigua modificado. Inicialmente se localiza y asigna un trozo de espacio contiguo, luego, si el espacio no es lo suficientemente grande, otro trozo de espacio contiguo, conocido como extensión es añadido.

![](_figures/contigua.png)

### Asignación Enlazada

La asignación enlazada vino a solventar los problemas de la asignación contigua. Con este método el archivo se visualiza como una lista enlazada de bloques en el disco duro sin importar si estos son contiguos o no. El directorio es quién contiene el puntero al primer y último bloque del archivo. Para crear un nuevo archivo, simplemente se crea una nueva entrada en el directorio y se actualiza el puntero final si se requiere.

Al representar los archivos como bloques dispersos, este método es únicamente eficiente para archivos de acceso secuencial, es decir archivos que requieran ser accesados completos de inicio a fin. Por ejemplo, si requerimos accesar un bloque específico del archivo, tendremos que recorrer la lista enlazada para localizar este bloque, lo cual es lento pues en cada accceso a un bloque se requiere una lectura de disco y en ocasiones una búsqueda.

![](_figures/enlazada.png)

Otro inconveniente es el espacio que se requiere para almacenar los punteros. Si un puntero necesita 4 bytes de un bloque de tamaño de 512 bytes, entonces el 0.78% del disco será utilizado por punteros de bloques de archivo, en lugar de información. Para resolver este inconveniente, se juntan los bloques en múltiplos, llamados clúster y en lugar de asignarle un bloque a un archivo se le asignan grupos de cluster. La definición del tamaño de un cluster queda a decisión del desarrollador y de la capacidad del sistema operativo para manejarlo. De esta forma, los punteros usan un porcentaje mas pequeño del espacio del disco.

Otra estrategia utilizada ha sido la creación de una tabla de localización de archivos
o FAT, que es un conjunto específico de bloques en disco, utilizado para almacenar todos los índices. Esta estrategia permite facilitar el acceso directo a los archivos pues las direcciones se buscan en la FAT que por lo general estará cargada en memoria, de esta forma no se requiere acceso a disco para la búsqueda del bloques de archivo. Sin embargo, un problema que esto conlleva es que a mayor tamaño de disco, mayor tamaño requerirá la FAT lo que implica que la carga en memoria será mas grande también.

### Asignación Indexada

Dados los problemas con el método de asignación enlazada, la solución fue mantener la lista de bloques para cada archivo de manera separada. Así, cada archivo tiene sus *bloques índice* que incluyen apuntadores a los bloques de archivos en disco.

![](_figures/indexada.png)

La ventaja que genera manejar la asignación de esta forma, es que basta con traer el bloque de índices del archivo a memoria. Así podemos encontrar rápidamente el puntero al bloque _b_ de un archivo y accederlo sin necesidad de ir bloque por bloque como en la asignación enlazada.

Aún con las ventajas descritas el costo a pagar por ello es el espacio extra requerido para los bloques de índices. Este problema fue resuelto posteriormente en UNIX BSD con un sistema de índices multinivel, el cuál es el utilizado actualmente en las distribuciones de Unix y Linux. Esta idea permitió almacenar archivos pequeños sin necesidad de crear un bloque de índices.

![](_figures/multinivel.png)

Otras alternativas desarolladas han sido la **asignación indexada enlazada** y la **asignación indexada conbinada**. En la primera, el nodo índice no solamente puede referenciar a bloques sino también a datos.

![](_figures/ienlazada.png)

La asignación combinada es la combinación de la multinivel y la enlazada.

![](_figures/icombinada.png)

## Directorios

Un archivo no es más que un conjunto de bytes relacionados que están en disco u otro medio, a los que se les asigna un nombre que se utilizara para referirse a este archivo. Un directorio no es más que un archivo común a los que se les ha impuesto una estructura particular.

Los directorios tienen información que apunta hacia la ubicación de los archivos reales. Esta información (tanto en los archivos y directorios) junto con el nombre del creador, tamaño, permisos, etc, es guardada en lo que se denomina TABLA DE INODOS en ciertos tipos de sistemas operativos. El sistema de archivos crea esta tabla que contendrá la mayoría de la información de los archivos.

Técnicamente el directorio almacena información acerca de los archivos que contiene: como los atributos de los archivos o dónde se encuentran físicamente en el dispositivo de almacenamiento.

En el entorno gráfico de los sistemas operativos modernos, el directorio se denomina metafóricamente carpeta y de hecho se representa con un icono con esta figura. Esta imagen se asocia con el ambiente administrativo de cualquier oficina, donde la carpeta de cartón encierra las hojas de papel de un expediente.

![](_figures/carpetas_de_windows.png)

Archivos y directorios no pueden ser diferenciados a través del nombre, sino solo a través de las herramientas del sistema operativo, las que además muestran otras propiedades de archivos y directorios, como fecha de creación, fecha de modificación, usuarios y grupos de usuarios que tienen acceso o derechos al archivo o directorio.

Se le llama directorio-padre al directorio que contiene dentro de si otros directorios para formar una jerarquía de directorios que mantenien estructurados todos los archivos propios de un programa o destinados a un propósito específico.

## Concepto de directorio

Un directorio es un objeto que relaciona de forma unívoca un nombre de archivo (dado por el usuario) con su descriptor interno, también organiza y proporciona información sobre la estructuración del sistema de archivos. Un directorio puede verse como una colección de listados que contienen información acerca de los archivos. Esta es una unidad de organización interna del sistema operativo que se utiliza para localizar archivos.

## Visión lógica de los directorios

Los directorios se caracterizan por estar organizados en un esquema jerárquico. Las acciones principales que se pueden realizar sobre un directorio son las siguientes:

* Crear (insertar) y borrar (eliminar) directorios.
* Abrir y cerrar directorios.
* Renombrar directorios.
* Combinar dos directorios distintos.

Cuando se pide abrir un archivo el Sistema Operativo busca el nombre en la estructura de dicho directorio.

La organización jerárquica de un directorio:

* Simplifica el nombrado de archivos ya que se le asignan nombre únicos que el usuario proporciona a su gusto.
* Proporciona una gestión de la distribución ya que agrupa archivos de forma lógica y a gusto del usuario.

## Estructura de los directorios

Tanto la estructura del directorio como los archivos residen en disco. Por tanto, los directorios se suelen implementar como archivos:

* Información en un directorio: nombre, tipo, dirección, longitud máxima y actual, tiempos de acceso y modificación, dueño, etc.
* Hay estructuras de directorio muy distintas. La información depende de esa estructura.

Dos alternativas principales:

* Almacenar atributos de archivo en entrada directorio.
* Almacenar <nombre, identificador>, con datos archivo en una estructura distinta. Ésta es mejor opción.

A los usuarios les interesa la forma de nombrar sus archivos, las operaciones que pueden efectuarse en ellos, el aspecto que tiene el árbol de directorios y cuestiones de interfaz por el estilo. A los implementadores les interesa como están almacenados los archivos y directorios, como se administra el espacio en disco y como puede hacerse para que todo funcione de forma eficiente y confiable.

## Implementación de Directorios

Cuando se abre un archivo, el sistema operativo usa el nombre de la ruta proporcionado por el usuario para localizar la entrada del directorio.

### Directorios en MS-DOS

Los directorios pueden tener otros directorios, dando lugar a un sistema de archivos jerárquicos. En este sistema operativo es común que los diferentes programas de aplicación comiencen por crear un directorio en el directorio raíz pongan ahí todos sus archivos, con objeto que no halla conflictos entre las aplicaciones.

### Directorios en UNIX

La estructura de directorios es extremadamente sencilla. Cuando se abre un archivo, el sistema de archivos debe tomar el nombre que se le proporciona y localizar sus bloques de disco.

## Administración del Espacio en Disco

Es de interés primordial para los diseñadores de sistemas de archivos. Hay dos posibles estrategias para almacenar un archivo  de n bytes: asignar n bytes consecutivos de espacio en disco, o dividir el archivo en varios bloques (no necesariamente) contiguos.

Tamaño de bloque  Una vez que se ha decidido almacenar archivos en bloques de tamaño fijo, surge la pregunta de qué tamaño deben tener los bloques. Dada la forma como están organizados los discos, el sector, la pista y el cilindro son candidatos obvios para utilizarse como unidad de asignación. En un sistema con paginación, el tamaño de página también es un contendiente importante.

Administración de bloques libres  Una vez que se ha escogido el tamaño de bloque, el siguiente problema es cómo seguir la pista a los bloques libres. Se utilizan ampliamente dos métodos.

El primero consiste en usar una lista enlazada de bloques de disco, en la que cada bloque contiene tantos números de bloques de disco libres como quepan en él.
El mapa de bits. Un disco con n bloques requiere un mapa de bits con n bits. Los bloques libres se representan con unos en el mapa, y los bloques asignados con ceros (o viceversa).

## Rendimiento del Sistema de Archivos

El acceso a un disco es mucho más lento que el acceso a la memoria. La lectura de una palabra de memoria por lo regular toma decenas de nanosegundos. La lectura de un bloque de un disco duro puede tardar 50 microsegundos. La técnica más común empleada para reducir los accesos a disco es el caché de bloques o el caché de buffer.

## Organización del directorio

Se debe tener en cuenta la eficiencia, es decir,  localizar un archivo rápidamente. El nombrado de los directorios debe ser conveniente y sencillo para los usuarios:

* Dos usuarios pueden tener el mismo nombre para archivos distintos.
* Los mismos archivos pueden tener nombres distintos.
* Nombres de longitud variable.
* Agrupación: agrupación lógica de los archivos según sus propiedades (por ejemplo: archivos del trabajo o universidad, juegos, etc.).
* Sencillez: la entrada de directorio debe ser lo más sencilla posible.

## Estructura física del directorio

Los directorios son una tabla contigua con entradas de tamaño fijo. Los directorios son poco flexibles. En grandes directorios la búsqueda es lenta.

## Funcionamiento del directorio

### Compartir archivos

En los sistemas operativos multiusuario, se puede desarrollar este tipo de actividad que he permitir a otros usuarios a accesar a los archivos que otro usuario distribuye. Siempre que tengan los derechos de acceso.

### Agrupación de registros

La ejecución de entradas y salidas, los registros se ubican en tres bloques:

### Bloque fijo

Los registros son guardados en un bloque por su longitud fija y por un número entero de registros, puede haber espacios sin utilizar en cada bloque.

### Bloque de longitud variable por tramos

Los registros son variables por su longitud y se agrupan en bloques no se dejan espacios.

### Bloque de longitud variable sin tramos

se usan registros de longitud variable pero no se dividen en tramos. Casi todos los bloques hay un espacio desperdiciado ya que no se aprovechan el espacio libre de este.

### Gestión de almacenamiento secundario

Es responsable de asignar los bloques a los archivos, pero esto crea dos problemas, uno es que el espacio del almacenamiento secundario se le asigna a los archivos, segundo, es la necesidad de dejar espacios libres para asignar de modo que estas dos tareas se relacionan entre sí, ya que esto influye en el método de gestión del espacio libre.

## Permisos de archivos y directorios

En cualquier sistema multiusuario, es preciso que existan métodos que impidan a un usuario no autorizado copiar, borrar, modificar algún archivo sobre el cual no tiene permiso.

En Linux las medidas de protección se basan en que cada archivo tiene un propietario (usualmente, el que creó el archivo). Además, los usuarios pertenecen a uno o más grupos, los cuales son asignados por el Administrador dependiendo de la tarea que realiza cada usuario; cuando un usuario crea un archivo, el mismo le pertenece también a alguno de los grupos del usuario que lo creó.

Así, un archivo en Linux le pertenece a un usuario y a un grupo, cada uno de los cuales tendrá ciertos privilegios de acceso al archivo. Adicionalmente, es posible especificar qué derechos tendrán los otros usuarios, es decir, aquellos que no son el propietario del archivo ni pertenecen al grupo dueño del archivo.

En cada categoría de permisos (usuario, grupo y otros) se distinguen tres tipos de accesos: lectura (Read), escritura (Write) y ejecución (eXecute), cuyos significados varían según se apliquen a un archivo o a un directorio.

En el caso de los archivos, el permiso R (lectura) habilita a quién lo posea a ver el contenido del archivo, mientras que el permiso W (escritura) le permite cambiar su contenido. El permiso X (ejecución) se aplica a los programas y habilita su ejecución.

Para los directorios, el permiso R permite listar el contenido del mismo (es decir, “leer” el directorio, mientras que el W permite borrar o crear nuevos archivos en su interior (es decir, modificar o “escribir” el directorio). El permiso X da permiso de paso, es decir, la posibilidad de transformar el directorio en cuestión en el directorio actual (ver comando cd).

En los listados de directorio, los permisos se muestran como una cadena de 9 caracteres, en donde los primeros tres corresponden a los permisos del usuario, los siguientes tres a los del grupo y los últimos, a los de los demás usuarios. La presencia de una letra (r, w o x) indica que el permiso está concedido, mientras que un guión (-) indica que ese permiso está denegado.

Los permisos de un archivo o directorio pueden cambiarse desde el administrador de archivos KFM utilizando la ventana de propiedades o utilizando el comando chmod

## Rutas de Directorios

### Nombres de ruta

Un nombre de ruta define de manera única a un archivo o directorio en particular especificando su ubicación. Los nombres de ruta son similares a un mapa de caminos o a un conjunto de instrucciones que le indican al usuario cómo ir de un lugar en la jerarquía de directorios a otro. Por ejemplo, si el alumno le estuviera dando a alguien de otro país instrucciones acerca de cómo llegar a él, tendría que especificar dónde vive. Los directorios de un sistema de archivos pueden compararse a un país, estado, ciudad, etcétera. Si la Tierra fuera un disco duro con un sistema de archivos, la consideraríamos la raíz del mismo. Si quisiéramos identificar la ubicación de una persona para decirle a alguien dónde vive, especificaríamos el nombre de ruta hasta llegar a dicha persona. Utilizaríamos un nombre de ruta totalmente calificado para que no existiera ninguna duda de que estamos hablando de esa persona del planeta Tierra, y no de cualquier otro planeta.

### Componentes de la ruta

Las barras dentro del nombre de ruta son delimitadoras entre nombres de objetos. Las barras actúan como separadores. Los nombres de objetos pueden ser directorios, subdirectorios o archivos. DOS y Windows indican los directorios utilizando una barra invertida (\). Todos los sistemas de archivos UNIX utilizan una barra (/) en los nombres de ruta. La barra se encuentra por lo general cerca de la tecla Shift derecha en la mayoría de los teclados. Una barra (/) en la primera posición de cualquier nombre de ruta representa al directorio raíz.

###Directorio de Árbol###
![](_figures/DirectorioArbol.png)

### Directorio de un solo Nivel ###
![](_figures/DirectorioUnNivel.png)

###Directorio de Dos Niveles###
![](_figures/DirectorioDosNiveles.png)

### Directorio de Grafo Aciclico ###
![](_figures/DirectorioGrafoAciclico.png)

###Directorio de Grafo General ###
![](_figures/DirectorioGrafoGeneral.png)

## Glosario

* **Bloque:** Unidad lógica de disco duro.

* **Caché:** Memoria pequeña de acceso rápido que almacena los datos usados por un periférico recientemente.

* **Cluster:** Conjunto de bloques.

* **FAT:** File Allocation Table o tabla de asignación de archivos.

* **Nodo Índice:** Bloque especial que almacena atributos de archivo y punteros hacia datos o hacia otros nodos índice.

##Referencias

* Giampaolo Dominic. Practical File System Design with the Be File System. Morgan Kaufmann Publishers INC, San Francisco California. 1999.

* Tanenbaum Andrew. Sistemas Operativos Modernos. Tercera Edición. PEARSON EDUCACIÓN, México, 2009.

* Downey, Allen. Think OS A Brief Introduction to Operating Systems. Green Tea Press. Needham. Massachusetts. 2014.

* Silberschatz, Abraham. Operating System Conceps. Wiley. Ninth Edition. United States. 2013.

* Overview of FAT, HPFS, and NTFS File Systems. (n.d.). Retrieved 24 April 2015, from https://support.microsoft.com/en-us/kb/100108

* UNIX File System. (n.d.). Retrieved 23 April 2015, from http://www.cis.rit.edu/class/simg211/unixintro/Filesystem.html

* Rutas de Directorios (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <https://sites.google.com/a/ingenieria.lm.uasnet.mx/so/t3#section-Archivos-RutasDeDirectorios>

* Directorios (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <http://es.wikipedia.org/wiki/Directorio>

* archivos Linux (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <http://www.investigacion.frc.utn.edu.ar/labsis/Publicaciones/apunte_linux/ma.html>

* Filesystem (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <http://www.ant.org.ar/cursos/curso_intro/filesystem.html>

* Sistema de archivos (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <http://laurel.datsi.fi.upm.es/_media/docencia/asignaturas/dso/sistemaarchivosdso_2011.pdf>

* Sistemas Operativos.: Sistemas de gestion de archivos. (2015, 12 de Marzo). Recuperado el 12 de Marzo del 2015, de <http://sistemasoperativos03-unefa.blogspot.com/2011/12/normal-0-21-false-false-false-es-ve-x.html>
