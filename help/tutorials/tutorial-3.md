---
title: 'Tutorial 3: Avance en el tablero'
description: Cree paneles con variables, fórmulas y varios widgets en Observability Insights.
source-git-commit: 91a61d33010192cedce7d40a45ffe86f32e4824b
workflow-type: tm+mt
source-wordcount: '879'
ht-degree: 3%
---

## Tutorial 3: Avance en el tablero {#tutorial-3-advance-your-dashboarding}

En este tutorial, creará los componentes de un panel de control de nivel de producción: puntuaciones de experiencia del usuario, tasas de contador, agrupación de casos, histogramas, mosaicos de KPI con umbrales, secciones, widgets de fórmula y variables.

### Experiencia del usuario de puntuación con `apdex`

Apdex convierte latencia y errores en una puntuación de 0 a 1. Con un umbral de 500 ms, una solicitud está _satisfecha_ a 500 ms o menos, _tolera_ hasta 2.000 ms y _se siente frustrada_ por encima de ese **o si da error**:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

Tendencias por servicio:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

Colóquelo en un widget **Number** con umbrales `0 → Red`, `0.7 → Amber`, `0.85 → Green`. La banda más alta gana, así que los puntajes bajos se mantienen rojos.

### Convertir contadores en tasas con `rate`

Las entidades de contador (✔ en el [diccionario de datos](#53-curated-metric-entities)) almacenan totales en constante aumento. `rate(value)` los convierte a valores por segundo. Necesita `TIMESERIES` y debe ser el único agregado:

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host, device TIMESERIES AUTO LIMIT 10
```

> **Advertencia**
>
> Use siempre la faceta `rate()` por `host` (y por `device` para los discos) o filtre a un host. Sin eso, las lecturas de diferentes hosts se mezclan y el resultado no tiene sentido.

Para obtener una tasa de **request** de los seguimientos, cuente las solicitudes en bloques fijos en su lugar. Esto proporciona solicitudes por minuto:

```sql
SELECT count(*) AS requestsPerMinute FROM Transaction TIMESERIES 1 minute
```

### Agrupar valores en casos con `FILTER`

`FACET CASES` de NRQL agrupa filas en bloques con nombre. En SQL, escriba una columna `FILTER` por cada caso. Estas son las clases de estado por servicio:

```sql
SELECT count(*) FILTER (WHERE http.statusCode < 400) AS ok, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS clientErrors, count(*) FILTER (WHERE http.statusCode >= 500) AS serverErrors FROM Transaction FACET service LIMIT 20
```

La misma idea funciona con el tiempo, dando una línea por caso:

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/graphql/%') AS graphql FROM Transaction TIMESERIES AUTO
```

### Creación de un histograma de latencia

SQL no tiene la función `histogram()`. Compile los contenedores usted mismo con `FILTER` y procese el resultado como un widget de **Barra** o **Tabla**:

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS 'le 100ms', count(*) FILTER (WHERE duration > 100 AND duration <= 250) AS '100-250ms', count(*) FILTER (WHERE duration > 250 AND duration <= 500) AS '250-500ms', count(*) FILTER (WHERE duration > 500 AND duration <= 1000) AS '0.5-1s', count(*) FILTER (WHERE duration > 1000 AND duration <= 2000) AS '1-2s', count(*) FILTER (WHERE duration > 2000) AS 'over 2s' FROM Transaction
```

Agregue `FACET service` para comparar las distribuciones de varios servicios uno al lado del otro.

### Generación de mosaicos y secciones de KPI

Un buen panel comienza con una fila de mosaicos de KPI y, a continuación, agrupa gráficos de detalles en secciones.

1. Agregue cuatro widgets de **Number**, cada uno con 3 columnas de ancho:

   | Título | Consulta | Unidad | Umbrales |
   | -------- | --------------------------------------------------------------------------- | ---- | ---------------------------------------- |
   | Solicitudes | `SELECT count(*) AS requests FROM Transaction` |      |                                          |
   | Error % | `SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction` | `%` | `0 → Green`, `1 → Amber`, `5 → Red` |
   | p95 | `SELECT percentile(duration, 95) AS p95 FROM Transaction` | `ms` | `0 → Green`, `800 → Amber`, `2000 → Red` |
   | Apdex | `SELECT apdex(duration, 500) AS apdex FROM Transaction` |      | `0 → Red`, `0.7 → Amber`, `0.85 → Green` |

2. Agregue un widget de **Texto**, marque _Convertir esto en encabezado de sección_ y denomínelo **LATENCIA**. Agregue gráficos de latencia debajo.
3. Agregue otro encabezado de sección, **ERRORES**, seguido de gráficos de errores. Contraer una sección con sus comillas angulares para centrarse en la otra.
4. Use **tabs** para distintas audiencias, por ejemplo _Información general_, _AEM_, _Infraestructura_. Solo la pestaña visible ejecuta sus consultas.

### Combinar consultas con widgets de fórmula

SQL no puede realizar operaciones matemáticas en `SELECT` y no tiene subconsultas. **Los widgets de fórmula** llenan ese vacío: ejecutan de 2 a 6 consultas con nombre y combinan sus resultados con `+ - * / ( )`.

**Ejemplo: solicitudes por host a lo largo del tiempo** (Fórmula, Línea)

1. **Agregar widget → fórmula**.
2. Asigne un nombre a la primera consulta `requests` y a la segunda `hosts`.
3. Escriba las consultas y la fórmula y, a continuación, haga clic en **Ejecutar fórmula**.

Consulta `requests`:

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

Consulta `hosts`:

```sql
SELECT uniqueCount(host) AS v FROM Transaction TIMESERIES AUTO
```

Fórmula:

```text
requests / hosts
```

**Ejemplo: porcentaje de tiempo empleado en la base de datos** (Fórmula, Número, unidad `%`)

Consulta `db`:

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != ''
```

Consulta `server`:

```sql
SELECT sum(duration) AS v FROM Transaction
```

Fórmula:

```text
(db / server) * 100
```

Reglas para consultas de fórmula:

| Regla | Detalles |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| Nombres | Letras, dígitos y `_`, que comienzan con una letra o `_`; hasta 40 caracteres; únicos |
| Expresión | `+ - * / ( )`, números y nombres de consulta; hasta 500 caracteres |
| Misma forma | Todas las consultas son totales sin formato (número), **o** y todas son `TIMESERIES` con **el mismo tamaño de contenedor** (línea/área/barra) |
| Una métrica cada uno | Cada consulta devuelve exactamente un agregado y no `FACET` |
| Faltan datos | Un valor que falta o una división por cero produce un punto vacío, nunca `Infinity` ni `NaN` |

>[!TIP]
>
> Use `TIMESERIES AUTO` en cada consulta de una fórmula de serie temporal. Dado que las consultas comparten una ventana, siempre se resuelven en el mismo tamaño de bloque.

### Hacer que los paneles se puedan reutilizar con variables

Las variables permiten que un tablero sirva para muchos servicios, niveles o umbrales. Cada variable es un selector en la barra de variables. Puede hacer referencia a él como `{{name}}` en los títulos de los widgets y widgets de texto (y, una vez corregida la limitación siguiente, en el SQL de los widgets).

**Crear una variable** (_org y superadministradores_): haga clic en **+ variable**.

| Campo | Notas |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Nombre** | Letras, dígitos y `_`, comenzando por una letra o `_`. Debe ser único. Se usó como `{{name}}`. |
| **Etiqueta** | Lo que muestra el selector (opcional). |
| **Tipo** | **Lista personalizada** (un valor por línea), **De una consulta** (una consulta SQL con un `FACET`; sus distintos valores se convierten en opciones) o **Cuadro de texto** (texto libre). |
| **Permitir la selección de varios valores** | Se expande a una lista separada por comas. Úselo con `IN (…)`. |
| **Los valores son números** | Inserta valores sin comillas (`500` en lugar de `'500'`). |

**Funcionamiento de la sustitución**

| Selección | `{{svc}}` se convierte en |
| ---------------------------------- | ------------------------------------------------------- |
| Cadena única `checkout` | `'checkout'` |
| Varios valores `checkout`, `cart` | `'checkout', 'cart'` |
| Número `500` | `500` |
| Valor que contiene `'` | Se escapó como `''` |
| Nada seleccionado | El widget espera y muestra _Seleccione un valor para{{svc}}_ |

```text
WHERE service = {{svc}}            single select
WHERE service IN ({{svc}})         multiple select
WHERE duration > {{slowMs}}        numeric text box
```

Para una variable controlada por consulta, **agregue siempre un`LIMIT`**. Una consulta con facetas devuelve solo los 10 valores principales de forma predeterminada. Esta lista incluye hasta 200 servicios:

```sql
SELECT count(*) FROM Transaction FACET service LIMIT 200
```

Las opciones se actualizan cuando cambia el inquilino o el selector de hora y se ordenan alfabéticamente. Cuando la selección actual ya no es válida, se selecciona automáticamente el primer valor (selección única) o todos los valores (selección múltiple). La consulta de una variable no puede hacer referencia a otras variables.

> **Advertencia**
>
> **Límite de compilación actual.** La sustitución se produce en la cuadrícula del panel, pero la vista previa de **Agregar widget** y la **comprobación de guardado del lado del servidor** reciben el token `{{name}}` sin procesar y lo rechazan con `Unexpected character '{'`. En la práctica:
>
> - `{{variables}}` en **títulos de widgets** y **widgets de texto** funcionan de extremo a extremo.
> - `{{variables}}` dentro de **widget SQL** se representa correctamente después de **Importar**, pero el tablero no se puede **guardar** mientras existan estos widgets.
>
> Hasta que esto se arregle, genere widgets por contexto con filtros literales (por ejemplo `WHERE service = 'checkout'`) y use **Duplicate** para clonarlos.

### Ponlo todo junto

Ahora tiene todos los bloques de creación utilizados en [Planes del panel](../examples/dashboard-blueprint.md): mosaicos de KPI con umbrales, secciones, percentiles, Apdex, tasas, columnas de mayúsculas y minúsculas, histogramas, fórmulas y variables. Los modelos muestran cómo combinarlos en paneles completos de señales doradas, clasificación de errores, AEM, infraestructura y JVM.
