# Desarrollo técnico reproducible (`cr-la-carpio`)

Espacio preparado para versionar los scripts (Google Earth Engine, Python, openEO, R o SQL espacial) que generen las capas publicables en `geovisor/` y las contribuciones a `atlas/products/`.

## Cuadernos disponibles

### 1. `score_multiamenaza_edificaciones.ipynb`
- **Propósito:** Evaluación de multiamenaza ponderada y exposición de edificaciones (techos) en La Carpio (San José, Costa Rica).
- **Entorno:** Diseñado para ejecutarse en **Google Colab** o Jupyter Notebook local.
- **Flujo de trabajo:**
  1. Configuración de entorno geoespacial abierto (`geopandas`, `rasterio`, `rasterstats`, `overturemaps`, `folium`).
  2. Proyección en **CRTM05 (EPSG:5367)** oficial de Costa Rica con resolución configurable (10 m / 30 m).
  3. Catálogo modular y extensible de amenazas (`CATALOGO_AMENAZAS`) para incorporar capas vectoriales y ráster (inundación, cauces, deslizamientos, fallas, incidentes históricos).
  4. Descarga abierta de edificaciones de **Overture Maps** (o capa local de techos).
  5. Reducción zonal para clasificar edificaciones en 5 niveles estándar de GEE (*Muy Bajo*, *Bajo*, *Medio*, *Alto*, *Muy Alto*).
  6. Visualización interactiva en Folium y descarga de paquete `.zip` con GeoTIFFs y GeoPackage.
