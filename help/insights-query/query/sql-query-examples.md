---
title: Ejemplos de consultas SQL
description: Explore ejemplos de consultas SQL para la telemetría de Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 10%
---

## Ejemplos de consultas SQL

Este es un ejemplo más completo. Para esta consulta:

- Usted elige `Transaction` como entidad.
- Utilice `SELECT` para calcular la duración del percentil 95.
- Agrupa los resultados por servicio con `FACET`.
- Se usa `TIMESERIES AUTO` para trazar los datos en un intervalo de tamaño automático.

```sql
FROM Transaction SELECT percentile(duration, 95) AS p95 FACET service TIMESERIES AUTO
```

Esto genera una línea por servicio en un gráfico de líneas, con los 10 servicios más ocupados mostrados de forma predeterminada.

Haga clic en cada ejemplo a continuación para ampliarlo.

<details>
<summary><strong>Consulta básica de solicitudes entrantes</strong></summary>

Tiempo medio de respuesta, en milisegundos, de todas las solicitudes entrantes:

```sql
SELECT average(duration) FROM Transaction
```

</details>

<details>
<summary><strong>Nombres de atributo que contienen puntos</strong></summary>

Los nombres de puntos no necesitan comillas:

```sql
SELECT count(*) FROM Transaction WHERE http.statusCode >= 500 FACET http.route
```

</details>

<details>
<summary><strong>La consulta devuelve varias columnas</strong></summary>

Varios agregados de una consulta se convierten en columnas en una tabla:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, percentile(duration, 95) AS p95, max(duration) AS maxMs FROM Transaction FACET service
```

</details>

<details>
<summary><strong>Agregados condicionales en un solo paso</strong></summary>

Total, errores y 5xx en paralelo:

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx FROM Transaction FACET service
```

</details>

<details>
<summary><strong>Consultando una métrica seleccionada</strong></summary>

Promedio de CPU por host a lo largo del tiempo:

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

</details>

<details>
<summary><strong>Consultando cualquier métrica de OpenTelemetry por nombre</strong></summary>

Use `MetricGauge` / `MetricSum` para métricas sin una entidad depurada:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool
```

</details>

<a id="what-sql-doesnt-do"></a>

## Qué no hace SQL

SQL es deliberadamente pequeño. Esto mantiene las consultas seguras, rápidas y aisladas del inquilino. Cuando necesite una de estas capacidades, utilice la alternativa.

| Capacidad | Estado | Alternativa |
| ------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------ |
| `SELECT *` | No compatible | Enumere los atributos que desee: `SELECT timestamp, name, duration, traceId …`. |
| Combinación de atributos sin procesar y agregados | No compatible | Use `FACET attr` o `latest(attr)` / `earliest(attr)`. |
| Matemáticas en `SELECT` (`a - b`, `count(*) / 60`) | No compatible | Widget de **Fórmula** (paneles). |
| Uniones y subconsultas | No compatible | **Widget de fórmula** o dos widgets uno al lado del otro. |
| `HAVING` | No compatible | `ORDER BY alias` + `LIMIT`. |
| `COMPARE WITH` | No compatible | Cambie el selector de tiempo o coloque dos widgets en pestañas diferentes. |
| `FACET CASES` | No compatible | Una columna `count(*) FILTER (WHERE …)` por caso (consulte [Tutorial 3](#83-group-values-into-cases-with-filter)). |
| `histogram()` | No compatible | Columnas `FILTER` agrupadas (consulte [Tutorial 3](#84-build-a-latency-histogram)). |
| `uniques()` / `keyset()` | No compatible | `FACET attr LIMIT n` enumera valores distintos. Las consultas de detección enumeran métricas y atributos rellenados. |
| `RLIKE` (regex) | No compatible | `LIKE` / `NOT LIKE` con `%` y `_` caracteres comodín, o `IN (…)`. |
| Comentarios (`--`, `//`, `/* */`) | No compatible | Coloque notas en el título del widget o en un widget de **texto**. |
| Fechas absolutas en `SINCE` | No compatible | Utilice el intervalo absoluto del selector de tiempo. |
| Conversión de tipos (`numeric()`, `string()`) | No se necesita | Los atributos se escriben en el catálogo. |
