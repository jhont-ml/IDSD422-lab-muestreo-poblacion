# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** 23/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Revisamos las 8 columnas de la base de fidelización y las clasificamos por criterio: 3 son numéricas (`edad`, `numero_hijos` e `ingresos_mensuales`) y 5 son categóricas, dos de ellas ordinales (`nivel_educacion` y `satisfaccion`); el código confirmó los tipos de cada una. Después estimamos el ingreso mensual promedio de los 400 clientes con muestras de 20 y tres diseños: conveniencia, aleatorio simple y estratificado por ciudad. El valor real es 1330.72 USD y las estimaciones fueron 1348.50, 1336.00 y 1287.50 USD, diferencias que con un solo sorteo no permiten nombrar un mejor diseño. Antes de decidir con una muestra, la gerencia debe saber que la de conveniencia dejó fuera a Guayaquil y Cuenca, y que cinco sorteos aleatorios dieron medias entre 1235.00 y 1491.50 USD: pesan más el diseño y el azar del sorteo que la cantidad de filas.

## Parte A — Clasificación de las variables

*Completa la tabla usando el **criterio** (¿tiene sentido la mitad del valor? ¿hay orden entre las categorías?), no el ejemplo. En «verificación» anota qué mostró el código: dtype, número de valores únicos y, si declaraste el orden, cómo lo comprobaste.*

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Categórica nominal (identificador) | Es un código que solo distingue a un cliente de otro. No hay orden entre los códigos y promediarlos no significa nada. | dtype `object` (texto) |
| `edad` | Numérica continua | La escala admite fracciones (años y meses) y la mitad de un valor tiene sentido, aunque se registre en años enteros; por eso se puede promediar. | dtype `int64` |
| `numero_hijos` | Numérica discreta | Sale de contar y solo toma valores enteros (0, 1, 2…); no existen 1.5 hijos. Se puede promediar, aunque es un conteo. | dtype `int64` |
| `genero` | Categórica nominal | Tiene 3 categorías (femenino, masculino y otro) y ninguna está por encima de otra; no se puede promediar. | dtype `object` (texto) |
| `nivel_educacion` | Categórica ordinal | Sus niveles siguen un orden natural (Primaria < Secundaria < Universidad < Posgrado), pero la distancia entre un nivel y otro no es medible; no se promedia. | dtype `object`; el orden se declara en código como categoría ordenada |
| `ciudad` | Categórica nominal | Quito, Guayaquil y Cuenca son grupos sin jerarquía entre sí; no se puede promediar. | dtype `object` (texto) |
| `ingresos_mensuales` | Numérica continua | Es un monto en dólares: se puede sumar, promediar y tomar la mitad de un valor. | dtype `int64` |
| `satisfaccion` | Categórica ordinal | Es una escala de 1 a 5 con orden claro, pero un 4 no equivale al doble de un 2; se resume mejor con mediana o moda que con media. | dtype `int64`; la escala 1–5 se declara en código como ordenada |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype de `satisfaccion` fue `int64`: pandas la leyó como número entero porque sus valores son dígitos. Pero el dtype solo dice cómo se almacenó el dato, no qué significa. Por criterio es ordinal: los niveles tienen orden, pero no se puede medir la distancia entre ellos, y un 4 no es "el doble" de un 2. Tratarla como numérica permitiría sacar una media que parece un resultado y no tiene una interpretación clara. Si una ordinal queda como texto sin declarar el orden, el lenguaje la ordena alfabéticamente, y operaciones como `.min()`, `.max()` o una comparación dan respuestas sin sentido (por ejemplo, "Posgrado" antes de "Primaria"). También conviene declarar los cinco niveles de la escala: en las 12 filas nadie respondió 1, y si no se declara ese nivel desaparece de la categoría.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes (la base de fidelización completa) |
| Tamaño de cada muestra (n) | 20 (5 % de N) |
| Variable de interés | `ingresos_mensuales` (USD) |
| Parámetro poblacional (media) | 1330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

Porque en la práctica casi nunca se tiene toda la población a mano, y medirla completa suele ser lento o caro. Aquí el parámetro se conoce a propósito: sirve de referencia para ver cuánto se equivoca cada diseño. En un caso real solo tendríamos la muestra, y el parámetro sería justamente lo que se busca estimar.

### Los tres diseños

*Corre tu script y registra las salidas. La composición por ciudad es la que delata el sesgo: anótala tal como salió.*

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1348.50 USD | Quito 20 | +17.78 USD |
| Aleatorio simple (`sample`) | 20 | 1336.00 USD | Cuenca 8, Quito 8, Guayaquil 4 | +5.28 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1287.50 USD | Quito 10, Guayaquil 6, Cuenca 4 | −43.22 USD |

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media (USD) | 1336.00 | 1405.00 | 1235.00 | 1491.50 | 1288.00 |
| Diferencia vs. parámetro (USD) | +5.28 | +74.28 | −95.72 | +160.78 | −42.72 |

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro es 1330.72 USD. La conveniencia estimó 1348.50 (+17.78), el aleatorio simple 1336.00 (+5.28) y el estratificado 1287.50 (−43.22). El aleatorio simple quedó más cerca, pero cada diseño se corrió una sola vez, así que ese resultado no demuestra que sea mejor: con otras semillas el mismo diseño se alejó hasta −95.72 USD (semilla 3) y +160.78 USD (semilla 4). La conveniencia también estuvo cerca, pero por casualidad: su muestra es 100 % de Quito y no representa a toda la base. Un diseño se juzga por cómo se comporta al repetirlo muchas veces, no por un único resultado.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

El defecto es un sesgo de selección. El archivo viene ordenado por ciudad y `head(20)` toma el inicio sin sortear, de modo que no todos los clientes tenían la posibilidad de entrar. La composición lo delata: las 20 filas son de Quito y no aparecen Guayaquil (30 % de la base) ni Cuenca (20 %), es decir, la mitad de la población queda fuera. Tomar más filas no lo arregla, porque seguiría siendo el mismo tramo del archivo; lo que lo corrige es el diseño, con un sorteo en el que todos tengan opción de entrar.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

Las cinco medias fueron 1336.00, 1405.00, 1235.00, 1491.50 y 1288.00 USD, con 256.50 USD de distancia entre la menor y la mayor. Una empresa que hace un solo estudio recibe solo uno de esos valores y no sabe cuánto se desvió del real: podría creer que el ingreso medio es 1235 o 1491.50 USD solo por la suerte del sorteo. Por eso la estimación de una muestra debe presentarse con su margen de incertidumbre, y una decisión importante no debería apoyarse en un número sacado de una sola muestra pequeña.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

- **Conveniencia:** para un piloto o una prueba rápida donde no se busca generalizar, por ejemplo, revisar si un cuestionario se entiende con los primeros 20 clientes que lleguen a la tienda.
- **Aleatorio simple:** cuando se necesita una conclusión sobre toda la población y no hay un subgrupo que deba estar garantizado, por ejemplo, estimar el ingreso medio de todos los clientes.
- **Estratificado:** cuando hay subgrupos que sí o sí deben aparecer, por ejemplo, evaluar una campaña por ciudad; fija la proporción de cada grupo (10 de Quito, 6 de Guayaquil y 4 de Cuenca).

## Conclusiones

1. **El diseño, y no el tamaño, decide si la muestra representa a la población.** Las tres muestras tuvieron 20 filas, pero la de conveniencia dejó fuera a dos de las tres ciudades. Agregar más filas del mismo tramo del archivo no habría corregido ese sesgo.

2. **Un solo resultado no sirve para juzgar un diseño ni para decidir.** El aleatorio simple quedó más cerca con la semilla 1, pero entre sus cinco sorteos hubo 256.50 USD de diferencia: la media de una muestra es una estimación con incertidumbre, no el valor real. El estratificado quedó más lejos en este sorteo (−43.22 USD), pero fue el único que aseguró la proporción por ciudad.

3. **Recomendación para la gerencia:** antes de aceptar un análisis basado en muestras, pedir que se explique cómo se eligió la muestra, de qué tamaño es y qué margen de incertidumbre tiene la estimación, y desconfiar de muestras armadas con "los primeros registros" de un archivo.