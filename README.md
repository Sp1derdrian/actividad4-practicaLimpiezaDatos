# actividad4-practicaLimpiezaDatos
# Limpieza y Curación de Datos: Cafe Sales Dataset

Este repositorio contiene el procedimiento aplicado para la transformación del conjunto de datos `dirty_cafe_sales.csv`, correspondiente al historial transaccional de una cafetería extraído de la página de Kaggle.

---

## 1. Descripción del Procedimiento

Cada paso tuvo el propósito de mejorar las inconsistencias encontradas en la base de datos:

1. **Inspección exploratoria y tipado inicial**: Se analizó el esquema de datos crudo identificando 10,000 registros. Todas las columnas fueron importadas inicialmente como tipo `object` (cadena de texto) debido a la presencia de literales de error del sistema.
2. **Estandarización de anomalías a nulos (`NaN`)**: Se detectaron cadenas de texto como `"ERROR"` y `"UNKNOWN"` en variables cuantitativas y categóricas. Estas cadenas fueron sustituidas por valores nulos estándar (`np.nan`) para que los datos fueran homogéneos.
3. **Conversión de tipos de datos**: 
   - `Quantity`, `Price Per Unit` y `Total Spent` fueron convertidas a formato numérico continuo (`float64` / entero).
   - `Transaction Date` fue convertida a estructura cronológica `datetime64[ns]`.
4. **Validación de duplicados**: Se buscó la existencia de filas duplicadas completas y se confirmó la ausencia de registros repetidos en la base de datos.
5. **Detección de valores atípicos (Outliers)**: Se utiliza el método IQR para encontrar los outliers mediante las diversas dispersiones numéricas. Se identificó que las variaciones correspondían al catálogo regular de productos y compras múltiples, descartando anomalías fuertes.
6. **Estrategia de reemplazo de datos escalonada**:
   - **Regla de negocio determinista**: Para valores faltantes en `Total Spent`, se recalculó directamente mediante la siguiente multiplicacioón:  
     $$\text{Total Spent} = \text{Quantity} \times \text{Price Per Unit}$$
   - **Reemplazo de datos numéricos**: Se aplicó la mediana muestral en cantidades y precios restantes debido a su robustez contra sesgos.
   - **Reemplazo de datos categóricos**: Se utilizó la moda para rellenar variables discretas en las columnas (`Item`, `Payment Method`, `Location`).
   - **Propagación cronológica**: En fechas faltantes se implementó un esquema combinado de arrastre temporal (`ffill` y `bfill`).
   - Con esta técnica se rellenan valores faltantes en series de tiempo. ffill copia el último valor válido hacia adelante. bfill copia el siguiente valor válido hacia atrás.
7. **Exportación de la base de datos limpia**: Se generó el archivo final depurado `cafe_sales_cleaned_final.csv` listo para análisis y modelado.

---

# 📊 Resumen de Problemas de Calidad y Decisiones Tomadas

| Problema detectado | Variables afectadas | Evidencia en datos crudos | Acción aplicada | Justificación |
| :--- | :--- | :--- | :--- | :--- |
| **Tipos de datos incorrectos** | `Quantity`, `Price Per Unit`, `Total Spent`, `Transaction Date` | Guardados como texto, bloqueando cálculos y fechas. | Se reemplazaron cadenas `"ERROR"`/`"UNKNOWN"` por `np.nan` y se convirtieron con `pd.to_numeric` y `pd.to_datetime`. | Permite hacer operaciones matemáticas y análisis temporal sin errores. |
| **Errores de formato / Literales de error** | Todas las columnas | Valores `"ERROR"` y `"UNKNOWN"` generados por el datalogger. | Se transformaron en valores nulos estándar (`np.nan`). | Evita que se procesen como categorías válidas o generen fallos en cálculos. |
| **Valores faltantes en métricas compuestas** | `Total Spent` | 173 nulos originales más los generados por valores `"ERROR"`. | Se recalculó multiplicando `Quantity * Price Per Unit`. | Garantiza consistencia contable antes de usar métodos estadísticos. |
| **Valores faltantes numéricos** | `Quantity`, `Price Per Unit` | Celdas vacías tras la limpieza inicial. | Se reemplazaron con la mediana de cada atributo. | Mantiene valores realistas y evita sesgos por valores extremos. |
| **Valores faltantes categóricos** | `Item`, `Payment Method`, `Location` | Pérdidas de hasta ~3,200 registros en ubicación y ~2,500 en método de pago. | Se completaron con el valor más frecuente (moda). | Refleja el comportamiento más común sin alterar la distribución global. |
| **Valores faltantes cronológicos** | `Transaction Date` | 159 registros con fechas vacías o corruptas. | Se aplicó arrastre temporal (`ffill` / `bfill`). | Conserva la secuencia y coherencia de las transacciones. |
| **Registros duplicados** | Todo el dataset | No se detectaron filas repetidas. | No se aplicó eliminación. | La base ya presenta unicidad en `Transaction ID`. |
| **Valores atípicos (Outliers)** | `Quantity`, `Total Spent` | Transacciones con valores altos según IQR. | Se conservaron sin cambios. | Representan consumos legítimos (ej. pedidos corporativos). |

