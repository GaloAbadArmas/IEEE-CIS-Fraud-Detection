# IEEE-CIS-Fraud-Detection
Aquí tienes la misma sección estructurada profesionalmente en español técnico, manteniendo los estándares de calidad que un reclutador o un equipo de ciencia de datos busca en un portfolio de alto nivel:

Markdown
## 1. Comprensión y Perfilado de Datos (Data Understanding)

### 🎯 Objetivo
El objetivo principal de esta fase fue analizar exhaustivamente la estructura del dataset, evaluar la calidad de los datos e identificar los desafíos técnicos antes de iniciar el preprocesamiento y el entrenamiento de los modelos de Aprendizaje Automático (Machine Learning) para la **Detección de Fraude**.

Durante esta etapa, se auditaron los siguientes aspectos:
* **Dimensiones del Dataset:** Evaluación de la escala y el volumen de las operaciones financieras.
* **Distribución de la Variable Objetivo:** Análisis del desbalanceo de clases (transacciones fraudulentas vs. legítimas).
* **Calidad y Completitud de los Datos:** Identificación de valores faltantes (NaNs) y anomalías estructurales.
* **Consistencia entre Tablas:** Asegurar la alineación estructural entre las bases de datos de transacciones y de identidad.
* **Tipado de Variables:** Clasificación de las características en variables categóricas, numéricas y de ingeniería de atributos.

---

### 📋 Descripción de Variables
El dataset está compuesto por dos estructuras principales: **Datos Transaccionales** y **Datos de Identidad**. Para cumplir con las normativas de privacidad y seguridad, múltiples variables han sido anonimizadas o enmascaradas por el proveedor de los datos.

#### A. Variables Transaccionales
Estas variables capturan la mecánica central de cada operación financiera:

| Variable | Descripción | Tipo |
| :--- | :--- | :--- |
| **TransactionDT** | Diferencia temporal (timedelta) desde una fecha de referencia (no es un timestamp real). | Numérica |
| **TransactionAmt** | Monto de la transacción en USD. | Numérica |
| **ProductCD** | Código del producto asociado a la transacción. | Categórica |
| **card1 - card6** | Atributos de la tarjeta de pago (tipo, categoría, banco emisor, país). | Categórica / Enmascarada |
| **addr1 - addr2** | Códigos de región de la dirección de facturación/envío. | Categórica |
| **dist1 - dist2** | Métricas de distancia relacionadas con la ubicación o el comportamiento transaccional. | Numérica |
| **P_emaildomain** | Dominio de correo electrónico del comprador (Purchaser). | Categórica |
| **R_emaildomain** | Dominio de correo electrónico del receptor (Recipient). | Categórica |
| **C1 - C14** | Variables de conteo que registran asociaciones de entidades (tarjeta, dirección, etc.). | Numérica / Anonimizada |
| **D1 - D15** | Deltas de tiempo (ej. días transcurridos desde una transacción previa). | Numérica / Anonimizada |
| **M1 - M9** | Variables de coincidencia que evalúan inconsistencias (nombre, dirección, titular). | Categórica (Binaria) |
| **V1 - V339** | Variables generadas por Vesta (rankings, conteos y relaciones entre entidades). | Numérica / Enmascarada |

> 🛠️ **Nota de Preprocesamiento Categórico:** Las siguientes variables transaccionales requerirán técnicas de codificación (como *Target Encoding* o *One-Hot Encoding*) durante el pipeline de ingeniería de características: `ProductCD`, `card1 - card6`, `addr1`, `addr2`, `P_emaildomain`, `R_emaildomain` y `M1 - M9`.

---

#### B. Tabla de Identidad (Identity Data)
El dataset de identidad contiene indicadores de la huella digital asociados al dispositivo o al entorno de red que ejecuta la transacción. Esta información es recopilada mediante sistemas antifraude y capas de ciberseguridad.

**Atributos clave capturados:**
* **Red y Conectividad:** Rastreo de direcciones IP, Proveedor de Servicios de Internet (ISP) e indicadores de uso de proxy o VPN.
* **Huella Digital del Dispositivo:** Tipo de navegador web, sistema operativo (OS), versión del software y características del hardware.

> 🔒 **Protección de Privacidad:** Debido a estrictas políticas de protección de datos, los nombres reales de estas variables están anonimizados (representados desde `id_12` hasta `id_38`) y no cuentan con un diccionario de metadatos explícito.

**Variables Categóricas de Identidad para el Pipeline:**
* `DeviceType` (ej. móvil, escritorio)
* `DeviceInfo` (ej. modelo del dispositivo, compilación del sistema operativo)
* `id_12` a `id_38`
