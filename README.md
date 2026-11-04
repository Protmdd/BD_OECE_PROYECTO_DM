# Segmentación de proveedores en contrataciones de municipalidades provinciales del Perú (2021–2025)

Proyecto de minería de datos que identifica segmentos naturales de proveedores que participan en procesos de contratación de municipalidades provinciales del Perú, a partir de sus patrones de participación, adjudicación, escala económica y alcance territorial.

**Grupo Diamond Finders**
- Gianella Silvestre Nina
- Andre de la Fuente
- Luis Atto Pintado

## Descripción

La información de contrataciones públicas registrada en el SEACE está distribuida entre múltiples fuentes (postores, adjudicaciones, convocatorias, consorcios, entidades contratantes y penalidades), lo que dificulta identificar los distintos perfiles de proveedores que operan en el mercado. El proyecto integra esas fuentes a nivel de proveedor (comportamiento agregado 2021–2025), construye una matriz analítica y segmenta a los proveedores con técnicas de reducción de dimensión y clustering.

**Unidad de análisis:** proveedor postor. Cada fila resume su comportamiento agregado: número de convocatorias, ítems adjudicados, tasa de adjudicación, monto total adjudicado, alcance territorial, rubros, trayectoria, participación en consorcios y penalidades.

**Alcance:** 47 281 proveedores que participaron entre enero de 2021 y diciembre de 2025 en las contrataciones de las 196 municipalidades provinciales, más la Municipalidad Metropolitana de Lima (197 entidades de las 3 363 registradas).

## Estructura del repositorio

```
BD_OECE_PROYECTO_DM/
├── Bases de Datos Proyecto DM/          # fuentes (Git LFS)
├── HITO_2_Proyecto_DM_OECE_FINAL.ipynb  # integración, calidad y matriz analítica
├── Hito3_Proyecto_OECE_DM_.ipynb        # PCA, NMF, clustering, perfiles y mapas
└── README.md
```

La carpeta `Bases de Datos Proyecto DM` contiene las bases de CONOSCE por año (2021–2025): `CONOSCE_POSTOR`, `CONOSCE_ADJUDICACIONES`, `CONOSCE_CONVOCATORIAS` y `CONOSCE_CONSORCIO`, además de `entidades_contratantes.csv` y `penalidades.csv`.

## Hito 2: integración y matriz analítica

Notebook: `HITO_2_Proyecto_DM_OECE_FINAL.ipynb`

1. Problema, usuario y objetivo de análisis
2. Fuentes de datos y unidad de análisis
3. Claves y plan de integración (diagrama entidad-relación)
4. Diagnóstico preliminar de calidad
5. Código central: carga, detección de faltantes, duplicados, errores de tipo, outliers (IQR) y categorías raras, e integración por niveles (ítem de convocatoria → filtro poblacional → colapso a proveedor → consorcios → penalidades)
6. Visualizaciones iniciales y hallazgos
7. Decisiones de limpieza, transformaciones justificadas y matriz analítica (variables derivadas, outliers, discretización, escalamiento Z-score y selección de variables)
8. Próximos pasos

**Salidas** (se guardan en `/content` al correr en Colab):

| Salida | Contenido |
|---|---|
| `proveedores_integrado_analisis_FINAL.xlsx` | Hojas `dataset_integrado` (una fila por proveedor), `calidad_por_columna`, `describe_numericas`, `outliers_iqr` y `correlaciones` |
| `matriz_analitica_OECE.csv` | `ruc_proveedor` (solo trazabilidad) + 7 variables continuas con Z-score + `participa_en_consorcio` |
| `base_interpretativa` (objeto en memoria) | Variables en unidades originales, derivadas y discretizadas (`grupo_frecuencia`, `grupo_monto`), para perfilar segmentos |

## Hito 3: reducción de dimensión, clustering y perfilamiento

Notebook: `Hito3_Proyecto_OECE_DM_.ipynb`

| N.° | Sección | Contenido |
|---:|---|---|
| 1 | Carga de datos y límites provinciales | Matriz integrada y límites provinciales; verificación de cobertura |
| 2 | Preparación de las variables | 7 variables, `log1p`, Z-score y correlaciones |
| 3 | PCA | Número de componentes (regla del 85 % y Kaiser), cargas e interpretación |
| 4 | NMF | Elección de k por error de reconstrucción y lectura de patrones |
| 5 | Matrices para clustering | Comparación de 7 variables, PCA y NMF |
| 6 | Métodos de clustering | K-means, DBSCAN, jerárquico (Ward) y comparación con silueta, Davies-Bouldin, Calinski-Harabasz y ARI |
| 7 | Selección provisional y estabilidad | K-means con k = 6 sobre PCA, cruce con DBSCAN y estabilidad |
| 8 | Perfil de los segmentos | Nombres, tabla de perfil, concentración del monto, consorcio y penalidades |
| 9 | Distribución territorial | Mapa interactivo por provincia (GeoPandas y Plotly), caso de Lima y composición por departamento |
| 10 | Conclusiones preliminares | Decisiones, parámetros y conclusiones |
| 11 | Problemas y trabajo pendiente | Limitaciones antes del Hito 4 |
| 12 | Guardado de resultados | Exportación a CSV |

### Segmentos identificados (K-means, k = 6, sobre PCA de 3 componentes)

| N.° | Segmento | % de proveedores | Cómo se define |
|---:|---|---:|---|
| 0 | Ganadores de una sola vez | 29,5 % | Una convocatoria y la ganó |
| 1 | Postores de una sola vez sin éxito | 48,3 % | Una convocatoria y no ganó |
| 2 | Postores recurrentes sin éxito | 7,1 % | Varias convocatorias en varios meses y ninguna adjudicación |
| 3 | Ganadores recurrentes | 8,5 % | Varias convocatorias y adjudicaciones en al menos una |
| 4 | Generalistas multirrubro | 3,8 % | Operan en más de un rubro (2,05 en promedio) |
| 5 | Grandes operadores multimunicipales | 2,8 % | Mediana de 20 convocatorias, 8 municipalidades y más de 3 años de trayectoria |

La solución es provisional y se validará en el Hito 4. K-means sobre PCA y Ward (k = 6) coinciden con un ARI de 0,989, y la partición se reproduce con otras semillas (ARI mayor a 0,999).

**Salidas** (se guardan en el directorio de trabajo): `etiquetas_segmentos.csv`, `comparacion_kmeans.csv` y `comparacion_metodos.csv`.

## Cómo ejecutarlos

Los dos notebooks están pensados para Google Colab. Ejecuta las celdas en orden, de arriba hacia abajo.

- **Hito 2:** la primera celda instala Git LFS, clona este repositorio en `/content/BD_OECE_PROYECTO_DM` (las bases se descargan solas) e instala `python-calamine` si falta. Las dependencias de mapas (`geopandas`) se instalan en la celda que las usa.
- **Hito 3:** clona dos repositorios de apoyo: [Matriz_Integrada_DM_OECE](https://github.com/Protmdd/Matriz_Integrada_DM_OECE) (`proveedores_integrado.xlsx`, hoja `dataset_integrado`, resultado del Hito 2) y [limites-peru-geojson](https://github.com/josedaniel-cb/limites-peru-geojson) (`LIM_PROVINCIAL_PERU.json`). No necesita descargar nada a mano.

### Ejecución local

```bash
git lfs install
git clone https://github.com/Protmdd/BD_OECE_PROYECTO_DM.git
cd BD_OECE_PROYECTO_DM
pip install pandas numpy matplotlib seaborn scikit-learn scipy geopandas plotly openpyxl python-calamine requests
jupyter notebook
```

Algunas celdas usan `display()` y comandos de shell (`!apt-get`, `!git lfs`, `!pip`), propios de Colab/Jupyter. Fuera de Colab, adapta esas líneas y las rutas que apuntan a `/content/...`.

## Notas

- Los montos se convierten a soles usando el tipo de cambio promedio anual del BCRP; queda documentado como aproximación conocida.
- El análisis cubre municipalidades provinciales y la Municipalidad Metropolitana de Lima, filtradas a partir de Entidades Contratantes (197 de 3 363 entidades).
- Las penalidades y la participación en consorcio quedan fuera del modelo: se usan solo para caracterizar los segmentos ya formados, para evitar circularidad.
- Los valores atípicos no se eliminan: son proveedores o procesos de gran escala real. Se controlan con `log1p` y Z-score; sin `log1p`, K-means aísla a un solo proveedor en un cluster propio.
- El 53,9 % de los proveedores tiene un perfil idéntico al de otro, lo que infla la silueta; por eso DBSCAN se ejecuta sobre perfiles únicos con pesos.
- `n_departamentos` se conserva en `base_interpretativa`, pero queda fuera de la matriz analítica por su alta correlación con `n_municipalidades`.
- Cada proveedor cuenta en todas las provincias donde participó, por lo que los conteos por provincia suman más que los 47 281 proveedores.
