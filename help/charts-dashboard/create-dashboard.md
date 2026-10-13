---
source-git-commit: 05ed98c07602488843c52cd262af2d8cc204fef1
workflow-type: tm+mt
source-wordcount: '1117'
ht-degree: 1%
---
### Creación de un tablero, paso a paso

#### Paso 1: Antes de comenzar

- **Seleccione un solo cliente** en el conmutador de inquilinos. **Crear tablero** está deshabilitado mientras que **Todas las cuentas** está seleccionado. SQL no se puede ejecutar con el inquilino de demostración.
- **Roles.** Cualquier persona con acceso al tablero puede añadir, editar y guardar widgets. Crear o editar **variables** e inspeccionar la consulta de un widget está limitado a **administradores de organización** y **superadministradores**.

#### Paso 2: Crear el tablero

1. Abra **Paneles** en el panel de navegación izquierdo.
2. Haga clic en **Crear tablero**.
3. Elija un punto de partida:

   | Opción | Lo que obtienes |
   | ---------------------------- | -------------------------------------------------------------------------------------------------------- |
   | **Crear un tablero personalizado** | Un tablero vacío con una pestaña. |
   | **Conceptos básicos de CPU** | Alojar y procesar CPU (usuario/sistema/iowait/robo), rendimiento, carga, Apdex. |
   | **Información general de todo en uno** | KPI (volumen, porcentaje de error, Apdex, p95), pasos de error/tráfico, tablas de CPU/memoria, subprocesos JVM, operaciones lentas. |
   | **Transacciones Principales** | Apdex, p95 por operación, carga por host/CPU, transacciones más lentas con ID de seguimiento, desglose de errores HTTP. |

4. Escriba un **nombre** y confirme. Se abre el lienzo del panel.

#### Paso 3: Conozca el lienzo

| Área | Controles |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Barra superior** | **Agregar widget**, **Guía** (hoja de trucos en la aplicación), **Importar**, **Exportar**, **Guardar**, selector de tiempo y actualización |
| **Barra variable** | Distintivo de inquilino, un selector por variable, **+ variable** (administradores) |
| **Barra de fichas** | Cambiar fichas; cambiar nombre (lápiz), eliminar (×), agregar (+) |
| **Cuadrícula** | 12 columnas. Arrastre los widgets por el control y cambie su tamaño desde la esquina (mínimo 2×2). |
| **Encabezado de widget** (al pasar el ratón por encima) | Descargar CSV, Descargar PNG, Editar, Duplicar, Eliminar, controlador de arrastre |

&#x200B;> [!IMPORTANT]
> **El guardado es explícito.** Los cambios de diseño, los nuevos widgets, las pestañas y las variables permanecerán en el explorador hasta que hagas clic en **Guardar**. El botón muestra _Guardando... → guardado_ o _Error al guardar_ (consulte [Solución de problemas](#troubleshooting)).

#### Paso 4: Añadir un widget

Haga clic en **Agregar widget**. El cuadro de diálogo tiene tres modos: **Consulta única**, **Fórmula** y **Texto**. También tiene su propio selector de tiempo, por lo que puede previsualizar el widget en cualquier ventana.

**Opción A: el generador (sin código)**

1. _(Opcional)_ **Comience con un asistente**: elija uno de los 57 ajustes preestablecidos, agrupados como Golden signals, APM, AEM, Transactions, Traces &amp; Spans, Infrastructure, Publishers, JVM &amp; runtime y Data discovery. Un ajuste preestablecido rellena el título, la consulta, el tipo de gráfico y la unidad.
2. **Mostrar**: elige `Count`, `Average`, `Percentile` (y su nivel), `Sum`, `Min`, `Max` o `Unique count`, y después un atributo.
3. **de**: elija la entidad.
4. **Filtro**: agregue filas de `attribute operator value` (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`). Todas las filas se combinan con `AND`.
5. **Agrupar por**: elige un atributo de faceta y **N** superior para mantener solamente las series más concurridas.
6. **Con el tiempo**: activa `TIMESERIES`, con un tamaño de bloque de **cada N minutos**.
7. Compruebe el SQL generado en el formulario y haga clic en **Ejecutar consulta**.

**Opción B: el editor SQL**

1. Cambie el conmutador de **Builder** a **SQL**. La edición de un widget existente siempre se abre en SQL.
2. Escriba o pegue una consulta. El editor autocompleta desde el catálogo en directo.
3. Utilice el panel **Guía de consulta** para examinar entidades, atributos y ejemplos.
4. Haga clic en **Ejecutar consulta**. Verá que la consulta _`<Entity>`es válida_, una vista previa en vivo y un resumen (`N rows · N time points · N groups`). Aparece una advertencia _Serie dispersa_ cuando hay menos de tres puntos de tiempo.

&#x200B;> [!TIP]
> El Generador de consultas abarca las consultas más comunes. Cambiar a SQL para `apdex`, `percentage`, `rate`, `FILTER`, varios agregados, `OR`, `IN`, varias facetas o `ORDER BY`.

#### Paso 5: Configurar el aspecto

| Campo | Notas |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Título** | Hasta 200 caracteres. Puede contener `{{variable}}` tokens. |
| **Tipo de gráfico** | `Line`, `Area`, `Bar`, `Pie / Donut`, `Number`, `Table`. Se recomienda un tipo de gráfico después de cada ejecución hasta que elija uno. |
| **Unidad** | Opcional; hasta 40 caracteres (`ms`, `%`, `req/s`, `MB`, `bytes`). Déjela vacía para utilizar la unidad deducida (por ejemplo `ms` para `duration`, `%` para `CpuUtilization`). |
| **Umbrales de color** | Solo widgets numéricos. Hasta 10 bandas con el formato _en o por encima de `value` → Verde / Ámbar / Rojo_. La banda más alta gana. |

#### Paso 6: Guardar el widget y luego el tablero

**Guardar widget** solo se habilita después de que una ejecución devuelva datos. Los nuevos widgets tienen 6 columnas × 3 filas y se empaquetan dos por fila. Organícelos y luego haga clic en **Guardar** en la barra superior.

### Elija el gráfico correcto

| Forma Consulta | Mejor gráfico | ¿Por qué? |
| ------------------------------------------------- | ------------------- | ---------------------------------------------- |
| Un agregado, no `FACET`, no `TIMESERIES` | **Número** | Un solo mosaico de KPI; agregue umbrales. |
| Agregar + `TIMESERIES` | **Línea** / **Área** | Una tendencia; una serie por valor de faceta. |
| Agregar + `FACET` (pocos valores) | **Barra** / **Circular** | Compara categorías; circular para compartir un todo. |
| Varios agregados + `FACET` | **Tabla** | Cada acumulado se convierte en una columna. |
| Atributos sin procesar | **Tabla** | Una lista de eventos. |
| Agregar + `FACET` + `TIMESERIES` en datos dispersos | **Barra** | Lee mejor que las líneas espaciadas. |

Cuando la fila única de un widget de número lleva un valor de texto (por ejemplo, `latest(host)`), el widget lo muestra como un subtítulo.

### Widgets de texto: notas y encabezados de sección

**Agregar widget → texto**:

- **Nota**: texto sin formato (se conservan hasta 4.000 caracteres y líneas en blanco). Utilícelo para vínculos de runbook, propietarios y definiciones de SLO. Se procesaron `{{variables}}`.
- **Encabezado de sección**: marca _Convertir esto en un encabezado de sección_. Abarca todo el ancho y sus comillas angulares contraen todos los widgets debajo de él, hasta el encabezado de la siguiente sección. Úselo para agrupaciones como **PUBLISH**, **AUTHOR** y **DISPATCHER**.

&#x200B;> [!TIP]
> Los títulos de widgets que terminan en `- PUBLISH` o `- AUTHOR` obtienen una sutil señal de color, lo que facilita la búsqueda por niveles de los paneles de AEM.

### El selector de tiempo

- Establece la ventana para **cada** widget y para las opciones de variables basadas en consultas.
- Anula `SINCE` / `UNTIL` en las consultas de widgets.
- La ventana máxima es de **31 días**.
- `TIMESERIES AUTO` adapta su tamaño de contenedor a la ventana.
- El subtítulo de cada widget muestra la ventana aplicada, por ejemplo _Desde hace una hora_.
- Los resultados se almacenan en caché durante 30 segundos. Utilice el botón de actualización para forzar una nueva ejecución.

### Variables

Las variables permiten que un tablero sirva a muchos contextos, como un servicio, entorno o umbral a la vez. Se tratan en profundidad en [Tutorial 3](#87-make-dashboards-reusable-with-variables).

&#x200B;> [!WARNING]
> **Limitación conocida en la versión actual:** `{{variables}}` funcionan en los widgets **títulos** y **widgets de texto** del widget. Dentro del widget **SQL**, se representan en el panel, pero la vista previa de **Agregar widget** y la comprobación de **Guardar** del lado del servidor rechazan el token `{{name}}` sin procesar con _carácter inesperado &#39;&lbrace;&#39;_. Hasta que esto se arregle, use valores literales en consultas de widget y use **Duplicate** para hacer variantes.

### Widgets de fórmula

Una fórmula combina aritméticamente de 2 a 6 consultas SQL con nombre, por ejemplo `(errors / total) * 100`. Utilice una cuando un KPI necesite más de una consulta, como una relación, un margen de ampliación, una suma de tasas o una tasa de grabación. Ver [Tutorial 3](#86-combine-queries-with-formula-widgets).

### Compartir, exportar e importar

| Acción | Qué hace |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Guardar** | Conserva el nombre, las pestañas, los widgets, el diseño y las variables. Cada consulta de widget se vuelve a validar en el lado del servidor; un widget no válido falla al guardar por completo. |
| **Exportar** | Descargas `<dashboard-name>.json` (pestañas, widgets, variables). |
| **Importar** | Carga una exportación de JSON **en el tablero que está abierto**, reemplazando sus fichas y variables. Luego haz clic en **Guardar**. Sugerencia: cree primero un tablero vacío y, a continuación, impórtelo. |
| **CSV / PNG** | Descargas por widget desde el encabezado del widget. |

&#x200B;> [!TIP]
> Exportar e importar permite controlar las versiones de los paneles en Git y promocionar un panel de un cliente a otro. El formato de archivo se describe en el [Apéndice](#142-export-file-format).

### Sugerencias para el rendimiento de tableros y gráficos

- **Use tablas para grandes conjuntos de resultados** en lugar de barras o líneas de varias series.
- **Dividir tableros ocupados en fichas.** Solo se ejecutan los widgets de la pestaña visible.
- **Siempre `LIMIT` consultas con facetas**, especialmente en atributos de alta cardinalidad como `urlPath`, `traceId` y `jcr.path`.
- **Filtrar antes** por atributos de baja cardinalidad (`service`, `kind`, `status`).
- **Prefiera`TIMESERIES AUTO`** a los contenedores pequeños fijos y evite `TIMESERIES MAX` excepto en unos minutos.
- **Diseñar tendencias de largo alcance para resúmenes.** Ver [Rendimiento](#performance).
