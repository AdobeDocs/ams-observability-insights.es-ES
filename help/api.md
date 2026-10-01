---
title: API pública de Observability Insights
description: La API pública de Observability Insights le permite extraer sus propios datos de observabilidad (solicitar descripciones generales, catálogos de servicios, seguimientos y métricas) directamente en sus propias herramientas, scripts y paneles.
source-git-commit: f9361af48539ab50bcde6265963247714621c0ce
workflow-type: tm+mt
source-wordcount: '1135'
ht-degree: 7%
---

# API pública de Observability Insights

La API pública de Observability Insights le permite extraer sus propios datos de observabilidad (solicitar descripciones generales, catálogos de servicios, seguimientos y métricas) directamente en sus propias herramientas, scripts y paneles.

- **URL base de API (API_BASE_URL):** `https://insights.adobecqms.net/`
- **Formato:** JSON a través de HTTPS
- **Autenticación:** clave de API (token de portador)

> Reemplace `{{API_BASE_URL}}` en todo este documento con el host de API de su instancia de Observability Insights, por ejemplo `https://insights.adobecqms.net/`.

## &#x200B;1. Obtención de una clave de API

Las claves de API son credenciales personales vinculadas a su cuenta y con un ámbito de una sola organización. Una clave solo puede leer datos de inquilinos que pertenecen a la organización para la que se creó; nunca puede ver los datos de otra organización.

### Generar una clave

1. Inicie sesión en el [panel de Observability Insights](https://insights.adobecqms.net/).
2. Abra el menú de perfil (parte superior derecha) → **Claves de API**.
   ![Menú de claves API](v2-assets/api-key.png)
3. En la ficha **Claves de API**, haga clic en **Generar clave**.
   ![Generar clave API](v2-assets/api-key-gen.png)
4. Asígnele un nombre descriptivo (p. ej. `CI pipeline`, `Grafana datasource`), elija la organización a la que debe establecerse el ámbito y, opcionalmente, establezca una fecha de caducidad.
5. Haga clic en **Generar clave**. La clave se muestra **una vez** con el formato:

   ```
   synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
   ```

   **Cópielo inmediatamente y guárdelo en un lugar seguro** (un administrador de secretos, almacén secreto de CI, etc.) — el tablero no puede mostrárselo de nuevo. Si lo pierde, revoque y genere uno nuevo.

### Administrar claves existentes

La sección Claves de API enumera todas las claves que ha creado, incluida su organización, fecha de creación, caducidad y marca de tiempo de la última utilización. Haga clic en el icono de papelera que hay junto a una clave para **revocarla**; la revocación es inmediata y no se puede deshacer.

### Seguridad de claves

- Trate una clave API exactamente como una contraseña. Cualquier persona con la clave puede leer todos los datos de observabilidad de cada inquilino en la organización a la que está vinculado, hasta que se revoque o caduque.
- Nunca confirme una clave para el control de código fuente ni la comparta en texto sin formato (chat, correo electrónico, tickets).
- Gire las claves periódicamente y revoque cualquier clave que ya no esté en uso.
- Si una clave está en peligro, revocarla inmediatamente de **Configuración de organización → Claves de API** y generar un reemplazo.


## &#x200B;2. Autenticando solicitudes

Cada solicitud a la API pública debe incluir su clave en el encabezado `Authorization`:

```
Authorization: Bearer synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
```

Las solicitudes sin una clave válida o con una clave caducada o revocada reciben `401 Unauthorized`. Se aceptan **no** los inicios de sesión (cookies/tokens de explorador) en esta API

## &#x200B;3. Conceptos básicos

### Inquilinos

Cada extremo requiere un parámetro de consulta `tenant_id` que identifique los datos del inquilino que se van a leer. Una clave solo puede consultar inquilinos que pertenezcan a la organización para la que se creó; si se solicita un inquilino fuera de esa organización, se devuelve `403 Forbidden`. No hay modo &quot;todos los inquilinos&quot; en esta API; pase siempre un `tenant_id` específico.

¿No está seguro de qué valores de `tenant_id` puede utilizar su clave? Llamar a [`GET /public/v1/tenants`](#get-publicv1tenants): enumera exactamente los inquilinos cuya clave está autorizada a consultar.

### Intervalos de tiempo

Los extremos que aceptan parámetros de `from` / `to` toman marcas de tiempo Unix (segundos), marcas de tiempo de milisegundos o cadenas de fecha y hora ISO 8601, por ejemplo:

```
from=1735689600
from=2025-01-01T00:00:00Z
```

Si se omite, la mayoría de los extremos tienen de forma predeterminada una ventana móvil reciente (vea cada extremo más abajo).

### Límites de velocidad

Las solicitudes están limitadas a una tasa por clave de API. Si supera el límite, recibirá lo siguiente:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{ "error": "Too Many Requests", "message": "Rate limit of 300 requests/60s exceeded" }
```

Atrás y vuelva a intentarlo después del número de segundos en el encabezado `Retry-After`. Póngase en contacto con el servicio de asistencia si su caso de uso necesita un límite más alto.

### Errores

Los errores se devuelven como JSON con un campo `error` y, normalmente, un `message` legible en lenguaje natural:

```json
{ "error": "Bad Request", "message": "tenant_id is required" }
```

| Estado | Significado |
| ------------------------- | ------------------------------------------------------------------ |
| `400 Bad Request` | Falta el parámetro o no es válido (por ejemplo, no hay `tenant_id`, intervalo de tiempo incorrecto) |
| `401 Unauthorized` | Clave de API faltante, no válida, caducada o revocada |
| `403 Forbidden` | La clave no está autorizada para el usuario solicitado |
| `429 Too Many Requests` | Se sobrepasó el límite de tarifa — ver `Retry-After` |
| `502 Bad Gateway` | Error en la consulta ascendente: seguro para reintentar |
| `503 Service Unavailable` | Servidor de datos temporalmente no disponible |

## &#x200B;4. Puntos finales

### `GET /public/v1/tenants`

Enumera los ID de inquilino que su clave está autorizada a consultar. Llame primero a esto — todos los demás extremos requieren uno de estos valores como `tenant_id`.

```bash
curl -s "{{API_BASE_URL}}/public/v1/tenants" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{ "tenants": ["tenant1", "tenant2"] }
```

### `GET /public/v1/overview`

KPI de estado de alto nivel para un inquilino durante un período determinado: volumen de solicitudes, tasa de errores y percentiles de latencia.

| Parámetro | Requerido | Descripción |
| ------------ | -------- | ------------------------------------------------------------------------- |
| `tenant_id` | Sí | Inquilino a consultar |
| `from`, `to` | No | Intervalo de tiempo (consulte [Intervalos de tiempo](#time-ranges)) |
| `minutes` | No | Abreviatura de &quot;últimos N minutos&quot; si no se dan `from`/`to` (predeterminado `15`) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/overview?tenant_id=<tenant_id>&minutes=30" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "total_spans": 48213,
  "errors": 112,
  "error_rate_pct": 0.23,
  "p50_ms": 34,
  "p95_ms": 210,
  "p99_ms": 480,
  "service_count": 12,
  "trace_count": 9021
}
```

### `GET /public/v1/services`

Enumera los distintos nombres de servicio que informan de un inquilino.

| Parámetro | Requerido | Descripción |
| ------------ | -------- | --------------------------------------------------------------------- |
| `tenant_id` | Sí | Inquilino a consultar |
| `from`, `to` | No | Restringir a los servicios vistos en esta ventana; el valor predeterminado son los últimos 7 días |

```bash
curl -s "{{API_BASE_URL}}/public/v1/services?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "services": ["checkout-api", "payments-worker", "web-frontend"]
}
```

### `GET /public/v1/traces`

Busca los seguimientos recientes de un inquilino, con filtros opcionales.

| Parámetro | Requerido | Descripción |
| ----------------- | -------- | ------------------------------------------------- |
| `tenant_id` | Sí | Inquilino a consultar |
| `from`, `to` | No | Intervalo de tiempo; el valor predeterminado es las últimas 24 horas |
| `limit` | No | Máximo de filas para devolver (1-200, valor predeterminado 100) |
| `offset` | No | Desplazamiento de paginación (predeterminado 0) |
| `service` | No | Filtrar por nombre de servicio |
| `app_name` | No | Filtrar por nombre de aplicación/instancia |
| `status` | No | Filtrar por estado de seguimiento: `ok`, `error` o `unset` |
| `search` | No | Búsqueda de texto libre en los nombres de span/operation |
| `min_duration_ms` | No | Solo los seguimientos que superen esta duración |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces?tenant_id=<tenant_id>&status=error&limit=25" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TraceId": "4bf92f3577b34da6a3ce929d0e0e4736",
      "ServiceName": "checkout-api",
      "DurationMs": 812,
      "StatusCode": "Error",
      "Timestamp": "2026-08-30T09:12:44Z"
    }
  ],
  "rows": 137,
  "limit": 25,
  "offset": 0
}
```

Use `rows` (el recuento total de coincidencias) junto con `limit`/`offset` para recorrer página a página los resultados.

### `GET /public/v1/traces/:traceId`

Devuelve la cascada de intervalo completo para un único seguimiento.

| Parámetro | Requerido | Descripción |
| ----------- | -------- | ---------------------------------------- |
| `tenant_id` | Sí | Usuario al que pertenece el seguimiento |
| `limit` | No | Número máximo de espacios que se van a devolver (1-500, valor predeterminado 500) |
| `offset` | No | Desplazamiento de paginación para trazos muy grandes |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces/4bf92f3577b34da6a3ce929d0e0e4736?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "spans": [
    {
      "SpanId": "00f067aa0ba902b7",
      "Name": "POST /checkout",
      "DurationMs": 812,
      "children": []
    }
  ],
  "totalDurationMs": 812,
  "spanCount": 14,
  "limit": 500,
  "offset": 0
}
```

### `GET /public/v1/metrics`

Devuelve puntos de datos de métricas sin procesar para un inquilino.

| Parámetro | Requerido | Descripción |
| ---------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `tenant_id` | Sí | Inquilino a consultar |
| `metric` | Uno de `metric`/`like` | Nombre exacto de la métrica |
| `like` | Uno de `metric`/`like` | Patrón SQL `LIKE` para que coincida con varios nombres de métricas |
| `type` | No | `gauge` (predeterminado) o `sum` |
| `from`, `to` | No | Intervalo de tiempo; el valor predeterminado es las últimas 24 horas |
| `service` | No | Filtrar por nombre de servicio |
| `host` | No | Filtre por nombre de host. Necesario para las métricas de host de infraestructura siguientes: sin ella, las lecturas de cada host del inquilino se mezclan |
| `attribute_key`, `attribute_value` | No | Filtrar por un atributo de métrica específico (deben usarse juntos) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=jvm.memory.used&type=gauge" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TimeUnix": "2026-08-30T09:00:00Z",
      "MetricName": "jvm.memory.used",
      "Value": 512482816,
      "ServiceName": "checkout-api",
      "host": ""
    }
  ],
  "rows": 1
}
```

#### Métricas de host de infraestructura

El mismo punto de conexión también sirve para las métricas de nivel de host que se muestran en el panel Infraestructura (CPU, memoria, promedio de carga, E/S de disco, E/S de red). Use estas combinaciones exactas de `metric` / `attribute_key` / `attribute_value`, siempre con `host`:

| Widget del panel | `metric` | `attribute_key` | `attribute_value` |
| --------------------- | ------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| CPU % | `system.cpu.utilization` | `state` | `idle` (restar de 1 para &quot;en uso&quot;) o consulta `user`/`system`/`iowait` por separado y suma |
| % de uso de memoria | `system.memory.utilization` | `state` | `used` |
| Carga media (1m) | `system.cpu.load_average.1m` | — | — |
| E/S de lectura de disco | `system.disk.io` (`type=sum`) | `direction` | `read` |
| E/S de escritura en disco | `system.disk.io` (`type=sum`) | `direction` | `write` |
| Operaciones de lectura de disco | `system.disk.operations` (`type=sum`) | `direction` | `read` |
| Operaciones de escritura en disco | `system.disk.operations` (`type=sum`) | `direction` | `write` |
| Red en | `system.network.io` (`type=sum`) | `direction` | `receive` |
| Salida de red | `system.network.io` (`type=sum`) | `direction` | `transmit` |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=system.cpu.utilization&type=gauge&attribute_key=state&attribute_value=idle&host=<host_name>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

**Importante: los valores de disco y de red son contadores sin procesar que aumentan constantemente, no tasas.** Los gráficos de &quot;bytes/s&quot; y &quot;operaciones/s&quot; del tablero se calculan tomando dos lecturas de contador consecutivas y dividiendo por el tiempo transcurrido:

```
rate = (value_at_t2 - value_at_t1) / (t2 - t1_in_seconds)
```

### `GET /public/v1/pages`

Páginas de contenido principales solicitadas (`.html`) por instancia de Dispatcher, clasificadas por recuento de solicitudes. Respaldado por la métrica `dispatcher.httpd.requests`: este extremo es específico de los registros de acceso de estilo Dispatcher/CDN de AEM, no una herramienta general de análisis de páginas.

| Parámetro | Requerido | Descripción |
| ------------ | -------- | -------------------------------------- |
| `tenant_id` | Sí | Inquilino a consultar |
| `from`, `to` | No | Intervalo de tiempo; el valor predeterminado es las últimas 24 horas |
| `limit` | No | Máximo de filas para devolver (1-500, valor predeterminado 50) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/pages?tenant_id=<tenant_id>&limit=50" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "data": [
    {
      "instance": "<instance_name>",
      "domain": "www.abc.com",
      "path": "/join-us/insights.html",
      "full_url": "https://www.abc.com/join-us/insights.html",
      "requests": 7
    }
  ],
  "rows": 1
}
```

## &#x200B;5. Qué no hace esta API

- **Sin acceso SQL sin procesar.** Todos los extremos devuelven formas de datos revisadas y creadas específicamente; no puede consultar directamente el almacén de datos subyacente.
- **No hay consultas entre inquilinos.** Cada solicitud tiene exactamente un ámbito de `tenant_id`.
- **Sin acceso de escritura.** La API pública es de solo lectura.

## &#x200B;6. Soporte

Si encuentra errores inesperados o tiene un caso de uso no cubierto por estos puntos de conexión, póngase en contacto con su ingeniero de éxito del cliente/habilitación para obtener más ayuda.
