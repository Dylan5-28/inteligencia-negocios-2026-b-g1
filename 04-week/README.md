<!-- CONFIG (FULL_NAME + GITHUB_USER) -->
**FULL_NAME:** Dylan Solano Salazar  
**GITHUB_USER:** Dylan5-28  

---

# Actividad Calificable - Corte 1: Modelo Estrella

## 1. Proceso de negocio, pregunta y KPI
* **Proceso de negocio:** Despacho y entrega de carga terrestre.
* **Pregunta de negocio:** ¿Cuál es el porcentaje de entregas a tiempo por tipo de cliente y zona destino en el último semestre?
* **KPI y Meta:** 
  * **KPI:** Tasa de entregas a tiempo ($[ \text{Envíos a tiempo} / \text{Total de envíos} ] \times 100$).
  * **Meta:** Mantener el indicador por encima del 92% mensual.

---
## 2. Clasificación OLTP vs. OLAP
* **Sistema de origen (OLTP):** Sistema transaccional de operaciones para el registro diario de guías, asignación de conductores y estados de envío en tiempo real. Está pensado para escrituras rápidas y operativas sin tumbar el sistema de flota.
* **Justificación OLAP / Warehouse:** Ejecutar reportes históricos de tiempos de entrega en el OLTP satura las consultas de los despachadores en vivo. El entorno OLAP permite organizar los datos consolidados en esquemas de analítica para revisar rendimiento histórico sin frenar la operación.

---

## 3. Diseño del Modelo Estrella

### Tabla de Hechos: `Hechos_Despachos`
* `ID_Despacho` (PK / Key degenerada)
* `ID_Tiempo` (FK)
* `ID_Conductor` (FK)
* `ID_Cliente` (FK)
* `ID_Ubicacion` (FK)
* **Medidas:**
  * `Cantidad_Paquetes` (Entero)
  * `Peso_Total_KG` (Decimal)
  * `Costo_Flete` (Decimal)
  * `Tiempo_Entrega_Horas` (Decimal)
  * `Entrega_A_Tiempo` (Entero: 1 = Sí, 0 = No)

### Tablas de Dimensiones:
* **`Dim_Tiempo`**: `ID_Tiempo` (PK), `Fecha`, `Dia`, `Mes`, `Nombre_Mes`, `Trimestre`, `Año`, `Es_Festivo`.
* **`Dim_Conductor`**: `ID_Conductor` (PK), `Nombre_Conductor`, `Tipo_Licencia`, `Tipo_Vehiculo`, `Estado_Vehiculo`.
* **`Dim_Ubicacion`**: `ID_Ubicacion` (PK), `Ciudad_Origen`, `Ciudad_Destino`, `Departamento`, `Zona_Ruta`.
* **`Dim_Cliente`**: `ID_Cliente` (PK), `Nombre_Empresa`, `Tipo_Contrato` (Corporativo, Pyme, Ocasional), `Ciudad_Sede`.

---

## 4. Esquema en Estrella y Preguntas

```text
       [Dim_Tiempo]
             |
             v
[Dim_Cliente]---> [Hechos_Despachos] <---[Dim_Conductor]
             ^
             |
     [Dim_Ubicacion]

### Preguntas que responde el modelo:
1. ¿Qué zona o ruta presenta el mayor promedio de retraso en horas durante época de lluvia o festivos?
2. ¿Cuál es el costo total de flete movilizado por tipo de contrato de cliente en cada departamento?

---

## Model & questions

We designed this data model to keep track of our day-to-day freight operations, keeping `Fact_Shipments`
 right at the center. In this table, we log core metrics like package counts, total shipment weight,
freight charges, delivery times, and whether an order arrived on time. To get a clear picture of what those numbers mean,
 we connected the fact table to four key dimensions: time, drivers, destinations, and clients.

Setting up this star schema solves a couple of real pain points we deal with in logistics. For starters,
 it helps us pinpoint exactly which delivery routes get hit with the worst delays during bad weather or peak holiday weeks.
 It also lets us see how different driver fleets perform when handling orders for our key corporate accounts, making it much easier to optimize route planning over time.
