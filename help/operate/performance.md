---
title: Rendimiento
description: Mejore el rendimiento de consultas, gráficos y tableros en Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '227'
ht-degree: 3%
---

## Rendimiento {#performance}

Cada consulta se ejecuta con un tiempo de espera de 15 segundos y un presupuesto de lectura de bytes. Este documento explica cómo Observability Insights acelera las consultas automáticamente y cómo diseñar paneles que se mantengan rápidos en cualquier intervalo de tiempo.

### Aceleración de acumulación (automática)

`Transaction` gráficos leen tablas agregadas previamente cuando **todos** de ellos contienen:

| Condición | Requisito |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Forma | La consulta tiene `TIMESERIES` (los totales de número/tabla siempre se leen sin procesar para comprobar su exactitud) |
| Ventana | ≥ 3 h (nivel de 1 min) · ≥ 1 d (nivel de 5 min) · ≥ 7 d (nivel de 1 h) |
| Agregados | solo `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | solo el `service`, `appInstance`, `kind` |

Los gráficos `CpuUtilization` y `MemoryUtilization` (`average(value)`) y `JvmCpuRecentUtilization` (`average(value)`) con `FACET host` / `WHERE host …` durante ≥ 3 h leen resúmenes de 1 minuto.

**Sugerencia de diseño:** para widgets de tendencias de 7 a 31 días, manténgase fiel al conjunto compatible con el resumen (p95, error %, Apdex @ 500, count, servicio FACET); se mantienen rápidos en cualquier rango. Guarde los filtros `FACET host`/`name`, `average(duration)` o `urlPath` para ventanas o tablas más cortas.

### Sugerencias generales

- Use `LIMIT` en todas las consultas con facetas, especialmente `FACET urlPath`, `traceId`, `jcr.path` (alta cardinalidad).
- Filtrar al principio con `WHERE` en atributos de baja cardinalidad (`service`, `kind`, `status`).
- Prefiera `Transaction` sobre `Span` cuando solo necesite solicitudes entrantes, muchas menos filas.
- `uniqueCount` es exacto; en atributos de alta cardinalidad en ventanas largas, es el agregado más caro.
- Evite `TIMESERIES MAX` excepto las ventanas de unos minutos.
- Los widgets del panel almacenan en caché los resultados durante 30 s; el botón de actualización fuerza a que se vuelva a ejecutar.
- Un widget que alcanza el tiempo de espera de 15 segundos o los errores de presupuesto de lectura de bytes: reduzca la ventana, agregue filtros o reduzca las facetas.
