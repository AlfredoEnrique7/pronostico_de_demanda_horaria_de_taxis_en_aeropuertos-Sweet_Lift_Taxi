# Pronóstico de Demanda Horaria de Taxis en Aeropuertos — Sweet Lift Taxi

![Python](https://shields.io)
![Pandas](https://shields.io)
![Scikit-Learn](https://shields.io)
![LightGBM](https://shields.io)

## Descripción del Proyecto
La compañía **Sweet Lift Taxi** ha recopilado datos históricos sobre los pedidos de taxis en los aeropuertos entre marzo y agosto de 2018 (registrados en intervalos de 10 minutos). Durante las horas pico, la demanda supera la oferta disponible, generando tiempos de espera prolongados y pérdida de viajes.

El objetivo de este proyecto es desarrollar y validar un modelo de **series temporales** capaz de **predecir la cantidad de pedidos de taxis para la próxima hora**, alcanzando un Error Cuadrático Medio Raíz (**RECM / RMSE**) **no superior a 48** en el conjunto de prueba, permitiendo así incentivar a los conductores con anticipación.

---

## Hallazgos Clave del Análisis Exploratorio
* **Estructura temporal:** El conjunto original cuenta con **26,496 registros** cada 10 minutos sin valores ausentes, los cuales fueron remuestreados a intervalos de **1 hora** (`4,416` observaciones).
* **Tendencia y estacionariedad:** La demanda presenta un crecimiento sostenido desde marzo (~55–60 pedidos/hora) hasta agosto (~160–170 pedidos/hora), periodo en el que la desviación estándar móvil casi se duplica. La prueba formal de **Dickey-Fuller Aumentada (ADF = `-3.0689`, `p-value = 0.0289`)** confirma que la serie es estacionaria alrededor de una tendencia determinista (*trend-stationary*).
* **Estacionalidad diaria dominante:** Existe un ciclo de 24 horas muy marcado con su **pico máximo a las 00:00 (medianoche)** (~144 pedidos/hora en promedio), un **valle mínimo a las 06:00 a.m.** (~25 pedidos/hora) y repuntes secundarios alrededor de las 10:00 y 16:00–17:00 horas.
* **Patrón semanal:** Los días con mayor promedio son los **viernes y lunes** (~90–91 pedidos/hora), representando un efecto secundario frente a la fuerte variación horaria.

---

## Metodología e Ingeniería de Características
Para evitar cualquier fuga de datos (*data leakage*), todas las variables históricas fueron desplazadas (`shift()`) antes de su cálculo:
1. **Variables de calendario:** Mes (`month`), día (`day`), día de la semana (`dayofweek`) y hora (`hour`).
2. **Desfases temporales (*Lags*):** 24 desfases horarios (`lag_1` a `lag_24`) para capturar tanto la inercia reciente como el comportamiento de la misma hora del día anterior.
3. **Media móvil desplazada:** Promedio móvil con ventana de 24 horas (`rolling_mean`).
4. **Validación cronológica:** División estricta sin mezclar (`shuffle=False`) en **90 % entrenamiento** (`3,950` registros tras eliminar nulos iniciales) y **10 % prueba** (`442` registros), optimizando hiperparámetros con `TimeSeriesSplit(n_splits=5)`.

---

## Resultados y Comparación de Modelos

| Modelo | Hiperparámetros Óptimos | RECM Validación (CV) | RECM Prueba (Test) | Estado (Umbral ≤ 48) |
| :--- | :--- | :---: | :---: | :---: |
| **Modelo Ingenuo** *(Hora anterior)* | — | — | `58.86` | ❌ No cumple |
| **Regresión Lineal** *(Baseline)* | Por defecto | `26.94` | `45.81` | ✅ Cumple |
| **Random Forest Regressor** | `n_estimators=200`, `min_samples_split=5`, `max_depth=None` | `25.13` | `43.46` | ✅ Cumple |
| **LightGBM Regressor** | `learning_rate=0.05`, `n_estimators=200`, `num_leaves=15` | **`24.77`** | **`40.83`** | ✅ **Mejor modelo** |

---

## Conclusiones y Recomendación de Negocio
* **Despliegue recomendado:** Implementar **LightGBM Regressor** en producción, ya que obtiene el menor error en datos no vistos (**RECM = 40.83**), superando en más de **18 pedidos/hora** al modelo ingenuo y dejando un margen de **7.17 puntos** respecto al límite máximo permitido (`48`) durante el mes de mayor demanda del año.
* **Consideración operativa:** Si bien el modelo reproduce con fidelidad el ritmo diario de 24 horas, tiende a suavizar los picos atípicos más extremos (~340–400 pedidos/hora). Se recomienda acompañar las alertas del modelo con un **margen de seguridad adicional de flota** en eventos extraordinarios y **reentrenar el modelo periódicamente** para adaptarse al crecimiento continuo de la demanda.

---

## Estructura del Repositorio y Ejecución
```text
├── data/
│   └── taxi.csv               # Conjunto de datos históricos (intervalos de 10 min)
├── notebook/
│   └── pronóstico_de_demanda_horaria_de_taxis_en_aeropuertos—Sweet_Lift Taxi.ipynb # Jupyter Notebook con análisis, modelado y evaluación
├── requirements.txt           # Dependencias del proyecto
└── README.md                  # Documentación principal
```

### Instalación rápida
```bash
git clone https://github.com
cd sweet-lift-taxi-forecasting
pip install pandas numpy matplotlib statsmodels scikit-learn lightgbm
```
