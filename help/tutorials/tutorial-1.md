---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '562'
ht-degree: 0%
---
## Tutorial 1: Sus primeras consultas {#tutorial-1-your-first-queries}

Bienvenido a SQL. En este tutorial escribirá sus primeras consultas, controlará la ventana de tiempo, trazará tendencias, filtrará y agrupará resultados y colocará su primer gráfico en un panel. Cada paso se basa en el anterior.

**En este tutorial:** [Antes de comenzar](#before-you-begin) · [6.1 Eventos de consulta](#61-query-events-with-select-from-and-limit) · [6.2 Agregado](#62-aggregate-your-data) · [6.3 Varios agregados](#63-return-several-aggregates-at-once) · [6.4 Ventana de tiempo](#64-set-the-time-window) · [6.5 SERIES DE TIEMPO](#65-plot-trends-with-timeseries) · [6.6 DÓNDE](#66-filter-with-where) · [6.7 FACETA](#67-group-with-facet) · [6.8 Guardar en un tablero](#68-save-your-query-to-a-dashboard)

### Antes de empezar

- Necesita acceso a Observability Insights y, al menos, un inquilino de cliente que envíe seguimientos o métricas.
- Seleccione ese **cliente** en el conmutador de inquilinos. SQL no se ejecuta mientras se seleccionan **Todas las cuentas**.
- Abra **Explorar** y cambie a la ficha **SQL**. Use **Ejecutar** (⌘/Ctrl + Entrar) después de cada consulta.

### Eventos de consulta con `SELECT`, `FROM` y `LIMIT`

Cada consulta necesita dos cláusulas: `SELECT` dice lo que debe devolverse y `FROM` dice qué entidad debe leerse. Comenzar con solicitudes entrantes:

```sql
SELECT timestamp, service, name, duration FROM Transaction
```

Esta es una **consulta de evento**. Cada fila es una solicitud. Sin `LIMIT` se obtienen hasta 100 filas. Pida menos:

```sql
SELECT timestamp, service, name, duration FROM Transaction LIMIT 10
```

Las filas se ordenan por la primera columna, en sentido de bajada, de modo que al colocar `timestamp` en primer lugar, se muestran las solicitudes más recientes en la parte superior.

> [!NOTE]
> SQL no tiene `SELECT *`. La lista de atributos mantiene los resultados pequeños y legibles. El [diccionario de datos](#data-dictionary) y el completado automático en el editor muestran lo que está disponible.

### Agregar sus datos

La mayoría de los paneles no muestran eventos individuales. Ellos los resumen. Reemplazar los atributos con una **función de agregador**:

```sql
SELECT count(*) FROM Transaction
```

Ese es el número de solicitudes entrantes en la ventana. Pruebe con algunas más:

```sql
SELECT average(duration) FROM Transaction
```

```sql
SELECT max(duration) FROM Transaction
```

```sql
SELECT uniqueCount(host) FROM Transaction
```

### Devolver varios acumulados a la vez

Separe los agregados con comas y asigne a cada uno `AS` nombres:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

> [!TIP]
> Los alias se convierten en encabezados de columna y etiquetas de leyenda y son a los que se hace referencia en `ORDER BY`. Toma el hábito de nombrar cada agregado.

### Configuración de la ventana de tiempo

De manera predeterminada, una consulta cubre la **última hora**. Use `SINCE` para ver más atrás:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 1 day ago
```

Use `UNTIL` para finalizar la ventana en el pasado. Esto cuenta el tráfico de ayer, de hace 48 a 24 horas:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

Las unidades pueden ser `second`, `minute`, `hour`, `day` o `week`. La ventana puede ser de 31 días como máximo.

> [!IMPORTANT]
> En **Explorar** y en **paneles**, el **selector de tiempo** en la parte superior de la página controla la ventana y anula `SINCE`/`UNTIL`. Para ver otro período, cambie el selector. El resto de esta guía excluye a `SINCE` de las consultas de panel por ese motivo.

### Trazar tendencias con `TIMESERIES`

Agregue `TIMESERIES` para convertir un solo número en una línea a lo largo del tiempo:

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

`AUTO` elige un tamaño de contenedor de modo que el gráfico tenga aproximadamente 120 puntos (2 minutos para una ventana de 1 hora, 15 minutos para 24 horas). También puede establecer el contenedor explícitamente:

```sql
SELECT average(duration) AS avgMs FROM Transaction TIMESERIES 5 minutes
```

### Filtrar con `WHERE`

`WHERE` mantiene solamente las filas que le interesan. Contar solo las solicitudes con errores:

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

Combinar condiciones con `AND`, `OR` y paréntesis:

```sql
SELECT count(*) AS slowOrFailed FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR duration > 2000)
```

Use `LIKE` con `%` para las coincidencias parciales:

```sql
SELECT count(*) AS contentRequests FROM Transaction WHERE urlPath LIKE '/content/%'
```

> [!NOTE]
> Los valores de cadena distinguen entre mayúsculas y minúsculas y utilizan comillas simples: `'Error'`, no `"Error"` ni `'error'`. Reemplazar `checkout` por un servicio de su inquilino. El siguiente paso muestra cómo enumerarlos.

### Grupo con `FACET`

`FACET` agrupa resultados, como el de SQL `GROUP BY`. Enumere los servicios y la disponibilidad de cada uno:

```sql
SELECT count(*) AS requests FROM Transaction FACET service
```

Una consulta con facetas devuelve los 10 grupos principales de forma predeterminada. Use `LIMIT` para ver más:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs FROM Transaction FACET service LIMIT 50
```

Combine `FACET` y `TIMESERIES` para obtener una línea por grupo:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Aquí, `LIMIT 5` mantiene las cinco series más concurridas.

### Guarde la consulta en un panel

1. Abra **Paneles → Crear panel → Crear un panel personalizado** y asígnele el nombre _Mi primer panel_.
2. Haga clic en **Agregar widget**, elija **Consulta única** y cambie el conmutador a **SQL**.
3. Pegue la consulta desde el paso 6.7 y haga clic en **Ejecutar consulta**.
4. Establece **Title** en _Solicitudes del servicio_ y elige el tipo de gráfico de **Línea**.
5. Haga clic en **Guardar widget** y luego en **Guardar** en la barra superior.

Ha creado su primer tablero.
