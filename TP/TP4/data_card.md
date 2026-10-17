# Data Card · census_income.csv

**Versión:** 1.0 · **Fecha:** 18/10/2026 · **Responsable de esta documentación:** Victor Gabriel Peralta


## Motivación y procedencia
Uso la copia local provista para la Actividad 4, con el objetivo  de predecir ingreso anual. La consigna ofrece un [enlace de descarga](https://drive.google.com/file/d/1Od_naN-d1B_mjJeIMDlL1dTvHEDX9PO6/view).

## Composición y esquema
10,000 filas y 9 columnas en origen; cada fila se interpreta como un registro de persona, sin identificador verificable. Target: `income` (ingreso anual continuo). Entradas numéricas: edad (`age`) y horas semanales (`hours_per_week`). Entradas categóricas: educación, tipo de empleador, estado civil, sexo registrado, raza registrada y nacimiento en EE. UU. Sus categorías son las del archivo, sin atribuirles identidades adicionales.

|                | tipo    |   faltantes_originales |   porcentaje_faltante |
|:---------------|:--------|-----------------------:|----------------------:|
| age            | int64   |                      0 |                  0.00 |
| hours_per_week | float64 |                    500 |                  5.00 |
| education      | object  |                    300 |                  3.00 |
| employer_type  | object  |                    400 |                  4.00 |
| marital_status | object  |                      0 |                  0.00 |
| sex            | object  |                      0 |                  0.00 |
| race           | object  |                      0 |                  0.00 |
| born_in_us     | object  |                      0 |                  0.00 |
| income         | float64 |                      0 |                  0.00 |

## Calidad y preparación
Encontré 86 duplicados exactos y conservé 9,914 registros únicos. El ingreso no tiene faltantes, es finito y estrictamente positivo. No eliminé extremos por magnitud ni confirmé que todos sean válidos. La imputación numérica usa mediana y la categórica `Sin dato`, aprendidas o aplicadas dentro del pipeline; la estandarización y codificación se ajustan en entrenamiento. No infiero el mecanismo de ausencia de los datos.

## Particiones y representación
División aleatoria 80/20 con semilla 42, después de quitar duplicados exactos. Cinco folds en desarrollo. No hay un conjunto externo independiente. Conteos por grupo tras la limpieza:

| atributo   | grupo             |   n_desarrollo |   n_test |
|:-----------|:------------------|---------------:|---------:|
| sex        | Female            |           3839 |      993 |
| sex        | Male              |           4092 |      990 |
| race       | Asian             |            343 |       88 |
| race       | Black             |           1865 |      460 |
| race       | Other or multiple |            346 |       83 |
| race       | White             |           5377 |     1352 |

Rangos numéricos observados en desarrollo (no son garantías de validez fuera de ellos):

|                |   min |   max |
|:---------------|------:|------:|
| age            |  17.0 |  92.0 |
| hours_per_week |   1.0 |  99.0 |

Los conteos permiten detectar menor cobertura de ciertas categorías, pero sin población de referencia no cuantifico subrepresentación respecto de un censo. No hay pesos muestrales, fechas o identificadores para verificar representatividad, dependencia entre personas o validez temporal.

## Usos, distribución y mantenimiento
Uso previsto: aprendizaje y reproducción de esta actividad. No sustenta inferencias causales ni decisiones reales individuales. Antes de redistribuir la base debe verificarse su licencia y procedencia; esta ficha no concede permisos. El repositorio mantiene la copia local utilizada; no hay una política de actualización del proveedor documentada. Ante cambios del CSV, actualizar hash, control de calidad, particiones, auditoría y documentación. No hubo contacto con el creador original.
