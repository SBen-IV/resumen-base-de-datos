# Resumen

## Concurrencia

La concurrencia es la posibilidad de ejecutar múltiples
transacciones (tareas, en la jerga de los sistemas operativos) en
forma simultánea.

- Consideraremos que nuestra base de datos está formada por
ítems.
- Un ítem puede representar:
  - El valor de un atributo en una fila determinada de una tabla.
  - Una fila de una tabla.
  - Un bloque del disco.
  - Una tabla.
- Las instrucciones atómicas básicas de una transacción sobre la
base de datos serán:
  - **leer_item(X)**: Lee el valor del ítem X , cargándolo en una variable
en memoria
  - **escribir_item(X)**: Ordena escribir el valor que está en memoria del
ítem X en la base de datos

## Transacción

Es una unidad lógica de trabajo en los SGBD.
Es una secuencia ordenada de instrucciones que deben ser
ejecutadas en su totalidad o bien no ser ejecutadas, al margen de
la interferencia con otras transacciones simultáneas.

## Propiedades ACID

- **Atomicidad**: Desde el punto de vista del usuario, las transacciones
deben ejecutarse de manera atómica. Esto quiere decir que, o bien
la transacción se realiza por completo, o bien no se realiza.
- **Consistencia**: Cada ejecución, por sí misma, debe preservar la
consistencia de los datos. La consistencia se define a través de
reglas de integridad: condiciones que deben verificarse sobre los
datos en todo momento.
- **aIslamiento**: El resultado de la ejecución concurrente de las
transacciones debe ser el mismo que si las transacciones se
ejecutaran en forma aislada una tras otra, es decir en forma serial.
La ejecución concurrente debe entonces ser equivalente a alguna
ejecución serial.
- **Durabilidad**: Una vez que el SGBD informa que la transacción se
ha completado, debe garantizarse la persistencia de la misma,
independientemente de toda falla que pueda ocurrir.

Para garantizar las propiedades ACID se agregan a la secuencia de instrucciones de cada transacción algunas instrucciones especiales:

- **begin**: Indica el comienzo de la transacción.
- **commit**: Indica que la transacción ha terminado exitosamente, y se
espera que su resultado haya sido efectivamente almacenado en
forma persistente.
- **abort**: Indica que se produjo algún error o falla, y que por lo tanto
todos los efectos de la transacción deben ser deshechos (rolled
back).


## Anomalías

- **Lectura sucia (dirty read)**: se presenta cuando una transacción $T_2$ lee un ítem que ha sido modificado por otra transacción $T_1$. Si luego $T_1$ se deshace, la lectura que hizo $T_2$ **no es válida** en el sentido de que la ejecución resultante puede no ser equivalente a una ejecución serial de las transacciones. Es un conflicto de tipo WR: $W_{T_1}(X)...R_{T_2}(X)...(a_{T_1} ó c_{T_1}).$

Ejemplo:

![Dirty Write](images/dirty_write.png)

- **Actualización perdida (lost update)**: ocurre cuando una transacci'on modifica un ítem que fue leído anteriormente por una primera transacción que aún no terminó. Si la primera transacción luego modifica y escribe el ítem que leyó, el valor escrito por la segunda se perderá.
- **Lectura no repetible (unrepeatable read)**:  Si en cambio la primera transacción volviera a leer el ítem luego de que la segunda lo escribiera, se encontraría con un valor distinto.

Ambas situaciones presentan un conflicto de tipo RW seguido por otro de tipo WW ó WR, respectivamente $R_{T_1}(X)...W_{T_2}(X)...(a_{T_1} ó c_{T_1})$

Ejemplo:

![Lost update](images/lost_update.png)

- **Escritura sucia (dirty write)**: ocurre cuando una transacción $T_2$ escribe un ítem que ya había sido escrito por otra transacción $T_1$ que luego se deshace. El problema se datá si los mecanismos de recuperación vuelven al ítem a su valor inicial, deshaciendo la modificación realizada por $T_2$. Es un conflicto WW: $W_{T_1}(X)...W_{T_2}(X)...(a_{T_1} ó c_{T_1})$

- **Fantasma (phantom)**: se produce cuando una transacción $T_1$ observa un conjunto de ítems que cumplen determinada condición, y luego dicho conjunto cambia porque algunos de sus ítems son modificados/creados/eliminados por otra transacción $T_2$. Si esta modificación se hace mientras $T_1$ aún se está ejecutando, $T_1$ podría encontrarse con que el conjunto de ítems que cumplen la condición cambió. Caracterización: $R_{T_1}(\{X|cond\})...W_{T_2}(X_{cond})...(a_{T_1} ó c_{T_1})$

## Serializabilidad

Notación:

- $R_T(X)$: La transacción T lee el ítem X.
- $W_T(X)$: La transacción T escribe el ítem X.
- $b_T$: Comienzo de la transacción T.
- $c_T$: La transacción T realiza el *commit*.
- $a_T$: Se aborta la transacción T (abort).

## Solapamiento

- Un solapamiento entre 2 transacciones $T_1$ y $T_2$ es una lista de $m(T_1) + m(T_2)$ instrucciones, en donde cada instrucción de $T_1$ y $T_2$ aparece una única vez, y las instrucciones de cada transacción consrvan el orden entre ellas dentro del solapamiento.
- Existen ${(m(T_1)+m(T_2))! \over m(T_1)!m(T_2)!}$ distintos solapamientos.


## Ejecución serial

- Dado un conjutno de transacciones $T_1, T_2, ..., T_n$ una ejecución serial es aquella en que las transacciones se ejecutan por completo una detrás de otra, en base a algún orden $T_{i_1}, T_{i_2}, ..., T_{i_n}$.
- Existen $n!$ distintas ejecuciones seriales.

## Serializabilidad

Un solapamiento de un conjunto de transacciones $T_1, T_2, ..., T_n$ es **serializable** cuando la ejecución de sus instrucciones en dicho orden deja a la base de datos en un estado *equivalente* a aquel en que la hubiera dejado alguna ejecución serial de $T_1, T_2, ..., T_n$.
La serializabilidad garantiza la propiedad de aislamiento.

## Equivalencia de solapamientos

- **Equivalencia de resultados**: Cuando, dado un estado inicial particular, ambos órdenes de ejcución dejan a la base de datos en el mismo estado.
- **Equivalencia de conflictos**: Cuando ambos órdenes de ejecución
poseen los mismos **conflictos** entre instrucciones. (No depende del estado inicial de la base de datos).
- **Equivalencia de vistas**: Cuando en cada órden de ejecución, cada
lectura $R_{T_i}(X)$ lee el valor escrito por la misma transacción $j$, $W_{T_j}(X)$. Además se pide que en ambos órdenes la última
modificación de cada ítem $X$ haya sido hecha por la misma
transacción.

## Conflictos

Dado un orden de ejecución, un conflicto es un par de instrucciones ($I_1, I_2$) ejecutadas por 2 transacciones distintas $T_i$ y $T_j$, tales que $I_2$ se encuentra más tarde que $I_1$ en el orden, y que respode a alguno de los siguientes esquemas:
  - ($R_{T_i}(X), W_{T_j}(X)$): Una transacción escribe un ítem que otra leyó.
  - ($W_{T_i}(X), R_{T_j}(X)$): Una transacción lee un ítem que otra escribió.
  - ($W_{T_i}(X), W_{T_j}(X)$): Dos transacciones escriben un mismo ítem.

Todo par de instrucciones consecutivas ($I_1, I_2$) de un solapamiento que no constituye un conflicto puede ser invertido en su ejecución obteniendo un solapamiento equivalente por conflictos al inicial.

## Grafo de precedencias

Dado un conjunto de transacciones $T_1, T_2, ..., T_n$ que acceden a determinados ítems $X_1, X_2, ..., X_p$, el grafo de precedencias es un *grafo dirigido simple* que se construye de la siguiente forma:

1. Se crea un nodo por cada transacción $T_1, T_2, ..., T_n$.
2. Se agrega un arco entre los nodos $T_i$ y $T_j$ (con $i \ne j$) si y sólo si existe algún conflicto de la forma $(R_{T_i}(X_k), W_{T_j}(X_k)), (W_{T_i}(X_k), R_{T_j}(X_k)) ó (W_{T_i}(X_k), W_{T_j}(X_k))$.

Cada arco ($T_i, T_j$) en el grafo representa una precedencia entre $T_i$ y $T_j$, e indica que para que el resultado sea equivalente por conflictos a una ejecución serial, entonces en dicha ejecución serial $T_i$ debe ejecutarse antes que $T_j$.

![Grafo precedencia](images/grafo_precedencias.png)

Un orden de ejecución es serializable por conflictos si y sólo si su grafo de precedencias no tiene ciclos.

## Control de concurrencia

- **Lock**

![Lock](images/lock.png)

En general, los SGBD impementan locks de varios tipos. Los dos tipos de locks principales son:
- Locks de escritura o "de acceso exclusivo" ($L_{EX}(X)$)
- Locks de lectura o "de acceso compartido" ($L_{SH}(X)$)

![Compatibilidad locks](images/compatibilidad_locks.png)

- **Protocolo de lock de dos fases (2PL)**: Una transacción no puede adquirir un lock luego de haber liberado un lock que había adquirido. Existen 2 fases:
  - Una de adquisición de locks, en la que la cantidad de locks adquiridos crece.
  - Una de liberación de locks, en que la cantidad de locks adquiridos decrece.

Este protocolo garantiza que cualquier orden de ejecución de un conjunto de transacciones sea serializable.

#### Deadlock

Condición en que un conjunto de transacciones quedan cada una de ellas bloqueada  la espera de recursos que otra de ellas posee.
- Mecanismos de prevención:
  1. Que cada transacción adquiera todo los locks que necesita antes de comenzar su primera instrucción, y en forma simultánea.
  2. Definir un ordenamiento de los recursos.
  3. Métodos basados en timestamps.

La inanición es una condición vinculada con el deadlock, y ocurre cuando una transacción no logra ejecutarse por un período de tiempo indefinido.


## Control de concurrencia basado en timestamps

- Se asigna a cada transacción $T_i$ un timestamp $TS(T_i)$ (p. ej> un número de secuencia o la fecha actual del reloj).
- Los timestamps deben ser únicos.
- Se permite la ocurrencia de conflictos, pero siempre que las transacciones de cada conflicto aparezcan de acuerdo al orden serial equivalente: $(W_{T_i}(X), R_{T_j}(X)) \rightarrow TS(T_i) < TS(T_j)$
- Este método está exento de deadlocks.
- Se debe mantener en todo instante, **para cada ítem X**, la siguiente información:
  - **read_TS(X)**: Es el TS(T) correpondiente a la transacción más joven -de mayor TS(T)- que leyó el ítem X.
  - **write_TS(X)**: Es el TS(T) correpondiente a la transacción más joven -de mayor TS(T)- que escribió el ítem X.
- Lógica:
  1. Cuando una transacción $T_i$ quiere ejecutar un $R(X)$:
     1. Si una transacción posterior $T_j$ modificó el ítem, $T_i$ deberá ser abortada (read too late).
     2. De lo contrario, actualiza $read\_TS(X)$ y lee.
  2. Cuando una transacción $T_i$ quiere ejecutar un $W(X)$:
     1. Si una transacción posterior $T_j$ leyó o escribió el ítem, $T_i$ deberá ser abortada (write too late).
     2. De lo contrario, actualiza $write\_TS(X)$ y escribe.

## Snapshot Isolation

Cuando 2 transacciones intentan modificar un mismo ítem de datos, generalmente gana aquella que hace primero su commit, mientras que la otra deberá ser abortada (first-commiter-wins).

## Solución a la anomalía del fantasma

1. Bajo control de concurrencia basado en locks:
   1. Locks de tablas: cuando se lee un X con una dada condición, bloquear el acceso a toda la tabla a la que X pertenece.
   2. Locks de predicados: bloquear todas aquellas tuplas que podrían cumplir la condición.
2. Bajo Snapshot Isolation
   1. Locks de predicados
3. Bajo control de concurrencia basado en timestamps:
   1. Utilizar índices de tipo árbol, y mantener registros $read\_TS(I)$ y $write\_TS(I)$ también para los nodos del árbol.

## Recuperabilidad

Un solapamiento es recuperable si y sólo si ninguna transacción T realiza el commit hasta tanto todas las transacciones que escribieron datos antes de que T los leyera hayan commiteado.

![Recuperabilidad](images/recuperabilidad_ejemplos.png)

El log almacena generalmente los siguientes registros:
- $(BEGIN, T_{id})$
- $(WRITE, T_{id}, X, X_{old}, X_{new})$
- $(READ, T_{id}, X)$
- $(COMMIT, T_{id})$
- $(ABORT, T_{id})$

#### Rollback

- Si las modificaciones hechas por $T_j$ no fueron leídas por nadie, entonces basta con procesar el log de $T_j$ en forma inversa para deshacer sus efectos (rollback).
- Pero si una transacción $T_i$ leyó un dato modificado por $T_j$, entonces será necesario hacer el rollback de $T_i$ para volverla a ejecutar.
- Que un solapamiento sera recuperable, no implica que no sea necesario tener que hacer rollbacks en cascada de transacciones que aún no commitearon.
- Para evitar rollbacks en cascada es necesario que no existan conflictos de la forma $(W_{T_i}(X); R_{T_j}(X))$ sin que en el medio haya un commit $c_{T_i}$.

#### Protocolo de lock de dos fases estricto (S2PL)

Una transacción no puede adquirir un lock luego de haber liberado un
lock que había adquirido, y además los locks de escritura sólo pueden
ser liberados después de haber commiteado la transacción.

#### Protocolo de lock de dos fases riguroso (R2PL)

Los locks sólo pueden ser liberados después del commit.

S2PL y R2PL garantizan que todo solapamiento sea
no sólo serializable, sino también recuperable, y que no se
producirán cascadas de rollbacks al deshacer una transacción.

## Niveles de aislamiento

1. Read Uncommitted: Es la carencia total de aislamiento: No se
emplean locks, y se accede a los ítems sin tomar ninguna
precaución.
2. Read Committed: Evita la anomalía de lectura sucia.
3. Repeatable Read: Evita la lectura no repetible y la lectura sucia.
4. Serializable: Evita todas las anomalías, y asegura que el resultado
de la ejecución de las transacciones es equivalente al de algún
orden serial.

![Niveles aislamiento](images/niveles_aislamiento.png)

## NoSQL

#### Fragmentación

- La fragmentación es la tarea de dividir un conjunto de agregados
entre un conjunto de nodos.
- Se realiza con dos objetivos:
  - Almacenar conjuntos muy grandes de datos que de lo contrario no
podrían caber en un único nodo.
  - Paralelizar el procesamiento, permitiendo que cada nodo ejecute
una parte de las consultas para luego integrar los resultados.
- Según la manera de fragmentar, podemos distinguir entre:
  - Fragmentación horizontal: Los agregados se reparten entre los
nodos, de manera que cada nodo almacena un subconjunto de
agregados. Generalmente se asigna el nodo a partir del valor de
alguno de los atributos del agregado.
  - Fragmentación vertical: Distintos nodos guardan un subconjunto
de atributos de cada agregado. Todos suelen compartir los
atributos que conforman la clave.

#### Replicación

- La replicación es el proceso por el cual se almacenan múltiples
copias de un mismo dato en distintos nodos del sistema.
- Nos brinda varias ventajas:
  - Es un mecanismo de backup: permite recuperar el sistema en caso
de fallas de disco o catastróficas.
  - Permite repartir la carga de procesamiento si permitimos que las
réplicas respondan consultas o actualizaciones.
  - Garantiza cierta disponibilidad del sistema aún si se caen algunos
nodos.
- Cuando las réplicas sólo funcionan como mecanismo de backup
se denominan réplicas secundarias. Cuando también pueden
hacer procesamiento, se las conoce como réplicas primarias.
- La replicación nos genera un nuevo problema a resolver: la
consistencia de los datos.
  - Puede darse la situación de que distintas réplicas almacenen (al
menos, temporalmente) distintos valores para un mismo dato.


## Clave-valor

Las bases de datos clave-valor (key-value stores) almacenan
vectores asociativos ó diccionarios, es decir conjuntos formados
por pares de elementos de forma (clave, valor).

- Este tipo de bases de datos tiene cuatro operaciones
elementales:
  - Insertar un nuevo par (put)
  - Eliminar un par existente (delete)
  - Actualizar el valor de un par (update)
  - Encontrar un par asociado a una clave particular (get)
- Sus ventajas son:
  - Simplicidad
    - No se define un esquema.
    - No hay DDL’s, restricciones de integridad, ni dominios.
    - El agregado es mínimo, y está limitado al par.
    - El objetivo es guardar y consultar grandes cantidades de datos, pero
no de interrelaciones entre los datos.
  - Velocidad
    - Ya que se prioriza la eficiencia de acceso por sobre la integridad de
los datos.
  - Escalabilidad
    - Generalmente proveen replicación (ya sea maestro-esclavo ó
distribuida) y permiten repartir las consultas entre los nodos.


## Hashing consistente

- Disponemos de una función de hash $h()$ que, dada una clave $k$,
devuelve un valor $h(k)$ entre $0$ y $2^M−1$, en donde $M$ representa la
cantidad de bits del resultado.
- El valor de la función de hash para un par dado es lo que
determina en cuál de los $S$ nodos el mismo será almacenado.
Esto es lo que se conoce como una tabla de hash distribuida
(DHT).
- A diferencia de otras DHT’s en que el nodo asignado se
determina como $h(k)$ $mod$ $S$, en Dynamo se utiliza una técnica
ligeramente distinta, conocida como hashing consistente.

Al identificador de cada nodo de procesamiento (generalmente,
su dirección IP) se le aplica la misma función de hash. A partir de
los hashes, los nodos se organizan virtualmente en una
estructura de anillo por hash creciente.

![Hashing consistente](images/hashing_consistente.png)

Regla: Un par $(k, v)$ se replicará en los N servidores siguientes a
$h(k)$, que conformarán el listado de preferencia para esa clave.

## Modelos de consistencia

- **Consistencia secuencial**: Partimos de una serie de procesos que ejecutan instrucciones de lectura, $R_{P_i}(X)$ y de escritura, $W_{P_i}(X)$, sobre una base de datos distribuida.
- Se dice que una base de datos distribuida tiene consistencia secuencial cuando “el resultado de cualquier ejecución concurrente de los procesos es equivalente al de alguna ejecución secuencial en que las instrucciones de los procesos se ejecutan una después de otra”.
- Para indicar el valor leído/escrito utilizaremos esta notación:
  - $R(X)a$ indica que el proceso leyó el valor $a$ del ítem $X$.
  - $W(X)b$ indica que el proceso escribió el valor $b$ en el ítem $X$.

![Consistencia secuencial](images/consistencia_secuencial.png)

- **Consistencia causal**: Si un evento b fue influenciado por un evento a, la causalidad requiere que todos vean al evento a antes que al evento b.

![Consistencia causal](images/consistencia_causal.png)

- **Consistencia eventual**: Decimos que una ejecución tiene consistencia eventual cuando “si en el sistema no se producen modificaciones (escrituras) por un tiempo suficientemente grande, entonces eventualmente todos los procesos verán los mismos valores”.

- Se definen dos parámetros adicionales:
  - W ≤ N: Quorum de escritura
  - R ≤ N: Quorum de lectura
- Quorum de escritura W
  - Un nodo puede devolver un resultado de escritura exitosa luego de
recibir la confirmación de escritura de otros W − 1 nodos del
listado de preferencia.
  - W = 2 ofrece un nivel de replicación mínimo.
- Quorum de lectura R
  - Un nodo puede devolver el valor de una clave leída luego de
disponer de la lectura de R nodos distintos (incluído él mismo).
  - En muchas situaciones R = 1 es ya suficiente.
  - Valores mayores de R brindan tolerancia a fallas como corrupción
de datos ó ataques externos, pero hacen más lenta la lectura.

Algunos valores comunes de R y W son:

|   N   |   R   |   W   |                                                                |
| :---: | :---: | :---: | :------------------------------------------------------------- |
|   3   |   2   |   2   | Buena durabilidad y latencia.                                  |
|   3   |   3   |   1   | Lectura más lenta, pero pobre durabilidad. Escritura rápida.   |
|   3   |   1   |   3   | Escrituras más lentas. Muy buena durabilidad y lectura rápida. |

## Bases de datos orientadas a documentos

- En las bases de datos orientadas a documentos, un documento
es una unidad estructural de información (agregado), que
almacena datos bajo una cierta estructura.
- Generalmente, un documento se define como
un conjunto de pares clave: valor que representan los atributos
del documento y sus valores. Se admiten atributos multivaluados,
y también se admite que el valor de un atributo sea a su vez un
documento.

### MongoDB

- El pipeline de agregación de MongoDB ofrece las siguientes
operaciones, entre otras:
  - match: Filtrado de resultados.
  - group: Agrupamiento de los resultados por uno o más atributos,
aplicando funciones de agregación.
  - sort: Ordenamiento de resultados.
  - limit: Limitado de resultados.
  - sample: Selección aleatoria de resultados.
  - unwind: Deconstrucción de un atributo de tipo vector.

#### Sharding

- MongoDB utiliza un modelo distribuido de procesamiento,
conocido como sharding.
- Se basa en el particionamiento horizontal de las colecciones en
chunks que se distribuyen en nodos denominados shards. Cada
shard contendrá un subconjunto de los documentos de cada
colección.
- Un sharding cluster de MongoDB está formado por distintos tipos
de nodos de ejecución:
  - Los **shards (fragmentos)**: Son los nodos en los que se distribuyen
los chunks de las colecciones. Cada shard corre un proceso
denominado mongod.
  - Los **routers**: Son los nodos servidores que reciben las consultas
desde las aplicaciones clientes, y las resuelven comunicándose
con los shards. Corren un proceso denominado mongos.
  - Los **servidores de configuración**: Son los que almacenan la
configuración de los routers y los shards.
- El particionado de las colecciones se realiza a partir de una shard
key. La shard key es un atributo ó conjunto de atributos de la
colección que se escoge al momento de construir el sharded
cluster.
- La asignación de documentos a shards se hace dividiendo en
rangos los valores de la shard key (range-based sharding), o bien
a partir de una función de hash aplicada sobre su valor (hashed
sharding).
- En un contexto de sharding es posible tener algunas colecciones
sharded (fragmentadas) y otras unsharded (no fragmentadas).
Las colecciones unsharded de una base de datos se
almacenarán en un shard particular del cluster, que será el shard
primario para esa base de datos.
- La realización de un sharding sobre una colección posee las
siguientes restricciones:
  1. Es conveniente que la shard key esté definida en todos los
documentos de la colección.
  1. La colección deberá tener un índice que comience con la shard
key.
  3. Desde MongoDB 5.0, una vez realizado el sharding se puede
cambiar la shard key y desde la versión 4.2 se puede cambiar
(update) su valor.
  4. No es posible defragmentar (unshard) una colección que ya fue
fragmentada (sharded) .
- El sharding permite:
  - Disminuir el tiempo de respuesta en sistemas con alta carga de
consultas, al distribuir el trabajo de procesamiento entre varios
nodos.
  - Ejecutar consultas sobre conjuntos de datos muy grandes que no
podrían caber en un único servidor.

#### Replicación

- El esquema de réplicas es de master-slave with automated
failover (maestro-esclavo con recuperación automática):
  - Cada shard pasa a tener un servidor mongod primario (master ), y
uno o más servidores mongod secundarios (slaves). El conjunto
de réplicas de un shard se denomina replica set.
  - Las réplicas eligen inicialmente un master a través de un algoritmo
distribuido.
  - Cuando el master falla, los slaves eligen entre sí a un nuevo
master.
- También los servidores de configuración se implementan como
replica sets.
- Todas las operaciones de escritura sobre el shard se realizan en
el master. Los slaves sólo sirven de respaldo.

Ejemplos:

```js
db.compras.aggregate([
	{ $match: { "productos.codigo_producto": { $in: [108431] }}},
	{ $unwind: "$productos" },
	{ $group: {
		_id: "$productos.codigo_producto",
		cantidad_tickets_compra: { $sum: 1},
	}},
	{ $match: { "_id": { $ne: 108431 }}},
	{ $sort: { count: -1 }},
	{ $limit: 5 }
])
```

```js
db.graduados.aggregate([
	{ $project: {
		_id: "$padron",
		promedio: { $trunc:
     [{ $divide: 
      [{ $sum: "$notas.nota_final" }, { $size: "$notas" } ]}, 1
     ]}}
	},
	{ $group: {
		_id: "$promedio", cantidad_alumnos: { $sum: 1 }
	}},
	{ $project: {
		_id: 0,
		nota_promedio: "$_id",
		cantidad_alumnos: "$cantidad_alumnos"
	}},
	{ $sort: { nota_promedio: 1} }
])
```

## Bases de datos wide column

### Cassandra

- Una fila está formada por:
  - Una clave compuesta (atributo ó conjunto de atributos)
  - Un conjunto de pares clave-valor ó columnas

![Cassandra](images/cassandra.png)

```sql
CREATE COLUMNFAMILY clientes (
nro_cliente int,
nombre text static,
ISBN bigint,
nombre_libro text,
primary key ((nro_cliente), ISBN));
```

- Para diseñar una base de datos en Cassandra debemos tener en
cuenta los siguientes puntos:
  1. No existe el concepto de junta. Si para alguna consulta típica
necesitamos el resultado de una junta, entonces debemos
guardarla como una tabla desnormalizada más desde el comienzo.
  2. No existe el concepto de integridad referencial. Si la necesitamos,
debe ser manejada desde el nivel de aplicación.
  3. Desnormalización de datos. En las bases de datos NoSQL el uso
de tablas no normalizadas está a la orden del día, y básicamente
por un único motivo: performance.
  4. Diseño orientado a las consultas.


![Cassandra modelo 1](images/cassandra_modelo_1.png)

![Cassandra modelo 2](images/cassandra_modelo_2.png)

- Cassandra está optimizado para altas tasas de escritura.
- Utiliza una estructura de búsqueda denominada LSM-tree
(log-structured merge tree), que mantiene parte de sus datos en
memoria, para diferir los cambios sobre el índice en disco.
- Se busca acceder en forma secuencial a disco, para mejorar el
trade-off entre el costo de hacer un disk seek y el costo de un
buffer en memoria. Esto ha sido bastante estudiado y se conoce
como regla de los cinco minutos (Five-minute Rule).

![Cassandra trees](images/cassandra_trees.png)

- Consideraciones:
  - Desde que se inserta una entrada en $C_0$ hasta que se traslada a
$C_1$ habrá una demora.
  - El costo de I/O de escritura en $C_0$ es nulo.
  - Cuando el tamaño de $C_0$ alcanza un umbral se inicia un proceso de
rolling merge (flush).
- El árbol $C_1$ suele tener una estructura similar a un B-tree.
- En cambio, como $C_0$ está en memoria no es relevante minimizar
su profundidad $\rightarrow$ suelen emplearse árboles balanceados como el
2-3 tree o el árbol AVL.

## Bases de datos basadas en grafos

- En las bases de datos basadas en grafos los elementos
principales son nodos y arcos (ejes).
- Estas bases de datos resultan útiles para modelar interrelaciones
complejas entre las entidades.
- Organizar nuestra base de datos de esta forma nos provee
ventajas para resolver problemas clásicos de grafos como:
  - Encontrar patrones de nodos conectados entre sí.
  - Encontrar caminos entre nodos.
  - Encontrar la ruta más corta entre dos nodos.
  - Calcular medidas de centralidad asociadas a los nodos.

## Teorema CAP

- **Consistencia**:
  - La propiedad de que en un instante determinado el sistema muestre un único valor de cada item de datos a los usuarios.
  - Su nivel máximo es la consistencia secuencial, en la que todas las operaciones de lectura/escritura distribuidas en el sistema pueden ordenarse de forma tal que toda lectura de un item siempre lea el último valor escrito en ese ítem.
  - Alcanzar consistencia secuencial requiere de un alto nivel de sincronización entre los nodos.

- **Disponibilidad**:
  - Consiste en que toda consulta que llega a un nodo del sistema distribuido que no está caído reciba una respuesta efectiva (es decir, sin errores).

- **Tolerancia a particiones**:
  - Consiste en que el sistema pueda responder una consulta aún cuando algunas conexiones entre algunos pares de nodos estén caídas.

- El Teorema CAP dice entonces que a lo sumo podremos ofrecer 2
de las 3 garantías:
  - AP: Si la red está particionada, podemos optar por seguir
respondiendo consultas aún cuando algunos nodos no respondan.
Garantizaremos disponibilidad, pero el nivel de consistencia no
será el máximo.
  - CP: Con la red particionada, si queremos garantizar consistencia
máxima no podremos garantizar disponibilidad. Es posible que no
podamos responder una consulta en forma efectiva porque
esperamos mensajes de confirmación desde nodos que no pueden
comunicarse.
  - CA: Si queremos consistencia y disponibilidad, entonces no
podremos tolerar que una cantidad indeterminada de enlaces se
caiga.

## BASE

- Las propiedades BASE representan un sistema distribuido con:
  - (BA) Disponibilidad básica (basic availability): El SGBD distribuido
está siempre en funcionamiento, aunque eventualmente puede
devolvernos un error, o un valor desactualizado.
  - (S) Estado débil (soft state): No es necesario que todas los nodos
réplica guarden el mismo valor de un ítem en un determinado
instante. No existe entonces un “estado actual de la base de
datos”.
  - (E) Consistencia eventual (eventual consistency): Si dejaran de
producirse actualizaciones, eventualmente todos los nodos réplica
alcanzarían el mismo estado.

