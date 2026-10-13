---
title: Apéndice
description: Información complementaria de referencia para consultas SQL y paneles de Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '145'
ht-degree: 5%
---

## Apéndice {#appendix}

**En este documento:** [14.1 Límites](#141-limits) · [14.2 Formato de archivo de exportación](#142-export-file-format) · [14.3 Hoja de trucos de una página](#143-one-page-cheat-sheet)

### Límites

| Elemento | Límite |
| -------------------------------- | ----------------------------------------------------------------------- |
| Texto de consulta | 4 KB |
| `FACET` atributos | 5 |
| `LIMIT` | 1 - 5 000 (valores predeterminados: número 1, faceta 10, eventos 100, serie temporal 5 000) |
| `IN (…)` valores | 1 - 100 |
| Ventana de tiempo | 31 días |
| Query timeout | 15 s (más un presupuesto de lectura de bytes) |
| `TIMESERIES AUTO` espacio mínimo | 2 minutos |
| Fórmula | 2 - 6 consultas; expresión ≤ 500 caracteres; nombres ≤ 40 caracteres |
| Panel de control | ≤ 40 fichas · ≤ 200 widgets por ficha · ≤ 50 variables |
| Widget | título ≤ 200 · subtítulo ≤ 500 · unidad ≤ 40 · ≤ 10 umbrales |
| Widget de texto | ≤ 4 000 caracteres |
| Variable | nombre ≤ 64 caracteres · ≤ 500 opciones/valores · consulta ≤ 4 KB |

### Exportar formato de archivo

Use **Exportar** para producir esto; **Importar** para cargarlo. Útil para controlar versiones de paneles o promocionarlos entre inquilinos.

```json
{
  "name": "Golden Signals",
  "templateVariables": [
    {
      "id": "var-1",
      "name": "svc",
      "label": "Service",
      "kind": "query",
      "multi": true,
      "query": "SELECT count(*) FROM Transaction FACET service LIMIT 200",
      "value": null
    }
  ],
  "pages": [
    {
      "id": "page-1",
      "name": "Overview",
      "widgets": [
        {
          "id": "sec-kpis",
          "title": "SLO KPIs",
          "queryLanguage": "text",
          "text": "",
          "isSection": true,
          "collapsed": false,
          "layout": { "x": 0, "y": 0, "w": 12, "h": 1 }
        },
        {
          "id": "w-error-rate",
          "title": "Error %",
          "queryLanguage": "SQL",
          "query": "SELECT percentage(*, WHERE status = 'Error') AS errorRate FROM Transaction",
          "vizType": "stat",
          "unit": "%",
          "thresholds": [
            { "value": 0, "status": "ok" },
            { "value": 1, "status": "warning" },
            { "value": 5, "status": "error" }
          ],
          "layout": { "x": 0, "y": 1, "w": 3, "h": 2 }
        },
        {
          "id": "w-throughput",
          "title": "Throughput by service",
          "queryLanguage": "SQL",
          "query": "SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 8",
          "vizType": "area",
          "layout": { "x": 3, "y": 1, "w": 9, "h": 4 }
        },
        {
          "id": "w-availability",
          "title": "Availability %",
          "queryLanguage": "SQL-formula",
          "formulaQueries": [
            {
              "name": "ok",
              "query": "SELECT count(*) AS v FROM Transaction WHERE status != 'Error'"
            },
            {
              "name": "total",
              "query": "SELECT count(*) AS v FROM Transaction"
            }
          ],
          "formulaExpression": "(ok / total) * 100",
          "vizType": "stat",
          "unit": "%",
          "layout": { "x": 0, "y": 3, "w": 3, "h": 2 }
        }
      ]
    }
  ],
  "exportedAt": "2026-10-01T00:00:00.000Z"
}
```

`vizType` valores: `timeseries` (línea), `area`, `bar`, `pie`, `stat` (número), `table`. Umbral `status`: `ok` (verde), `warning` (ámbar), `error` (rojo).

### Hoja de trucos de una página

```text
SHAPE        SELECT agg FROM Entity                         -> Number
             ... FACET a[, b..e] LIMIT n                     -> Bar / Pie / Table
             ... TIMESERIES AUTO                             -> Line / Area
             ... FACET a TIMESERIES AUTO LIMIT n             -> top-n series
             SELECT attr1, attr2 FROM Entity LIMIT n         -> event Table

AGGREGATES   count(*)  average(x)  sum(x)  min(x)  max(x)  uniqueCount(x)
             percentile(x, 95)  stddev(x)  latest(x[, rank])  earliest(x[, rank])
             percentage(*, WHERE c)  apdex(duration, 500)  percentOfTotal(x)
             rate(value)        -- counters, TIMESERIES only
             derivative(value)  -- gauges,   TIMESERIES only
             agg FILTER (WHERE c) AS alias

WHERE        = != < <= > >=   LIKE / NOT LIKE ('%', '_')   IN / NOT IN (...)
             IS [NOT] NULL   AND OR NOT ( )   strings in 'single quotes'
             string attr present?  ->  attr != ''

ORDER/LIMIT  ORDER BY alias [ASC|DESC]   LIMIT 1..5000

VARIABLES    = {{name}}   IN ({{name}})   numeric: > {{name}}

FORMULA      2-6 named queries, one metric each, no FACET, same shape
             expression: + - * / ( ) numbers names
```
