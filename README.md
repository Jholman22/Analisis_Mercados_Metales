# 🥇 Análisis y Predicción del Mercado del Oro (XAUUSD) en Temporalidad de 1 Hora

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Jholman22/Proyecto_Analisis_Mercados_Metales/blob/main/ANALISIS_MERCADO_ORO.ipynb)

Proyecto de *machine learning* aplicado a series de tiempo financieras. Entrena modelos **LightGBM** para predecir los precios **High** y **Low** de cada vela horaria del oro (XAUUSD), con el fin de apoyar estrategias de trading intradía.

---

## 📌 Descripción

El notebook recorre el flujo completo de un proyecto de predicción:

1. Descarga de datos históricos del oro desde **MetaTrader 5** (velas de 1 hora).
2. Selección de retardos (*lags*) mediante **información mutua**.
3. Ingeniería de características estadísticas sobre ventanas móviles.
4. Optimización de hiperparámetros con **LGBMTuner** (basado en Optuna).
5. Evaluación sobre un conjunto de prueba y predicción del siguiente valor.
6. Visualización interactiva de resultados con **Plotly**.

## 🧠 Metodología

### Datos
- **Activo:** XAUUSD (oro vs. dólar)
- **Temporalidad:** 1 hora (H1)
- **Fuente:** MetaTrader 5 (`copy_rates_range`), con actualización incremental de un CSV local (`XAUUSD_H1_data.csv`)
- **Periodo:** último año por defecto
- **Columnas:** Open, High, Low, Close, Volume, spread, real_volume
- **Limpieza:** se eliminan filas con valores en cero en OHLC

### Ingeniería de características
| Técnica | Descripción |
|---|---|
| **Lags** | Valores rezagados del precio (por defecto 2, 3, 4 y 5 periodos) |
| **Información mutua** | `mutual_information_lag` puntúa hasta 100 retardos y selecciona los *k* más relevantes |
| **Estadísticas móviles** | Media y varianza en ventana deslizante |
| **Momentos L (L1–L5)** | Medidas robustas de forma de la distribución (librería `lmoments3`) |
| **Coeficiente de Gini** | Desigualdad de los valores dentro de la ventana |

Las características se calculan para Open, Close, Volume y para el precio complementario (Low para predecir High, y viceversa).

### Modelo
- **Algoritmo:** `LGBMRegressor` mediante `verstack.LGBMTuner`
- **Métrica de optimización:** RMSE
- **Ensayos:** 150
- **Espacio de búsqueda:** `bagging_fraction`, `min_sum_hessian_in_leaf`, `num_leaves`, `feature_fraction`, `learning_rate`
- **Validación:** las últimas 90 velas se reservan como conjunto de prueba

## 📊 Resultados

Ejecución para el precio **High**:

| Métrica | Valor |
|---|---|
| Mejor RMSE de validación (Optuna, trial 105) | 2.14 |
| MAE (prueba) | 2.67 |
| RMSE (prueba) | 3.49 |
| Tiempo de entrenamiento | ~2 min 44 s |
| Predicción High | 2924.29 |
| Porcentaje de error* | 14.7 % |

\* Cifra reportada en el informe del notebook como error de dirección; el cálculo no aparece en el código del notebook.

El notebook genera además tres gráficas HTML interactivas: `low_prediction.html`, `high_prediction.html` y `full_analysis.html` (High, Low y Close con selector de rango).

## 📈 Estrategia de trading propuesta

Descrita a nivel conceptual en el informe (no está implementada en el código):

- **Compra** al alcanzar el nivel *Low* predicho.
- **Cierre parcial** de 5 lotes antes de llegar al nivel *High*.
- **Venta** al alcanzar el nivel *High* predicho.
- **Cierre parcial** de 5 lotes antes de descender al nivel *Low*.
- **Stop loss** de 3 lotes.

## 🛠️ Tecnologías

- Python 3
- pandas, NumPy, SciPy
- scikit-learn
- LightGBM, verstack (LGBMTuner / Optuna)
- lmoments3
- MetaTrader5
- Matplotlib, Plotly

## 🚀 Instalación y uso

```bash
git clone https://github.com/Jholman22/Proyecto_Analisis_Mercados_Metales.git
cd Proyecto_Analisis_Mercados_Metales
pip install pandas numpy scipy scikit-learn lightgbm verstack lmoments3 matplotlib plotly MetaTrader5
jupyter notebook ANALISIS_MERCADO_ORO.ipynb
```

> ⚠️ El paquete `MetaTrader5` solo funciona en **Windows** y requiere la terminal de MetaTrader 5 instalada e iniciada. En Google Colab o Linux, omite esa celda y carga un CSV propio con columnas `Open, High, Low, Close, Volume` e índice de fecha.

Después ejecuta las celdas en orden: carga de datos → funciones de características → entrenamiento (`prediction(...)`) → gráficas.

## ⚠️ Limitaciones y mejoras pendientes

- **Conjunto de prueba pequeño:** 90 velas (~4 días de mercado). Se recomienda validación *walk-forward* con periodos más largos.
- **Posible fuga de información:** las columnas originales Open, Close, Volume y Low de la misma vela forman parte de las variables de entrada al predecir su High. En operación real, Close y Low solo se conocen al cerrar la vela. Conviene usar únicamente valores rezagados.
- **Definición de "siguiente hora":** la predicción final usa las características de la última vela disponible; revisar que corresponda realmente a la vela futura.
- **Costos de operación:** no se consideran spread, comisiones ni deslizamiento.
- **Sin *backtesting*:** la estrategia de lotes y stop loss aún no está simulada.
- **Variables externas:** no se incluyen noticias, calendario macroeconómico ni datos del dólar o rendimientos de bonos.
- **Nombres de columnas:** `Roll_Stats` calcula varianza pero la nombra `_std`.

## 🗂️ Estructura

```
├── ANALISIS_MERCADO_ORO.ipynb   # Notebook principal
├── XAUUSD_H1_data.csv           # Datos descargados (generado al ejecutar)
├── low_prediction.html          # Gráfica interactiva Low (generada)
├── high_prediction.html         # Gráfica interactiva High (generada)
└── full_analysis.html           # Gráfica combinada (generada)
```

## ⚖️ Aviso legal

Proyecto con fines **educativos y de investigación**. No constituye asesoría financiera. El trading con apalancamiento implica un alto riesgo de pérdida de capital.

## 👤 Autor

**Jholman22** — [GitHub](https://github.com/Jholman22)
