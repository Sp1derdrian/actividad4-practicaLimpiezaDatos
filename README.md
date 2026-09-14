# actividad4-practicaLimpiezaDatos
# Limpieza y Curación de Datos: Cafe Sales Dataset

Este repositorio contiene el procedimiento de inspección, diagnóstico de calidad, depuración y transformación del conjunto de datos `dirty_cafe_sales.csv`, correspondiente al historial transaccional de una cafetería.

---

## 1. Breve Descripción del Procedimiento

El flujo de procesamiento se diseñó para corregir inconsistencias sintácticas y semánticas sin alterar la integridad del negocio:

1. **Inspección exploratoria y tipado inicial**: Se analizó el esquema de datos crudo identificando 10,000 registros. Todas las columnas fueron importadas inicialmente como tipo `object` (cadena de texto) debido a la presencia de literales de error del sistema.
2. **Estandarización de anomalías a nulos (`NaN`)**: Se detectaron etiquetas sintácticas como `"ERROR"` y `"UNKNOWN"` en variables cuantitativas y categóricas. Estas cadenas fueron sustituidas por valores nulos estándar (`np.nan`) para permitir el casteo adecuado.
3. **Conversión de tipos de datos**: 
   - `Quantity`, `Price Per Unit` y `Total Spent` fueron convertidas a formato numérico continuo (`float64` / entero).
   - `Transaction Date` fue convertida a estructura cronológica `datetime64[ns]`.
4. **Validación de unicidad (Duplicados)**: Se auditó la existencia de filas duplicadas completas y se confirmó la ausencia de registros redundantes.
5. **Detección de valores atípicos (Outliers)**: Mediante el método del Rango Intercuartílico (IQR), se evaluaron las dispersiones numéricas. Se identificó que las variaciones correspondían al catálogo regular de productos y compras múltiples, descartando anomalías extremas no plausibles.
6. **Estrategia de imputación escalonada**:
   - **Regla de negocio determinista**: Para valores faltantes en `Total Spent`, se recalculó directamente mediante la ecuación:  
     $$\text{Total Spent} = \text{Quantity} \times \text{Price Per Unit}$$
   - **Imputación estadística numérica**: Se aplicó la mediana muestral en cantidades y precios restantes debido a su robustez contra sesgos.
   - **Imputación categórica**: Se utilizó la moda (valor más frecuente) para rellenar variables discretas (`Item`, `Payment Method`, `Location`).
   - **Propagación cronológica**: En fechas faltantes se implementó un esquema combinado de arrastre temporal (`ffill` y `bfill`).
7. **Exportación estructurada**: Se generó el archivo final depurado `cafe_sales_cleaned_final.csv` listo para análisis exploratorio y modelado.

---

## 2. Resumen de Problemas de Calidad y Decisiones Tomadas

| Problema de calidad detectado | Variable(s) afectada(s) | Evidencia en datos crudos | Decisión / Técnica aplicada | Justificación técnica |
| :--- | :--- | :--- | :--- | :--- |
| **Tipos de datos incorrectos** | `Quantity`, `Price Per Unit`, `Total Spent`, `Transaction Date` | Almacenados como `object` (texto) impidiendo operaciones aritméticas. | Reemplazo de cadenas `"ERROR"`/`"UNKNOWN"` por `np.nan` y casteo con `pd.to_numeric` y `pd.to_datetime`. | Permite habilitar operaciones matemáticas vectorizadas y análisis de series temporales. |
| **Errores de formato / Literales de error** | Todas las columnas cuantitativas y categóricas | Celdas con valores texto `"ERROR"` y `"UNKNOWN"` generados por el datalogger. | Homologación global hacia valores nulos estándar (`np.nan`). | Evita que etiquetas de error sean procesadas como categorías legítimas o generen excepciones en cálculos. |
| **Valores faltantes en métricas compuestas** | `Total Spent` | 173 nulos originales más los generados por valores `"ERROR"`. | Recálculo lógico multiplicando `Quantity * Price Per Unit`. | Prioriza la consistencia contable determinista antes de recurrir a aproximaciones estadísticas arbitrarias. |
| **Valores faltantes numéricos** | `Quantity`, `Price Per Unit` | Celdas nulas tras la limpieza inicial de errores. | Imputación mediante la mediana muestral del atributo. | Mantiene cantidades discretas realistas y protege el valor central contra sesgos producidos por outliers. |
| **Valores faltantes categóricos** | `Item`, `Payment Method`, `Location` | Pérdidas de hasta ~3,200 registros en ubicación y ~2,500 en método de pago. | Imputación por la moda (frecuencia más alta). | Asigna el comportamiento operativo modal observado en la cafetería sin alterar la distribución global predominante. |
| **Valores faltantes cronológicos** | `Transaction Date` | 159 registros con fechas vacías o corruptas. | Propagación direccional (`ffill` / `bfill`). | Conserva la secuencia temporal y la vecindad transaccional del bloque de ventas. |
| **Registros duplicados** | Totalidad del dataset | 0 filas redundantes detectadas. | Mantenimiento del set sin exclusión forzada. | La base presenta unicidad transaccional adecuada en su identificador `Transaction ID`. |
| **Valores atípicos (Outliers)** | `Quantity`, `Total Spent` | Transacciones de volumen superior en la distribución IQR. | Conservación de los valores sin truncamiento. | Corresponden a consumos legítimos dentro del rango comercial del establecimiento (e.g., pedidos corporativos). |
