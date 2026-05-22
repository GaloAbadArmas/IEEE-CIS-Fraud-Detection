# IEEE-CIS Fraud Detection

Proyecto de detección de fraude en transacciones electrónicas, desarrollado sobre el dataset **IEEE-CIS Fraud Detection**. El objetivo es construir modelos capaces de identificar transacciones fraudulentas minimizando pérdidas económicas y reduciendo falsos negativos.

---

# Descripción general del proyecto

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

- Información de pago
- Datos del dispositivo
- Comportamiento transaccional
- Atributos temporales
- Relaciones entre entidades
- Indicadores de riesgo anonimizados

---

# 01. Data Understanding

## Objetivo

El objetivo de esta fase fue comprender la estructura del dataset, evaluar la calidad de los datos e identificar desafíos relevantes antes del modelado predictivo.

Durante esta etapa se analizaron:

- Dimensiones del dataset
- Tipos de variables
- Distribución de fraude
- Consistencia entre datasets
- Valores faltantes
- Variables dominadas
- Características generales de los datos

---

## Descripción de Variables

El dataset contiene información **transaccional** y de **identidad digital**. Muchas variables se encuentran anonimizadas por motivos de privacidad.

### Variables

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

- Información de red
- Proveedor de internet (*ISP*)
- Uso de proxy
- Huella digital del dispositivo
- Navegador
- Sistema operativo
- Versión del software

Estas variables fueron recopiladas por el sistema antifraude de **Vesta** y socios de seguridad digital. Los nombres reales de varias variables se encuentran anonimizados por privacidad y acuerdos contractuales.

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

Especialmente sobre la clase minoritaria (*fraude*).


---

## Missing Values

Se identificaron múltiples variables con altos porcentajes de valores faltantes, especialmente en atributos de identidad y comportamiento.

Algunas variables presentan más del **90% de missing values**.

La gran cantidad de valores faltantes sugiere:

- Baja cobertura de ciertas variables
- Información no disponible para todas las transacciones
- Necesidad de imputación o selección de variables

En fraude, la ausencia de datos también puede representar señal predictiva.


---

## Dominated Variables

Se identificaron múltiples variables donde más del **95% de los registros contienen el mismo valor**.

Estas variables presentan baja variabilidad y potencialmente menor capacidad predictiva. Sin embargo, dado el contexto de fraude, no se eliminarán automáticamente sin evaluación previa.


---

## Data Consistency

Se detectaron inconsistencias en nombres de columnas entre train y test (`id_01` vs `id-01`). Este hallazgo representa un posible riesgo de incompatibilidad durante el modelado y requiere estandarización previa.


---
Puedes ver el data understandig [en este archivo](notebooks/01_data_understanding.ipynb).
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

# 02. Análisis Exploratorio de Datos (EDA)

## Objetivo

El objetivo de esta etapa fue explorar el comportamiento de las transacciones fraudulentas y no fraudulentas para identificar patrones, diferencias relevantes y posibles señales predictivas útiles para el modelado.

Durante esta fase se analizaron:

- Distribución del monto de transacción
- Fraude por tipo de producto
- Fraude por red de tarjeta
- Dominios de correo electrónico
- Patrones temporales
- Variables de coincidencia (`M1-M9`)
- Valores faltantes como señal predictiva
- Correlaciones con la variable objetivo

---

# Distribución del Monto de Transacción

## Análisis realizado

Se analizó la variable `TransactionAmt` para comparar el comportamiento del monto de transacciones legítimas y fraudulentas.

### Resultados

<img width="975" height="597" alt="image" src="https://github.com/user-attachments/assets/3dd218ed-a1aa-4422-a628-aabc90abbb52" />
<img width="975" height="515" alt="image" src="https://github.com/user-attachments/assets/4f5f52fb-3f7b-4df3-9423-64e6964f98e1" />

| Clase | Media | Mediana | Percentil 75 |
|--------|--------|----------|---------------|
| No fraude | 134.51 USD | 68.50 USD | 120 USD |
| Fraude | 149.24 USD | 75 USD | 161 USD |

Se identificaron diferencias en la distribución de TransactionAmt entre transacciones fraudulentas y legítimas. Las transacciones fraudulentas presentan mayor concentración en montos bajos, lo cual puede ser consistente con estrategias de validación de tarjetas (“card testing”), así como una cola de distribución asociada a montos más altos. Esto sugiere que el monto transaccional puede contener señal predictiva relevante.

---

# Tasa de Fraude por Producto (`ProductCD`)

## Análisis realizado

Se calculó la proporción de fraude para cada categoría de producto (`ProductCD`) con el fin de identificar segmentos de mayor riesgo.

### Resultados
<img width="975" height="668" alt="image" src="https://github.com/user-attachments/assets/926ae380-f5fa-43c3-bca3-be8b93a4c505" />

| ProductCD | Fraud Rate |
|------------|------------|
| C | **11.69%** |
| S | 5.90% |
| H | 4.77% |
| R | 3.78% |
| W | **2.04%** |


La variable ProductCD muestra diferencias importantes en la tasa de fraude. La categoría C presenta una tasa de fraude de aproximadamente 11.7%, significativamente superior al promedio global (3.5%), lo que sugiere que ciertos tipos de transacción presentan mayor vulnerabilidad al fraude y que esta variable puede aportar capacidad predictiva relevante al modelo.

---

# Tasa de Fraude por Red de Tarjeta (`card4`)

## Análisis realizado

Se evaluó el porcentaje de fraude según la red de tarjeta (`card4`).

### Resultados

<img width="975" height="833" alt="image" src="https://github.com/user-attachments/assets/6bd78e86-f9de-43d7-aecd-15e7121a78ee" />

| Red de Tarjeta | Fraud Rate |
|----------------|-------------|
| Discover | **7.73%** |
| Visa | 3.48% |
| Mastercard | 3.43% |
| American Express | **2.87%** |

### Distribución de tarjetas

| Tarjeta | Volumen aproximado |
|----------|-------------------|
| Visa | 384,767 |
| Mastercard | 189,217 |
| American Express | 8,328 |
| Discover | 6,651 |


La red `Discover` presenta la mayor tasa de fraude observada, mientras que `American Express` muestra el menor porcentaje.

No obstante, es importante considerar que algunas categorías poseen menor cantidad de observaciones, lo que puede generar mayor variabilidad estadística.

Aun así, las transacciones asociadas a discover presentan una mayor probabilidad observada de fraude, posiblemente relacionada con características del tipo de comercio, perfil de usuario, región o interacción con otras variables del sistema.

El tipo de tarjeta muestra diferencias en la incidencia de fraude y podría aportar información predictiva relevante, aunque debe interpretarse considerando el volumen de transacciones de cada categoría.

---

# Dominios de Correo Electrónico

## Análisis realizado

Se estudió la tasa de fraude según el dominio de correo electrónico del comprador (`P_emaildomain`).

### Resultados relevantes

<img width="988" height="605" alt="image" src="https://github.com/user-attachments/assets/4c36d437-68f7-459d-8bf5-b4853e6b5733" />


| Dominio | Fraud Rate |
|----------|-------------|
| outlook.com | **9.46%** |
| hotmail.com | 5.30% |
| gmail.com | 4.35% |


Se observan diferencias relevantes entre dominios de correo.

Algunos proveedores presentan tasas de fraude considerablemente superiores al promedio del dataset, lo que podría reflejar diferencias de comportamiento, perfiles de usuario o patrones operativos.

Sin embargo, estos resultados no implican causalidad y deben interpretarse con cautela.

---

# Patrones Temporales del Fraude

## Análisis realizado

Se exploró la evolución temporal del fraude utilizando la variable `TransactionDT`.

<img width="1143" height="547" alt="image" src="https://github.com/user-attachments/assets/236e66eb-12e7-4092-8ad8-f0c75b88eb87" />

La tasa de fraude presenta variabilidad significativa a lo largo del tiempo, lo que sugiere posibles cambios en el comportamiento transaccional y patrones de ataque. Debido a esta dependencia temporal, un split aleatorio tradicional podría introducir leakage de información, por lo que se considerará una estrategia de validación temporal en etapas posteriores del modelado.

Además, se analizó el volumen total de transacciones a lo largo del tiempo.
<img width="1169" height="547" alt="image" src="https://github.com/user-attachments/assets/bf3ccbb9-d126-487d-81ab-ed6a99ae7c8e" />

El volumen de transacciones no se mantiene constante a lo largo del tiempo. Durante los aproximadamente 183 días analizados, se observan períodos con alrededor de 2.000 transacciones diarias y otros en los que la actividad aumenta hasta unas 6.500–7.000 transacciones por día. Esta variación evidencia que el comportamiento transaccional presenta cambios significativos a lo largo del periodo estudiado.

---

# Variables de Coincidencia (`M1-M9`)

## Análisis realizado

Se evaluó la relación entre las variables `M1-M9` y la tasa de fraude.

Estas variables representan indicadores de coincidencia (*match features*) entre atributos relacionados con la transacción.

### Hallazgo relevante

Ejemplo observado en `M9`:

| Valor | Fraud Rate |
|--------|-------------|
| F | 3.00% |
| T | 1.78% |

Este constituye un hallazgo importante, ya que sugiere que las variables de consistencia contienen una señal predictiva real para la detección de fraude. En particular, las variables M2, M3, M4, M6, M8 y M9 muestran patrones relevantes donde, en general, una mayor consistencia se asocia con una menor probabilidad de fraude. 

Este comportamiento resulta coherente desde la perspectiva de negocio, dado que un comprador legítimo suele mantener información consistente entre sus distintos datos de identificación y transacción. En contraste, las operaciones fraudulentas tienden a presentar discrepancias, como diferencias entre la dirección de facturación y envío, inconsistencias entre el correo electrónico y el dispositivo utilizado, o desajustes en la dirección registrada, lo que convierte a estas variables en indicadores valiosos para identificar transacciones sospechosas.

---

# Missing Values como Señal Predictiva

## Análisis realizado

Se investigó si la presencia o ausencia de información en variables de identidad estaba relacionada con fraude.

### Resultados observados

Ejemplo evaluado: `DeviceType`.

Porcentaje de registros con información disponible:

| Clase | Información presente |
|--------|----------------------|
| No fraude | 77.26% |
| Fraude | 45.74% |

Ejemplo evaluado: `dist1`.

Porcentaje de registros con información disponible:

| Clase | Información presente |
|--------|----------------------|
| No fraude | 59.00% |
| Fraude | 77.00% |

Se observó que la ausencia de información no ocurre aleatoriamente. Variables relacionadas con dispositivo e identidad (DeviceType, DeviceInfo, id_31) presentan diferencias sustanciales en sus tasas de missing entre fraude y no fraude. Asimismo, variables de distancia (dist1, dist2) muestran patrones diferenciados, sugiriendo que la falta de información puede actuar como una señal predictiva relevante y no únicamente como un problema de calidad de datos.

---

# Correlación con la Variable Objetivo

## Análisis realizado

Se calcularon correlaciones entre variables numéricas y `isFraud` para identificar posibles señales predictivas o casos de *data leakage*.

### Variables más correlacionadas

| Variable | Correlación |
|------------|-------------|
| V257 | 0.383 |
| V246 | 0.367 |
| V244 | 0.364 |
| V242 | 0.361 |
| V201 | 0.328 |
| V200 | 0.319 |


No se identificaron correlaciones extremadamente altas que sugieran fuga directa de información (*data leakage*).

Sin embargo, múltiples variables `Vxxx` presentan correlaciones moderadamente altas con fraude, lo que sugiere que contienen señal predictiva importante. Las variables `Vxxx` parecen capturar patrones transaccionales relevantes y podrían desempeñar un papel importante en el rendimiento del modelo.

---

# Conclusión General del EDA

El análisis exploratorio permitió identificar patrones relevantes asociados al fraude. Variables como `ProductCD`, `card4`, dominios de correo electrónico y atributos temporales mostraron diferencias entre transacciones legítimas y fraudulentas. Asimismo, la ausencia de información del dispositivo emergió como una posible señal predictiva. Estos hallazgos respaldan la necesidad de un pipeline de feature engineering orientado a capturar relaciones temporales, categóricas y comportamentales para mejorar el desempeño de los modelos de detección de fraude.

---
Puedes ver el análisis técnico detallado [en este archivo](./02_eda_fraud_detection.ipynb).
---

# 03. Preprocesamiento de Datos

## Objetivo

El objetivo de esta etapa fue preparar el dataset para el entrenamiento de modelos de Machine Learning, resolviendo problemas de calidad de datos y transformando las variables a un formato adecuado para modelado.

Durante esta fase se realizaron:

- Análisis de tipos de variables
- Evaluación de valores faltantes
- Eliminación de variables con missing extremo
- Creación de señales derivadas de missing values
- Imputación de datos
- Codificación de variables categóricas
- División temporal de entrenamiento y validación

---

# Inventario de Variables

## Análisis realizado

Se realizó una inspección inicial para identificar el tipo de variables disponibles después del merge del dataset.

### Resultados

| Tipo de variable | Cantidad |
|------------------|-----------|
| Numéricas | 403 |
| Categóricas | 31 |

El dataset presenta una clara predominancia de variables numéricas, lo que facilita el entrenamiento de algoritmos de Machine Learning.

Sin embargo, las variables categóricas requieren transformación antes del modelado, especialmente aquellas con alta cardinalidad.

---

# Perfil de Missing Values

## Análisis realizado

Se evaluó el porcentaje de valores faltantes por variable para identificar atributos problemáticos.

### Variables con mayor porcentaje de missing

| Variable | Missing (%) |
|-----------|--------------|
| `id_24` | 99.20% |
| `id_25` | 99.13% |
| `id_07` | 99.13% |
| `id_08` | 99.13% |
| `id_21` | 99.13% |
| `id_26` | 99.13% |
| `id_22` | 99.12% |
| `id_23` | 99.12% |
| `id_27` | 99.12% |


Se observó un grupo de variables con más del **99% de datos faltantes**, lo que implica una cobertura extremadamente baja.

En estas condiciones, resulta difícil extraer señal útil consistente, especialmente cuando la información disponible es mínima.

---

# Eliminación de Variables con Missing Extremo

## Análisis realizado

Se identificaron variables con más del **95% de valores faltantes**.

Posteriormente, se verificó si la ausencia de información estaba relacionada con fraude antes de eliminarlas.

### Resultados

Se encontraron **9 variables** con más del **95% missing**:

```text
id_24
id_25
id_07
id_08
id_21
id_26
id_22
id_23
id_27
```

Después de evaluar su señal predictiva, estas variables fueron eliminadas.

### Shape posterior

| Dataset | Shape |
|----------|--------|
| Train | (590,540, 425) |

Aunque en fraude los valores faltantes pueden contener señal predictiva, en estas variables las diferencias observadas entre fraude y no fraude fueron pequeñas (≈1%).

Esto sugiere una baja capacidad discriminativa frente al costo de mantener variables extremadamente incompletas.

---

# Cardinalidad de Variables Categóricas

## Análisis realizado

Se evaluó el número de categorías únicas por variable categórica para definir una estrategia de codificación adecuada.

### Variables de alta cardinalidad

```text
DeviceInfo
id_33
id_31
id_30
R_emaildomain
P_emaildomain
```

### Variables de baja cardinalidad

Incluyen:

```text
ProductCD
card4
card6
M1-M9
id_12
id_15
id_16
id_28
id_29
id_34-id_38
DeviceType
```

Las variables con muchas categorías presentan un reto importante:

- One-Hot Encoding puede generar demasiadas columnas
- Aumentar dimensionalidad,
- Provocar sparsity.

Por ello se definió una estrategia diferenciada según cardinalidad. Se separaron variables categóricas de alta y baja cardinalidad para aplicar técnicas de codificación más eficientes y evitar explosión dimensional.

---

# División Temporal Train / Validation

## Análisis realizado

El dataset fue ordenado utilizando `TransactionDT` y posteriormente dividido de forma temporal:

- **80% entrenamiento**
- **20% validación**

### Resultados

| Dataset | Shape |
|----------|--------|
| Train | (472,432, 425) |
| Validation | (118,108, 425) |

### Fraud Rate

| Dataset | Fraud Rate |
|----------|-------------|
| Train | 3.51% |
| Validation | 3.44% |

La división temporal evita **data leakage temporal**, una práctica especialmente importante en fraude financiero.

En escenarios reales, el modelo siempre predice sobre eventos futuros, por lo que una separación aleatoria podría sobreestimar el rendimiento.

Además, la tasa de fraude se mantuvo estable entre entrenamiento y validación.

### Aclaración

Este split temporal divide el dataset respetando el orden cronológico de las transacciones. Primero, los datos se ordenan según TransactionDT, que representa el tiempo relativo de cada operación, y luego se separan en un 80% para entrenamiento (train_df) y un 20% para validación (valid_df). De esta manera, el modelo aprende únicamente con transacciones “del pasado” y se evalúa con transacciones “del futuro”, simulando un escenario real de producción donde los fraudes futuros no son conocidos durante el entrenamiento.

El motivo por el que este enfoque es importante es que evita data leakage o fuga de información. Todo el preprocessing —como imputación de medianas, escalado o frequency encoding— debe calcularse exclusivamente usando train_df y después aplicarse sobre valid_df. Si se utilizara todo el dataset antes del split, el modelo estaría incorporando información futura de manera indirecta, generando métricas artificialmente optimistas.

---

# Creación de Indicadores de Missing Values

## Análisis realizado

Se generaron variables binarias para capturar la ausencia de información en columnas seleccionadas.

Ejemplos:

```text
DeviceType
DeviceInfo
id_31
dist1
```


Durante el EDA se observó que la ausencia de información estaba asociada con fraude. Por esta razón, se preservó explícitamente esta señal mediante variables indicadoras.

La ausencia de datos fue tratada como posible señal predictiva mediante indicadores binarios de missing values.

---

# Imputación de Variables Categóricas

## Análisis realizado

Las variables categóricas de baja cardinalidad fueron imputadas con una categoría especial:

```text
"Missing"
```

Esta estrategia evita pérdida de información y permite conservar el significado potencial de la ausencia de datos. En problemas de fraude, los missing values pueden representar comportamiento anómalo. Los valores faltantes categóricos fueron preservados como categoría explícita para evitar pérdida de señal predictiva.

---

# Frequency Encoding

## Análisis realizado

Las variables categóricas de alta cardinalidad fueron transformadas mediante **Frequency Encoding**.

Variables tratadas:

```text
DeviceInfo
id_33
id_31
id_30
P_emaildomain
R_emaildomain
```
Frequency Encoding permite representar categorías mediante su frecuencia de aparición en el dataset.

Esta estrategia:

- Reduce dimensionalidad
- Evita miles de columnas adicionales
- Conserva información estadística útil

---

# Imputación de Variables Numéricas

## Análisis realizado

Las variables numéricas fueron imputadas utilizando la **mediana**.

### Resultados

- **405 variables numéricas tratadas**
- **0 missing values restantes**


La mediana es robusta frente a outliers, una propiedad especialmente relevante en datos financieros.

Después de la imputación, el dataset quedó completamente libre de valores faltantes.

---

# One-Hot Encoding

## Análisis realizado

Las variables categóricas de baja cardinalidad fueron transformadas mediante **One-Hot Encoding**.

### Resultados

| Dataset | Shape |
|----------|--------|
| Train | (472,432, 486) |
| Validation | (118,108, 486) |

Verificaciones finales:

- Sin missing values
- Mismas columnas entre train y validation


La codificación permitió convertir variables categóricas en un formato compatible con algoritmos de Machine Learning. Además, se verificó consistencia estructural entre entrenamiento y validación.

---

# Conclusión General del Preprocesamiento

La etapa de preprocesamiento permitió transformar un dataset complejo y altamente incompleto en un conjunto de datos consistente y apto para Machine Learning. Se eliminaron variables con baja utilidad, se preservó la señal de missing values, se aplicaron estrategias diferenciadas de codificación categórica y se implementó una validación temporal para evitar leakage. Como resultado, se obtuvo un dataset limpio, sin valores faltantes y listo para modelado predictivo.

---
Puedes ver el procesamiento de datos [en este archivo](./03_preprocessing.ipynb).
---

# 04. Modelado

## Objetivo

El objetivo de esta etapa fue construir y evaluar modelos para establecer un punto de referencia en la detección de fraude.

Se compararon distintos enfoques de Machine Learning con el fin de:

- Medir capacidad predictiva
- Evaluar sensibilidad al fraude
- Comparar métricas relevantes
- Analizar el impacto de reducción de dimensionalidad
- Establecer una línea base antes de optimización avanzada

Debido al fuerte desbalanceo del dataset, se priorizó la evaluación mediante:

- **ROC-AUC**
- **Recall (Fraude)**
- **Precision (Fraude)**
- **F1-score**
- **Matriz de Confusión**

más allá de accuracy.

---

# Preparación del Dataset para Modelado

## Análisis realizado

Antes del entrenamiento se eliminaron variables no relevantes para clasificación:

```text
TransactionID
isFraud
```

### Shape final del dataset

| Dataset | Shape |
|----------|--------|
| Train | (472,432, 484) |
| Validation | (118,108, 484) |

El dataset quedó compuesto por **484 variables predictoras**, incluyendo variables originales, variables codificadas y señales derivadas de missing values.

---

# Modelo Baseline: Random Forest

Se entrenó un modelo **Random Forest** como baseline inicial utilizando el conjunto completo de variables.

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | **0.875** |
| Accuracy | 0.88 |
| Precision (Fraude) | 0.18 |
| Recall (Fraude) | **0.69** |
| F1-score (Fraude) | 0.28 |

### Matriz de confusión

| | Pred No Fraude | Pred Fraude |
|---|---:|---:|
| No fraude | 101,092 | 12,952 |
| Fraude | 1,249 | 2,815 |

El modelo logró un **ROC-AUC sólido (0.875)**, indicando buena capacidad para separar transacciones fraudulentas y legítimas.

Hallazgos relevantes:

### Recall alto en fraude (0.69)

El modelo logra detectar aproximadamente: **69% de los fraudes reales**

Esto es positivo en un contexto financiero, donde perder fraudes (*false negatives*) suele ser costoso.

### Precision baja (0.18)

Sin embargo, solo: **18% de las alertas de fraude realmente eran fraude**

Esto implica una gran cantidad de **falsos positivos**.

En términos de negocio:

- Más revisiones manuales
- Fricción para usuarios legítimos
- Mayor costo operativo

 El Random Forest mostró una buena capacidad de detección de fraude, priorizando recall sobre precision. Aunque genera muchos falsos positivos, representa una base sólida para un sistema antifraude.

---

# Importancia de Variables (Feature Importance)

 En este análisis, la selección de variables se realiza a partir de la importancia calculada por el modelo Random Forest, específicamente mediante el atributo feature_importances_. Este indicador mide cuánto contribuye cada variable a la reducción de la impureza (como el índice Gini) a lo largo de todos los árboles del modelo, es decir, qué tan útil es cada feature para separar correctamente las clases de fraude y no fraude durante el entrenamiento. A partir de estos valores, las variables se ordenan de mayor a menor importancia y se seleccionan las 20 más relevantes para su visualización.

### Variables más relevantes

Entre las principales:

<img width="840" height="682" alt="image" src="https://github.com/user-attachments/assets/6b13fe76-37f8-494c-b554-6a980093c0b6" />

```text
V69
V257
C13
C5
C14
V70
C10
V219
V264
C1
```

Las variables tipo:

- `Vxxx`
- `Cxxx`
- `Dxxx`

aparecen como las más influyentes.

Esto sugiere que:

- Las variables generadas por **Vesta** contienen fuerte señal predictiva
- Variables de comportamiento transaccional tienen alto valor
- Relaciones históricas parecen importantes para detectar fraude

---

# Overfitting Check

Se comparó desempeño entre entrenamiento y validación utilizando ROC-AUC.

### Resultados

| Dataset | ROC-AUC |
|----------|----------|
| Train | 0.913 |
| Validation | 0.875 |

Existe una diferencia moderada entre entrenamiento y validación:

```text
0.913 → 0.875
```

Esto indica un **leve sobreajuste (*overfitting*)**, aunque todavía dentro de un rango razonable.

El modelo generaliza relativamente bien sobre datos futuros.

---

# Logistic Regression Baseline

Se entrenó una **Regresión Logística** utilizando un subconjunto balanceado para evitar "muertes" del kernel y **variables escaladas**.

### Dataset utilizado (Mini dataset experimental)

| Dataset | Shape |
|----------|--------|
| Train sample | (100,000, 484) |
| Validation sample | (25,000, 484) |

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | 0.819 |
| Accuracy | 0.78 |
| Precision (Fraude) | 0.11 |
| Recall (Fraude) | **0.73** |
| F1-score | 0.19 |

La regresión logística consiguió un **recall superior (0.73)** respecto al Random Forest.

Sin embargo:

- Precision extremadamente baja
- Demasiados falsos positivos
- Menor capacidad de separación global

Aunque es un modelo interpretable, parece limitado para capturar relaciones complejas presentes en fraude.

La regresión logística mostró alta sensibilidad al fraude, aunque con bajo poder discriminativo y una elevada cantidad de falsas alarmas.

---

# Linear Discriminant Analysis

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | 0.809 |
| Accuracy | 0.95 |
| Precision (Fraude) | 0.32 |
| Recall (Fraude) | 0.33 |
| F1-score | 0.32 |

Aunque LDA alcanzó una accuracy elevada (95%), su recall sobre la clase fraudulenta fue limitado (33%), indicando baja capacidad para detectar fraude. Si bien presentó la mejor precisión entre los modelos evaluados, su comportamiento conservador generó una elevada tasa de falsos negativos, limitando su utilidad en un contexto antifraude donde la sensibilidad suele ser prioritaria.

---

# Quadratic Discriminant Analysis 

Se entrenó el modelo **Quadratic Discriminant Analysis** utilizando un subconjunto balanceado para evitar "muertes" del kernel y **variables escaladas (top 50)**.

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | 0.795 |
| Accuracy | 0.94 |
| Precision (Fraude) | 0.23 |
| Recall (Fraude) | 0.29 |
| F1-score | 0.26 |

Aunque QDA ofrece mayor flexibilidad al modelar relaciones no lineales mediante matrices de covarianza independientes por clase, presentó el desempeño más bajo entre los modelos evaluados. Esto puede explicarse por la alta dimensionalidad y complejidad del dataset, lo cual dificulta una estimación robusta de covarianzas y afecta su capacidad de generalización.

Esto significa que el modelo comete menos falsas alarmas, pero deja escapar más fraudes reales.

En fraude financiero, esta pérdida de recall suele ser problemática.

---

# KNN

Se entrenó el modelo **Quadratic Discriminant Analysis** utilizando un subconjunto balanceado para evitar "muertes" del kernel y **variables escaladas (top 20)**

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | 0.711 |
| Accuracy | 0.97 |
| Precision (Fraude) | **0.65** |
| Recall (Fraude) | 0.24 |
| F1-score | 0.35 |

El modelo logró una precision alta: **65% de las alertas eran realmente fraude**

Sin embargo solo detectó: **24% de los fraudes reales**

---

# Modelo XGBoost

## Qué se hizo

Se entrenó un modelo **XGBoost**, ampliamente utilizado en problemas tabulares y fraude financiero.

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | **0.903** |
| Accuracy | 0.89 |
| Precision (Fraude) | 0.20 |
| Recall (Fraude) | **0.75** |
| F1-score | **0.31** |

### Matriz de Confusión

| | Pred No Fraude | Pred Fraude |
|---|---:|---:|
| No fraude | 101,585 | 12,459 |
| Fraude | 1,029 | 3,035 |

XGBoost fue el modelo con mejor desempeño observado.

Hallazgos relevantes:

### Mejor ROC-AUC (0.903)

Indica mejor capacidad global de separación entre clases.

### Mejor recall (0.75)

Detecta aproximadamente **75% de los fraudes reales**

reduciendo pérdidas potenciales.

### Precision aún baja (0.20)

Aunque mejora ligeramente frente a Random Forest, el modelo sigue generando falsos positivos.

XGBoost mostró el mejor equilibrio entre sensibilidad al fraude y capacidad predictiva general, posicionándose como el modelo baseline más prometedor.

---

# Optimización de Threshold

## Qué se hizo

Se evaluó el impacto de distintos umbrales de clasificación sobre precision y recall.

### Resultados observados

| Threshold | Precision | Recall |
|------------|------------|--------|
| 0.1 | 0.044 | **0.987** |
| 0.2 | 0.072 | 0.941 |
| 0.3 | 0.108 | 0.880 |
| 0.4 | 0.147 | 0.820 |
| 0.5 | 0.196 | 0.747 |

Se observa un trade-off claro:

### Threshold bajo

- Recall muy alto
- Muchos falsos positivos

### Threshold alto

- Precision mejora
- Recall disminuye

La elección del threshold depende del objetivo de negocio:

- Minimizar pérdidas → priorizar recall
- Minimizar fricción → priorizar precision

---

# Selección de Variables con L1 Regularization

## Qué se hizo

Se aplicó **regularización L1 (Lasso)** para seleccionar variables relevantes.

### Resultados

- Variables iniciales: **484**
- Variables seleccionadas: **420**

Variables relevantes observadas:

```text
TransactionDT
TransactionAmt
card3
C2
C3
D2
D8
D10
V47
V70
V156
V262
ProductCD_R
dist1_missing
```

La reducción de variables fue relativamente pequeña.

Esto indica que gran parte del dataset contiene información útil.

No hubo una reducción agresiva de dimensionalidad.


---

# Random Forest con Variables Seleccionadas

## Qué se hizo

Se reentrenó Random Forest utilizando únicamente las variables seleccionadas por L1.

### Resultados

| Métrica | Valor |
|----------|--------|
| ROC-AUC | **0.881** |
| Accuracy | 0.86 |
| Precision (Fraude) | 0.16 |
| Recall (Fraude) | **0.73** |
| F1-score | 0.27 |

El desempeño se mantuvo muy similar al Random Forest original.

Esto sugiere que:

- Se logró simplificar parcialmente el modelo
- Sin deterioro severo del rendimiento

No obstante, la mejora fue marginal.

---

# Sobre el uso de...

**1. K-Fold**
No es recomendable usar K-Fold aleatorio en fraude porque puede mezclar pasado y futuro (leakage temporal). Es mejor un temporal split o validación temporal, ya que simula un entorno real.

**2. Bootstrap**
Sirve para medir estabilidad e incertidumbre de las métricas, pero en este proyecto suele ser innecesario o demasiado complejo (overkill) si no se requiere análisis estadístico avanzado.

**3. Forward / Backward Selection**
No son prácticos en datasets con muchas variables como IEEE-CIS Fraud Detection. Son costosos y poco eficientes; se prefieren métodos como importancia de variables o regularización.

**4. PCA**
No suele ser adecuado en fraude porque reduce la interpretabilidad y puede eliminar patrones importantes. Por eso se prefieren variables originales y modelos como XGBoost que manejan alta dimensionalidad.

---
Puedes ver el modelado [en este archivo](./04_modeling_baseline.ipynb).
---
# 05. Comparación de Modelos y Resultados Finales

## Objetivo

El objetivo de esta etapa fue comparar el desempeño de los distintos modelos entrenados para identificar cuál ofrece el mejor equilibrio entre:

- Capacidad predictiva
- Detección de fraude
- Precisión operativa

Debido al fuerte desbalanceo del problema, la comparación se centró principalmente en:

- **ROC-AUC**
- **Recall de fraude**
- **Precision de fraude**
- **F1-score**

Más allá de accuracy, ya que esta métrica puede resultar engañosa en problemas altamente desbalanceados.

---

# Comparación General de Modelos

## Resultados obtenidos

| Modelo | ROC-AUC | Precision (Fraude) | Recall (Fraude) | F1-score |
|----------|----------|--------------------|-----------------|-----------|
| **XGBoost (Full)** | **0.903** | 0.20 | **0.75** | 0.31 |
| XGBoost (L1) | 0.881 | 0.16 | 0.73 | 0.27 |
| Random Forest | 0.875 | 0.18 | 0.69 | 0.28 |
| Logistic Regression | 0.819 | 0.11 | 0.73 | 0.19 |
| LDA | 0.810 | 0.32 | 0.33 | 0.32 |
| QDA | 0.795 | 0.23 | 0.29 | 0.26 |
| KNN | 0.711 | **0.65** | 0.24 | **0.35** |

---

# Comparación por ROC-AUC

## Qué se hizo

Se comparó la capacidad de separación entre fraude y no fraude utilizando **ROC-AUC**, una métrica especialmente útil en datasets desbalanceados.

El mejor resultado fue obtenido por:

### **XGBoost (Full)**

```text
ROC-AUC = 0.903
```

Esto indica que el modelo posee una **excelente capacidad para diferenciar transacciones fraudulentas y legítimas**.

También destacan:

- **XGBoost (L1)** → 0.881
- **Random Forest** → 0.875

Mientras que los modelos lineales y basados en distancia mostraron menor capacidad discriminativa. Los modelos basados en árboles de decisión (*tree-based models*) mostraron mejor desempeño global frente a modelos lineales, probablemente debido a su capacidad para capturar relaciones no lineales complejas presentes en fraude financiero.

---

# Comparación Recall vs Precision

Se comparó el trade-off entre:

- **Recall de fraude** → capacidad de detectar fraudes reales.
- **Precision de fraude** → calidad de las alertas generadas.

Esto es especialmente importante en fraude financiero.

### Interpretación de negocio

#### Recall alto

Significa detectar más fraudes reales. Pero normalmente implica más falsos positivos.

#### Precision alta

Significa que las alertas generadas son más confiables. Pero puede implicar dejar escapar más fraudes.

---

# Visualización de Resultados

## ROC-AUC por Modelo

<img width="846" height="618" alt="image" src="https://github.com/user-attachments/assets/0ac6514c-de01-4951-94bc-fd6ad896374d" />

La comparación visual de ROC-AUC confirma que:

1. **XGBoost (Full)** domina el desempeño global.
2. Los modelos basados en árboles superan consistentemente a modelos lineales.
3. KNN presenta el peor desempeño de separación.

## Precision vs Recall

<img width="779" height="547" alt="image" src="https://github.com/user-attachments/assets/38dcd339-99cc-4445-a35f-a8c8caf6c919" />

La visualización de precision vs recall mostró claramente:

- **XGBoost** como mejor balance global.
- **KNN** como modelo altamente preciso pero poco sensible.
- **Logistic Regression** como modelo sensible pero poco preciso.

---

# Modelo Seleccionado

###  XGBoost (Full)**

### Resultados principales

| Métrica | Valor |
|----------|--------|
| ROC-AUC | **0.903** |
| Recall Fraude | **0.75** |
| Precision Fraude | 0.20 |
| F1-score | 0.31 |

El modelo fue seleccionado porque:

- Obtuvo el **mejor ROC-AUC**
- Alcanzó el **mayor recall de fraude**
- Mantuvo un balance razonable entre recall y precision

En un problema de fraude detectar la mayor cantidad posible de fraudes suele ser prioritario frente a minimizar falsas alarmas.

---
Puedes ver un resumen de los resultados [en este archivo](./05_model_comparison.ipynb).
---
