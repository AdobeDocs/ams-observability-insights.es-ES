---
title: Preguntas más frecuentes
description: Preguntas comunes y puntos de partida de investigación para Observability Insights en AEM Managed Services.
feature: Operations
role: Admin
product_v2:
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
    internal-label: Experience Manager
feature_v2:
  - id: 8a70d214-ab7b-58c1-b001-2ed2e5d6303d
    internal-label: Operations
role_v2:
  - id: c66ffd68-0f65-42bb-aa23-b4020f12e0bd
    internal-label: Admin
source-git-commit: ba4f816187e77f3f7b708942b3f14535417c2fdc
workflow-type: tm+mt
source-wordcount: '253'
ht-degree: 0%
---

# Preguntas más frecuentes {#faq}

Utilice esta página como punto de partida cuando no esté seguro de por dónde empezar o necesite una respuesta rápida durante una investigación activa.

## ¿Por qué no puedo acceder a Observability Insights? {#cannot-access-observability-insights}

Comience con [Administración de acceso y cuenta](../get-started/access-and-accounts.md). Si el aprovisionamiento está incompleto o desactualizado, póngase en contacto con su ingeniero de éxito del cliente (Customer Success Engineer, CSE) para solicitar el acceso o una actualización.

## ¿Por qué veo &quot;Permisos de carga&quot; cuando intento iniciar sesión? {#loading-permissions-error}

Esto suele indicar un problema con el aprovisionamiento de usuarios. Póngase en contacto con su ingeniero de éxito del cliente (Customer Success Engineer, CSE), que puede trabajar con los equipos relevantes para resolver el problema de acceso.

## ¿Cómo puedo determinar si un problema está relacionado con la aplicación o la infraestructura? {#application-or-infrastructure}

Comience con [Supervisión del rendimiento de la aplicación](/help/applications.md) para revisar las tasas de solicitud, las tasas de error y la latencia en Autor o Publicación. Si las señales de la aplicación están elevadas, use [Hosts](/help/hosts.md) para comprobar si la presión de recursos en el nivel de host (CPU, memoria, disco o red) explica o compone lo que está viendo.

## ¿Qué datos recopila realmente Observability Insights? {#what-data-is-collected}

Consulte [Cobertura, entornos y retención de datos](../get-started/coverage-and-data.md) para supervisar el ámbito, la representación de la aplicación, los períodos de retención y las implicaciones operativas.

## ¿Por qué puedo iniciar sesión con otros navegadores pero no con Google Chrome?

Para permitir cookies de terceros para Observability Insights e intentar iniciar sesión de nuevo:

1. Para obtener acceso a la configuración de cookies, abra `chrome://settings/cookies` en Google Chrome.
2. Para permitir cookies para Observability Insights, en **Sitios con permiso para usar cookies de terceros**, seleccione **Agregar** e introduzca `https://insights.adobecqms.net`.
3. Para comprobar si el problema está resuelto, inicie sesión en Observability Insights en una ventana de incógnito.