# 🔥 MergeGeo: Incendios Forestales en Chile × Clima

**Python · PySpark · XGBoost · Folium · Plotly · NetCDF**

---

## 📌 Descripción del Proyecto

MergeGeo es un pipeline de análisis masivo que cruza **60.530 registros de incendios forestales** de CONAF (2010-2019) con datos climáticos reales de ERA5 (temperatura y precipitación diaria en grilla de 0.25° sobre todo Chile) para detectar anomalías de estrés hídrico y clasificar la severidad de incendios a nivel de comuna.

El objetivo es responder una pregunta concreta: **¿el clima anómalo (más seco y más caliente de lo histórico) se asocia con incendios más severos?** A través de agregación con Apache Spark, modelos predictivos con XGBoost y mapas geoespaciales interactivos, el proyecto construye evidencia cuantitativa de la relación entre estrés hídrico y severidad de incendios forestales en Chile.

El resultado es un **dashboard HTML autocontenido** que integra KPIs, mapas coropléticos por comuna, mapas de puntos por evento, gráficos de tendencia nacional y un modelo de riesgo clasificatorio — todo listo para presentar a una audiencia no técnica.

---

## 📊 Dataset y Origen de los Datos

El proyecto integra **3 fuentes de datos** principales:

| Fuente | Descripción | Volumen | Período |
|--------|-------------|---------|---------|
| **CONAF** (vía `datospararesiliencia.cl`) | Incendios individuales: fecha, coordenadas, causa, superficie quemada por tipo de vegetación | 60.530 eventos | 2010-2019 |
| **ERA5** (Copernicus Climate Data Store) | Temperatura (`t2m`) y precipitación (`tp`) diaria en grilla 0.25° sobre todo Chile | Archivos NetCDF | 2000-2025 |
| **CONAF — Archivo 6** | Serie histórica nacional mensual de ocurrencia de incendios | 480 meses | 1985-2024 |

**Notas técnicas:**
- Los datos de CONAF pasaron por un proceso de descarga (notebooks 01-02) que combinó 18 temporadas de la API y limpió coordenadas/fechas, resultando en 60.530 eventos válidos dentro de Chile continental.
- ERA5 fue pre-extraído a archivos `.nc` en un notebook independiente, cubriendo todo Chile en un grid regular de 0.25° (~3.600 celdas).
- El dataset de mtys (Archivo 6) es una serie compilada por expertos de la CONAF con registro mensual de ocurrencia nacional.

---

## 🎯 Hipótesis

**¿El estrés hídrico a nivel comunal (temperatura anómalamente alta + precipitación anómalamente baja) se asocia con incendios de mayor severidad?**

La hipótesis central es que comunas que experimentan meses climáticamente más secos y cálidos de lo histórico deberían presentar una mayor superficie quemada y mayor frecuencia de eventos catastróficos. Para testear esto se construyó un **índice de estrés hídrico** (z-score de anomalía de temperatura menos z-score de anomalía de precipitación) y se entrenó un modelo de clasificación que predice la categoría de riesgo de cada comuna-año usando **solo variables climáticas**, sin información previa sobre incendios, para evitar data leakage.

---

## 🔧 Pipeline de Procesamiento

El proyecto sigue un pipeline secuencial de 7 notebooks:

```
01  Descarga de incendios CONAF (API)
         ↓
02  Limpieza y filtrado → 60.530 eventos válidos
         ↓
03  Cruce con ERA5 (temp. + precip. diaria por evento)
         ↓
    ┌────┴────┐
    ↓         ↓
04  Agregación      05  Serie nacional mensual
    Spark              (mtys × ERA5, XGBoost)
    comuna×año
    ↓         ↓
06  Mapas       07  Mapa de puntos
    coropléticos    de importancia
    ↓         ↓
08  Modelo de riesgo   →  09  Dashboard final
    (XGBClassifier)       (HTML autocontenido)
```

### Notebooks

| # | Notebook | Qué hace | Output |
|---|----------|----------|--------|
| 01 | `descargar_incendios_conaf` | Descarga 18 temporadas de CONAF vía API | `incendios_conaf_raw.csv` |
| 02 | `preparar_incendios_para_era5` | Limpieza, filtrado a Chile continental 2010-2019 | `incendios_conaf_2010_2020.xls` |
| 03 | `cruzar_incendios_era5` | Cruza cada evento con clima diario de ERA5 + anomalías mensuales | `incendios_conaf_era5_2010_2020.csv` |
| 04 | `agregacion_spark_comuna_anio` | Agrega a comuna×año con PySpark, calcula estrés hídrico y ranking de severidad | `spark_comuna_anio/csv/` |
| 05 | `serie_nacional_mensual_mtys_era5` | Cruza serie 1985-2024 con ERA5 nacional, entrena XGBoost con anti-extrapolación | `incendios_nacional_mensual_era5.csv` |
| 06 | `mapa_georreferenciado_comuna` | Mapas coropléticos: severidad histórica + estrés hídrico por comuna (Folium) | 2 HTML interactivos |
| 07 | `mapa_puntos_importancia` | Heatmap de densidad + puntos catastróficos/estrés extremo (Folium) | 1 HTML interactivo |
| 08 | `modelo_riesgo_comuna_anio` | Clasificación de riesgo (bajo/normal/alto/catastrófico) con XGBClassifier | `modelo_riesgo_comuna_anio.csv` |
| 09 | `dashboard` | Dashboard HTML con KPIs, mapas, gráficos y resultados del modelo | `dashboard.html` |

---

## 📈 Métricas Clave del Proyecto

| Métrica | Valor |
|---------|-------|
| Total de eventos analizados | 60.530 |
| Período de incendios | 2010-2019 |
| Cubrimiento climático | 2000-2025 (ERA5) |
| Comunas con datos | ~346 (GeoJSON Chile) |
| Variables climáticas por evento | 4 (t2m, tp, anomalía t2m, anomalía tp) |
| Categorías de riesgo | 4 (bajo, normal, alto, catastrófico) |
| Modelo de severidad | XGBClassifier (300 árboles, lr=0.05) |
| Modelo de tendencia nacional | XGBoost Regressor (anti-extrapolación) |

---

## 🛠️ Tecnologías Utilizadas

| Categoría | Herramientas |
|-----------|-------------|
| **Lenguaje** | Python 3 |
| **Procesamiento masivo** | Apache Spark (PySpark, local mode) |
| **Ciencia de datos** | Pandas, NumPy, Scikit-learn |
| **Modelos ML** | XGBoost (clasificación y regresión) |
| **Datos climáticos** | xarray, NetCDF4 (lectura de ERA5) |
| **Geoespacial** | Folium, branca, GeoJSON |
| **Visualización** | Plotly, Matplotlib |
| **Formato de datos** | Parquet, CSV |
| **Ejecución** | Google Colab, Google Drive |

---

## 📂 Estructura del Repositorio

```
MergeGeo/
├── 01_descargar_incendios_conaf.ipynb      # Descarga incendios CONAF vía API (pyDataverse)
├── 02_preparar_incendios_para_era5.ipynb   # Limpia/filtra a 2010-2019
├── 03_cruzar_incendios_era5.ipynb          # Cruza cada evento con clima diario ERA5
├── 04_agregacion_spark_comuna_anio.ipynb   # Agregación PySpark comuna×año
├── 05_serie_nacional_mensual_mtys_era5.ipynb  # Serie nacional mtys corregida con clima real
├── 06_mapa_georreferenciado_comuna.ipynb   # Mapas coropléticos por comuna
├── 07_mapa_puntos_importancia.ipynb        # Mapa de puntos por evento
├── 08_modelo_riesgo_comuna_anio.ipynb      # Clasificación de riesgo por comuna-año
├── 09_dashboard.ipynb                      # Ensambla el dashboard.html final
├── era5_extraccion.ipynb           # Documenta cómo se generaron los .nc de ERA5
│                                   # (no se ejecuta como parte del pipeline 01-09)
├── incendios_conaf_raw.xls         # Output crudo del notebook 01 (input del 02)
├── incendios_conaf_2010_2020.xls   # Output del notebook 02 (input del 03)
├── 6.- Ocurrencia Nacional de Incendios Forestales según Mes, 1985 - 2024_octubre.xls
│                                   # Serie histórica CONAF, input del notebook 05
├── MergeGeo_informe.pdf            # Informe técnico completo del proyecto
├── MergeGeo_presentacion.pptx      # Presentación utilizada en la exposición
├── .gitignore                      # Excluye los .nc de ERA5 (superan el límite de 100MB de GitHub)
├── README.md
└── datos_procesados/               # Outputs generados por los notebooks 03-09
    ├── incendios_conaf_era5_2010_2020.csv/.parquet  # Output del notebook 03
    ├── incendios_nacional_mensual_era5.csv           # Output del notebook 05
    ├── spark_comuna_anio/                            # Output del notebook 04 (Spark)
    │   ├── csv/
    │   └── parquet/año=2010.../año=2019/
    ├── mapa_severidad_historica.html   # Output del notebook 06
    ├── mapa_estres_hidrico.html        # Output del notebook 06
    ├── mapa_puntos_importancia.html    # Output del notebook 07
    ├── modelo_riesgo_comuna_anio.csv   # Output del notebook 08
    └── dashboard.html                  # Output del notebook 09 (junta todo lo anterior)
```

---

## 🚀 Instrucciones de Ejecución

**Requisitos previos:**
- Google Colab (recomendado) o entorno local con Python 3.8+
- Los `.nc` de ERA5 (`era5_t2m_chile_2000_2025.nc`, `era5_tp_chile_2000_2025.nc`) **no están en
  este repo** — superan el límite de 100MB de GitHub. Están disponibles en el Drive del equipo;
  hay que descargarlos de ahí y ubicarlos junto al resto de los inputs.
- Todos los demás archivos de entrada (`incendios_conaf_2010_2020.xls`,
  `incendios_conaf_raw.xls`, `6.- Ocurrencia Nacional...xls`) **ya están en la raíz de este
  repo** — no hace falta conseguirlos aparte.

**Orden de ejecución:**
1. Los notebooks **01 y 02 ya están corridos** — sus outputs (`incendios_conaf_raw.xls`,
   `incendios_conaf_2010_2020.xls`) ya están en el repo. Si quieres volver a correr el
   **01**, ahora pide la API key por teclado (`getpass`, no hardcodeada) — consíguela gratis
   en `plataformadedatos.cl/user/developer`. El **02** puede correrse solo, sin pasar por el
   01, usando `incendios_conaf_raw.xls` como input directo.
2. Los `.nc` de ERA5 **ya están extraídos** (ver `era5_extraccion.ipynb` para el detalle de
   cómo) — no es necesario re-correrlo, solo conseguir los `.nc` del Drive del equipo.
3. Ejecutar en orden: **03 → 04 → 05 → 06 → 07 → 08 → 09**
4. El notebook **09 (dashboard)** debe ejecutarse al final, ya que reúne todos los outputs
   anteriores

**Nota importante:** `dashboard.html` debe quedar en la misma carpeta que los 3 archivos de mapas HTML — los embebe por `iframe` con ruta relativa.

---

## ☁️ Cómo clonar este repo y correrlo en Google Colab

Los notebooks `03` a `09` están pensados para correr en Google Colab, leyendo sus archivos
desde una carpeta de Google Drive (`DRIVE_BASE`) — no desde el repo directamente. Pasos para
dejarlo andando desde cero:

1. **Consigue los `.nc` de ERA5** (no están en este repo por su tamaño — ver nota en
   "Estructura del Repositorio"): descárgalos desde esta carpeta de Drive del equipo:
   👉 **[era5_t2m / era5_tp — Google Drive](https://drive.google.com/drive/folders/1WJsjzEDG0r5h_7ulisdxdnxW4UjZJ-SX?usp=sharing)**
2. **Clona (o descarga como ZIP) este repo** en tu computador:
   ```bash
   git clone https://github.com/Mtys24/MergeGeo.git
   ```
3. **Crea una carpeta en tu propio Google Drive** (el nombre no importa, será tu `DRIVE_BASE`
   — por ejemplo `MergeGeo_datos`) y sube ahí adentro:
   - Los 2 archivos `.nc` que descargaste en el paso 1.
   - `incendios_conaf_2010_2020.xls`, `incendios_conaf_raw.xls` y
     `6.- Ocurrencia Nacional...xls` (ya vienen en la raíz de este repo, del paso 2).
4. **Abre cada notebook (03 → 09) en Colab**: `Archivo → Subir notebook` (o `Archivo → Abrir
   notebook → GitHub`, pegando la URL de este repo) y edita la variable `DRIVE_BASE` en la
   celda de configuración de cada uno para que apunte a la carpeta que creaste en el paso 3
   (reemplaza el placeholder `CAMBIAR_A_TU_RUTA`).
5. Corre los notebooks en orden: **03 → 04 → 05 → 06 → 07 → 08 → 09**. `datos_procesados/`
   se genera solo dentro de tu carpeta de Drive a medida que avanzas.

---

## 🧑‍💻 Equipo de Trabajo

| Integrante | Rol en el Proyecto | GitHub |
|------------|-------------------|--------|
| **Fabian Valdés** | Consultas y exploración de datos | [@favc-5](https://github.com/favc-5) |
| **Jairo Arias Valenzuela** | Extracción, limpieza y consolidación de registros CONAF (1985-2024) y consumo estructurado de la API meteorológica Open-Meteo | [@jairoarias208-beep](https://github.com/jairoarias208-beep) |
| **Matías Manríquez** | Visualización de datos: mapas geoespaciales interactivos en Folium y gráficos analíticos en Plotly | [@Mtys24](https://github.com/Mtys24) |
| **Javiera González Mardones** | Visualización de datos | [@Zelaznog-J](https://github.com/Zelaznog-J) |
| **José Salgado Escalona** | Redacción del informe: consolidación de la estructura narrativa, redacción técnica y síntesis de hallazgos | [@JoseRicardoSE](https://github.com/JoseRicardoSE) |
