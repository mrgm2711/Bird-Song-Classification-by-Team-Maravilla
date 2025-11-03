# Bird-Song-Classification-by-Team-Maravilla
En este proyecto se entrena un modelo de clasificación de canto de aves de acuerdo con descriptores espectrales de audios etiquetados de cantos de aves.

Primero construye un dataset de **descriptores espectrales** a partir de grabaciones de cantos de aves para entrenar modelos de **clasificación multiclase** (especies). El flujo incluye: **mapeo de etiquetas**, **extracción de features acústicas**, **limpieza**, **normalización opcional**, **filtrado por correlación** y **EDA**.

---

## 📦 Datos fuente

- **Grabaciones**: archivos locales `*.mp3` y `*.flac` (Xeno-canto y British Birdsong).
- Todos los archivos de audio se encontraban con una frecuencia de muestreo de 44100 Hz.  
- **Metadatos**:
  - `Xenocanto_metadata_qualityA_selection.csv` → columnas relevantes: `[id, label]` (columnas 1 y 5).
  - `birdsong_metadata.csv` (British BirdSong) → columnas relevantes: `[id, label]` (columnas 1 y 4).

Cada archivo de audio contiene un **ID** con el patrón `XC<id>` (ej. `XC12345_...`), usado para enlazar con la etiqueta de especie.

---

## 🛠 Extracción de características

Con el fin de mejorar el costo computacional, se remuestrean todos los audios a una frecuencia de muestreo de 32 kHz, teniendo en cuenta que el espectro frecuencial del canto de aves es significativo hasta 12-14 kHz aproximadamente.

Posterior a ello, se realiza con MATLAB la extracción de las características espectrales de los audios, ya que contiene una librería de funciones de extracción validadas y referenciadas; con lo que se garantiza fidelidad de los datos.
Con estas funciones se extraen las características espectrales de cada audio, por ventanas de tiempo hamming de 100 ms con solapamiento de 30 ms.

Cada descriptor espectral se reduce a tres valores estadísticos significativos Media (Xm), desviación estándar (Xd) y coeficiente de variación(Xv).

| Descriptor    | Significado            | # Columna  |
| ------------- | ---------------------- | ---------- |
| Cm, Cd, Cv    | Spectral Centroid      | 3, 4, 5    |
| Sm, Sd, Sv    | Spectral Spread        | 6, 7, 8    |
| SKm, SKd, SKv | Spectral Skewness      | 9, 10, 11  |
| Km, Kd, Kv    | Spectral Kurtosis      | 12, 13, 14 |
| Em, Ed, Ev    | Spectral Entropy       | 15, 16, 17 |
| Fm, Fd, Fv    | Spectral Flatness      | 18, 19, 20 |
| CRm, CRd, CRv | Spectral Crest         | 21, 22, 23 |
| FLm, FLd, FLv | Spectral Flux          | 24, 25, 26 |
| SLm, SLd, SLv | Spectral Slope         | 27, 28, 29 |
| DCm, DCd, DCv | Spectral Decrease      | 30, 31, 32 |
| RPm, RPd, RPv | Spectral Rolloff Point | 33, 34, 35 |

El dataset final contiene **35 columnas**:  
`Name`, `Label` + 33 descriptores numéricos.

---

## 🧾 Archivos generados

- `DATASET_No_normalizado_resultado.xlsx` → datos crudos (sin normalizar).  
---

## 🧹 Limpieza y preparación

1. **Valores faltantes (`NaN`)**  
   - Visualización: `missingno.matrix()` y `missingno.heatmap()`.  
   - Columnas con >40 % de NaN → eliminadas.  
   - Imputación con **mediana** (o por clase en análisis exploratorio).

2. **Normalización (opcional)**  
   - Escalado 0–1 con `MinMaxScaler`.  
   - En pipelines para PCA o SVM se usa `StandardScaler`.

3. **Filtrado por correlación**  
   - Se calcula la matriz |ρ|;  
   - Si **ρ > 0.8**, se elimina una de las dos variables correlacionadas.

---

## 🔎 EDA (Análisis Exploratorio)

- **Distribución de clases (`Label`)**.  
- **Estadística descriptiva** de todas las variables.  
- **Boxplots** globales y por especie.  
- **Matriz de correlación** de descriptores.  
- **PCA (2 componentes)** para visualizar separación entre especies.

---

## 🧠 Codificación de etiquetas

Se usa `LabelEncoder` para transformar especies (texto) en números:

```python
from sklearn.preprocessing import LabelEncoder
le = LabelEncoder()
y = le.fit_transform(df['Label'])
