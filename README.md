# 🔥 MergeGeo: Incendios Forestales en Chile × Clima

**Python · PySpark · XGBoost · Folium · Plotly · NetCDF**

---

## 📌 Descripción del Proyecto

MergeGeo es un pipeline de análisis masivo que cruza **60.530 registros de incendios forestales** de CONAF (2010-2019) con datos climáticos reales de ERA5 (temperatura y precipitación diaria en grilla de 0.25° sobre todo Chile) para detectar anomalías de estrés hídrico y clasificar la severidad de incendios a nivel de comuna.

El objetivo es responder una pregunta concreta: **¿el clima anómalo (más seco y más caliente de lo histórico) se asocia con   incendios más severos?** A través de agregación condictivos con XGBoost y mapas geoespacialesinteractivos, el proyecto construye evidencia cuantitativa de la relación entre estrés hídrico y severidad de incendios foresten Chile.
                                                                                                                              El resultado es un **dashboard HTML autocontenido**oropléticos por comuna, mapas de puntos por evento,gráficos de tendencia nacional y un modelo de riesgo clasificatorio — todo listo para presentar a una audiencia no técnica.   
---                                                                                                                           
## 📊 Dataset y Origen de los Datos                                                                                           
El proyecto integra **3 fuentes de datos** principales:                                                                       
| Fuente | Descripción | Volumen | Período |                                                                                  |--------|-------------|---------|---------|
| **CONAF** (vía `datospararesiliencia.cl`) | Incendios individuales: fecha, coordenadas, causa, superficie quemada por tipo dvegetación | 60.530 eventos | 2010-2019 |
| **ERA5** (Copernicus Climate Data Store) | Temperatura (`t2m`) y precipitación (`tp`) diaria en grilla 0.25° sobre todo ChilArchivos NetCDF | 2000-2025 |
| **CONAF — Archivo 6** | Serie histórica nacional mensual de ocurrencia de incendios | 480 meses | 1985-2024 |               
**Notas técnicas:**                                                                                                           - Los datos de CONAF pasaron por un proceso de desc combinó 18 temporadas de la API y limpiócoordenadas/fechas, resultando en 60.530 eventos válidos dentro de Chile continental.                                         - ERA5 fue pre-extraído a archivos `.nc` en un notendo todo Chile en un grid regular de 0.25° (~3.600celdas).                                                                                                                      - El dataset de mtys (Archivo 6) es una serie compiNAF con registro mensual de ocurrencia nacional.
                                                                                                                              ---
                                                                                                                              ## 🎯 Hipótesis
                                                                                                                              **¿El estrés hídrico a nivel comunal (temperatura atación anómalamente baja) se asocia con incendios demayor severidad?**                                                                                                            
La hipótesis central es que comunas que experimentan meses climáticamente más secos y cálidos de lo histórico deberían presentuna mayor superficie quemada y mayor frecuencia de a testear esto se construyó un **índice de estréshídrico** (z-score de anomalía de temperatura menos z-score de anomalía de precipitación) y se entrenó un modelo de clasificacque predice la categoría de riesgo de cada comuna-a climáticas**, sin información previa sobreincendios, para evitar data leakage.                                                                                          
---                                                                                                                           
## 🔧 Pipeline de Procesamiento                                                                                               
El proyecto sigue un pipeline secuencial de 7 notebooks:                                                                      
01  Descarga de incendios CONAF (API)                                                                                                  ↓
02  Limpieza y filtrado → 60.530 eventos válidos                                                                                       ↓
03  Cruce con ERA5 (temp. + precip. diaria por evento)                                                                                 ↓
    ┌────┴────┐                                                                                                                   ↓         ↓
04  Agregación      05  Serie nacional mensual                                                                                    Spark              (mtys × ERA5, XGBoost)
    comuna×año                                                                                                                    ↓         ↓
06  Mapas       07  Mapa de puntos                                                                                                coropléticos    de importancia
    ↓         ↓                                                                                                               08  Modelo de riesgo   →  09  Dashboard final
    (XGBClassifier)       (HTML autocontenido)                                                                                
### Notebooks                                                                                                                 
| # | Notebook | Qué hace | Output |                                                                                          |---|----------|----------|--------|
| 01 | `descargar_incendios_conaf` | Descarga 18 temporadas de CONAF vía API | `incendios_conaf_raw.csv` |                    | 02 | `preparar_incendios_para_era5` | Limpieza, fl 2010-2019 | `incendios_conaf_2010_2020.xls` |
| 03 | `cruzar_incendios_era5` | Cruza cada evento con clima diario de ERA5 + anomalías mensuales |                           `incendios_conaf_era5_2010_2020.csv` |
| 04 | `agregacion_spark_comuna_anio` | Agrega a comuna×año con PySpark, calcula estrés hídrico y ranking de severidad |      `spark_comuna_anio/csv/` |
| 05 | `serie_nacional_mensual_mtys_era5` | Cruza serie 1985-2024 con ERA5 nacional, entrena XGBoost con anti-extrapolación | `incendios_nacional_mensual_era5.csv` |
| 06 | `mapa_georreferenciado_comuna` | Mapas coropléticos: severidad histórica + estrés hídrico por comuna (Folium) | 2 HTML interactivos |
| 07 | `mapa_puntos_importancia` | Heatmap de densidad + puntos catastróficos/estrés extremo (Folium) | 1 HTML interactivo |  | 08 | `modelo_riesgo_comuna_anio` | Clasificación o/catastrófico) con XGBClassifier |`modelo_riesgo_comuna_anio.csv` |                                                                                             | 09 | `dashboard` | Dashboard HTML con KPIs, mapasl modelo | `dashboard.html` |
                                                                                                                              ---
                                                                                                                              ## 📈 Métricas Clave del Proyecto
                                                                                                                              | Métrica | Valor |
|---------|-------|                                                                                                           | Total de eventos analizados | 60.530 |
| Período de incendios | 2010-2019 |                                                                                          | Cubrimiento climático | 2000-2025 (ERA5) |
| Comunas con datos | ~346 (GeoJSON Chile) |                                                                                  | Variables climáticas por evento | 4 (t2m, tp, ano
| Categorías de riesgo | 4 (bajo, normal, alto, catastrófico) |                                                               | Modelo de severidad | XGBClassifier (300 árboles,
| Modelo de tendencia nacional | XGBoost Regressor (anti-extrapolación) |                                                     
---                                                                                                                           
## 🛠️ Tecnologías Utilizadas                                                                                                  
| Categoría | Herramientas |                                                                                                  |-----------|-------------|
| **Lenguaje** | Python 3 |                                                                                                   | **Procesamiento masivo** | Apache Spark (PySpark,
| **Ciencia de datos** | Pandas, NumPy, Scikit-learn |                                                                        | **Modelos ML** | XGBoost (clasificación y regresi
| **Datos climáticos** | xarray, NetCDF4 (lectura de ERA5) |                                                                  | **Geoespacial** | Folium, branca, GeoJSON |
| **Visualización** | Plotly, Matplotlib |                                                                                    | **Formato de datos** | Parquet, CSV |
| **Ejecución** | Google Colab, Google Drive |                                                                                
---                                                                                                                           
## 📂 Estructura del Repositorio                                                                                              
MergeGeo/                                                                                                                     ├── 01_descargar_incendios_conaf.ipynb
├── 02_preparar_incendios_para_era5.ipynb                                                                                     ├── 03_cruzar_incendios_era5.ipynb
├── 04_agregacion_spark_comuna_anio.ipynb                                                                                     ├── 05_serie_nacional_mensual_mtys_era5.ipynb
├── 06_mapa_georreferenciado_comuna.ipynb                                                                                     ├── 07_mapa_puntos_importancia.ipynb
├── 08_modelo_riesgo_comuna_anio.ipynb                                                                                        ├── 09_dashboard.ipynb
├── era5_extraccion.ipynb          # Documenta cómo se generaron los .nc de ERA5                                              │                                   # (no se ejecut01-09)
├── incendios_conaf_raw.xls         # Output crudo del notebook 01 (input del 02)                                             ├── incendios_conaf_2010_2020.xls   # Output del no
├── 6.- Ocurrencia Nacional de Incendios Forestales según Mes, 1985 - 2024_octubre.xls                                        │                                   # Serie históriok 05
├── MergeGeo_informe.pdf                                                                                                      ├── .gitignore                      # Excluye los .ite de 100MB de GitHub)
├── README.md                                                                                                                 └── datos_procesados/               # Outputs gener09
    ├── incendios_conaf_era5_2010_2020.csv/.parquet                                                                               ├── incendios_nacional_mensual_era5.csv
    ├── spark_comuna_anio/                                                                                                        │   ├── csv/
    │   └── parquet/año=2010.../año=2019/                                                                                         ├── mapa_severidad_historica.html
    ├── mapa_estres_hidrico.html                                                                                                  ├── mapa_puntos_importancia.html
    ├── modelo_riesgo_comuna_anio.csv                                                                                             └── dashboard.html
                                                                                                                              ---
                                                                                                                              ## 🚀 Instrucciones de Ejecución
                                                                                                                              **Requisitos previos:**
- Google Colab (recomendado) o entorno local con Python 3.8+                                                                  - Los `.nc` de ERA5 (`era5_t2m_chile_2000_2025.nc`,nc`) **no están en
  este repo** — superan el límite de 100MB de GitHub. Están disponibles en el Drive del equipo;                                 hay que descargarlos de ahí y ubicarlos junto al
- Todos los demás archivos de entrada (`incendios_conaf_2010_2020.xls`,                                                         `incendios_conaf_raw.xls`, `6.- Ocurrencia Naciona raíz de este
  repo** — no hace falta conseguirlos aparte.                                                                                 
**Orden de ejecución:**                                                                                                       1. Los notebooks **01 y 02 ya están corridos** — suf_raw.xls`,
   `incendios_conaf_2010_2020.xls`) ya están en el repo. Si quieres volver a correr el                                           **01**, ahora pide la API key por teclado (`getpnsíguela gratis
   en `plataformadedatos.cl/user/developer`. El **02** puede correrse solo, sin pasar por el                                     01, usando `incendios_conaf_raw.xls` como input
2. Los `.nc` de ERA5 **ya están extraídos** (ver `era5_extraccion.ipynb` para el detalle de                                      cómo) — no es necesario re-correrlo, solo consegl equipo.
3. Ejecutar en orden: **03 → 04 → 05 → 06 → 07 → 08 → 09**                                                                    4. El notebook **09 (dashboard)** debe ejecutarse as los outputs
   anteriores                                                                                                                 
**Nota importante:** `dashboard.html` debe quedar en la misma carpeta que los 3 archivos de mapas HTML — los embebe por `iframcon ruta relativa.
                                                                                                                              ---
                                                                                                                              ## 🧑‍💻 Equipo de Trabajo
                                                                                                                              | Integrante | Rol en el Proyecto | GitHub |
|------------|-------------------|--------|                                                                                   | **Fabian Valdés** | Consultas y exploración de dathub.com/favc-5) |
| **Jairo Arias Valenzuela** | Extracción, limpieza y consolidación de registros CONAF (1985-2024) y consumo estructurado de lAPI meteorológica Open-Meteo | [@jairoarias208-beeparias208-beep) |
| **Matías Manríquez** | Visualización de datos: mapas geoespaciales interactivos en Folium y gráficos analíticos en Plotly | [@Mtys24](https://github.com/Mtys24) |
| **Javiera González Mardones** | Visualización de datos | [@Zelaznog-J](https://github.com/Zelaznog-J) |
| **José Salgado Escalona** | Redacción del informeuctura narrativa, redacción técnica y síntesis dehallazgos | [@JoseRicardoSE](https://github.com/JoseRicardoSE) |
