---
title: Preguntas más frecuentes
description: Preguntas comunes y puntos de partida de investigación para Observability Insights en AEM Managed Services.
feature: Operations
role: Admin
source-git-commit: 3e9cd3734665dc06a4b90902b229dffb8f5421df
workflow-type: tm+mt
source-wordcount: '216'
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

## ¿Cómo debo entender un gráfico o una métrica específicos? {#understand-graph-or-metric}

Utilice las páginas de referencia del tablero para ver descripciones panel por panel, nombres de métricas, unidades y capturas de pantalla:

- [Referencia de panel de APM](../reference/apm-dashboard-reference.md)
- [Referencia del tablero de infraestructura](../reference/infrastructure-dashboard-reference.md)

## ¿Qué datos recopila realmente Observability Insights? {#what-data-is-collected}

Consulte [Cobertura, entornos y retención de datos](../get-started/coverage-and-data.md) para supervisar el ámbito, la representación de la aplicación, los períodos de retención y las implicaciones operativas.
