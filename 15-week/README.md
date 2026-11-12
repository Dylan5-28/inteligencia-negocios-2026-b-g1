# Proyecto de Aula: Inteligencia de Negocios (Fase 3)

## 1. Ficha del Proyecto
* **Institución:** Corporación Universitaria del Huila (CORHUILA)
* **Asignatura:** Business Intelligence
* **Ubicación en Repositorio:** `15-week/proyecto-aula/`
* **Integrantes del Equipo:**
  * Dylan Solano Salazar
  * Juan Diego Escobar
  * Juan Esteban González
* **Pregunta de Negocio:** ¿Cómo influye la distribución por categorías de productos y el comportamiento temporal en los ingresos netos y la rotación de inventario de la compañía para optimizar la toma de decisiones gerenciales?

---

## 2. Diccionario de Datos (Esquema en Estrella)

El modelo analítico implementa una arquitectura dimensional tipo **Estrella (Star Schema)** con relaciones formales de uno a varios ($1:*$) desde las dimensiones hacia las tablas de hechos[cite: 2].

### Tabla de Hechos: `Fact_Ventas`
| Columna / Campo | Tipo de Dato | Descripción | Rol en el Modelo |
| :--- | :--- | :--- | :--- |
| `ID_Venta` | Texto / Entero | Identificador único de cada transacción comercial | Llave Primaria (PK) |
| `Fecha` | Fecha (`Date`) | Fecha en la que se registró la operación de venta | Llave Foránea (`Dim_Calendario`) |
| `Codigo_Producto` | Texto | Código identificador del artículo comercializado | Llave Foránea (`Dim_Producto`) |
| `Producto` | Texto | Nombre descriptivo del producto vendido | Atributo |
| `Categoria` | Texto | Línea o categoría comercial a la que pertenece el artículo | Atributo |
| `Canal` | Texto | Canal de distribución o punto de venta | Atributo |
| `Sede` | Texto | Sucursal o ubicación operativa de la venta | Atributo |
| `Estado` | Texto | Estado de la transacción (ej. Completada, Facturada) | Atributo |
| `Cantidad` | Número Entero | Número de unidades vendidas en la transacción | Métrica Base |
| `Costo_Unitario` | Decimal | Costo unitario asociado al producto en la operación | Métrica Base |
| `Precio_Unitario` | Decimal | Precio unitario de venta al público | Métrica Base |
| `Costo_Venta` | Decimal | Valor monetario total registrado en la venta | Métrica Base |

### Tabla de Dimensión: `Dim_Producto`
| Columna / Campo | Tipo de Dato | Descripción | Rol en el Modelo |
| :--- | :--- | :--- | :--- |
| `Codigo_Producto` | Texto | Código único de referencia en el catálogo | Llave Primaria (PK) |
| `Producto` | Texto | Nombre oficial del producto | Atributo Descriptivo |
| `Categoria` | Texto | Clasificación o familia comercial del producto | Atributo de Agrupación |
| `Costo_Unitario_Catalogo` | Decimal | Costo estándar de referencia en inventario | Atributo Numérico |

### Tabla de Dimensión: `Dim_Calendario`
| Columna / Campo | Tipo de Dato | Descripción | Rol en el Modelo |
| :--- | :--- | :--- | :--- |
| `Date` (Fecha) | Fecha (`Date`) | Serie continua de fechas generada mediante `CALENDAR()` | Llave Primaria (PK) |
| `Año / Mes / Trimestre` | Texto / Entero | Jerarquía temporal para análisis evolutivo | Atributos Temporales |

---

## 3. Medidas DAX y Cifras de Control

Se desarrollaron cinco medidas analíticas en lenguaje DAX para evaluar el rendimiento financiero y operativo del negocio:

| Nombre de la Medida | Expresión DAX | Cifra / Criterio de Control | Propósito Analítico |
| :--- | :--- | :--- | :--- |
| **`Total_Ventas`** | `SUM(Fact_Ventas[Costo_Venta])` | Debe coincidir exactamente con la sumatoria bruta de la columna `Costo_Venta` en el origen de datos sin filtros aplicados. | Calcula el acumulado monetario total de las operaciones comerciales. |
| **`Total_Cantidad`** | `SUM(Fact_Ventas[Cantidad])` | Validada contra el conteo total de unidades físicas despachadas en la tabla transaccional (`Fact_Ventas`). | Suma el volumen total de unidades comercializadas. |
| **`Venta_Bruta`** | `SUMX(Fact_Ventas, Fact_Ventas[Cantidad] * RELATED(Dim_Producto[Costo_Unitario_Catalogo]))` | Consistente con el producto cruzado entre cantidades vendidas y el costo base del catálogo `Dim_Producto`. | Determina el valor bruto antes de aplicar deducciones o ajustes. |
| **`Venta_Neta`** | `[Venta_Bruta] - [Descuentos_Aplicados]` | Siempre menor o igual a `Venta_Bruta`; verifica que los descuentos no generen márgenes negativos. | Representa el ingreso real tras aplicar descuentos comerciales. |
| **`KPI_Principal`** | `DIVIDE([Total_Ventas], [Total_Cantidad])` | Controlada con `DIVIDE()` para retornar `BLANK()` o `0` ante divisiones por cero al aplicar filtros vacíos. | Indicador clave que mide el ticket o valor promedio por unidad vendida. |

---

## 4. Capturas del Tablero y Modelo



<img width="1919" height="944" alt="Captura de pantalla 2026-11-11 161653" src="https://github.com/user-attachments/assets/c953c754-afbd-46d3-82d1-53302be5467a" /><img width="556" height="470" alt="Captura de pantalla 2026-11-12 171008" src="https://github.com/user-attachments/assets/31a5ca15-f070-4249-8743-e93229538f86" />
<img width="504" height="477" alt="Captura de pantalla 2026-11-12 171002" src="https://github.com/user-attachments/assets/b710c433-8f8d-443f-a2ed-adf752675830" />
<img width="1919" height="1013" alt="Captura de pantalla 2026-11-11 161709" src="https://github.com/user-attachments/assets/87f982f9-141a-41ab-9e89-17da762f6c2d" />


---

## 5. Overview & Insights

**Problem:** The organization faced significant difficulties in consolidating transactional sales, purchasing, and inventory records, which hindered real-time visibility into product category performance and profit margins[cite: 2, 6].  
**Data and ETL:** To solve this, we extracted and transformed historical operational datasets using Power Query, standardizing text formats, removing duplicate entries, enforcing strict data types, and ensuring full data anonymization.  
**Model:** We implemented an optimized star schema in Power BI centered on the `Fact_Ventas` fact table connected via single-direction one-to-many ($1:*$) relationships to `Dim_Producto` and a dynamically generated `Dim_Calendario` dimension[cite: 2, 6].  
**DAX & Interactivity:** Five core DAX measures—led by `KPI_Principal` using safe `DIVIDE` logic—were integrated alongside category and date slicers and a drill-down hierarchy to enable dynamic analytical exploration.  
**Key Insight:** The interactive dashboard revealed a high revenue concentration in a small subset of core product categories, while secondary product lines exhibited low turnover and underutilized inventory.  
**Recommendation:** Consequently, we recommend that the commercial and marketing departments launch targeted cross-selling promotions and bundled offers for low-rotation categories starting in the next monthly operating cycle to balance cash flow and optimize stock turnover[cite: 6].
