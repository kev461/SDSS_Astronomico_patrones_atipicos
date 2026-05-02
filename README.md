# SDSS Astronómico — Detección de Patrones Atípicos

Análisis de objetos astronómicos con valores extremos de magnitud usando **PySpark** e **Isolation Forest**, sobre un dataset de **2.000.000 de registros** del Sloan Digital Sky Survey (SDSS).

---

## Contexto

El SDSS es uno de los catálogos astronómicos más completos del mundo. Cada objeto registrado tiene coordenadas espaciales (RA, DEC) y magnitudes de brillo en 5 bandas del espectro electromagnético: **u, g, r, i, z**.

El objetivo de este proyecto es detectar objetos con comportamientos fotométricos atípicos —como quásares, objetos extremadamente tenues o brillantes, o errores de medición— analizando combinaciones de magnitudes y su distribución espacial.

---

## Dataset

- **Fuente:** SDSS PhotoObj (consultas a la API oficial)
- **Volumen:** 2.000.000 de filas (4 consultas de 500.000 filas unidas)
- **Columnas:** `objID`, `ra`, `dec`, `modelMag_u`, `modelMag_g`, `modelMag_r`, `modelMag_i`, `modelMag_z`

---

## Tecnologías

- **PySpark** — procesamiento distribuido del dataset completo
- **Scikit-learn** — Isolation Forest aplicado sobre muestra en Pandas
- **Pandas / NumPy** — transformación y análisis estadístico
- **Matplotlib / Seaborn** — visualización de resultados
- **Google Colab** — entorno de ejecución con simulación de HDFS local

---

## Pipeline del análisis

### 1. Carga y exploración con PySpark
Se carga el CSV completo con Spark y se realiza un análisis inicial: schema, estadísticas descriptivas, conteo de nulos y detección de valores inválidos (-9999, usados por SDSS para mediciones no obtenidas).

```python
dfSDSS = spark.read.csv(ruta_local, header=True, inferSchema=True)
dfSDSS.describe().show()
```

### 2. Limpieza de datos
- Identificación de valores -9999 por columna (80 registros afectados en total)
- Reemplazo por `None` para cálculos de promedio
- Conservación de valores -9999 para el análisis de Isolation Forest, ya que representan comportamientos extremos relevantes

### 3. Filtrado por umbrales estadísticos
Filtro inicial basado en el promedio general de magnitudes (18.22) con rangos superior e inferior, encontrando:
- **18 objetos** con magnitudes extremadamente altas (posiblemente tenues o lejanos)
- **13 objetos** con magnitudes extremadamente bajas (posiblemente muy brillantes o cercanos)

### 4. Isolation Forest — detección de anomalías
Se aplica en dos escenarios:

**a) Solo magnitudes** — detecta objetos con combinaciones fotométricas inusuales sin considerar posición.

**b) Magnitudes + ubicación espacial (RA, DEC)** — detecta anomalías considerando también dónde están ubicados en el cielo.

Dado que 2.000.000 de filas no es manejable directamente con Pandas, se toma una muestra del 40% (~800.000 filas) para el modelo.

```python
iso = IsolationForest(contamination=0.01)
iso.fit(muestra)
```

Resultados: ~8.000 objetos clasificados como anomalías (-1) sobre ~800.000 analizados.

### 5. Análisis de colores fotométricos
Se calculan índices de color combinando bandas:

| Índice | Significado |
|--------|-------------|
| u - g  | Radiación ultravioleta |
| g - r  | Temperatura estelar |
| r - i  | Transición rojo-infrarrojo |
| i - z  | Objetos fríos |

### 6. Visualizaciones
- **Diagrama color-color (u-g vs g-r):** muestra en qué zonas del espacio fotométrico se concentran las anomalías
- **Distribución espacial (RA vs DEC):** ubica las anomalías en el plano del cielo
- **Mapa con categorías:** diferencia entre objetos normales, anomalías limpias y objetos con valores -9999 en alguna magnitud

---

## Para tener en cuenta

- Los objetos con valores -9999 se pueden ver graficamente que se focalizan en cietas coordenadas del espacio. Es interesante analziar esas coordenadas para encontrar información o un patrón especial.

---


## Cómo ejecutar

1. Abrir el notebook en Google Colab
2. Subir el archivo `SDSS_analisisespacial.csv` cuando se solicite
3. Ejecutar las celdas en orden

> El dataset no está incluido en el repo por su tamaño. Se puede obtener directamente desde el portal del [SDSS SkyServer](http://skyserver.sdss.org/).
