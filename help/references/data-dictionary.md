---
title: Diccionario de datos
description: Haga referencia a las entidades y atributos disponibles para las consultas SQL de Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '598'
ht-degree: 21%
---

## Diccionario de datos {#data-dictionary}

Esta es cada entidad y atributo que puede consultar. El catálogo en vivo, con descripciones, también está en el panel **Guía de consulta** en Explorar y en el cuadro de diálogo Agregar widget.

### Seguimiento de entidades

| Entidad | Contiene | Ámbito fijo | Resúmenes |
| ------------- | --------------------------------------------------------- | ---------------------------- | ------------------------------------------ |
| `Transaction` | Solicitudes de servidor entrantes | `kind = Server` | 1m / 5m / 1h ([Rendimiento](#performance)) |
| `AppRequest` | Solicitudes de consumidor de mensajes del servidor entrante **y** | `kind IN (Server, Consumer)` | — |
| `Span` | Cada intervalo (servidor, cliente, productor, consumidor, interno) | — | — |

<details open>
<summary><strong>Atributos de seguimiento</strong> (idénticos en las tres entidades)</summary>

| Atributo | Tipo | Descripción |
| --------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `timestamp` | timestamp | Hora de inicio del intervalo |
| `traceId`, `spanId`, `parentSpanId` | cadena | Identificadores (los espacios raíz tienen `parentSpanId = ''`) |
| `name` | cadena | Nombre de operación o transacción |
| `kind` | cadena | `Server`, `Client`, `Producer`, `Consumer`, `Internal` |
| `service` | cadena | Emitiendo nombre de servicio |
| `serviceKey` | cadena | Identidad de servicio efectiva (vuelve a `appName` cuando `service` está en blanco o es un elemento principal no válido) |
| `appName` | cadena | Nombre de aplicación de Observability Insights |
| `appInstance` | cadena | Identidad por instancia: prefijo de host de autor/publicación, nombre de aplicación y, por último, servicio |
| `hostTierPrefix` | cadena | Nombre de host truncado en su sufijo de rol `author` / `publish` / `dispatcher` |
| `duration` | número (ms) | Duración del intervalo en milisegundos |
| `status` | cadena | `Ok`, `Error`, `Unset` |
| `statusMessage` | cadena | Descripción del estado |
| `host` | cadena | Nombre de host |
| `environment` | cadena | Entorno de implementación |
| `region` | cadena | Región de nube |
| `cloud.provider` | cadena | AWS/Azure/GCP |
| `http.method` | cadena | `GET`, `POST`, ... |
| `http.statusCode` | número | Estado de respuesta HTTP |
| `http.route` | cadena | Plantilla de ruta normalizada |
| `url` | cadena | URL completa |
| `urlPath` | cadena | Parte de ruta de acceso de la dirección URL (`/content/site/en.html`) |
| `externalDomain` | cadena | Dominio de una dirección URL saliente (se utiliza en los intervalos del cliente) |
| `peer.service` | cadena | Servicio descendente llamado |
| `db.system`, `db.operation`, `db.namespace` | cadena | Tecnología, operación y esquema de bases de datos |
| `messaging.system`, `messaging.destination` | cadena | Kafka, RabbitMQ, ... y tema o cola |
| `rpc.system`, `rpc.method` | cadena | gRPC y similar |
| `error.type` | cadena | Excepción o tipo de error |
| `sling.script`, `sling.servlet`, `sling.requestUri` | cadena | Detalles de AEM Sling |
| `jsp.path` | cadena | AEM JSP ejecutado |
| `jcr.operation`, `jcr.path` | cadena | Ruta y operación del repositorio JCR de AEM |
| `graphql.operation` | cadena | Nombre de operación de GraphQL |

</details>

### Entidades de métricas genéricas

| Entidad | Contiene |
| ------------- | -------------------------------------------------- |
| `MetricGauge` | Cada muestra de medición de OpenTelemetry en un momento dado |
| `MetricSum` | Cada muestra de suma de OpenTelemetry acumulativa o delta |

Úselos para **discovery** o para métricas que no tengan una entidad depurada: `WHERE metric = 'jvm.gc.duration'`.

<details open>
<summary><strong>Atributos de métrica</strong> (compartidos por todas las entidades de métrica)</summary>

| Atributo | Tipo | Descripción |
| ----------------------------------------------------------------------- | --------- | ------------------------------------------------------- |
| `timestamp` | timestamp | Tiempo de muestra |
| `metric` | cadena | Nombre de métrica de OpenTelemetry |
| `value` | número | Valor de muestra (convertido en entidades depuradas; consulte a continuación) |
| `unit` | cadena | Unidad métrica |
| `service`, `appName`, `host`, `environment`, `region`, `cloud.provider` | cadena | Identidad del recurso |
| `state` | cadena | Modo CPU o estado de memoria |
| `cpu`, `device`, `direction`, `mountpoint` | cadena | Por CPU, por dispositivo, dirección de E/S, sistema de archivos |
| `pool` | cadena | Grupo de memoria JVM o grupo de conexión |
| `topic` | cadena | Tema de Kafka |
| `process.pid`, `process.name`, `process.owner` | cadena | Identidad del proceso |

</details>

### Entidades de métricas depuradas

Cada entidad depurada ajusta una métrica de OpenTelemetry con el filtro correcto y la conversión de unidad, por lo que `value` ya tiene sentido. **Contador** entidades (✔) contienen totales acumulativos. Consúltelos con `rate(value)` y `TIMESERIES`.

| Entidad | `value` significa | Unidad | Contador | Notas |
| ------------------------------------------------------------------------------ | ----------------------------------- | --------- | ------- | -------------------------------------------- |
| `CpuUtilization` | CPU ocupado | % |         | Resumen de 1 minuto de `average(value)` por host |
| `CpuUserPercent` / `CpuSystemPercent` / `CpuIowaitPercent` / `CpuStealPercent` | Tiempo de CPU por modo | % |         |                                              |
| `MemoryUtilization` | Memoria utilizada | % |         | Resumen de 1 minuto para `average(value)` |
| `LoadAverage1m` / `LoadAverage5m` / `LoadAverage15m` | Cargar promedio | — |         |                                              |
| `LoadAverageWithFallback` | Carga de 5m, volviendo a caer a 1m | — |         |                                              |
| `DiskUtilization` | Disco utilizado por montaje | % |         | Faceta por `mountpoint` |
| `DiskFreeBytes` | Sistema de archivos raíz libre | bytes |         |                                              |
| `DiskReadRate` / `DiskWriteRate` | Rendimiento de disco | bytes/s | ✔ |                                              |
| `DiskReadOpsRate` / `DiskWriteOpsRate` | IOPS de disco | ops/s | ✔ |                                              |
| `NetworkReceiveRate` / `NetworkTransmitRate` | rendimiento eth0 | bytes/s | ✔ |                                              |
| `NetworkReceiveRateAllDevices` / `NetworkTransmitRateAllDevices` | Todos los dispositivos | bytes/s | ✔ |                                              |
| `NetworkIoRate` | Todas las direcciones y dispositivos combinados | bytes/s | ✔ |                                              |
| `NetworkReceiveErrorRate` / `NetworkTransmitErrorRate` | Errores de red | errores/s | ✔ |                                              |
| `NetworkReceivePacketRate` / `NetworkTransmitPacketRate` | Paquetes | paquetes/s | ✔ |                                              |
| `ProcessCpuUtilization` | CPU por proceso | % |         | Faceta por `process.name` |
| `ProcessMemoryUsage` | Memoria por proceso | MB |         |                                              |
| `JvmHeapUsed` / `JvmHeapLimit` | Montón de JVM | MB |         | Notificado por grupo de memoria |
| `JvmThreadCount` | Hilos de JVM | — |         |                                              |
| `JvmCpuRecentUtilization` | CPU de proceso JVM | % |         | Resumen de 1 minuto |
| `DispatcherRequestRate` | Solicitudes HTTP del distribuidor de AEM | req/s | ✔ |                                              |

>[!NOTE]
>
> Las métricas de memoria de JVM se registran **por grupo de memoria**, por lo que `average(value)` en `JvmHeapUsed` / `JvmHeapLimit` promedios entre grupos. Use `FACET host, pool` para obtener una vista exacta por grupo.

### `SpanBreakdown`: composición de transacciones agregada previamente

`SpanBreakdown` contiene la composición por minuto de intervalo secundario de cada transacción. Se calcula durante la ingesta, por lo que es barato consultarlo.

| Atributo | Tipo | Descripción |
| --------------- | --------- | ---------------------------------------------------------------------- |
| `timestamp` | timestamp | Depósito de un minuto |
| `service` | cadena | Servicio |
| `rootOperation` | cadena | Nombre de transacción principal |
| `segment` | cadena | Nombre de span secundario |
| `callCount` | número | Llamadas de este segmento en el bloque |
| `avgDurationMs` | número | Duración media del segmento en el bloque (establezca la unidad en `ms` manualmente) |
