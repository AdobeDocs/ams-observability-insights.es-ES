---
title: Referencia SQL
description: Haga referencia a la sintaxis, las cláusulas y las funciones SQL que admite Observability Insights.
source-git-commit: 91a61d33010192cedce7d40a45ffe86f32e4824b
workflow-type: tm+mt
source-wordcount: '1505'
ht-degree: 11%
---

## Referencia SQL {#sql-reference}

Esta referencia cubre sintaxis, cláusulas, funciones y operadores SQL. Expanda cada entrada para obtener detalles y ejemplos.

### Componentes de consulta

```text
SELECT <select-item> [, <select-item> ...]
FROM   <Entity>
[WHERE      <condition>]
[FACET      <attribute> [, <attribute> ...]]          -- up to 5
[SINCE      <n> <unit> [AGO]]
[UNTIL      NOW | <n> <unit> [AGO]]
[TIMESERIES AUTO | MAX | <n> <unit>]
[ORDER BY   <alias-or-selected-name> [ASC | DESC]]
[LIMIT      <1..5000>]

<select-item> := <function>(<args>) [FILTER (WHERE <condition>)] [AS <alias>]
               | <attribute> [AS <alias>]
<unit>        := second | seconds | minute | minutes | hour | hours | day | days | week | weeks
```

> **Nota**
>
> Las `--` anotaciones anteriores son solo documentación. SQL en sí no acepta comentarios.

### Cláusulas obligatorias

#### `SELECT` — qué se debe devolver

+++Ver detalles

`SELECT` enumera **atributos sin procesar** (una consulta de evento) o **funciones de agregador** (una consulta de agregado). No se pueden mezclar los dos.

| Formulario | Ejemplo | Resultado |
| ------------ | -------------------------------------------------- | ------------------------------------------------- |
| Atributos | `SELECT timestamp, name, duration` | Filas de eventos individuales (valor predeterminado 100) |
| Agregados | `SELECT count(*), average(duration)` | Un valor por acumulado (por faceta/bloque de tiempo) |
| Con alias | `SELECT count(*) AS requests` | Etiqueta de columna `requests` |
| Alias citado | `SELECT percentile(duration, 95) AS 'p95 latency'` | Etiqueta con espacios |

Lista de eventos:

```sql
SELECT timestamp, service, name, duration, status, traceId FROM Transaction WHERE status = 'Error' LIMIT 50
```

Varios agregados:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

Reglas:

- `SELECT *` no es compatible. Enumere los atributos que desee.
- No se admiten las coincidencias entre atributos o agregados (`sum(a) / count(*)`). Usar un [widget de fórmula](#86-combine-queries-with-formula-widgets).
- La combinación de atributos y agregados produce un error con _No se pueden mezclar atributos sin procesar con agregados; use FACET para agrupar los resultados_.

+++

#### `FROM` — qué entidad consultar

+++Ver detalles

`FROM` nombra exactamente **una** entidad del [diccionario de datos](#data-dictionary). El nombre de entidad no distingue entre mayúsculas y minúsculas.

```sql
SELECT count(*) FROM Transaction
```

```sql
FROM Span SELECT count(*) FACET kind
```

Cada entidad tiene un ámbito fijo. Por ejemplo, `Transaction` es siempre el servidor expandido y `CpuUtilization` es siempre la métrica de CPU ocupado con `value` convertido en porcentaje. No se puede consultar más de una entidad en una sola consulta. Para combinar entidades, usa un [widget de fórmula](#86-combine-queries-with-formula-widgets).

+++

### Cláusulas opcionales

#### `AS` — etiquetar una columna

+++Ver detalles

Use `AS` para dar un nombre legible a un agregado o atributo. El alias se convierte en el encabezado de columna, la etiqueta de leyenda y el nombre al que hace referencia en `ORDER BY`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET service ORDER BY p95 DESC LIMIT 10
```

Utilice una cadena entre comillas simples para las etiquetas con espacios:

```sql
SELECT percentile(duration, 95) AS 'p95 latency (ms)' FROM Transaction TIMESERIES AUTO
```

+++

#### `FACET` — resultados del grupo

+++Ver detalles

`FACET` agrupa los resultados por hasta **5** atributos, como SQL `GROUP BY`.

```sql
SELECT count(*) FROM Transaction FACET service, http.method, http.statusCode LIMIT 30
```

| Consulta | Qué significa `LIMIT n` | Predeterminado |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `FACET` sin `TIMESERIES` | Número de filas | 10 |
| `FACET` con `TIMESERIES` | **Serie n superior**, clasificada por la suma del primer agregado en toda la ventana. Cada periodo de tiempo se conserva para esas series. | Todas las series (con un límite de 5000 filas) |

Serie temporal con facetas, top 5:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Los valores de faceta pueden ser cadenas o números. Las cadenas en blanco forman su propio grupo. Excluirlos con `WHERE attr != ''`.

+++

#### `FILTER` - agregados condicionales

+++Ver detalles

Anexe `FILTER (WHERE condition)` a cualquier agregado para calcularlo en un subconjunto de filas. Esto permite comparar varias condiciones en un solo paso.

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentile(duration, 95) FILTER (WHERE http.method = 'GET') AS getP95 FROM Transaction FACET service LIMIT 20
```

`FILTER` no se permite con `apdex`, `rate` o `derivative`. `percentage` ya cumple su propia condición.

+++

#### `LIMIT`: cuántas filas o series

+++Ver detalles

`LIMIT` toma un entero positivo. Los valores superiores a 5.000 se fijan a 5.000.

| Forma Resultado | Predeterminado |
| ------------ | ---------- |
| Valla | 1 |
| Faceta | 10 |
| Serie temporal | 5.000 filas |
| Eventos | 100 |

```sql
SELECT count(*) FROM Transaction FACET urlPath LIMIT 50
```

>[!TIP]
>
> Establezca siempre `LIMIT` en variables controladas por consultas y en facetas de alta cardinalidad (`urlPath`, `traceId`, `jcr.path`, `url`).

+++

#### `ORDER BY` — ordenar resultados

+++Ver detalles

`ORDER BY` ordena por un **alias o atributo seleccionado**. La dirección predeterminada es `DESC`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

| Forma Resultado | Orden predeterminado |
| ----------------- | --------------------------------------------------------------------------- |
| Valla publicitaria/faceta | Primer agregado, `DESC` |
| Serie temporal | Hora, `ASC` |
| Eventos | Primera columna seleccionada, `DESC` (así que ponga `timestamp` primero para &quot;más reciente primero&quot;) |

Reglas:

- La referencia a algo que no está seleccionado falla con _ORDER BY &#39;x&#39; debe hacer referencia a un valor o alias seleccionado_.
- No agregue `ORDER BY` a una consulta `TIMESERIES`. Anula el orden de tiempo que necesitan los gráficos.

+++

#### `SINCE` — inicio de la ventana

+++Ver detalles

`SINCE n unit [AGO]` establece hasta dónde empieza la ventana. Unidades: `second`, `minute`, `hour`, `day`, `week` (singular o plural). `AGO` es opcional.

```sql
SELECT count(*) FROM Transaction SINCE 6 hours ago
```

- El valor predeterminado es `SINCE 1 hour ago`.
- La ventana no puede exceder de **31 días** (_El intervalo de tiempo no puede exceder de 31 días_).
- Las fechas absolutas (`SINCE '2026-01-01'`) y `TODAY` / `YESTERDAY` no son compatibles.

>[!IMPORTANT]
>
> En los paneles y en Explorar, el selector de tiempo **anula** `SINCE` y `UNTIL`. Consulte [Prioridad de la ventana de tiempo](#time-window-precedence).

+++

#### `TIMESERIES` — trazar con el tiempo

+++Ver detalles

`TIMESERIES` devuelve una fila por bloque de tiempo (por valor de faceta), que está lista para gráficos de líneas, áreas o barras.

| Formulario | Tamaño del cubo |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIMESERIES 5 minutes` | Exactamente lo que usted pide (cualquier unidad) |
| `TIMESERIES AUTO` | Sobre la ventana ÷ 120, redondeado a 1s / 5s / 10s / 15s / 30s / 1m / 5m / 10m / 15m / 30m / 1h / 3h / 6h / 12h / 1d. **Nunca menos de 2 minutos.** |
| `TIMESERIES MAX` | Contenedores de 1 segundo. Usar solo durante unos minutos. |

`AUTO` contenedores habituales:

| Ventana | 1 h | 6 h | 24 h | 7 d | 30 d |
| ------ | ----- | ----- | ------ | --- | ---- |
| Cubo | 2 min | 5 min | 15 min | 3 h | 6 h |

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction TIMESERIES AUTO
```

```sql
SELECT count(*) FROM Transaction TIMESERIES 1 hour SINCE 1 day ago
```

>[!TIP]
>
> Preferir `AUTO` en los paneles. Mantiene alrededor de 120 puntos sea lo que sea que diga el selector de tiempo. También evita líneas por host &quot;destrozadas&quot;, ya que las métricas de host se eliminan aproximadamente cada 90 segundos.

+++

#### `UNTIL` — final de la ventana

+++Ver detalles

`UNTIL NOW` (predeterminado) o `UNTIL n unit [AGO]`. Combínelo con `SINCE` para ver una ventana anterior:

```sql
SELECT count(*) AS yesterday FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

`SINCE` debe ser anterior a `UNTIL`. De lo contrario, la consulta falla con _SINCE debe ser anterior a UNTIL_.

+++

#### `WHERE` — filas de filtro

+++Ver detalles

`WHERE` mantiene solamente las filas que cumplen una condición. Combinar condiciones con `AND`, `OR`, `NOT` y paréntesis. Consulte [Operadores y condiciones](#45-operators-and-conditions).

```sql
SELECT count(*) FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR http.statusCode >= 500)
```

`WHERE` solo puede aparecer una vez. Ponga todas las condiciones en él.

+++

### Funciones de agregador

| Función | Argumentos | Devuelve | Funciona en |
| -------------------------------------- | ----------------------------------- | ---------------------- | ------------------------------------- |
| [`apdex`](#fn-apdex) | duración numérica, umbral `T` | 0-1 | Seguimiento de entidades |
| [`average` / `avg`](#fn-average) | numérico | Misma unidad que la entrada | Cualquiera |
| [`count`](#fn-count) | `*` o atributo | Número de filas | Cualquiera |
| [`derivative`](#fn-derivative) | numérico | Cambio por segundo | Cualquiera (necesita `TIMESERIES`) |
| [`earliest`](#fn-latest) | cualquier [, rango] | Valor de atributo | Cualquiera |
| [`latest`](#fn-latest) | cualquier [, rango] | Valor de atributo | Cualquiera |
| [`max` / `min`](#fn-minmax) | cualquiera | Misma unidad que la entrada | Cualquiera |
| [`percentage`](#fn-percentage) | `*` o atributo, `WHERE` condición | % | Cualquiera |
| [`percentile`](#fn-percentile) | numérico, nivel `p` | Misma unidad que la entrada | Cualquiera |
| [`percentOfTotal`](#fn-percentoftotal) | numérico | % | Cualquiera (no `TIMESERIES`) |
| [`rate`](#fn-rate) | contador numérico | Por segundo | Entidades de contador (necesita `TIMESERIES`) |
| [`stddev`](#fn-stddev) | numérico | Misma unidad que la entrada | Cualquiera |
| [`sum`](#fn-sum) | numérico | Misma unidad que la entrada | Cualquiera |
| [`uniqueCount`](#fn-uniquecount) | cualquiera | Valores distintos | Cualquiera |

<a id="fn-apdex"></a>

#### `apdex(attribute, T)`

+++Ver detalles

Calcula una puntuación de [Apdex](https://www.apdex.org/) de 0 (frustrado) a 1 (satisfecho), usando el umbral `T` en la unidad del atributo (milisegundos para `duration`).

| Cubo | Condición | Grosor |
| ---------- | ------------------------------------ | ------ |
| Satisfecho | `duration ≤ T` y no es un error | 1 |
| Tolerante | `T < duration ≤ 4T` y no es un error | 0.5 |
| Frustrado | `duration > 4T`, **o cualquier error** | 0 |

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service
```

- Solo funciona en entidades de seguimiento (`Transaction`, `AppRequest`, `Span`).
- `T` debe ser mayor que 0. `FILTER` no está permitido.
- `apdex(duration, 500)` en `Transaction` está acelerado con el resumen en ventanas largas.

+++

<a id="fn-average"></a>

#### `average(attribute)` / `avg(attribute)`

+++Ver detalles

La media aritmética de un atributo numérico.

```sql
SELECT average(duration) AS avgMs FROM Transaction FACET service TIMESERIES AUTO
```

+++

<a id="fn-count"></a>

#### `count(*)` / `count(attribute)`

+++Ver detalles

- `count(*)` cuenta filas.
- `count(attribute)` cuenta filas donde el atributo es **no NULL**. Para los atributos de cadena, los valores que faltan están en blanco (`''`) en lugar de ser NULL, por lo que se siguen contando (consulte [Tratamiento de tipos](#46-type-handling-blank-vs-null)).

```sql
SELECT count(*) AS requests, count(http.statusCode) AS withStatusCode FROM Transaction
```

+++

<a id="fn-derivative"></a>

#### `derivative(attribute)`

+++Ver detalles

El cambio por segundo de `average(attribute)` entre bloques consecutivos: (este bloque − el anterior) ÷ segundos de bloque. El resultado puede ser negativo. Utilícelo en indicadores para ver si un valor está creciendo o reduciéndose.

```sql
SELECT derivative(value) AS heapGrowthPerSec FROM JvmHeapUsed WHERE host = 'publish-1' TIMESERIES 5 minutes
```

- Requiere `TIMESERIES` y debe ser el agregado **only**. `FILTER` no está permitido.
- El primer bloque no tiene predecesor.

+++

<a id="fn-latest"></a>

#### `latest(attribute [, rankAttribute])` / `earliest(…)`

+++Ver detalles

- `latest(attr)` devuelve el valor de la fila más reciente. `earliest(attr)` devuelve el valor de la fila más antigua.
- Con un segundo argumento, ese atributo elige la fila en lugar de tiempo. `latest(traceId, duration)` devuelve el identificador de seguimiento de la solicitud **más lenta**. `earliest(traceId, duration)` devuelve el valor más rápido.

```sql
SELECT latest(value) AS current FROM MemoryUtilization FACET host
```

```sql
SELECT max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 10
```

+++

<a id="fn-minmax"></a>

#### `max(attribute)` / `min(attribute)`

+++Ver detalles

El valor más grande o más pequeño. Funciona en números, cadenas y marcas de tiempo.

```sql
SELECT min(duration) AS fastestMs, max(duration) AS slowestMs, max(timestamp) AS lastSeen FROM Transaction FACET service
```

+++

<a id="fn-percentage"></a>

#### `percentage(*, WHERE condition)`

+++Ver detalles

El recurso compartido de filas que coinciden con `condition`, de 0 a 100.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO
```

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

`percentage` tiene su propia condición, por lo que no requiere `FILTER`.

+++

<a id="fn-percentile"></a>

#### `percentile(attribute, p)`

+++Ver detalles

El percentil p-th aproximado, donde `0 < p ≤ 100`. Use un(a) `percentile` por nivel.

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

p50, p95 y p99 en `Transaction` se aceleran mediante acumulación en ventanas largas.

+++

<a id="fn-percentoftotal"></a>

#### `percentOfTotal(attribute)`

+++Ver detalles

Cuota de cada fila de faceta de **suma total** de un atributo numérico. Las filas suman 100%.

```sql
SELECT sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfTime FROM Transaction FACET name LIMIT 15
```

No se permite con `TIMESERIES`.

+++

<a id="fn-rate"></a>

#### `rate(attribute)`

+++Ver detalles

Frecuencia por segundo de **contador acumulado**. En cada contenedor, `rate` obtiene la lectura de contador más alta por serie y la diferencia con el contenedor anterior. Los restablecimientos de contador tienen un piso de 0 y el primer bloque es 0.

```sql
SELECT rate(value) AS bytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

- Solo funciona en entidades de contador (marcadas ✔ en el [diccionario de datos](#53-curated-metric-entities)).
- Requiere `TIMESERIES` y debe ser el agregado **only**. `FILTER` no está permitido.

>[!WARNING]
>
> Siempre `FACET host` o filtre a un solo host. Un elemento `rate()` sin facetas en muchos hosts compara los contadores de hosts diferentes y produce números sin sentido. Los contadores de discos son por dispositivo, por lo que `FACET host, device` es el más preciso.

+++

<a id="fn-stddev"></a>

#### `stddev(attribute)`

+++Ver detalles

La desviación estándar de muestra. Utilícelo con `average` para juzgar la latencia coherente.

```sql
SELECT average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name LIMIT 20
```

+++

<a id="fn-sum"></a>

#### `sum(attribute)`

+++Ver detalles

El total de un atributo numérico.

```sql
SELECT sum(callCount) AS calls FROM SpanBreakdown FACET segment LIMIT 20
```

+++

<a id="fn-uniquecount"></a>

#### `uniqueCount(attribute)`

+++Ver detalles

El número **exacto** de valores distintos.

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

+++

Restricciones de función de un vistazo:

| Restricción | Funciones |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| Atributo numérico obligatorio | `average`, `sum`, `percentile`, `stddev`, `percentOfTotal`, `rate`, `derivative`, `apdex` |
| No `FILTER (WHERE …)` | `apdex`, `rate`, `derivative`, `percentage` |
| Requiere `TIMESERIES`, solo agregado | `rate`, `derivative` |
| No con `TIMESERIES` | `percentOfTotal` |
| Rastrear entidades solamente | `apdex` |
| Solo entidades de contador | `rate` |

### Operadores y condiciones

| Operador | Significado | Ejemplo |
| ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `=` | Signo igual | `status = 'Error'` |
| `!=`, `<>` | Distinto a | `kind != 'Internal'` |
| `<`, `<=`, `>`, `>=` | Comparación | `http.statusCode >= 500` |
| `LIKE` | Coincidencia de comodín: `%` = cualquier carácter, `_` = un carácter. Distinción entre mayúsculas y minúsculas. | `urlPath LIKE '/content/%'` |
| `NOT LIKE` | Carácter comodín anulado | `urlPath NOT LIKE '%.css'` |
| `IN (…)` | Coincide con cualquiera de los valores 1 a 100 | `service IN ('checkout', 'cart')` |
| `NOT IN (…)` | No coincide con ningún valor | `http.method NOT IN ('OPTIONS', 'HEAD')` |
| `IS NULL` / `IS NOT NULL` | Comprobación nula | `http.statusCode IS NOT NULL` |
| `AND`, `OR`, `NOT` | Lógica | `NOT (status = 'Ok')` |
| `( … )` | Agrupación | `(a = 1 OR b = 2) AND c = 3` |

```sql
SELECT count(*) FROM Transaction WHERE (status = 'Error' OR duration > 2000) AND NOT host LIKE '%dispatcher%' AND http.method IN ('GET', 'POST')
```

> **Advertencia**
>
> `attr = NULL` es un error. Usar `attr IS NULL`.

### Tratamiento de tipos: en blanco frente a NULO

Cada atributo tiene un tipo en el catálogo: `string`, `number`, `boolean` o `timestamp`. El tipo SQL comprueba las funciones en él (por ejemplo, `average(service)` se rechaza). No se necesitan ni están disponibles funciones de conversión de tipos.

> **Importante**
>
> **En blanco frente a NULL.** La mayoría de los atributos de cadena provienen de asignaciones de atributos OpenTelemetry, donde una clave _missing_ se lee como una cadena vacía `''`, no como `NULL`. Para mantener solamente los espacios que llevan un atributo, use **`attr != ''`** en lugar de `IS NOT NULL`. Los atributos numéricos como `http.statusCode` son `NULL` de verdad cuando están ausentes, por lo que `IS NOT NULL` funciona para ellos.

```sql
SELECT count(*) AS jcrCalls FROM Span WHERE jcr.operation != '' FACET jcr.operation
```

### Traducir SQL a SQL

| SQL | SQL |
| ------------------------------------------------------------ | ------------------------------------------------------------------- |
| `WHERE ts > now() - INTERVAL 1 HOUR` | Nada; el selector de tiempo (o `SINCE 1 hour ago`) establece la ventana |
| `GROUP BY service` | `FACET service` |
| `GROUP BY toStartOfInterval(ts, INTERVAL 5 MINUTE), service` | `FACET service TIMESERIES 5 minutes` |
| `quantile(0.95)(duration)` | `percentile(duration, 95)` |
| `countIf(status = 'Error')` | `count(*) FILTER (WHERE status = 'Error')` |
| `100 * countIf(x) / count()` | `percentage(*, WHERE x)` |
| `count(DISTINCT host)` | `uniqueCount(host)` |
| `argMax(traceId, duration)` | `latest(traceId, duration)` |
| `x / sum(x) OVER ()` | `percentOfTotal(x)` |
| `ORDER BY p95 DESC LIMIT 20` | `ORDER BY p95 DESC LIMIT 20` |
| `HAVING count() > 100` | _No compatible_. Ordene y `LIMIT` en su lugar. |
| `JOIN` / subconsulta | _No compatible_. Usar un widget **Formula**. |
| `CASE WHEN` | _No compatible_. Use `FILTER` columnas o `WHERE … LIKE` + `FACET`. |

### Importación de paneles existentes en Insights

Si conoce la consulta existente, a continuación verá los detalles de la migración

| Consulta existente | SQL | Notas |
| ------------------------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `FROM Transaction SELECT count(*)` | Igual | Se admite el primer pedido de `FROM` |
| `SINCE 1 day ago` | Igual | Se ignora en los paneles; el selector gana |
| `TIMESERIES AUTO` / `MAX` / `5 minutes` | Igual | `AUTO` piso es de 2 minutos |
| `FACET appName` | `FACET appName` / `FACET service` |                                                                                  |
| `percentage(count(*), WHERE error IS true)` | `percentage(*, WHERE status = 'Error')` |                                                                                  |
| `filter(count(*), WHERE …)` | `count(*) FILTER (WHERE …)` | Formulario estándar de SQL |
| `apdex(duration, t: 0.5)` | `apdex(duration, 500)` | `duration` está en **ms** |
| `latest(x)` / `earliest(x)` | Igual | Además de un formulario clasificado: `latest(x, rankAttr)` |
| `uniqueCount(x)` | Igual | Exacto |
| `rate(count(*), 1 minute)` | `count(*)` con `TIMESERIES 1 minute` | SQL `rate()` es solo para contadores |
| `derivative(x, 1 minute)` | `derivative(x)` + `TIMESERIES` | Por segundo |
| `uniques(x)` | `count(*) … FACET x LIMIT n` |                                                                                  |
| `keyset()` / `SHOW EVENT TYPES` | [Consultas de detección](../examples/query-cookbook.md#data-discovery) o el panel Guía de consulta |                                                                                  |
| `histogram(duration, …)` | `FILTER` contenedores | Ver [Tutorial 3](../tutorials/tutorial-3.md#build-a-latency-histogram) |
| `FACET CASES (…)` | `FILTER` columnas | Ver [Tutorial 3](../tutorials/tutorial-3.md#group-values-into-cases-with-filter) |
| `COMPARE WITH 1 week ago` | No compatible | Cambie el selector de tiempo o use `SINCE … UNTIL …` mediante la API |
| `RLIKE` | `LIKE` / `IN` |                                                                                  |
| Matemáticas en `SELECT` / agregación anidada | Widget **Fórmula** |                                                                                  |
| `WITH TIMEZONE`, `EXTRAPOLATE`, `SLIDE BY` | No compatible |                                                                                  |
