---
title: 'Tutorial 2: Control de datos'
description: Aprenda a filtrar, agrupar y controlar los resultados de las consultas en Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '361'
ht-degree: 0%
---

## Tutorial 2: Control de datos {#tutorial-2-control-your-data}

En este tutorial, aprenderá las funciones y cláusulas que convierten los recuentos sin procesar en respuestas: recuentos distintos, valores más recientes, porcentajes, percentiles, caracteres comodín, ordenación y agrupación multidimensional.

**En este tutorial:** [7.1 uniqueCount](#71-count-distinct-values-with-uniquecount) · [7.2 más reciente/más reciente](#72-find-the-latest-and-earliest-values) · [7.3 porcentaje](#73-calculate-percentages) · [7.4 percentil](#74-understand-latency-with-percentile) · [7.5 Etiquetas](#75-make-results-readable-with-labels) · [7.6 Comodines y listas](#76-match-patterns-and-lists) · [7.7 Clasificación](#77-sort-and-rank-with-order-by) · [7.8 Múltiples facetas](#78-group-by-several-attributes) · [7.9 windows](#79-look-at-a-past-window)

### Contar valores distintos con `uniqueCount`

¿Cuántos hosts y seguimientos distintos tiene cada servicio?

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

`uniqueCount` es exacto, no una estimación.

Para **enumerar** los valores distintos (lo que NRQL llama `uniques()`), haga lo siguiente en el atributo:

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 100
```

### Buscar los valores más recientes y más antiguos

`latest(attr)` devuelve el valor más reciente y `earliest(attr)` el más antiguo. Son útiles para tablas de &quot;estado actual&quot;:

```sql
SELECT latest(value) AS currentCpu FROM CpuUtilization FACET host
```

```sql
SELECT earliest(timestamp) AS firstSeen, latest(timestamp) AS lastSeen FROM Transaction FACET service
```

### Cálculo de porcentajes

`percentage(*, WHERE condition)` devuelve el recurso compartido de filas que cumplen una condición. Esta es la tasa de error:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction
```

Tendencias por servicio:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

Y realice un seguimiento de la proporción de solicitudes lentas:

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction FACET service
```

### Comprender la latencia con `percentile`

Los promedios ocultan periféricos. Los percentiles muestran lo que experimentan la mayoría de los usuarios (p50) y lo que experimentan los desafortunados (p95, p99):

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

>[!TIP]
>
> p50, p95 y p99 de `Transaction.duration` se sirven desde resúmenes en ventanas largas, por lo que permanecen rápidas para las vistas semanales.

### Hacer que los resultados sean legibles con etiquetas

Use `AS` con comillas simples en las etiquetas que contengan espacios:

```sql
SELECT count(*) AS 'Requests', percentile(duration, 95) AS 'p95 (ms)' FROM Transaction FACET service
```

>[!NOTE]
>
> Los alias se aplican solamente a `SELECT` elementos. Las columnas de faceta siempre se etiquetan con el nombre del atributo.

### Hacer coincidir patrones y listas

`LIKE` usa `%` para cualquier número de caracteres y `_` para exactamente uno:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%.html' FACET urlPath LIMIT 20
```

Excluir recursos estáticos con `NOT LIKE`:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath NOT LIKE '%.js' AND urlPath NOT LIKE '%.css' AND urlPath NOT LIKE '%.png' FACET urlPath LIMIT 20
```

Coincidencia con una lista con `IN` y `NOT IN` (hasta 100 valores):

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.method IN ('POST', 'PUT', 'DELETE') FACET service
```

>[!NOTE]
>
> SQL no tiene expresiones regulares (`RLIKE`). Combine `LIKE` patrones con `OR` o use `IN`.

### Ordenar y clasificar con `ORDER BY`

De forma predeterminada, los resultados con facetas se ordenan por el primer acumulado. Use `ORDER BY` para clasificarse por otra cosa, como las operaciones más lentas por p95:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

O los servicios menos utilizados primero:

```sql
SELECT count(*) AS requests FROM Transaction FACET service ORDER BY requests ASC LIMIT 10
```

>[!TIP]
>
> No hay `HAVING`. Para buscar solo operaciones &quot;ocupadas&quot;, clasifíquelas por `requests` y mantén el N superior, o agrega un `WHERE` que elimine el ruido (por ejemplo, `WHERE http.method != 'OPTIONS'`).

### Agrupar por varios atributos

`FACET` acepta hasta cinco atributos. Cada combinación se convierte en una fila o serie:

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET service, http.method, http.statusCode LIMIT 30
```

### Mirar una ventana pasada

No hay `COMPARE WITH`. En los paneles y en Explorar, cambie el **selector de tiempo**. Cuando llama a la API sin un intervalo explícito, `SINCE … UNTIL …` le muestra cualquier ventana anterior:

```sql
SELECT count(*) AS lastWeekSameHour FROM Transaction SINCE 169 hours ago UNTIL 168 hours ago
```
