# Datos para la Actividad 3

Utilicé **Human Activity Recognition Using Smartphones**, de Reyes-Ortiz, Anguita, Ghio, Oneto y Parra, disponible en [UCI](https://archive.ics.uci.edu/dataset/240/human+activity+recognition+using+smartphones), DOI [10.24432/C54S4K](https://doi.org/10.24432/C54S4K).

Descargué el archivo oficial el 5 de octubre de 2026. Su SHA256 es `c00b803081a5c797cd5e4b83700a9810b38d53d9d84e01917e090e1fdbc81031`.

Conservé en `har_features.npz` las 10.299 filas y 561 características publicadas: primero las 7.352 de train y después las 2.947 de test. Leí los TXT con `numpy.loadtxt` y guardé los números como float64 con compresión sin pérdida, sin redondeo adicional, selección de filas ni imputación. El archivo incluye arrays separados `X`, `y`, `subject`, `split` y `names`. Las etiquetas y sujetos se consultan en el notebook solo después de seleccionar el clustering.

Mantuve `README.txt`, `features.txt`, `features_info.txt` y `activity_labels.txt` del archivo original. El README original usa codificación Latin-1. No incluí las señales inerciales crudas, que no se usan en esta actividad.

Referencia de la publicación: Anguita, D., Ghio, A., Oneto, L., Parra, X. y Reyes-Ortiz, J. L. (2013). *A Public Domain Dataset for Human Activity Recognition Using Smartphones*. ESANN 2013, Bruges, Belgium.

Si el NPZ no está disponible, puedo reconstruirlo desde la raíz del proyecto:

```bash
uv run python tarea/preparar_datos_actividad_3.py
```

El preparador descarga el ZIP oficial y verifica su huella. También admite `--archive /ruta/al/zip` y conserva el NPZ si ya existe.
