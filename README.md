# Bird-Song-Classification-by-Team-Maravilla
Ene ste proyecto se entrena un modelo de clasificación de canto de aves por medio de coeficientes espectrales normalizados de 0 a 1, resumidos en su promedio, desviación estándar y coeficiente de variación.

EPrimero construye un dataset de **descriptores espectrales** a partir de grabaciones de cantos de aves para entrenar modelos de **clasificación multiclase** (especies). El flujo incluye: **mapeo de etiquetas**, **extracción de features acústicas**, **limpieza**, **normalización opcional**, **filtrado por correlación** y **EDA**.

---

## 📦 Datos fuente

- **Grabaciones**: archivos locales `*.mp3` y `*.flac` (Xeno-canto y British Birdsong).  
- **Metadatos**:
  - `Xenocanto_metadata_qualityA_selection.csv` → columnas relevantes: `[id, label]` (columnas 1 y 5).
  - `birdsong_metadata.csv` (British BirdSong) → columnas relevantes: `[id, label]` (columnas 1 y 4).

Cada archivo de audio contiene un **ID** con el patrón `XC<id>` (ej. `XC12345_...`), usado para enlazar con la etiqueta de especie.

---

## 🛠 Extracción de características

La extracción se realiza con MATLAB, ya que contiene funciones de extracción de características validadas y referenciadas. Con estas funciones se extraen las características espectrales de cada audio, por ventanas de tiempo hamming de 100 ms con solapamiento de 30 ms.

| Descriptor | Significado | Estadísticos |
|-------------|-------------|---------------|
| Cm, Cd, Cv | Spectral Centroid | Media, desviación, coef. variación |
| Sm, Sd, Sv | Spectral Spread | 〃 |
| SKm, SKd, SKv | Spectral Skewness | 〃 |
| Km, Kd, Kv | Spectral Kurtosis | 〃 |
| Em, Ed, Ev | Spectral Entropy | 〃 |
| Fm, Fd, Fv | Spectral Flatness | 〃 |
| CRm, CRd, CRv | Spectral Crest | 〃 |
| FLm, FLd, FLv | Spectral Flux | 〃 |
| SLm, SLd, SLv | Spectral Slope | 〃 |
| DCm, DCd, DCv | Spectral Decrease | 〃 |
| RPm, RPd, RPv | Spectral Rolloff Point | 〃 |

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
