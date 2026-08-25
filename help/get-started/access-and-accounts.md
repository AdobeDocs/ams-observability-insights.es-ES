---
title: Acceso y administración de cuentas
description: Comprenda cómo se aprovisionan las cuentas de Observability Insights, quién administra el acceso y qué nivel de control tienen los equipos de clientes.
feature: Operations
role: Admin
source-git-commit: 6526a90a017147ac3483c0b2b626b9aa903819ba
workflow-type: tm+mt
source-wordcount: '232'
ht-degree: 0%

---


# Acceso y administración de cuentas {#access-and-account-management}

Adobe aprovisiona y administra la cuenta de Observability Insights para su organización de AEM Managed Services. Los equipos de clientes utilizan Adobe IMS para iniciar sesión y recibir visibilidad de solo lectura de los datos supervisados.

## Modelo de propiedad de cuenta {#account-ownership-model}

- Adobe Managed Services posee la cuenta de Observability Insights.
- Los equipos de clientes tienen acceso de solo lectura.
- Los cambios administrativos, el aprovisionamiento y las actualizaciones de acceso se administran mediante Adobe.

## Cómo obtienen acceso los usuarios {#how-users-get-access}

El acceso a Observability Insights requiere el aprovisionamiento de Adobe IMS.

Para solicitar o actualizar el acceso:

1. Póngase en contacto con su ingeniero de éxito del cliente (Customer Success Engineer, CSE).
2. Proporcione los detalles de usuario necesarios para el aprovisionamiento de Adobe IMS.
3. Confirme que se han asignado la organización y el ámbito de acceso correctos.

Una vez completado el aprovisionamiento, inicie sesión en [insights.adobecqms.net](https://insights.adobecqms.net).

## Lo que los usuarios pueden hacer {#what-users-can-do}

Los usuarios del cliente suelen poder:

- Ver paneles de APM
- Ver paneles de infraestructura
- Inspeccionar las métricas de host y aplicación supervisadas
- Participar en investigaciones utilizando paneles y seguimientos compartidos

## Lo que los usuarios no pueden hacer {#what-users-cannot-do}

Los usuarios clientes deben suponer que el control administrativo permanece en Adobe Managed Services a menos que Adobe documente explícitamente lo contrario.

Algunos ejemplos comunes son:

- Administración de la propiedad de cuenta
- Cambio del aprovisionamiento a nivel de plataforma
- Modificación del comportamiento de instrumentación gestionada

## Información que se agregará más adelante {#information-to-add-later}

Utilice esta sección cuando estén disponibles más detalles del proceso:

- Requisitos previos de IMS de Adobe
- Tiempos de respuesta de aprovisionamiento esperados
- Contactos y rutas de escalación
- Administración del ciclo vital de usuario para iniciadores y salientes
