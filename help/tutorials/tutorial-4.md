---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '491'
ht-degree: 1%
---
## Tutorial 4: Funciones avanzadas {#tutorial-4-advanced-functions}

Este tutorial cubre las funciones y entidades que impulsan investigaciones más profundas: variabilidad, crecimiento, búsquedas del &quot;peor ejemplo&quot;, proporción del total, composición de transacciones y métricas arbitrarias de OpenTelemetry.

**En este tutorial:** [9.1 stddev](#91-measure-consistency-with-stddev) · [9.2 derivado](#92-spot-growth-with-derivative) · [9.3 Último clasificado](#93-find-the-worst-example-with-ranked-latest) · [9.4 por ciento del total](#94-find-where-time-goes-with-percentoftotal) · [9.5 SpanBreakdown](#95-break-down-transactions-with-spanbreakdown) · [9.6 Cualquier métrica](#96-discover-and-query-any-metric) · [9.7 Elige la entidad](#97-choose-the-right-trace-entity) · [9.8 Agregación anidada](#98-work-around-nested-aggregation)

### Medir coherencia con `stddev`

Dos operaciones pueden compartir un promedio y seguir comportándose de forma muy diferente. La desviación estándar muestra la cantidad de latencia que varía:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY stddevMs DESC LIMIT 20
```

Una desviación estándar mayor que la media apunta a una operación con solicitudes ocasionales muy lentas. Mira su p99 a continuación.

### Crecimiento puntual con `derivative`

`derivative(attr)` devuelve el cambio por segundo del promedio del atributo entre bloques. Es útil para detectar un medidor que está subiendo, como el uso de la pila antes de una fuga:

```sql
SELECT derivative(value) AS heapMBPerSec FROM JvmHeapUsed FACET host TIMESERIES 5 minutes
```

Una línea positiva sostenida significa crecimiento. Los valores negativos significan que el indicador está cayendo (por ejemplo, después de la recolección de basura). Al igual que `rate`, `derivative` necesita `TIMESERIES` y debe ser el único agregado.

### Busque el peor ejemplo con la clasificación `latest`

El segundo argumento de `latest` y `earliest` cambia &quot;más reciente&quot; a &quot;más alto por este atributo&quot;. `latest(traceId, duration)` devuelve el identificador de seguimiento de la solicitud más lenta, para que pueda pegarlo en la búsqueda de seguimiento:

```sql
SELECT count(*) AS requests, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace, latest(urlPath, duration) AS slowestPath FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 15
```

Use `earliest(traceId, duration)` para obtener el ejemplo más rápido para la comparación.

### Buscar adónde va el tiempo con `percentOfTotal`

`percentOfTotal(attr)` muestra la participación de cada grupo en el total. Aplicado a `duration`, muestra qué operaciones consumen la mayor parte del tiempo del servidor. Esto suele ser más útil que las operaciones más lentas:

```sql
SELECT count(*) AS requests, sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfServerTime FROM Transaction FACET name LIMIT 15
```

`percentOfTotal` no se puede combinar con `TIMESERIES`.

### Desglose de transacciones con `SpanBreakdown`

`SpanBreakdown` se acumula previamente en la ingesta. Registra en qué operaciones secundarias (`segment`) pasa tiempo cada transacción (`rootOperation`), minuto a minuto. Responde a &quot;¿qué hace este extremo?&quot; barato:

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE '%GET%' FACET rootOperation, segment LIMIT 25
```

Tendencia de un segmento con el tiempo:

```sql
SELECT average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET segment TIMESERIES AUTO LIMIT 8
```

> [!NOTE]
> `avgDurationMs` no se ha etiquetado automáticamente con una unidad. Establezca el widget **Unit** en `ms`.

### Descubra y consulte cualquier métrica

SQL no tiene `keyset()` ni `SHOW EVENT TYPES`. En su lugar, enumere todas las métricas que estén generando informes, junto con su unidad y el valor más reciente:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

A continuación, consulte cualquier métrica por su nombre. Por ejemplo, memoria JVM utilizada, por grupo:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 50
```

Para ver qué **atributos span** se rellenan para cada servicio, cuente los valores que no estén en blanco:

```sql
SELECT count(*) FILTER (WHERE http.route != '') AS withRoute, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE peer.service != '') AS withPeer, count(*) FILTER (WHERE error.type != '') AS withErrorType FROM Span FACET service LIMIT 50
```

> [!TIP]
> Cuando existe una entidad depurada (por ejemplo, `CpuUtilization`), preferirla sobre `MetricGauge`. Aplica el filtro y la conversión de unidad adecuados, y puede acelerarse con el resumen.

### Elija la entidad de seguimiento adecuada

| Uso | Cuando quieras | Ejemplo |
| ------------- | ------------------------------------------------------------------------------------------- | --------------------------------- |
| `Transaction` | KPI de solicitudes HTTP/RPC entrantes (rendimiento, latencia, errores, Apdex). Acumulación acelerada. | Señales de oro |
| `AppRequest` | Los mismos KPI, **incluidos los consumidores de mensajes** (Kafka y similares) | Servicios impulsados por eventos |
| `Span` | Cualquier cosa por debajo de la solicitud: HTTP de salida, base de datos, mensajería, AEM Sling/JCR/JSP, intervalos internos | Análisis de dependencias y rutas de código |

Cuente cada tipo de intervalo para ver qué se ha instrumentado:

```sql
SELECT count(*) AS spans, percentile(duration, 95) AS p95 FROM Span FACET kind
```

Llamadas salientes por dominio externo:

```sql
SELECT count(*) AS calls, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain LIMIT 20
```

### Solucionar la agregación anidada

NRQL permite agregar un agregado con una subconsulta, por ejemplo, &quot;el promedio de los máximos por host&quot;. SQL no lo hace. Estas son las alternativas:

| Meta | Enfoque SQL |
| --------------------------------------- | ---------------------------------------------------------- |
| Un valor por grupo y, a continuación, clasifique los grupos | `FACET group` + `ORDER BY alias` + `LIMIT` |
| Una proporción de dos agregados | [Widget de fórmulas](#86-combine-queries-with-formula-widgets) |
| Un valor por host y, a continuación, el &quot;peor host&quot; | `FACET host ORDER BY alias DESC LIMIT 1` |
| Un promedio por host a lo largo del tiempo | `FACET host TIMESERIES AUTO` y lea la serie |

Ejemplo: el peor host por p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET host ORDER BY p95 DESC LIMIT 1
```
