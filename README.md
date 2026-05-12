#  Detección de Fraude en Accidentes de Vehículos

Proyecto de Machine Learning orientado a detectar si una reclamación de accidente vehicular es fraudulenta o no, utilizando el dataset `fraud_oracle.csv`.

---

## Estructura del Proyecto

```
Fraude_accidentes/
│
├── fraud_oracle.csv        # Dataset original con 15,420 registros y 33 columnas
├── Fase1_datos.ipynb       # Notebook principal: exploración y preparación de datos
└── README.md               # Este archivo
```

---

##  Descripción del Dataset

El dataset contiene información sobre reclamaciones de seguros de automóviles. Cada fila representa una reclamación individual.

- **Total de registros:** 15,420
- **Total de columnas originales:** 33
- **Variable objetivo:** `FraudFound_P` (0 = No Fraude, 1 = Fraude)
- **Desbalance de clases:** ~94% No Fraude / ~6% Fraude

---

##  Fase 1 — Exploración y Preparación de Datos

### Lo que se hizo en el notebook:

#### 1. Carga y exploración inicial
- Carga del dataset con `pandas`.
- Visualización de las primeras filas (`df.head()`).
- Estadísticas descriptivas (`df.describe()`) — se identificó que `FraudFound_P` tiene media de 0.059, confirmando el 5.9% de fraudes.
- Revisión de tipos de datos (`df.info()`).

#### 2. Calidad de los datos
- **Valores nulos:** No se encontraron (`df.isnull().sum()` → todo en 0).
- **Duplicados:** No se encontraron registros duplicados.
- **Columna `Age` con valores 0:** Se detectaron 320 filas con `Age = 0`. Se confirmó que todas corresponden a titulares de póliza en el rango `"16 to 17"` en `AgeOfPolicyHolder`. Dado que no hay referencia real de la edad exacta en ese rango, se reemplazó el 0 por la **mediana** (mejor opción que la media ante outliers detectados en el boxplot).

#### 3. Eliminación de columnas irrelevantes
Se eliminaron columnas que no aportan valor predictivo o que representan información redundante:
- `PolicyNumber` (identificador único sin duplicados → no predictiva).
- Columnas de fecha: `Month`, `WeekOfMonth`, `DayOfWeek`, `MonthClaimed`, `WeekOfMonthClaimed`, `DayOfWeekClaimed` (ya que `Days_Policy_Accident` y `Days_Policy_Claim` capturan mejor esta información).
- `RepNumber`, `AgentType`, `AddressChange_Claim`, `Year`, `NumberOfSuppliments`, `MaritalStatus`, `AgeOfPolicyHolder` (redundante tras corregir `Age`).

**El dataset pasó de 33 a 19 columnas.**

#### 4. Codificación de variables categóricas binarias
Las siguientes columnas fueron convertidas a binario (1/0):

| Columna           | Codificación                       |
|-------------------|------------------------------------|
| `Sex`             | Female → 1, Male → 0              |
| `AccidentArea`    | Urban → 1, Rural → 0              |
| `Fault`           | Policy Holder → 1, Third Party → 0|
| `PoliceReportFiled`| Yes → 1, No → 0                  |
| `WitnessPresent`  | Yes → 1, No → 0                   |

#### 5. Codificación de variables ordinales (rangos de texto → número)
Columnas con rangos de texto se mapearon a valores numéricos representativos:

| Columna                | Ejemplo de Mapeo                             |
|------------------------|----------------------------------------------|
| `VehiclePrice`         | "less than 20000" → 15000, "more than 69000" → 70000 |
| `AgeOfVehicle`         | "new" → 1, "7 years" → 7, "more than 7" → 10 |
| `PastNumberOfClaims`   | "none" → 0, "1" → 1, "2 to 4" → 3, "more than 4" → 5 |
| `NumberOfCars`         | "1 vehicle" → 1, "2 vehicles" → 2, etc.    |
| `Days_Policy_Accident` | "none" → 0, "1 to 7" → 4, "more than 30" → 40 |
| `Days_Policy_Claim`    | "none" → 0, "more than 30" → 40            |

#### 6. Limpieza y agrupación de la columna `Make`
- Se corrigieron errores tipográficos: `Accura → Acura`, `Nisson → Nissan`, `Porche → Porsche`, `Mecedes → Mercedes`.
- Las 8 marcas más frecuentes se mantuvieron; el resto se agrupó bajo `"Other"`.

#### 7. One-Hot Encoding (variables nominales)
Se aplicó `pd.get_dummies` a: `Make`, `PolicyType`, `VehicleCategory`, `BasePolicy`.

**El dataset final quedó con 35 columnas, todas numéricas.**

#### 8. Análisis de correlación
Se calculó la correlación de cada variable con `FraudFound_P`. Las variables con mayor correlación (|r| > 0.03):

| Variable                     | Correlación |
|------------------------------|-------------|
| `Fault`                      | +0.131      |
| `PolicyType_Sport - Collision` | +0.050    |
| `BasePolicy_Collision`       | +0.044      |
| `PastNumberOfClaims`         | -0.057      |
| `VehicleCategory_Sport`      | -0.136      |
| `PolicyType_Sedan - Liability` | -0.153    |
| `BasePolicy_Liability`       | -0.154      |

>  Las correlaciones lineales son bajas en general

---

##  Tecnologías utilizadas

- **Python 3.12**
- **Pandas** — Manipulación y análisis de datos
- **NumPy** — Operaciones numéricas
- **Matplotlib** — Visualizaciones básicas (boxplot)
- **Seaborn** — Visualizaciones estadísticas

---

##  Próximos pasos

- [ ] Exploración visual adicional (heatmap de correlación completo, distribuciones por clase).
- [ ] Selección de características relevantes.
- [ ] Entrenamiento de modelos de clasificación (Árbol de decisión, Random Forest, XGBoost, etc.).
- [ ] Manejo del desbalance de clases (SMOTE, class_weight, etc.).
- [ ] Evaluación de métricas (Recall, Precision, AUC-ROC).
---

##  Actualizacion: comparacion con PyCaret

Al final de `Fase1_datos.ipynb` se anadio una celda simplificada que:

- Usa el datasheet ya procesado durante el notebook (`df`).
- Entrena y compara modelos con PyCaret.
- Muestra solo las metricas por modelo:
  `Model`, `Accuracy`, `AUC`, `Recall`, `Prec.`, `F1`, `Kappa`, `MCC`.

Significado de metricas:
- `Accuracy`: proporcion total de predicciones correctas.
- `AUC`: capacidad del modelo para separar clases (mas alto, mejor discriminacion).
- `Recall`: porcentaje de fraudes reales detectados.
- `Prec.` (Precision): porcentaje de alertas de fraude que realmente son fraude.
- `F1`: balance entre precision y recall.
- `Kappa`: acuerdo del modelo respecto a la verdad, ajustado por azar.
- `MCC`: correlacion entre predicciones y clases reales, robusta en datos desbalanceados.
