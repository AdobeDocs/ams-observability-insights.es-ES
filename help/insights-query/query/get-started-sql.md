---
title: Introducción a SQL Server
description: Aprenda a consultar la telemetría de Observability Insights mediante SQL.
source-git-commit: 148bf6480461a0109b221b8e7c2f9ab2035a7d68
workflow-type: tm+mt
source-wordcount: '687'
ht-degree: 1%
---

# Introducción

**Adobe Observability Insights Query Language (SQL)**: consulte, grafique y tablero su telemetría en Adobe Observability Insights.

|                |                                                                  |
| -------------- | ---------------------------------------------------------------- |
| **Se aplica a** | Interfaz de usuario de Insights → **Explorar** y **Paneles** (paneles personalizados) |
| **Público** | Ingenieros, SRE y usuarios avanzados que conocen SQL |

> **Sugerencia**
>
> Cada bloque de código marcado `sql` es una consulta SQL que puede pegar directamente en **Explorar → SQL** o en **Paneles → Agregar widget → SQL**.

## Introducción a SQL: el idioma de la telemetría {#get-started-with-sql-the-language-of-your-telemetry}

El lenguaje de consulta de perspectivas es un lenguaje de consulta similar a SQL para los datos de Observability Insights. Es el mismo idioma detrás de cada widget de panel personalizado y de la página **Explorar**, para que puedas crear las vistas que tu equipo necesita.

SQL funciona con **entidades**, que son vistas lógicas descriptivas como `Transaction`, `Span` o `CpuUtilization`. Nunca consulta directamente las tablas de almacenamiento. Observability Insights compila cada consulta SQL en una consulta de almacén parametrizada y segura. Siempre abarca esa consulta al inquilino y la ventana de tiempo seleccionada.

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

### Qué puede hacer con SQL

Utilice SQL para lo siguiente:

- **Cree gráficos** de rendimiento, tasa de error, latencia, Apdex, CPU, memoria, montón de JVM, disco y red a lo largo del tiempo.
- **Responda preguntas específicas**, como &quot;¿qué rutas de URL en el nivel de publicación tienen la peor p95?&quot; o &quot;¿qué dominio descendente falla?&quot;
- **Investigue incidentes** explorando desde un KPI hasta solicitudes individuales e ID de seguimiento.
- **Generar paneles** con KPI, tendencias, tablas, fórmulas, notas, encabezados de sección y variables.
- **Cargas de trabajo de perfil de AEM** (Adobe Experience Manager) con atributos de Sling, JSP, JCR, Dispatcher y GraphQL.

SQL admite **consultas de eventos**, que devuelven filas de solicitud individuales, y **consultas de agregados**, que devuelven totales, percentiles, tasas, desgloses y series temporales.

> **Sugerencia**
>
> Las plantillas de **Paneles → Crear panel** se han creado completamente a partir de SQL. Abra cualquier widget de plantilla con el icono de lápiz para ver su consulta y aprender de ella.

### Qué puede consultar

SQL puede consultar cuatro familias de datos. La lista completa está en el [diccionario de datos](/help/references/data-dictionary.md).

| Familia | Entidades | Preguntas habituales |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Seguimientos (APM)** | `Transaction` (solicitudes entrantes), `AppRequest` (solicitudes + consumidores de mensajes), `Span` (cada intervalo) | Rendimiento, latencia, errores, Apdex, llamadas descendentes, rutas de código AEM |
| **Métricas de tiempo de ejecución y host revisadas** | `CpuUtilization`, `MemoryUtilization`, `LoadAverage5m`, `DiskUtilization`, `NetworkReceiveRate`, `JvmHeapUsed`, `JvmThreadCount`, `DispatcherRequestRate` y más | Saturación, capacidad, estado de JVM, carga de Dispatcher |
| **Métricas genéricas** | `MetricGauge`, `MetricSum` | Cualquier métrica de OpenTelemetry por nombre, detección de métricas |
| **Composición de span preagregada** | `SpanBreakdown` | Qué operaciones secundarias constituyen una transacción |

> **Nota**
>
> Los registros, las definiciones de alertas y las relaciones de inventario no están disponibles.

### Dónde usar SQL

| Donde | Cómo abrirlo. | ¿Para qué sirve? |
| ----------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Explorar** | → de navegación izquierda **Explorar** | Análisis ad hoc. Use el **Generador de imágenes** o la ficha **SQL**, **Validar**, luego **Ejecutar** (⌘/Ctrl + Entrar). Incluye consultas **History** y **Saved** y un panel de **guía de consulta**. |
| **Paneles** | El → de navegación izquierdo **Paneles** → abrir o crear un panel → **Agregar widget** | Gráficos, KPI y tablas persistentes, combinados en paneles. |
| **Widgets de fórmula** | **Agregar widget → fórmula** | Combine de 2 a 6 consultas SQL con aritméticas (relaciones, margen de ampliación, tasas de grabación). |
| **Variables de panel** | **+ → de variable de una consulta** | Rellenar un selector desde los distintos valores de una consulta SQL `FACET`. |
| **Inspección de consultas** | Pase el ratón sobre un gráfico → inspeccionar consulta | Consulte el SQL detrás de un gráfico integrado o de panel. |

### Estructura de consulta SQL

SQL sigue una sintaxis basada en cláusulas. Esta consulta cuenta todas las solicitudes entrantes en la ventana seleccionada:

```sql
SELECT count(*) FROM Transaction
```

Estas son todas las cláusulas disponibles:

```text
SELECT  function(attribute) [FILTER (WHERE condition)] [AS alias] [, ...]
        | attribute [AS alias] [, ...]
FROM    Entity
  [WHERE      condition [AND | OR ...]]
  [FACET      attribute [, attribute ...]]        -- up to 5
  [SINCE      n unit [AGO]]
  [UNTIL      NOW | n unit [AGO]]
  [TIMESERIES AUTO | MAX | n unit]
  [ORDER BY   alias [ASC | DESC]]
  [LIMIT      number]                              -- 1 to 5000
```

Estas son las reglas que SQL sigue:

| Regla | Detalles |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cláusulas requeridas** | Se requieren `SELECT` y `FROM`. Todas las demás cláusulas son opcionales. Puede empezar con: `FROM Transaction SELECT count(*)` también es válido. |
| **Orden de la cláusula** | Las cláusulas posteriores a `FROM` pueden aparecer en cualquier orden. Cada cláusula puede aparecer una sola vez. |
| **Tamaño de consulta** | Una consulta debe tener 4 KB o menos. |
| **Distinción de mayúsculas y minúsculas** | Las palabras clave, los nombres de función, los nombres de entidad y los nombres de atributo **no** distinguen entre mayúsculas y minúsculas. Los valores de cadena **distinguen entre mayúsculas y minúsculas** (`'Error'` ≠ `'error'`). |
| **Cadenas** | Use comillas simples: `WHERE status = 'Error'`. Para incluir un presupuesto, debe duplicarlo: `'O''Brien'`. No se aceptan comillas dobles. |
| **Literales** | Cadenas, números (incluidos negativos y decimales), `TRUE`, `FALSE`, `NULL`. |
| **Nombres de atributos** | Los nombres pueden contener letras, dígitos, `_` y `.` (por ejemplo `http.statusCode`). No se necesitan palos traseros. |
| **Agregados vs. atributos** | Una consulta selecciona atributos sin procesar (una lista de eventos) o agregados, nunca ambos. Para agrupar, use `FACET`. |
| **Ventana de tiempo** | El valor predeterminado es la última hora. El máximo es de 31 días. En los paneles y en Explorar, el **selector de tiempo** establece la ventana (consulte [Ventana de tiempo](how-to-query-sql.md#time-window-precedence)). |
| **No compatible** | Punto y coma, comentarios, subconsultas, combinaciones, `SELECT *`, matemáticas entre atributos en `SELECT`, `HAVING`, `COMPARE WITH`. Vea [Lo que SQL no hace](sql-query-examples.md#what-sql-doesnt-do) para obtener alternativas. |
