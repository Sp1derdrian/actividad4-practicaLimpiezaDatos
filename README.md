# actividad4-practicaLimpiezaDatos

<div align="center">

# Limpieza y Curacion de Datos: Cafe Sales Dataset

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Cleaning-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Kaggle](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

<br/>

Este repositorio documenta el pipeline aplicado para la auditoria, depuracion y transformacion del conjunto de datos **`dirty_cafe_sales.csv`**, correspondiente al registro transaccional de una cafeteria extraido de Kaggle.

</div>

---

## Indice

- [Descripcion del Procedimiento](#1-descripcion-del-procedimiento)
- [Matriz de Problemas de Calidad y Decisiones](#2-matriz-de-problemas-de-calidad-y-decisiones)
- [Estructura del Proyecto](#3-estructura-del-proyecto)

---

## 1. Descripcion del Procedimiento

Cada etapa del pipeline abordo las inconsistencias estructurales detectadas en la base de datos cruda:

<details open>
<summary><b>Fase 1: Auditoria y Homologacion de Errores Sintacticos</b></summary>

- **Inspeccion exploratoria y tipado inicial**: Se audito el esquema estructural sobre un total de **10,000 registros**. Todas las columnas fueron cargadas originalmente como tipo `object` (texto) debido a cadenas de error generadas por el sistema.
- **Estandarizacion de anomalias a nulos (`NaN`)**: Se aislaron valores espurios como `"ERROR"` y `"UNKNOWN"` en campos cuantitativos y categoricos, reemplazandolos por valores nulos estandar (`np.nan`) para habilitar su procesamiento posterior.
</details>

<details open>
<summary><b>Fase 2: Casteo de Tipos y Validacion Estructural</b></summary>

- **Conversion de tipos de datos**:
  - `Quantity`, `Price Per Unit` y `Total Spent` se convirtieron a precision numerica continua (`float64` / entero).
  - `Transaction Date` se transformo al estandar cronologico `datetime64[ns]`.
- **Validacion de duplicados**: Se verifico la unicidad transaccional; no se detectaron filas repetidas a nivel global ni duplicidades en el identificador `Transaction ID`.
- **Deteccion de valores atipicos (Outliers)**: Se aplico el metodo del Rango Intercuartilico (IQR). Los picos observados en volumen correspondieron al catalogo comercial vigente y compras por lote, descartando anomalias fisicamente imposibles.
</details>

<details open>
<summary><b>Fase 3: Imputacion Escalonada y Exportacion</b></summary>

- **Regla de negocio determinista**: Para valores faltantes en `Total Spent`, se reconstruyo el calculo exacto de la venta:
  
  $$\text{Total Spent} = \text{Quantity} \times \text{Price Per Unit}$$

- **Reemplazo de datos numericos**: Se imputo la **mediana muestral** en las cantidades y precios faltantes remanentes para evitar sesgos provocados por distribuciones asimetricas.
- **Reemplazo de datos categoricos**: Se imputo la **moda** estadistica en las variables discretas `Item`, `Payment Method` y `Location`.
- **Propagacion cronologica**: Se empleo el metodo combinado `ffill` (propagacion del ultimo valor valido hacia adelante) y `bfill` (propagacion hacia atras) para resolver vacios en `Transaction Date`.
- **Exportacion del artefacto depurado**: Se genero el archivo final **`cafe_sales_cleaned_final.csv`**, optimizado para modelado predictivo y analitica descriptiva.
</details>

---

## 2. Matriz de Problemas de Calidad y Decisiones

| Dimension Evaluada | Variable(s) Afectada(s) | Evidencia en Datos Crudos | Accion Aplicada | Justificacion Tecnica |
| :--- | :--- | :--- | :--- | :--- |
| **Tipos de datos incorrectos** | `Quantity`, `Price Per Unit`, `Total Spent`, `Transaction Date` | Almacenamiento generico como texto, bloqueando operaciones aritmeticas y filtros de tiempo. | Reemplazo de cadenas `"ERROR"`/`"UNKNOWN"` por `np.nan` y conversion con `pd.to_numeric` y `pd.to_datetime`. | Habilita calculos vectorizados y analisis de series temporales sin excepciones en tiempo de ejecucion. |
| **Errores de formato / Cadenas de error** | Todas las columnas | Presencia reiterada de literales de error (`"ERROR"`, `"UNKNOWN"`) generados por el datalogger. | Homologacion uniforme a valores nulos estandar (`np.nan`). | Evita que codigos de falla se interpreten como categorias analiticas validas o generen sesgo en los calculos. |
| **Faltantes en metricas compuestas** | `Total Spent` | 173 celdas vacias nativas sumadas a los registros invalidados por cadenas de error. | Recalculo determinista mediante `Quantity * Price Per Unit`. | Asegura consistencia contable exacta antes de aplicar aproximaciones estadisticas. |
| **Faltantes numericos simples** | `Quantity`, `Price Per Unit` | Celdas nulas restantes tras el saneamiento inicial. | Imputacion mediante la mediana muestral de cada columna. | Mantiene cantidades enteras congruentes y mitiga el impacto de valores extremos en la distribucion. |
| **Faltantes categoricos** | `Item`, `Payment Method`, `Location` | Perdida de informacion de ~3,200 registros en canal/ubicacion y ~2,500 en metodo de pago. | Imputacion mediante el valor estadisticamente mas frecuente (moda). | Modela el comportamiento modal dominante de la sucursal sin distorsionar la distribucion global. |
| **Faltantes cronologicos** | `Transaction Date` | 159 transacciones con marca temporal nula o corrupta. | Imputacion direccional combinada (`ffill` y `bfill`). | Preserva la secuencia operativa y el orden relativo en las transacciones registradas. |
| **Registros duplicados** | Integridad total de la base | 0 observaciones redundantes detectadas. | Conservacion intacta de los registros. | Se confirma la unicidad e integridad referencial de `Transaction ID`. |
| **Valores atipicos (Outliers)** | `Quantity`, `Total Spent` | Observaciones situadas en la cola superior de la distribucion segun la regla IQR. | Mantenimiento de las transacciones sin truncamiento. | Corresponden a compras corporativas o de grupos reales plenamente validas segun el catalogo. |

---

## 3. Estructura del Proyecto

```plaintext
├── dirty_cafe_sales.csv            <- Archivo crudo con anomalias de entrada
├── cafe_sales_cleaned_final.csv    <- Dataset depurado, tipado y validado
├── a4_practicaDeLimpiezaDeDatos.py <- Código de python empleado para la limpieza de datos
└── README.md                       <- Documentacion tecnica del procedimiento
