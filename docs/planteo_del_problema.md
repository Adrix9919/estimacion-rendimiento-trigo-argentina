# Estimación del rendimiento del trigo en distintas regiones de Argentina

## Descripción del proyecto

El proyecto propone desarrollar un modelo de aprendizaje automático que permita predecir el rendimiento del cultivo de trigo en distintas regiones de Argentina, a partir de la ubicación geográfica, la campaña agrícola y la superficie sembrada.

Para ello se utilizan datos históricos oficiales de siembra, cosecha, producción y rendimiento, y se evalúa la capacidad de distintos modelos para estimar el rendimiento en kg/ha (kilogramos por hectárea).

## Pregunta de investigación

¿Es posible predecir el rendimiento del cultivo de trigo en distintas regiones de Argentina a partir de la ubicación geográfica, la campaña agrícola y la superficie sembrada?

## Contexto y relevancia

El rendimiento agrícola, expresado en kilogramos por hectárea, permite conocer la producción obtenida en relación con la superficie cultivada. Su estimación puede contribuir a comparar resultados entre regiones y campañas agrícolas.

Mediante el aprendizaje automático se busca identificar relaciones en los registros históricos que permitan estimar el rendimiento y evaluar qué tan precisas son las predicciones.

## Objetivos

**Objetivo general:** desarrollar y evaluar un modelo de regresión para predecir el rendimiento del cultivo de trigo en distintas regiones de Argentina, a partir de variables geográficas, temporales y de superficie sembrada.

**Objetivos específicos:**

- Preparar los datos seleccionados, identificando valores faltantes e inconsistencias.
- Seleccionar las variables predictoras adecuadas para estimar el rendimiento, evitando la fuga de información (data leakage).
- Entrenar modelos de regresión para predecir el rendimiento del trigo.
- Evaluar y comparar el desempeño de los modelos mediante métricas de error.

## Tipo de problema

Se trata de un problema de regresión supervisada, ya que se busca predecir un valor numérico: el rendimiento agrícola en kg/ha. Los modelos aprenden a partir de registros históricos que contienen las variables de entrada y el rendimiento observado.

## Modelos a utilizar

- **Regresión lineal:** estima el rendimiento a partir de relaciones lineales entre las variables.
- **Árbol de decisión para regresión:** aprende relaciones y patrones mediante divisiones sucesivas de los datos.

Los resultados de ambos se comparan para determinar cuál ofrece mejores estimaciones sobre datos no utilizados durante el entrenamiento.

## Dataset y variables

Se utiliza el conjunto de datos oficial *Trigo: siembra, cosecha, producción y rendimiento*, publicado en el portal de datos de la Secretaría de Agricultura, Ganadería y Pesca de Argentina.

- Fuente: https://datos.magyp.gob.ar/dataset/trigo-siembra-cosecha-produccion-rendimiento
- Archivo utilizado: https://datos.magyp.gob.ar/dataset/trigo-siembra-cosecha-produccion-rendimiento/archivo/50f0edcc-4dfc-4afb-b78a-b164601d36ae
- **Variable objetivo:** rendimiento en kg/ha.
- **Variables predictoras:** provincia, departamento, año de la campaña y superficie sembrada.
- **Variables excluidas:** la producción y la superficie cosechada no se utilizan como predictoras, ya que permiten calcular directamente el rendimiento y generarían fuga de información (data leakage).
- **Período de estudio:** campañas 2000 a 2025.

La descripción completa del dataset, sus variables y los criterios del período se encuentran en