# Estimación del rendimiento del trigo en Argentina

<a target="_blank" href="https://cookiecutter-data-science.drivendata.org/">
    <img src="https://img.shields.io/badge/CCDS-Project%20template-328F97?logo=cookiecutter" />
</a>

Modelos de regresión para estimar el rendimiento del trigo (kg/ha) de un departamento de Argentina en una campaña agrícola, al finalizar la siembra, a partir de datos oficiales de siembra, cosecha y producción.

## Pregunta del proyecto

¿Es posible estimar el rendimiento del trigo de un departamento en una campaña a partir de su ubicación, la campaña agrícola, la superficie sembrada y el rendimiento de campañas anteriores?

## Datos

- **Fuente:** *Trigo: siembra, cosecha, producción y rendimiento*, Secretaría de Agricultura, Ganadería y Pesca (https://datos.magyp.gob.ar/dataset/trigo-siembra-cosecha-produccion-rendimiento). Licencia CC BY 4.0.
- **Archivo:** `data/raw/trigo-serie-1927-2025.csv` (25.231 filas y 11 columnas). Cada fila es un departamento en una campaña.
- **Período utilizado:** campañas 2000 a 2025.
- **Descripción completa del dataset:** `docs/descripcion_dataset.md`.

## Método

- **Variable objetivo:** rendimiento en kg/ha.
- **Variables predictoras:** provincia, departamento, año de la campaña, superficie sembrada, rendimiento de la campaña anterior y promedio de las tres campañas anteriores del mismo departamento. No se usan la producción ni la superficie cosechada, porque revelan el rendimiento (fuga de información).
- **Separación de los datos:** por fecha. Entrenamiento con las campañas 2000 a 2019 y prueba con 2020 a 2025.
- **Modelos:** regresión lineal y árbol de decisión (profundidad 3, elegida con validación dentro del entrenamiento).
- **Referencias:** predecir siempre el promedio de entrenamiento y repetir el rendimiento de la campaña anterior.

## Resultados en la prueba (2020 a 2025)

| Modelo | MAE (kg/ha) | RMSE (kg/ha) | R² |
|---|---|---|---|
| Referencia: promedio de entrenamiento | 1.173 | 1.366 | 0,00 |
| Referencia: campaña anterior | 757 | 1.092 | 0,36 |
| Regresión lineal | 682 | 912 | 0,55 |
| Árbol de decisión (profundidad 3) | 685 | 906 | 0,56 |

## Conclusión

Los dos modelos estiman el rendimiento mejor que las dos referencias sencillas, con un error promedio de unos 680 kg/ha y explicando algo más de la mitad de la variación. La diferencia entre ambos es mínima, por lo que no se puede afirmar que uno sea mejor que el otro.

Los modelos siguen el nivel general del rendimiento pero no anticipan cambios bruscos entre campañas: sobreestimaron 2022, año de caída fuerte, y subestimaron 2025, el de mayor rendimiento. El dataset no incluye variables climáticas, por lo que no es posible determinar a qué se deben esas variaciones. Las relaciones observadas son asociaciones y no implican causalidad.

## Documentos y notebooks

- `docs/planteo_del_problema.md`: objetivos, tipo de problema y modelos.
- `docs/descripcion_dataset.md`: origen, variables, calidad de los datos y período de estudio.
- `notebooks/modelo_rendimiento_trigo.ipynb`: análisis exploratorio, preparación de los datos, modelos y resultados.
- `notebooks/exploracion_inicial.ipynb`: revisión inicial del archivo.

## Cómo reproducirlo

1. Instalar las dependencias: `pip install -r requirements.txt`
2. Abrir `notebooks/modelo_rendimiento_trigo.ipynb` y ejecutar las celdas en orden. El archivo CSV se lee con codificación `latin-1`.

## Organización del proyecto

El proyecto sigue la plantilla Cookiecutter Data Science.

```
├── data
│   ├── external       <- Datos de terceros
│   ├── interim        <- Datos intermedios
│   ├── processed      <- Datos finales para modelar
│   └── raw            <- Datos originales, sin modificar
├── docs               <- Documentos del proyecto
├── models             <- Modelos entrenados
├── notebooks          <- Análisis y modelos
├── references         <- Materiales de referencia
├── reports
│   └── figures        <- Gráficos
├── requirements.txt   <- Dependencias
└── parcial_aprendizaje_automatico   <- Código fuente de la plantilla
```