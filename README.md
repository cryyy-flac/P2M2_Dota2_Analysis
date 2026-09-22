# ☢️ NERV DATA ANALYSIS DIVISION

## PROJECT FINAL · MÓDULO 02
### DOTA 2 · MATCH WINNER PREDICTION

```text
╔══════════════════════════════════════════════╗
║ STATUS      :: COMPLETE                      ║
║ OPERATOR    :: JUAN                          ║
║ PROJECT     :: DOTA 2 WIN PREDICTION         ║
║ MODEL       :: GRADIENT BOOSTING             ║
║ OBJECTIVE   :: BINARY CLASSIFICATION         ║
║ FRAMEWORK   :: PYTHON / SCIKIT-LEARN         ║
╚══════════════════════════════════════════════╝
```

> **NERV DATA ANALYSIS DIVISION**  
> *Supervised Learning System — Dota 2 Professional Matches*

---

## 01 · BRIEFING

Este repositorio contiene el desarrollo del **Proyecto Final del Módulo 2: Aprendizaje Supervisado** del Diplomado en Ciencia de Datos.

El proyecto aborda un problema de **clasificación binaria** aplicado a partidas profesionales de **Dota 2**, cuyo objetivo es predecir qué equipo resultará ganador: **Radiant o Dire**.

El modelo utiliza estadísticas agregadas del desarrollo de las partidas, como diferencias de asesinatos, asistencias y patrimonio en oro, además de información relacionada con la liga, región y formato de la serie.

```text
MISSION OBJECTIVE
──────────────────────────────────────────────
INPUT  :: Match statistics
TASK   :: Binary classification
TARGET :: Radiant Win / Dire Win
OUTPUT :: Predicted winning team
```

---

## 02 · PROBLEM STATEMENT

Dota 2 es un videojuego competitivo de estrategia en tiempo real en el que dos equipos de cinco jugadores se enfrentan en una partida.

El problema abordado consiste en utilizar información estadística de una partida para estimar si **Radiant** o **Dire** será el equipo ganador.

La variable objetivo utilizada es:

```text
radiant_win
    1 → Radiant wins
    0 → Dire wins
```

El proyecto sigue un flujo completo de aprendizaje supervisado:

1. Obtención y limpieza de los datos.
2. Análisis exploratorio.
3. Ingeniería de variables.
4. Procesamiento y transformación de datos.
5. Entrenamiento de modelos de clasificación.
6. Optimización de hiperparámetros.
7. Evaluación mediante métricas.
8. Comparación de modelos.
9. Selección del modelo.
10. Interpretación de la importancia de variables.
11. Conclusiones y trabajo futuro.

---

## 03 · DATASET

### Dota 2 Matches — Pro Leagues

El conjunto de datos utilizado es **Dota 2 Matches (Pro Leagues)**, disponible públicamente en Kaggle.

**Fuente:**

https://www.kaggle.com/datasets/darianogina/dota-2-matches-pro-leagues

Los datos fueron obtenidos originalmente a partir de **OpenDota** y contienen información de partidas profesionales de Dota 2.

### Dataset original

```text
MATCHES       :: 193,773
VARIABLES     :: 130
PERIOD        :: 2011 – October 2024
SOURCE        :: OpenDota / Kaggle
```

Cada registro representa una partida completa e incluye información de la liga y serie, datos generales de la partida y estadísticas individuales de los diez jugadores.

---

## 04 · DATA PROCESSING

Antes del entrenamiento de los modelos se realizaron diferentes etapas de limpieza y transformación.

### 4.1 Eliminación de duplicados

Se eliminaron **307 registros duplicados** identificados mediante `match_id`.

### 4.2 Valores ausentes

Se eliminaron registros con valores nulos en variables clave como:

- `winner_id`
- `radiant_team_id`
- `dire_team_id`

Estos registros representaban menos del 1 % del conjunto original.

### 4.3 Partidas atípicas

Se descartaron partidas con una duración inferior a **5 minutos**, consideradas abandonos o remakes que no representan un desarrollo normal de la partida.

### 4.4 Resultado del procesamiento

Después de la limpieza y transformación, el conjunto utilizado para el análisis quedó conformado por:

```text
CLEAN DATASET
──────────────────────────────
MATCHES :: 102,743
```

---

## 05 · FEATURE ENGINEERING

Uno de los componentes principales del proyecto fue la creación de variables que representaran la **ventaja relativa entre Radiant y Dire**.

A partir de las estadísticas individuales de los cinco jugadores de cada equipo se calcularon agregaciones y posteriormente diferencias entre ambos equipos.

### Variables numéricas

| Variable | Descripción |
|---|---|
| `match_duration_seconds` | Duración total de la partida |
| `first_blood_time_seconds` | Segundo en que ocurrió el primer asesinato |
| `kills_diff` | Diferencia de asesinatos: Radiant − Dire |
| `assists_diff` | Diferencia de asistencias: Radiant − Dire |
| `networth_diff` | Diferencia de patrimonio: Radiant − Dire |
| `networth_ratio` | Razón de patrimonio: Radiant / Dire |
| `kda_diff` | Diferencia del KDA promedio entre equipos |

### Variables categóricas

| Variable | Descripción |
|---|---|
| `league_tier` | Nivel del torneo |
| `league_region` | Región de la liga |
| `series_type` | Formato de la serie |

### Variable objetivo

| Variable | Descripción |
|---|---|
| `radiant_win` | 1 si gana Radiant, 0 si gana Dire |

---

## 06 · TARGET DISTRIBUTION

La variable objetivo presentó una distribución prácticamente balanceada:

```text
RADIANT :: 50.8 %
DIRE    :: 49.2 %
```

Debido a este balance, no fue necesario aplicar técnicas adicionales de balanceo de clases.

```text
╔══════════════════════════════════════════════╗
║ TARGET DISTRIBUTION                           ║
╠══════════════════════════════════════════════╣
║ RADIANT WIN     :: 50.8%                     ║
║ DIRE WIN        :: 49.2%                     ║
║ CLASS BALANCE   :: ACCEPTABLE                ║
╚══════════════════════════════════════════════╝
```

---

## 07 · EXPLORATORY DATA ANALYSIS

El análisis exploratorio permitió identificar diferentes características del conjunto de datos.

### Duración de las partidas

La duración promedio de las partidas fue de aproximadamente:

```text
MEAN     :: 2,017 seconds
         :: ≈ 33.6 minutes

STD      :: 588 seconds
```

### Diferencial de patrimonio

El `networth_diff` presentó una relación clara con el resultado de la partida.

En términos generales:

```text
Radiant Networth Advantage
            ↓
     Higher Win Tendency

Dire Networth Advantage
            ↓
     Lower Radiant Win Tendency
```

### Nivel de las ligas

La mayor parte de las partidas correspondió a ligas clasificadas como **profesionales**, representando aproximadamente el 89 % del conjunto analizado.

---

## 08 · CORRELATION ANALYSIS

Se calculó una matriz de correlación de Pearson entre las variables numéricas.

Uno de los principales hallazgos fue la correlación prácticamente perfecta entre:

```text
kills_diff ↔ deaths_diff
```

Esta relación es esperable debido a que los asesinatos de un equipo están directamente relacionados con las muertes del equipo contrario.

Por esta razón, `deaths_diff` fue eliminado para evitar redundancia y multicolinealidad.

### Variables con mayor correlación con el objetivo

```text
networth_diff     :: r = 0.90
networth_ratio    :: r = 0.85
assists_diff      :: r = 0.84
kills_diff        :: r = 0.83
```

Estas relaciones son coherentes con la dinámica del juego: una ventaja en recursos y estadísticas de combate suele estar asociada con el resultado final.

---

## 09 · TRAIN / TEST SPLIT

Debido a la capacidad de cómputo disponible, se tomó una muestra aleatoria estratificada de:

```text
MODEL DATASET :: 30,000 matches
```

La partición utilizada fue:

```text
TRAINING :: 80% → 24,000 matches
TEST     :: 20% →  6,000 matches
```

La división se realizó de forma estratificada respecto a `radiant_win`.

### Feature Scaling

Las variables numéricas fueron estandarizadas mediante:

```text
StandardScaler
```

El escalamiento fue ajustado únicamente sobre el conjunto de entrenamiento para evitar fuga de información.

### Categorical Encoding

Las variables categóricas fueron transformadas mediante:

```text
One-Hot Encoding
```

Variables:

- `league_tier`
- `league_region`
- `series_type`

No se aplicó reducción de dimensionalidad, debido al número reducido de variables predictoras.

---

## 10 · MACHINE LEARNING MODELS

Se entrenaron y compararon tres algoritmos de clasificación supervisada.

```text
┌─────────────────────────────────────────────┐
│                 NERV MODELS                 │
├─────────────────────────────────────────────┤
│ 01 :: LOGISTIC REGRESSION                  │
│ 02 :: RANDOM FOREST                        │
│ 03 :: GRADIENT BOOSTING                    │
└─────────────────────────────────────────────┘
```

### 10.1 Logistic Regression

Modelo lineal utilizado como baseline.

Se utilizó regularización L2 y se optimizó el hiperparámetro `C`.

**Mejor configuración:**

```text
C :: 10
Penalty :: L2
```

---

### 10.2 Random Forest

Modelo de ensamble basado en múltiples árboles de decisión.

Se optimizaron principalmente:

- `n_estimators`
- `max_depth`

**Mejor configuración:**

```text
n_estimators :: 300
max_depth    :: 10
```

---

### 10.3 Gradient Boosting

Modelo de ensamble basado en boosting, donde los árboles son construidos secuencialmente para corregir los errores de los modelos anteriores.

Se utilizó:

```text
HistGradientBoostingClassifier
```

Los hiperparámetros optimizados fueron:

- `max_iter`
- `learning_rate`
- `max_depth`

**Mejor configuración:**

```text
max_iter       :: 200
learning_rate  :: 0.1
max_depth      :: 3
```

---

## 11 · HYPERPARAMETER OPTIMIZATION

Para los tres modelos se utilizó:

```text
GridSearchCV
Cross Validation :: k = 3
Optimization     :: ROC-AUC
```

El ROC-AUC fue utilizado como métrica de selección para evaluar la capacidad discriminativa de los clasificadores en el problema de clasificación binaria.

---

## 12 · MODEL COMPARISON

### Test Performance

| Modelo | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.9878 | 0.9879 | 0.9882 | 0.9880 | 0.9981 |
| Random Forest | 0.9878 | 0.9904 | 0.9856 | 0.9880 | 0.9989 |
| Gradient Boosting | 0.9880 | 0.9885 | 0.9878 | 0.9882 | 0.9989 |

```text
╔══════════════════════════════════════════════╗
║              MODEL EVALUATION                ║
╠══════════════════════════════════════════════╣
║ LOGISTIC REGRESSION   :: ROC-AUC 0.9981     ║
║ RANDOM FOREST         :: ROC-AUC 0.9989     ║
║ GRADIENT BOOSTING     :: ROC-AUC 0.9989     ║
╚══════════════════════════════════════════════╝
```

Los tres modelos presentaron un desempeño muy alto y similar entre sí, con valores de accuracy de prueba cercanos al 98.8 % y ROC-AUC cercanos a 0.999.

---

## 13 · SELECTED MODEL

Para el proyecto se seleccionó:

# ⚡ GRADIENT BOOSTING

El modelo presentó:

```text
ROC-AUC    :: 0.9989
F1-SCORE   :: 0.9882
ACCURACY   :: 0.9880
```

La diferencia entre las métricas de entrenamiento y prueba fue reducida:

```text
TRAIN ACCURACY :: 0.9903
TEST ACCURACY  :: 0.9880
```

```text
╔══════════════════════════════════════════════╗
║ NERV PREDICTION SYSTEM                       ║
╠══════════════════════════════════════════════╣
║ SELECTED MODEL :: GRADIENT BOOSTING         ║
║ ROC-AUC        :: 0.9989                    ║
║ F1-SCORE       :: 0.9882                    ║
║ TEST ACCURACY  :: 0.9880                    ║
║ STATUS         :: OPERATIONAL               ║
╚══════════════════════════════════════════════╝
```

---

## 14 · FEATURE IMPORTANCE

Para interpretar el modelo seleccionado se calculó **Permutation Importance**, utilizando la disminución del ROC-AUC producida al permutar cada variable.

Las variables con mayor importancia fueron:

```text
01 :: networth_diff
02 :: networth_ratio
03 :: assists_diff
```

Las variables categóricas relacionadas con la liga, región y formato de serie, así como la duración de la partida, tuvieron una contribución menor en comparación con las variables que representan la ventaja de recursos entre equipos.

---

## 15 · ⚠️ IMPORTANT MODEL CONSIDERATION

Una consideración importante del proyecto es la naturaleza temporal de las variables utilizadas.

Variables como:

- `networth_diff`
- `kills_diff`
- `assists_diff`
- `kda_diff`

representan información que se genera durante el desarrollo de la partida.

Por lo tanto, el problema abordado por el modelo debe interpretarse principalmente como:

```text
CURRENT MATCH STATE
        ↓
WIN PROBABILITY ESTIMATION
        ↓
RADIANT / DIRE
```

y no como una predicción del ganador **antes del inicio de la partida**.

El modelo aprovecha información que representa la ventaja acumulada entre ambos equipos durante el encuentro. Esto también ayuda a contextualizar el desempeño elevado obtenido por los clasificadores.

---

## 16 · CONCLUSIONS

El proyecto permitió implementar un flujo completo de aprendizaje supervisado sobre partidas profesionales de Dota 2.

Se realizó:

```text
DATA ACQUISITION
       ↓
DATA CLEANING
       ↓
EDA
       ↓
FEATURE ENGINEERING
       ↓
DATA PREPROCESSING
       ↓
MODEL TRAINING
       ↓
HYPERPARAMETER SEARCH
       ↓
MODEL EVALUATION
       ↓
FEATURE IMPORTANCE
```

Los tres modelos evaluados alcanzaron un desempeño elevado y similar.

El modelo de **Gradient Boosting** obtuvo:

```text
ROC-AUC :: 0.9989
F1      :: 0.9882
Accuracy:: 0.9880
```

El análisis de importancia de variables mostró que `networth_diff` y `networth_ratio` fueron las variables con mayor contribución al modelo.

Estos resultados son consistentes con la naturaleza de las variables utilizadas, ya que representan directamente la ventaja acumulada entre ambos equipos durante una partida.

---

## 17 · FUTURE WORK

Como continuación del proyecto se plantean diferentes líneas de trabajo.

### 01 · Pre-match Prediction

Incorporar variables disponibles **antes del inicio de la partida**, especialmente:

- Héroes seleccionados.
- Composición de los equipos.
- Información del draft.
- Historial reciente de los equipos.

El objetivo sería estudiar qué tan predecible es el resultado únicamente a partir de información previa al comienzo del encuentro.

### 02 · Historical Team Performance

Incorporar variables relacionadas con el rendimiento histórico reciente de los equipos, por ejemplo:

```text
Recent Win Rate
Recent Match Performance
Team Historical Statistics
```

### 03 · Full Dataset Training

Entrenar los modelos utilizando las aproximadamente **193 mil partidas originales** en un entorno con mayor capacidad de procesamiento.

### 04 · Real-Time Prediction

Explorar la construcción de un sistema capaz de estimar la probabilidad de victoria durante una transmisión de una partida profesional.

```text
LIVE MATCH
    ↓
CURRENT STATISTICS
    ↓
ML MODEL
    ↓
WIN PROBABILITY
    ↓
RADIANT / DIRE
```

---

## 18 · TECHNOLOGY STACK

```text
LANGUAGE
────────
Python

DATA ANALYSIS
─────────────
Pandas
NumPy

VISUALIZATION
─────────────
Matplotlib
Seaborn

MACHINE LEARNING
────────────────
Scikit-learn

MODELS
──────
Logistic Regression
Random Forest
HistGradientBoostingClassifier

DATA SOURCE
───────────
Kaggle
OpenDota

ENVIRONMENT
───────────
Jupyter Notebook
```

---

## 19 · PROJECT STRUCTURE

```text
DOTA2-WINNER-PREDICTION/
│
├── data/
│   └── dota2_matches.csv
│
├── notebooks/
│   └── dota2_winner_prediction.ipynb
│
├── figures/
│   ├── target_distribution.png
│   ├── match_duration.png
│   ├── networth_distribution.png
│   ├── league_distribution.png
│   ├── correlation_matrix.png
│   ├── model_comparison.png
│   ├── confusion_matrix.png
│   └── feature_importance.png
│
├── README.md
└── requirements.txt
```

---

## 20 · REFERENCES

### Dataset

Kaggle. (2024). *Dota 2 Matches (Pro Leagues)*.

https://www.kaggle.com/datasets/darianogina/dota-2-matches-pro-leagues

### Machine Learning

Géron, A. (2022). *Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow* (3.a ed.). O'Reilly Media.

Hastie, T., Tibshirani, R., & Friedman, J. (2009). *The Elements of Statistical Learning: Data Mining, Inference, and Prediction* (2.a ed.). Springer.

James, G., Witten, D., Hastie, T., & Tibshirani, R. (2021). *An Introduction to Statistical Learning with Applications in Python*. Springer.

Pedregosa, F., et al. (2011). *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research, 12, 2825–2830.

Ke, G., et al. (2017). *LightGBM: A Highly Efficient Gradient Boosting Decision Tree*. Advances in Neural Information Processing Systems, 30.

---

# ☢️ NERV FINAL REPORT

```text
╔══════════════════════════════════════════════╗
║              MISSION COMPLETE                ║
╠══════════════════════════════════════════════╣
║                                              ║
║  DOTA 2 DATASET        :: LOADED             ║
║  DATA PROCESSING       :: COMPLETE           ║
║  FEATURE ENGINEERING   :: COMPLETE           ║
║  MODEL TRAINING        :: COMPLETE           ║
║  MODEL EVALUATION      :: COMPLETE           ║
║  GRADIENT BOOSTING     :: ONLINE             ║
║                                              ║
║  ROC-AUC               :: 0.9989             ║
║  TEST ACCURACY         :: 98.80%             ║
║                                              ║
╚══════════════════════════════════════════════╝
```

> **MISSION STATUS :: SUCCESS**

```text
OPERATOR VERIFIED
MODEL TRAINING COMPLETE
PREDICTION SYSTEM ONLINE

OPERATOR
────────
JUAN

END OF REPORT
```

---

## 🎮 DOTA 2 · DATA SCIENCE SYSTEM

```text
"The Ancient awaits."

NERV DATA ANALYSIS DIVISION
SUPERVISED LEARNING PROJECT
DOTA 2 WINNER PREDICTION

[ SYSTEM SHUTDOWN ]
```