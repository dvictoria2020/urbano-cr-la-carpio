# Metodología de Estimación de Población — La Carpio, San José, Costa Rica

**Proyecto:** Copernicus Centroamérica — Intervención Urbanística Integral La Carpio  
**Elaborado por:** Jacqueline Vargas Bogantes  
**Fecha:** Noviembre 2026  
**Estado:** Versión 1.0

> **Nota de consistencia y uso:** este documento resume el procedimiento y los resultados registrados en los archivos disponibles; no constituye un censo ni una validación institucional. Las fuentes presentan dos diferencias que deben resolverse antes de usar las cifras como una única línea base: el total exportado de techos se registra como 4.127 en la descripción de la extracción y como 4.125 en la tabla de comparación y el resumen JSON; además, la suma del escenario base por sector es 25.209 personas, mientras que la malla hexagonal suma 25.241 (diferencia: 32). Hasta reconciliar versiones y proceso, se deben conservar ambas cifras con su procedencia.

---

## 1. Contexto y objetivo

El asentamiento informal de La Carpio (La Uruca, San José) carece de un censo actualizado de población y vivienda a nivel intra-sectorial. Este documento describe el procedimiento utilizado para estimar la población residente a partir de huellas de techo detectadas por satélite, calibradas con datos locales, y distribuidas espacialmente mediante una malla hexagonal.

**Objetivo principal:** Generar una estimación de población por sector y por celda hexagonal de 30 m, como insumo para el Plan Maestro de Intervención Urbanística Integral.

---

## 2. Fuentes de datos

| Capa | Fuente | Formato | CRS |
|------|--------|---------|-----|
| Techos locales (encuesta) | Municipalidad San José / IMAS | Shapefile (`Techos.shp`) | CRTM05 (EPSG:5367) |
| Techos satelitales | Google Open Buildings v3 (GEE) | GeoJSON | WGS84 (EPSG:4326) |
| Sectores del asentamiento | Delimitación operativa interna | Shapefile (`Sectores_Carpio.shp`) | CRTM05 (EPSG:5367) |
| Usos de suelo | Levantamiento de campo | Shapefile (`UsodelaTierra_AID.shp`) | CRTM05 (EPSG:5367) |
| Límite de estudio | Google Earth Engine Asset | Vector (GEE) | WGS84 |

### 2.1 Extracción de techos con Google Earth Engine

Se utilizó el siguiente script de Google Earth Engine para exportar los polígonos de Open Buildings dentro del límite del asentamiento:

```javascript
// Script GEE — Exportar Open Buildings dentro del límite de La Carpio
var limite = ee.FeatureCollection("projects/ee-jakyvargasbg/assets/Limite_Carpio");
var openBuildings = ee.FeatureCollection(
  "GOOGLE/Research/open-buildings/v3/polygons"
);

// Filtrar por intersección con el límite
var techos = openBuildings.filterBounds(limite.geometry());

// Filtrar por confianza mínima (opcional, recomendado ≥ 0.65)
var techosConfiables = techos.filter(ee.Filter.gte("confidence", 0.65));

// Exportar a Google Drive
Export.table.toDrive({
  collection: techosConfiables,
  description: "techos_open_buildings_la_carpio",
  fileFormat: "GeoJSON",
  folder: "GEE_Exports"
});
```

**Resultado:** 4,127 polígonos exportados (confidence ≥ 0.65), guardados en:  
`Informacion base/techos_open_buildings_la_carpio.geojson`

---

## 3. Proceso de calibración

### 3.1 Comparación de capas

Se realizó un cruce espacial entre los techos locales (encuesta) y los techos satelitales (Google Open Buildings), agrupados por sector, para verificar la coherencia entre ambas fuentes.

| Sector | Techos locales | Techos Google | Ratio G/L |
|--------|---------------|---------------|-----------|
| Central | 643 | 592 | 0.92 |
| La Libertad | 371 | 377 | 1.02 |
| Las Brisas | 343 | 359 | 1.05 |
| María Auxiliadora | 816 | 825 | 1.01 |
| Pequeña Gran Ciudad | 586 | 615 | 1.05 |
| Prodesarrollo | 106 | 321 | 3.03 ⚠️ |
| Roble Norte | 547 | 565 | 1.03 |
| Roble Sur | 263 | 255 | 0.97 |
| San Vicente | 5 | 216 | 43.2 ⚠️ |
| **TOTAL** | **3,680** | **4,125** | **0.988** |

> ⚠️ **Nota sobre Prodesarrollo y San Vicente:** La capa local presentó datos incompletos en estos sectores (posiblemente por cobertura parcial del levantamiento). Para la estimación se utilizó la capa de Google Open Buildings como fuente primaria en todos los sectores, dado que el ratio global (0.988) indica una alta concordancia general.

### 3.2 Conclusión de calibración

El ratio global Google/Local reportado es **0.988** (prácticamente 1:1). Esta concordancia agregada no demuestra por sí sola la exactitud de cada sector: Prodesarrollo y San Vicente presentan diferencias grandes y la documentación atribuye parte de ellas a una posible cobertura local incompleta. Los datos históricos del IMAS (1993) registraban 2,135 viviendas; la estimación modelada identifica **5,602 viviendas** (~2.6× de crecimiento en ~33 años). La comparación temporal es indicativa y requiere confirmar comparabilidad de cobertura, definiciones y fuentes.

---

## 4. Modelo de estimación de viviendas

No todos los polígonos de techo corresponden a una única vivienda. Se aplicó un **modelo piecewise por área** para convertir techos en viviendas:

| Rango de área | Factor | Justificación |
|--------------|--------|---------------|
| < 12 m² | 0.5 viviendas | Estructura accesoria o muy pequeña |
| 12 – 120 m² | 1.0 vivienda | Vivienda unifamiliar típica |
| 120 – 200 m² | 1.8 viviendas | Posible subdivisión (vecindad / inquilinato) |
| > 200 m² | área / 75 | Estructura colectiva o multifamiliar |

**Total estimado de viviendas:** 5,602

---

## 5. Escenarios de personas por vivienda

Se utilizó una tasa base de **4.5 personas/vivienda** como promedio general para asentamientos informales en Costa Rica. Se construyeron escenarios adicionales a partir de referencias censales:

| Escenario | Personas/vivienda | Referencia |
|-----------|------------------|------------|
| Mínimo | 3.4 | Promedio nacional urbano — INEC CRI 2011 |
| Bajo | 3.9 | Promedio GAM (Gran Área Metropolitana) — INEC 2011 |
| **Base** | **4.5** | **Referencia para asentamientos precarios (MIVAH)** |
| Alto | 5.0 | Estudios previos IMAS — La Carpio 2000-2005 |
| Máximo | 5.4 | Límite superior documentado en asentamientos similares CR |

---

## 6. Resultados por sector (escenario base 4.5)

| Sector | Techos Google | Viv. Estimadas | Pob. 3.4 | Pob. 3.9 | **Pob. 4.5** | Pob. 5.0 | Pob. 5.4 |
|--------|-------------|---------------|---------|---------|------------|---------|---------|
| Central | 592 | 849 | 2,887 | 3,311 | **3,820** | 4,245 | 4,585 |
| La Libertad | 377 | 418 | 1,421 | 1,630 | **1,881** | 2,090 | 2,257 |
| Las Brisas | 359 | 582 | 1,979 | 2,270 | **2,619** | 2,910 | 3,143 |
| María Auxiliadora | 825 | 1,142 | 3,883 | 4,454 | **5,139** | 5,710 | 6,167 |
| Pequeña Gran Ciudad | 615 | 694 | 2,360 | 2,707 | **3,123** | 3,470 | 3,748 |
| Prodesarrollo | 321 | 555 | 1,887 | 2,165 | **2,498** | 2,775 | 2,997 |
| Roble Norte | 565 | 694 | 2,360 | 2,707 | **3,123** | 3,470 | 3,748 |
| Roble Sur | 255 | 336 | 1,142 | 1,310 | **1,512** | 1,680 | 1,814 |
| San Vicente | 216 | 332 | 1,129 | 1,295 | **1,494** | 1,660 | 1,793 |
| **TOTAL** | **4,125** | **5,602** | **19,048** | **21,849** | **25,209** | **28,010** | **30,252** |

---

## 7. Malla hexagonal de distribución espacial

Para visualizar la distribución intra-sectorial de la población, se generó una **malla hexagonal con celdas de ~30 m** (distancia entre vértices opuestos).

### 7.1 Parámetros de generación

| Parámetro | Valor |
|-----------|-------|
| Tamaño de celda | 30 m (distancia entre vértices) |
| Área aproximada por celda | ~779 m² |
| CRS de trabajo | CRTM05 (EPSG:5367) |
| CRS de exportación | WGS84 (EPSG:4326) |
| Hexágonos generados | 304 (dentro del límite) |
| Hexágonos con población > 0 | 274 |

### 7.2 Metodología de asignación de población a hexágonos

1. **Proyección:** Todos los polígonos de techo de Google Open Buildings se reprojectan a CRTM05.
2. **Intersección espacial:** Cada techo se asigna al hexágono que contiene su centroide.
3. **Suma de viviendas por celda:** Se aplica el modelo piecewise de área por cada techo asignado a la celda.
4. **Cálculo de población:** `población = viviendas_celda × personas_por_vivienda` (para cada escenario).
5. **Densidad:** `densidad = (población / área_celda_ha)` en hab/ha.
6. **Sector dominante:** Se asigna el sector con mayor número de techos dentro de cada celda.

### 7.3 Resultados de la malla hexagonal (escenario base 4.5)

| Indicador | Valor |
|-----------|-------|
| Población total estimada | **25,241 hab.** |
| Densidad máxima por celda | **731 hab/ha** |
| Celda con más población | **171 hab** |

### 7.4 Campos del GeoJSON exportado

| Campo | Descripción |
|-------|-------------|
| `hex_id` | Identificador único de celda |
| `sector` | Sector dominante asignado |
| `techos_count` | Número de techos Google en la celda |
| `viviendas_est` | Viviendas estimadas (modelo piecewise) |
| `pob_esc_3_4` | Población — escenario 3.4 pers/viv |
| `pob_esc_3_9` | Población — escenario 3.9 pers/viv |
| `pob_base_4_5` | Población — escenario base 4.5 pers/viv |
| `pob_esc_5_0` | Población — escenario 5.0 pers/viv |
| `pob_esc_5_4` | Población — escenario 5.4 pers/viv |
| `densidad_hab_ha` | Densidad poblacional (hab/ha), escenario base |

---

## 8. Archivos generados

```
Informacion base/
├── techos_open_buildings_la_carpio.geojson     ← Export Google Earth Engine
└── Resultados_Estimacion/
    ├── estimacion_viviendas_la_carpio.json      ← Resumen JSON por sector
    ├── Sectores_Carpio_Estimacion.shp           ← Shapefile con estimaciones sectoriales
    ├── Sectores_Carpio_Estimacion.geojson       ← GeoJSON sectores (WGS84)
    └── malla_hexagonal_poblacion.geojson        ← Malla hexagonal (WGS84)

urbano-cr-la-carpio/                            ← Repositorio del geovisor
└── geovisor/
    └── capas/
        └── malla_hexagonal_poblacion.geojson   ← Copia para el geovisor web
```

---

## 9. Scripts de procesamiento

Todos los scripts están en Python 3.12. Requieren los paquetes:  
`shapely`, `pyshp`, `pyproj`, `numpy`

```
pip install shapely pyshp pyproj numpy
```

| Script | Función |
|--------|---------|
| `analisis_techos.py` | Cruce espacial por sector; cálculo de ratios |
| `comparativa_capas.py` | Tabla comparativa Google vs. local |
| `calcular_estimaciones.py` | Modelo piecewise y escenarios de población |
| `exportar_capas.py` | Exporta shapefile y GeoJSON de sectores con estimaciones |
| `generar_hexagonos.py` | Genera malla hexagonal y asigna población por celda |

> Los scripts scratch están en el directorio de sesión del agente. Para reproducir el análisis desde cero en otra máquina, se recomienda mover estos scripts a `scripts/analisis/` dentro del repositorio y adaptar las rutas de entrada.

---

## 10. Limitaciones y recomendaciones

1. **Verificación de campo:** Los resultados son estimaciones basadas en detección remota. Se recomienda validar con al menos una muestra de terreno (10-15% de los sectores).
2. **Sector San Vicente:** La capa local original tenía solo 5 registros frente a 216 de Google; la diferencia puede reflejar un levantamiento incompleto en ese sector.
3. **Sector Prodesarrollo:** La capa local registraba 106 techos frente a 321 de Google; mismo fenómeno.
4. **Techos no residenciales:** Google Open Buildings detecta cualquier estructura. Se recomienda aplicar una máscara de uso de suelo (capa `UsodelaTierra_AID.shp`) para excluir usos no residenciales en versiones futuras.
5. **Actualización:** Los datos de Google Open Buildings corresponden a imágenes de 2023 aproximadamente. Se recomienda actualizar el análisis con imágenes más recientes a medida que estén disponibles.
6. **CRS:** Todos los archivos de análisis se procesan en CRTM05 (EPSG:5367); los GeoJSON para web están en WGS84 (EPSG:4326).

---

## 11. Reproducción del análisis en otra computadora

### Requisitos de software

- Python 3.10 o superior
- Git
- GitHub CLI (`gh`) — para sincronizar el repositorio

### Pasos

```powershell
# 1. Clonar el repositorio del geovisor
gh repo clone jakyvargasb-ops/urbano-cr-la-carpio

# 2. Instalar dependencias Python
pip install shapely pyshp pyproj numpy

# 3. Reconstruir el sitio
cd urbano-cr-la-carpio
python scripts/build_site.py --output site

# 4. Validar
python scripts/validate.py --source
python scripts/validate.py --site site

# 5. Levantar servidor local
python -m http.server 8000 --directory site
# Abrir: http://localhost:8000/aplicacion/
```

### Rutas de datos de entrada
Los shapefiles de análisis están almacenados localmente en:
```
C:\DIRECCION DE DESARROLLO URBANO\Copernicus II Fase\Informacion base\
```
Esta carpeta **no está en el repositorio Git** (datos sensibles/pesados). Para reproducir el análisis completo en otra máquina, copiar esta carpeta y ajustar las rutas en los scripts de estimación.

---

*Documento generado con apoyo de Antigravity (Google DeepMind) · Noviembre 2026*
