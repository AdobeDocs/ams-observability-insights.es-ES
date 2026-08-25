---
title: Cobertura, entornos y retención de datos
description: Consulte qué monitoriza Observability Insights en AEM Managed Services, cómo se representan las aplicaciones y cuánto tiempo se retienen los datos de monitorización.
feature: Operations
role: Admin
source-git-commit: 1d54a6a398360b040221db5b2780d301722894bf
workflow-type: tm+mt
source-wordcount: '267'
ht-degree: 1%

---


# Cobertura, entornos y retención de datos {#coverage-environments-and-data-retention}

Esta página resume qué datos se recopilan en Observability Insights para AEM Managed Services y cómo se organizan esos datos.

## Monitorización de cobertura {#monitoring-coverage}

Monitores Adobe:

- Niveles de AEM Author con el complemento Java de Observability Insights de APM
- Niveles de publicación de AEM con el complemento Java de APM Observability Insights
- Servidores alojados en la topología administrada con el agente de infraestructura de Observability Insights

La monitorización personalizada de la infraestructura y la APM está habilitada tanto en entornos de Managed Services de producción como de no producción.

## Representación de las aplicaciones {#how-applications-are-represented}

Cada entorno de Managed Services de AEM suele incluir lo siguiente:

- Una aplicación de APM para Author
- Una aplicación de APM para publicación

Todas las topologías de un contrato de Managed Services informan en una cuenta de Observability Insights.

## Retención de datos {#data-retention}

Las métricas de APM, las métricas de infraestructura y los eventos relacionados se conservan por un máximo de **30 días**.

## Tablas de resumen {#summary-tables}

| Área de cobertura | Qué se supervisa |
| -------------- | ------------------------------------------ |
| APM | Aplicaciones de autor y publicación de AEM |
| Infraestructura | Todos los servidores alojados en la topología administrada |

| Elemento | Representación |
| ------------------------------ | ------------------------------------------------------------- |
| Entorno de AEM | Una aplicación APM de autor y una aplicación APM de publicación |
| Cuenta de Observability Insights | Una cuenta administrada por Adobe por ámbito de cliente de Managed Services |

| Tipo de datos | Retención |
| --------------------------------- | ------------- |
| Métricas y eventos de APM | Hasta 30 días |
| Métricas y eventos de infraestructura | Hasta 30 días |

## Qué significa esto operacionalmente {#what-this-means-operationally}

- Observability Insights es adecuado para el análisis operativo, los incidentes activos y la comparación de tendencias recientes.
- El análisis histórico más allá del período de retención debe gestionarse mediante otros procesos de creación de informes o archivo, si fuera necesario.
- Cuando investigue problemas recurrentes, capture capturas de pantalla o evidencias exportadas antes de que los datos caduquen.
