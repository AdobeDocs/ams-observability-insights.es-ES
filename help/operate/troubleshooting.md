---
title: Resolución de problemas
description: Solucione consultas SQL, gráficos y tableros en Observability Insights.
source-git-commit: c45d62d2af62e6bd0c88e210b31c8954b75533be
workflow-type: tm+mt
source-wordcount: '335'
ht-degree: 5%
---

## Resolución de problemas {#troubleshooting}

Los mensajes de error aparecen en el editor cuando **Valida** o **Ejecuta** una consulta y en el widget mismo en un panel. Busque el mensaje en la primera columna.

| Mensaje/síntoma | Causa | Fix |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Unknown entity 'X'` | Nombre de error tipográfico o que no es de catálogo | Compruebe [Diccionario de datos](#data-dictionary) / el panel de guía de consultas. |
| `Attribute 'x' is not available on Entity` | El atributo no está en esa entidad (por ejemplo `duration` en `CpuUtilization`) | Atributos de seguimiento frente a atributos de métricas: consulte [Diccionario de datos](#data-dictionary). |
| `Raw attributes cannot be mixed with aggregates; use FACET to group results` | `SELECT service, count(*)` | `SELECT count(*) … FACET service`. |
| `FACET and TIMESERIES require an aggregate query` | Consulta de eventos con FACET/TIMESERIES | Utilice un acumulado. |
| `FACET supports at most five attributes` | > 5 facetas | Reducir. |
| `average requires a numeric attribute` | e.g. `average(service)` | Use `uniqueCount`/`latest` o un atributo numérico. |
| `rate requires TIMESERIES` / `rate is only available for counter measures` | `rate()` sin series temporales o en un medidor | Agregar `TIMESERIES AUTO`; usar una entidad de contador ✔ ([Entidades de métricas revisadas](#53-curated-metric-entities)); para indicadores usar `derivative()`. |
| `rate cannot be combined with other aggregates` | Varios elementos con `rate()` | Un `rate()` por widget; combinar con una fórmula. |
| `percentOfTotal cannot be combined with TIMESERIES` | — | ELIMINAR SERIES TEMPORALES. |
| `apdex requires an entity with a status attribute` | `apdex` en una entidad de métrica | Usar `Transaction`/`Span`/`AppRequest`. |
| `ORDER BY 'x' must reference a selected value or alias` | Ordenar por algo no seleccionado | Asigne un alias al agregado y ordene por el alias. |
| `IN requires between 1 and 100 values` | — | Dividir o usar `LIKE`. |
| `Use IS NULL or IS NOT NULL for null comparisons` | `attr = NULL` | Usar `IS NULL`. |
| `Unexpected character '{'` | `{{variable}}` en la vista previa del widget Agregar o al guardar | Ver [limitación del tutorial 3](../tutorials/tutorial-3.md#make-dashboards-reusable-with-variables). |
| `Unexpected character ';'` / `'"'` | Punto y coma o comillas dobles | Quitar `;`; usar comillas simples. |
| `Expected a positive duration such as 30 minutes` | `TIMESERIES 5m` | Escriba `TIMESERIES 5 minutes`. |
| `Time range cannot exceed 31 days` | Selector > 31 d | Reduzca la ventana. |
| `Select a customer before running a SQL query` | Sin inquilino / inquilino de demostración | Elija un cliente en el conmutador de inquilinos. |
| `All queries in a formula must be the same kind…` | Mezcla de total y SERIES TEMPORALES, o una FACETA | Hacer todas las subconsultas simples o todas las `TIMESERIES AUTO`; eliminar FACET. |
| `Query 'x' must select exactly one metric…` | La subconsulta tiene dos acumulados | Un acumulado por subconsulta. |
| `Unknown query name 'x' - must be one of: …` | La fórmula hace referencia a un nombre que falta | Igualar nombres exactamente (distingue mayúsculas de minúsculas). |
| **Error al guardar** | Una consulta de widget no es válida del lado del servidor (a menudo `{{var}}`) | Abra cada widget editado recientemente y **ejecútelo**; corrija o elimine el no válido. |
| _Seleccionar un valor para{{x}}_ | La variable no tiene selección | Elija un valor en la barra de variables. |
| _No hay actividad en los últimos..._ | No hay filas en la ventana | Ampliar la ventana; compruebe los filtros con una consulta de detección ([Detección de datos](../examples/query-cookbook.md#data-discovery)). |
| Líneas divididas en puntos | Cubo menor que la cadencia de datos | Use `TIMESERIES AUTO` (≥ 2 min) o un gráfico de barras. |
| La faceta muestra un grupo de cadenas vacías | Falta atributo en algunas filas | Agregar `WHERE attr != ''`. |
| _Este panel SQL heredado ya no se ejecuta_ | Panel del antiguo generador de raw-SQL | Edite → vuelva a generar en SQL. |
