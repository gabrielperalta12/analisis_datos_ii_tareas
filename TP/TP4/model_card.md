# Model Card · Regresión de ingreso anual

**Versión:** 1.1 · **Fecha:** 18/10/2026 · **Responsable:** Victor Gabriel Peralta

## Propósito y alcance
Mi modelo estima ingreso anual continuo como ejercicio académico de target engineering, prevención de leakage y auditoría. No está validado para decisiones individuales de contratación, remuneración, crédito o acceso a servicios.

## Detalles técnicos y reproducibilidad
Regresión lineal ordinaria de scikit-learn, con intercepto, sin regularización y sin búsqueda de hiperparámetros. Modelo log-lineal: ln(income) = intercepto + combinación lineal de entradas procesadas; la salida se obtiene con exp. La no linealidad en escala original proviene de esta inversa. One-hot elimina la primera categoría de cada entrada y usa handle_unknown='ignore'; no se construyeron interacciones.

Python 3.12.13; versiones: {'numpy': '1.26.4', 'pandas': '2.3.3', 'scikit-learn': '1.9.1', 'fairlearn': '0.15.0', 'matplotlib': '3.11.2'}. Entorno definido por `../pyproject.toml` y `../uv.lock`, sincronizado con `uv sync`. Semilla de partición, folds, bootstrap y permutaciones: 42. No exporté un modelo serializado ni desplegué un servicio.

## Datos, entradas y salida
Archivo local `census_income.csv` provisto con la actividad. Su origen, año, moneda y mecanismo de muestreo no están documentados en el CSV; no aseguro representatividad censal. Hay 10,000 filas originales y 9,914 registros tras remover duplicados exactos. Desarrollo: 7,931; test: 1,983 (80/20, semilla 42).

Entradas: age, hours_per_week, education, employer_type, marital_status, sex, race, born_in_us. Salida: estimación de `income` en unidades originales de ingreso anual. Se utilizan directamente `sex` y `race`, atributos sensibles; otras entradas también pueden funcionar como proxies.

La [Data Card](data_card.md) documenta esquema, faltantes, duplicados, representación por grupo, rangos de desarrollo y aspectos desconocidos de procedencia y licencia.

## Método y prevención de leakage
Modelo elegido: **Lineal log**. Imputación numérica por mediana y estandarización; imputación categórica con `Sin dato` y one-hot. Todo el preprocesamiento se ajusta dentro de cada fold. La variante log usa `TransformedTargetRegressor` con log/exp. Selección por MAE en cinco folds idénticos de desarrollo; test reservado para evaluación final. No hay variables derivadas de income entre las entradas. La disponibilidad de las características al momento de predecir es un supuesto que debería comprobarse en una aplicación real.

### Checklist de leakage realizado
- Target fuera de las entradas y sin variables construidas a partir de ella.
- Duplicados exactos removidos antes de dividir; no se demuestra independencia de personas con registros diferentes.
- Índices de desarrollo y test disjuntos, comprobados con una aserción.
- Imputación, estandarización y one-hot ajustados dentro de cada fold mediante pipeline.
- Log sin parámetros estimados; inversión automática al predecir. No se ajustó Box-Cox/Yeo-Johnson globalmente.
- Mismos cinco folds para los candidatos; selección por MAE en desarrollo, sin usar test para seleccionar.
- Test usado para evaluación y diagnósticos del modelo fijo. Cualquier nuevo ajuste motivado por esos diagnósticos requeriría nueva evaluación independiente.
- Disponibilidad temporal de entradas, parentescos y entidades repetidas: no verificables con el archivo; no los doy por resueltos.

## Desempeño
Validación cruzada (MAE y RMSE en escala original; MAE_sd es la desviación estándar del MAE entre folds):

| modelo           |   MAE_medio |   MAE_sd |   RMSE_medio |
|:-----------------|------------:|---------:|-------------:|
| Lineal log       |    29547.82 |  1605.24 |     61697.15 |
| Lineal original  |    31431.65 |  1043.57 |     58558.84 |
| Baseline mediana |    36611.36 |  3309.98 |     70577.36 |

Test final: MAE **29,439.16**; RMSE **60,908.87**; R² **0.172**. Baseline mediana: MAE **35,590.13**.

La transformación redujo la magnitud de la skewness de 4,409 a 1,005, pero dejó asimetría negativa; no se estableció normalidad. Mejoró el MAE medio de validación un 5,99%, mientras el RMSE medio aumentó de 58.558,84 a 61.697,15. La elección prioriza MAE y no implica una mejora en todas las métricas. El R² de test de 0,172 muestra una capacidad explicativa limitada.

## Auditoría en test
Error medio = predicho - real. MAE/ingreso medio contextualiza escala, no es una medida suficiente de equidad.

### Sexo
| sex    |      n |      MAE |     RMSE |   Error_medio |   Ingreso_medio |   MAE_sobre_ingreso_medio |
|:-------|-------:|---------:|---------:|--------------:|----------------:|--------------------------:|
| Female | 993.00 | 21721.34 | 41952.23 |      -7720.73 |        40993.92 |                      0.53 |
| Male   | 990.00 | 37180.35 | 75270.84 |     -12326.03 |        66637.48 |                      0.56 |

### Raza
| race              |       n |      MAE |     RMSE |   Error_medio |   Ingreso_medio |   MAE_sobre_ingreso_medio |
|:------------------|--------:|---------:|---------:|--------------:|----------------:|--------------------------:|
| Asian             |   88.00 | 27640.53 | 40156.02 |      -7487.58 |        50515.80 |                      0.55 |
| Black             |  460.00 | 20098.07 | 35977.85 |      -8886.91 |        42020.00 |                      0.48 |
| Other or multiple |   83.00 | 17846.13 | 32548.79 |       3190.28 |        32399.88 |                      0.55 |
| White             | 1352.00 | 33446.11 | 69505.08 |     -11381.18 |        59330.10 |                      0.56 |

### Incertidumbre de diferencias de MAE
| atributo   | grupo             | referencia   |   diferencia_MAE |   IC95_inf |   IC95_sup |
|:-----------|:------------------|:-------------|-----------------:|-----------:|-----------:|
| sex        | Female            | Male         |        -15459.01 |  -20118.81 |  -10838.52 |
| race       | Asian             | White        |         -5805.58 |  -12962.78 |     938.18 |
| race       | Black             | White        |        -13348.04 |  -17350.29 |   -9050.41 |
| race       | Other or multiple | White        |        -15599.97 |  -22111.89 |   -8042.55 |

En test, ambos grupos de sexo presentan subestimación media; es mayor en valor absoluto en `Male`. Por raza, hay subestimación media en `Asian`, `Black` y `White`, y sobreestimación en `Other or multiple`. Los grupos `Asian` y `Other or multiple` tienen 88 y 83 observaciones, respectivamente. Los intervalos bootstrap son exploratorios, condicionados al modelo, y no tienen ajuste por comparaciones múltiples.

Las brechas describen errores respecto de los ingresos observados. No prueban discriminación causal ni certifican equidad. Las etiquetas pueden contener desigualdades históricas. No se aplicó mitigación de sesgos.

## Explicabilidad: variables principales
Importancia por permutación sobre test con diez repeticiones, modelo final fijo y scoring de MAE en unidades originales. Se permuta cada columna original antes de su preprocesamiento. El aumento medio del MAE indica dependencia predictiva; valores negativos o cercanos a cero no demuestran irrelevancia ni ausencia de sesgo.

| variable       |   aumento_MAE |   sd_permutaciones |
|:---------------|--------------:|-------------------:|
| hours_per_week |       5647.82 |             370.46 |
| education      |       5047.33 |             358.38 |
| age            |       2745.07 |             466.61 |
| employer_type  |        761.89 |             105.09 |
| marital_status |        524.70 |             112.52 |
| sex            |        368.86 |             122.40 |
| race           |         27.28 |              68.10 |
| born_in_us     |          4.74 |              19.33 |

Variables con mayor aumento medio de MAE: **hours_per_week, education, age**. Las correlaciones entre entradas limitan la interpretación. No hice análisis causal, SHAP por grupo ni evaluación contrafactual; la permutación no explica por sí sola las causas de las disparidades.

## Consideraciones éticas y factores no evaluados
Usuarios previstos: estudiante y docentes para evaluación académica. Personas potencialmente afectadas por un uso indebido: individuos cuyos ingresos se estimen y grupos definidos por atributos sensibles. Los errores podrían generar perjuicios si se reutilizan para fijar salarios, excluir candidatos o asignar crédito; no recomiendo esos usos.

La auditoría usa sexo y raza registrados; no evalúa intersecciones, bandas de edad, nacimiento en EE. UU. ni todos los contextos socioeconómicos. No se comprobó si el modelo amplifica diferencias respecto de un mecanismo causal o un criterio normativo. La métrica principal es MAE por interpretabilidad en unidades de ingreso; no se definió un umbral de equidad universal. No se aplicó mitigación ni se certifica aptitud para producción. Revisión humana, verificación de procedencia y una evaluación contextual serían necesarias antes de cualquier aplicación distinta de la académica.

## Limitaciones y uso responsable
El modelo lineal limita las relaciones e interacciones representables; los errores en ingresos extremos pueden ser grandes. La transformación log modifica la función objetivo y su inversa no corrige automáticamente el sesgo para estimar la media. La eliminación de duplicados puede remover personas distintas con los mismos atributos. Sin identificadores y fechas no pude verificar dependencia entre observaciones ni desempeño futuro. Las categorías observadas no representan necesariamente todas las identidades. Los grupos pequeños y las intersecciones no evaluadas requieren más datos. La auditoría no considera incertidumbre por reentrenamiento.

## Seguimiento propuesto
Si se evaluara un uso posterior, verificaría primero procedencia, consentimiento, moneda y disponibilidad temporal de las entradas. Con nuevos datos revisaría mensualmente faltantes, categorías nuevas, cambios en las distribuciones y, cuando haya ingresos reales, MAE global, por grupo e intersecciones y error con signo. Compararía con esta línea base y revisaría el modelo ante deterioros persistentes; los umbrales deben acordarse para el contexto de uso.

### Responsabilidad y respuesta ante deterioros
No existe monitoreo en ejecución: lo siguiente es una propuesta para un eventual uso validado. Responsable actual de reproducir y actualizar las fichas: estudiante; en una aplicación real habría que designar un propietario del modelo y revisores del dominio. Revisaría esquema y categorías en cada lote, y mensualmente distribuciones, faltantes y cobertura por grupo. Cuando estén disponibles los ingresos reales, recalcularía MAE, RMSE, error medio, brechas e incertidumbre por grupo. Antes de cada reentrenamiento repetiría el checklist de leakage y la auditoría, registrando versión, fecha y resultados.

Los umbrales de alerta no se fijaron ni validaron con este único dataset. Ante deterioro persistente, cambios de población o categorías sin cobertura, investigaría datos y medición, revisaría el uso y evitaría reutilizar el modelo hasta completar una evaluación independiente. Registraría las decisiones y actualizaría Model Card y Data Card.ß
