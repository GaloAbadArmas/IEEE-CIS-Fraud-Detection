# IEEE-CIS Fraud Detection

Proyecto de detección de fraude financiero, desarrollado sobre el dataset **IEEE-CIS Fraud Detection**. El objetivo es construir modelos capaces de identificar transacciones fraudulentas minimizando pérdidas económicas y reduciendo falsos negativos.

---

# Project Overview

La detección de fraude representa un problema altamente desafiante debido al **desbalanceo extremo de clases**, la alta dimensionalidad y la presencia de grandes cantidades de datos faltantes.

En este proyecto se desarrolla un pipeline completo de **Data Science y Machine Learning**, incluyendo:

- Data Understanding
- Exploratory Data Analysis (EDA)
- Data Preprocessing
- Feature Engineering
- Selección de Variables
- Modelado Predictivo
- Evaluación de Modelos

---

# Dataset

Se utilizó el dataset **IEEE-CIS Fraud Detection**, compuesto por información transaccional y variables de identidad asociadas a cada operación.

El dataset incluye:

- información de pago,
- datos del dispositivo,
- comportamiento transaccional,
- atributos temporales,
- relaciones entre entidades,
- e indicadores de riesgo anonimizados.

---

# Data Understanding

## Objetivo

El objetivo de esta fase fue comprender la estructura del dataset, evaluar la calidad de los datos e identificar desafíos relevantes antes del modelado predictivo.

Durante esta etapa se analizaron:

- dimensiones del dataset,
- tipos de variables,
- distribución de fraude,
- consistencia entre datasets,
- valores faltantes,
- variables dominadas,
- y características generales de los datos.

---

## Descripción de Variables

El dataset contiene información **transaccional** y de **identidad digital**. Muchas variables se encuentran anonimizadas por motivos de privacidad.

### Transaction Table

| Variable | Descripción |
|----------|-------------|
| `TransactionDT` | Diferencia temporal (*timedelta*) desde una fecha de referencia. No representa un timestamp real. |
| `TransactionAmt` | Monto de la transacción en USD. |
| `ProductCD` | Código del producto asociado a la transacción. |
| `card1 - card6` | Información de la tarjeta: tipo, categoría, banco emisor, país y otros atributos anonimizados. |
| `addr1 - addr2` | Información relacionada con dirección. |
| `dist1 - dist2` | Variables de distancia relacionadas con comportamiento o ubicación. |
| `P_emaildomain` | Dominio del correo electrónico del comprador. |
| `R_emaildomain` | Dominio del correo electrónico del receptor. |
| `C1 - C14` | Variables de conteo relacionadas con asociaciones entre entidades. |
| `D1 - D15` | Variables temporales (*timedeltas*), por ejemplo tiempo entre eventos. |
| `M1 - M9` | Variables de coincidencia (*match features*). |
| `Vxxx` | Variables generadas por Vesta (*engineered features*), relacionadas con rankings, conteos y relaciones entre entidades. |

### Categorical Features

Las siguientes variables categóricas requieren codificación para ser utilizadas en modelos de Machine Learning:

```text
ProductCD
card1 - card6
addr1, addr2
P_emaildomain
R_emaildomain
M1 - M9
```

---

## Identity Table

La tabla de identidad contiene información del entorno digital asociado a las transacciones.

Incluye:

- información de red,
- proveedor de internet (*ISP*),
- uso de proxy,
- huella digital del dispositivo,
- navegador,
- sistema operativo,
- versión del software.

Estas variables fueron recopiladas por el sistema antifraude de **Vesta** y socios de seguridad digital.

> Los nombres reales de varias variables se encuentran anonimizados por privacidad y acuerdos contractuales.

### Identity Categorical Features

```text
DeviceType
DeviceInfo
id_12 - id_38
```

---

## Dataset Dimensions

### Raw datasets

| Dataset | Rows | Columns |
|----------|------|----------|
| Train Transaction | 590,540 | 394 |
| Train Identity | 144,233 | 41 |
| Test Transaction | 506,691 | 393 |
| Test Identity | 141,907 | 41 |

### After Merge

| Dataset | Shape |
|----------|--------|
| Train | (590,540, 434) |
| Test | (506,691, 433) |

El dataset presenta una **alta dimensionalidad**, con más de 400 variables disponibles para modelado.

Además, no todas las transacciones contienen información de identidad, generando una importante presencia de datos faltantes que deberán ser tratados en etapas posteriores.

---

## Fraud Distribution

Distribución de la variable objetivo `isFraud`:

| Class | Percentage |
|--------|------------|
| No Fraud | 96.5% |
| Fraud | 3.5% |

El dataset presenta un **desbalanceo severo de clases**, típico en escenarios reales de fraude financiero.

Esto implica que métricas como **accuracy pueden ser engañosas**, ya que un modelo podría obtener alta precisión global prediciendo únicamente transacciones legítimas.

Por esta razón, el proyecto prioriza métricas como:

- Recall
- Precision
- F1-score
- ROC-AUC

especialmente sobre la clase minoritaria (*fraude*).


---

## Missing Values

Se identificaron múltiples variables con altos porcentajes de valores faltantes, especialmente en atributos de identidad y comportamiento.

Algunas variables presentan más del **90% de missing values**.

### Interpretation

La gran cantidad de valores faltantes sugiere:

- baja cobertura de ciertas variables,
- información no disponible para todas las transacciones,
- necesidad de imputación o selección de variables.

En fraude, la ausencia de datos también puede representar señal predictiva.


---

## Dominated Variables

Se identificaron múltiples variables donde más del **95% de los registros contienen el mismo valor**.

Estas variables presentan baja variabilidad y potencialmente menor capacidad predictiva. Sin embargo, dado el contexto de fraude, no se eliminarán automáticamente sin evaluación previa.


---

## Data Consistency

Se detectaron inconsistencias en nombres de columnas entre train y test (`id_01` vs `id-01`).

### Interpretation

Este hallazgo representa un posible riesgo de incompatibilidad durante el modelado y requiere estandarización previa.


---

# Tech Stack

- Python
- Pandas
- NumPy
- Scikit-Learn
- XGBoost
- LightGBM
- Matplotlib
- Seaborn

---
