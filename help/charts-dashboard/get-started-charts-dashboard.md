---
source-git-commit: 05ed98c07602488843c52cd262af2d8cc204fef1
workflow-type: tm+mt
source-wordcount: '292'
ht-degree: 3%
---
## Gráficos y paneles: visualización de los datos {#charts-and-dashboards-visualizing-your-data}

Las consultas encuentran la información que necesita. Los gráficos y paneles hacen que esa información sea útil para su organización. Este documento muestra cómo los resultados de SQL se convierten en gráficos, y cómo crear, organizar y compartir paneles.

### Gráficos

Cada resultado SQL se puede representar como uno de los seis tipos de gráficos.

| Tipo de gráfico | Úselo para | Forma Consulta |
| --------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Línea** | Tendencias con el tiempo; una línea por valor de faceta | Agregar + `TIMESERIES` |
| **Área** | Volumen con el tiempo, como rendimiento por servicio | Agregar + `TIMESERIES` |
| **Barra** | Comparar categorías; series temporales dispersas | Agregar + `FACET` (± `TIMESERIES`) |
| **Circular / Anillo** | Participación de un todo (combinación de código de estado, tráfico por nivel) | Un agregado + `FACET`, pocos valores |
| **Número** | Un solo KPI, con umbrales de color opcionales | Vallas (solo agregados) |
| **Tabla** | Varios agregados por grupo o listas de eventos | Varios agregados + `FACET` o atributos sin procesar |

Una vez que un widget se encuentra en un tablero, puede hacer lo siguiente:

- **Editar** su consulta, título, tipo de gráfico y unidad (icono de lápiz).
- **Duplicar** la prueba para crear una variante, como el mismo gráfico para otro nivel.
- **Descargar CSV** de las filas actuales o **PNG** con una resolución de 2×.
- **Arrástrelo** por el controlador de agarre y **cambie su tamaño** desde la esquina inferior derecha.

### Paneles de control

Los gráficos funcionan bien por sí solos, pero los paneles los unen. Un tablero es una colección con nombre y ámbito de inquilino de **tabulaciones**. Cada ficha es una cuadrícula de 12 columnas de **widgets**.

| Concepto | Significado |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Panel de control** | Una colección con nombre de 1-40 pestañas, almacenadas del lado del servidor para el inquilino seleccionado. |
| **Pestaña** | Una cuadrícula de widgets (hasta 200). Cambiarle el nombre con el lápiz, eliminarlo con ×, agregar uno con **+**. Debe quedar al menos una pestaña. |
| **Widget** | Una **consulta única**, una **fórmula** o **texto** (una nota o un encabezado de sección contraíble). |
| **Variable** | Un selector sobre la cuadrícula. Su valor se inserta como `{{name}}`. |
| **Selector de tiempo** | Ventana de cada widget del tablero. |
