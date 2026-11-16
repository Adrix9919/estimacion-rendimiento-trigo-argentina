# Estimación del rendimiento del trigo en Argentina

## Origen de los datos

| Aspecto | Detalle |
|---|---|
| Nombre del dataset | Trigo – siembra, cosecha, producción, rendimiento |
| Archivo | `trigo-serie-1927-2025.csv` |
| Organismo | Secretaría de Agricultura, Ganadería y Pesca (Dirección de Estimaciones Agrícolas) |
| Portal | https://datos.magyp.gob.ar/dataset/trigo-siembra-cosecha-produccion-rendimiento |
| Licencia | Creative Commons Attribution 4.0 |
| Frecuencia de actualización | Anual |
| Última actualización en el portal | 7 de mayo de 2026 |
| Fecha de descarga | 26-10-2026 |

El archivo original se conserva sin modificaciones en `data/raw/`. Cualquier limpieza se hará sobre copias en `data/interim/` y `data/processed/`.

## Descripción del dataset

### Variables

| Variable | Tipo | Descripción |
|---|---|---|
| `cultivo` | Texto | Cultivo (siempre "trigo") |
| `anio` | Entero | Año de inicio de la campaña agrícola |
| `campania` | Texto | Campaña agrícola (ej. "1927/1928") |
| `provincia` | Texto | Nombre de la provincia |
| `provincia_id` | Texto | Código de la provincia (ej. "06") |
| `departamento` | Texto | Nombre del departamento |
| `departamento_id` | Texto | Código del departamento (ej. "06007") |
| `superficie_sembrada_ha` | Entero | Superficie sembrada, en hectáreas |
| `superficie_cosechada_ha` | Decimal | Superficie cosechada, en hectáreas |
| `produccion_tm` | Decimal | Producción, en toneladas |
| `rendimiento_kgxha` | Decimal | Rendimiento, en kilogramos por hectárea |

- Cada fila representa un departamento en una campaña agrícola. En las provincias sin desagregación por departamento, la fila representa a la provincia completa.
- Las tres últimas son decimales porque tienen valores faltantes. Los códigos de provincia y departamento son texto para conservar los ceros.

### Dimensiones

- Cantidad de instancias (filas): 25.231
- Cantidad de características (columnas): 11
- Período cubierto: campañas con año de inicio entre 1927 y 2025
- Cantidad de provincias: 22
- Cantidad de departamentos distintos: 478

### Valores faltantes

| Variable | Valores faltantes |
|---|---|
| `departamento` | 119 |
| `departamento_id` | 119 |
| `superficie_cosechada_ha` | 484 |
| `produccion_tm` | 484 |
| `rendimiento_kgxha` | 486 |

Las demás variables no tienen valores faltantes. Las filas sin departamento corresponden a provincias que la fuente informa solo a nivel provincial.

## Planteo del modelo

### Unidad de análisis

Cada fila es un departamento en una campaña agrícola. Los datos están agregados, por lo que las predicciones corresponden al rendimiento medio de un departamento y no al de un establecimiento o lote particular.

### Momento de la predicción

El modelo estima el rendimiento al finalizar la siembra de cada campaña, antes de la cosecha. En ese momento se conocen la ubicación, la campaña y la superficie sembrada, pero no la superficie cosechada ni la producción.

### Variable objetivo

`rendimiento_kgxha`: rendimiento en kilogramos por hectárea.

### Variables predictoras

- `provincia` y `departamento`
- `anio`
- `superficie_sembrada_ha`
- Variables derivadas del propio dataset: rendimiento de la campaña anterior y promedio de las tres campañas anteriores del mismo departamento (o de las que existan), usando solo campañas previas a la que se predice.

### Variables excluidas

- `superficie_cosechada_ha` y `produccion_tm`: el rendimiento se calcula a partir de ellas (producción sobre superficie cosechada). Por ejemplo, en el primer registro del archivo, 91.350 toneladas sobre 105.000 hectáreas dan 870 kg/ha. Usarlas permitiría reconstruir el objetivo y generaría fuga de información (data leakage). Además, no se conocen al momento de la predicción.
- `cultivo`, `campania`, `provincia_id`: son constantes o repiten información de otras columnas. `departamento_id` se usa solo para identificar departamentos.

### Relación entre las predictoras y el objetivo

- La ubicación resume condiciones propias de cada zona (suelo, clima medio, tipo de producción), por lo que unas zonas rinden sistemáticamente más que otras.
- El año captura la tendencia de largo plazo del rendimiento (cambios tecnológicos y de manejo).
- El rendimiento de campañas anteriores aporta el nivel de productividad propio de cada departamento.

### Limitación

El dataset no incluye variables climáticas (lluvias, temperaturas, heladas), que explican gran parte de la variación del rendimiento entre campañas. Se espera que el modelo capture el componente estructural y la tendencia, pero no los extremos de campañas con clima excepcional. Incorporar datos meteorológicos públicos queda como posible ampliación.

## Calidad de los datos

Se encontraron los siguientes puntos a tratar antes de entrenar:

- **Valores faltantes:** 484 filas sin superficie cosechada ni producción y 486 sin rendimiento (ver tabla de valores faltantes). Las filas sin rendimiento no sirven para entrenar, porque no tienen valor objetivo.
- **Filas sin departamento:** 119 filas corresponden a provincias informadas solo a nivel provincial.
- **Registros repetidos:** 89 filas repiten la combinación de campaña y departamento. Se deben revisar para decidir si se eliminan o si corresponden a casos distintos.
- **Rendimientos inconsistentes:** en 39 filas, el rendimiento informado difiere en más de un 5 % del que resulta de dividir la producción por la superficie cosechada. Por ejemplo, Iriondo (Santa Fe, campaña 1927/1928) figura con 11.000 kg/ha cuando el cálculo da 1.100, y General Alvear (Buenos Aires, campaña 1929/1930) con 8.700 cuando el cálculo da 800. Probablemente sean errores de carga.

## Período de estudio

Se trabajará con las campañas de 2000 a 2025 (6.053 filas). El resto del archivo se conserva en `data/raw/` pero no se usa en el modelo.

Criterios de elección:

- **Calidad del objetivo:** desde 1990 ninguna campaña tiene rendimiento faltante, mientras que en las décadas de 1930 y 1940 hay campañas con decenas de faltantes.
- **Cobertura estable:** entre 2000 y 2025 cada campaña tiene entre 196 y 265 departamentos, con un solo registro por departamento.
- **Menor confiabilidad de los datos antiguos:** las primeras campañas tienen rendimientos redondeados y menor cobertura, y reflejan una tecnología de cultivo muy distinta a la actual.
- **Tamaño suficiente:** el período aporta unas 6.000 filas sin arrastrar décadas de datos poco comparables.

División de los datos: se entrena con las campañas 2000 a 2019 y se evalúa con 2020 a 2025, respetando el orden temporal para simular una predicción hacia adelante. Como las variables derivadas necesitan la campaña anterior del mismo departamento, se descartan 416 de las 6.053 filas que no la tienen (las de 2000 y otras cuyo departamento no tiene registro en la campaña anterior). Quedan 5.637 filas para el modelo: 4.128 para entrenar y 1.509 para evaluar.

Revisión de los datos dentro del período:

- **Provincias:** solo 14 de las 22 provincias del archivo tienen registros entre 2000 y 2025. Buenos Aires concentra 2.523 de las 6.053 filas.
- **Rendimiento igual a cero:** 44 filas tienen superficie sembrada y producción nula (25 en Santiago del Estero). Se conservan, porque el modelo predice al finalizar la siembra y en ese momento no se sabe si el lote se cosechará.
- **Calidad:** en este período no hay rendimientos inconsistentes ni registros repetidos.

## Análisis y modelos
El análisis exploratorio, la preparación de los datos, el entrenamiento de los modelos y sus resultados están en `notebooks/modelo_rendimiento_trigo.ipynb`.

## Fuente
Secretaría de Agricultura, Ganadería y Pesca (MAGyP). *Trigo – siembra, cosecha, producción, rendimiento*. Datos Agricultura, Ganadería y Pesca. Licencia CC BY 4.0. https://datos.magyp.gob.ar/dataset/trigo-siembra-cosecha-produccion-rendimiento