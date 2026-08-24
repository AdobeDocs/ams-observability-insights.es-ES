---
title: Supervise su entorno de Managed Services de AEM con  [!DNL Synoptryx]
description: 'Información general sobre la monitorización de  [!DNL Synoptryx] Adobe [!DNL Experience Manager] Managed Services: qué supervisa Adobe, cómo se configura su cuenta y cómo obtiene acceso su equipo.'
feature: Operations
role: Admin
source-git-commit: e8de2213d91e09da68a8f7014b075f81bd7f07ef
workflow-type: tm+mt
source-wordcount: '618'
ht-degree: 0%

---


# Supervise su entorno de Managed Services de AEM con [!DNL Synoptryx] {#synoptryx-monitoring}

[!DNL Synoptryx] proporciona a su equipo visibilidad sobre el rendimiento de la aplicación, el estado de la infraestructura y la experiencia del usuario final, sin necesidad de configurar una plataforma de supervisión independiente.

>[!NOTE]
>
> Hay disponible un documento de información general sobre el producto [!DNL Synoptryx] para obtener la información general completa sobre la supervisión y la observabilidad de AEM Managed Services, ideal para compartirlo con las partes interesadas o para revisarlo sin conexión.

## Información general {#overview}

[!DNL Synoptryx] es la plataforma de observabilidad de próxima generación de Adobe diseñada para proporcionar visibilidad unificada en el rendimiento de las aplicaciones, el estado de la infraestructura y la monitorización sintética. Permite la monitorización proactiva de servicios empresariales críticos a través de una experiencia única e integrada. [!DNL Synoptryx] combina la supervisión del rendimiento de las aplicaciones (APM), la supervisión de la infraestructura y la supervisión del Recorrido sintético del usuario para ayudar a identificar y resolver los problemas antes de que afecten a los usuarios finales. La plataforma ofrece un seguimiento profundo de transacciones, perspectivas de JVM, telemetría de infraestructura y diagnósticos avanzados para un análisis de causas raíz más rápido. Basado en modernas tecnologías de observabilidad, proporciona una monitorización escalable y segura en entornos empresariales complejos. [!DNL Synoptryx] ofrece retención de datos extendida, paneles enriquecidos y análisis inteligentes para admitir la excelencia operativa. La experiencia de inicio de sesión perfecta con [!DNL Adobe IMS] garantiza el acceso y el control seguros. La plataforma está diseñada para mejorar la fiabilidad del servicio, acelerar la resolución de problemas y mejorar la experiencia del cliente. Como solución de observabilidad estratégica de Adobe, [!DNL Synoptryx] proporciona una base preparada para el futuro para la supervisión, la automatización y las perspectivas operativas en entornos de servicios administrados.

[!DNL Synoptryx] se incluye con Adobe [!DNL Experience Manager] Managed Services; no se requiere ninguna plataforma de supervisión ni licencia por separado. Adobe supervisa la disponibilidad y el rendimiento de su entorno como parte de nuestra oferta estándar, y [!DNL Synoptryx] es la plataforma específica que su equipo puede utilizar para comprender el rendimiento de su aplicación de Adobe [!DNL Experience Manager] (AEM) y de su infraestructura de soporte.

Esta guía explica qué se supervisa, cómo se configura su cuenta de [!DNL Synoptryx] y cómo navegar por los paneles que utiliza para el análisis y la solución de problemas diarios.

## De un vistazo {#at-a-glance}

Como parte de AEM Managed Services, recibirá lo siguiente:

- **Cuenta [!DNL Synoptryx] dedicada**: aprovisionada y supervisada por Adobe Managed Services, con acceso de solo lectura para su equipo.
- **Supervisión profunda de transacciones de AEM**: el agente de APM [!DNL Synoptryx] rastrea transacciones significativas hasta llamadas de método (incluidos números de línea), dependencias externas y operaciones del repositorio.
- **Vista unificada de aplicaciones e infraestructura**: combine métricas de APM y de nivel de host para optimizar el rendimiento de forma integral.

## Lo que Adobe supervisa con [!DNL Synoptryx] {#what-we-monitor}

Adobe supervisa los niveles de AEM **Author** y **Publish** con el complemento Java de APM [!DNL Synoptryx]. Todos los servidores hospedados de su topología se supervisan con el agente de infraestructura [!DNL Synoptryx]. La monitorización personalizada de APM e infraestructura está habilitada tanto en entornos de Managed Services de producción como de no producción.

![Diagrama que muestra la monitorización de la infraestructura y la APM de Synoptryx en los servidores de AEM Author, Publish y alojados](assets/image6.png)

### Aplicaciones en su cuenta {#applications-in-your-account}

Su cuenta de [!DNL Synoptryx] está vinculada a una sola cuenta maestra de Adobe y puede recibir datos de varias aplicaciones, entre ellas:

- Una aplicación APM para el nivel **Author** por entorno de Managed Services de AEM
- Una aplicación APM para el nivel **Publish** por entorno de AEM Managed Services

Cada aplicación tiene su propia clave de licencia. Todas las topologías del informe de contrato de Managed Services en una cuenta de [!DNL Synoptryx]. Las métricas y los eventos de APM e infraestructura se conservarán por un período máximo de **30 días**.

## Acceso y su cuenta de {#access}

Los datos de supervisión se consolidan en una cuenta de [!DNL Synoptryx] que Adobe aprovisiona y administra. Su equipo recibe **acceso completo de solo lectura** a todas las métricas de APM e infraestructura recopiladas por los agentes. Adobe Managed Services conserva la propiedad y el control administrativo de la cuenta.

>[!NOTE]
>
> **Obtener acceso:** El acceso a [!DNL Synoptryx] requiere el aprovisionamiento de [!DNL Adobe IMS]. Su ingeniero de éxito del cliente (Customer Success Engineer, CSE) puede proporcionar y administrar el acceso de los usuarios a su organización.

Una vez que el CSE haya aprovisionado la cuenta, podrá iniciar sesión en [synoptryx.adobecqms.net](https://synoptryx.adobecqms.net).

## Siguientes pasos {#whats-next}

Continúe con los paneles de monitorización que su equipo utiliza día a día:

- [Supervisión del rendimiento de las aplicaciones (APM)](application-performance-monitoring.md): realice un seguimiento de las transacciones de AEM, analice el comportamiento de JVM e inspeccione los servicios externos.
- [Supervisión de infraestructura](infrastructure-monitoring.md): revise las métricas de sistema, red, proceso y almacenamiento en el nivel de host.

