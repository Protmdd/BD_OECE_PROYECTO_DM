# Segmentación de proveedores en contrataciones de municipalidades provinciales del Perú (2021–2025)

Proyecto de Minería de Datos que identifica perfiles de proveedores que participan en las contrataciones de municipalidades provinciales del Perú, según sus patrones de participación, adjudicación, escala económica y alcance territorial, y representa su distribución por provincia en un mapa interactivo.

**Curso:** Data Mining, Universidad del Pacífico. **Docente:** Soledad Espezúa Llerena.

**Grupo Diamond Finders**
- Gianella Silvestre Nina
- Andre de la Fuente
- Luis Atto Pintado

## Pregunta de análisis

¿Qué perfiles de proveedores se distinguen por sus patrones de participación y adjudicación en las municipalidades provinciales del Perú durante 2021–2025, y cómo se distribuye su participación entre provincias?

- **Usuario principal:** OECE (gestión de información sobre el mercado de proveedores).
- **Usuarios secundarios:** municipalidades provinciales y empresas que evalúan participar.
- **Alcance:** análisis descriptivo. No predice adjudicaciones ni evalúa legalidad o cumplimiento.

## Resultado final (Hito 4)

**Técnica:** K-means con k = 6 sobre las 7 variables originales, transformadas con `log1p` (solo las de asimetría > 1) y estandarizadas, **sin PCA**, para conservar la interpretación de cada variable.

**Datos:** 47 281 proveedores (1 fila = 1 RUC), participaciones entre el 4 de enero de 2021 y el 31 de diciembre de 2025, 196 provincias.

| Métrica (k = 6) | Valor |
|---|---|
| Silueta (muestra fija de 5 000) | 0,813 (máxima entre k = 2 y 8) |
| Davies-Bouldin | 0,761 (mínimo) |
| Calinski-Harabasz | 58 924 (máximo) |
| Cluster más pequeño | 2,70 % de los proveedores |
| ARI entre semillas (mínimo) | 0,9999 |
| ARI frente a un modelo entrenado en una submuestra de 22 000 | 0,9998 |
| ARI sin la variable `n_items_adjudicados` (6 variables) | 0,993 |

### Segmentos

| Segmento | Proveedores | % | % del monto adjudicado | Cómo se define |
|---|---:|---:|---:|---|
| S1 Ganadores de una sola vez | 13 847 | 29,29 | 67,47 | Una convocatoria y la ganó |
| S2 Postores de una sola vez sin éxito | 22 712 | 48,04 | 0,00 | Una convocatoria y no ganó |
| S3 Postores recurrentes sin éxito | 3 504 | 7,41 | 0,00 | Mediana de 3 convocatorias y ninguna adjudicación |
| S4 Ganadores recurrentes | 4 173 | 8,83 | 13,74 | Mediana de 4 convocatorias y adjudicación en todos |
| S5 Generalistas multirrubro | 1 769 | 3,74 | 6,19 | 2,05 rubros en promedio |
| S6 Grandes operadores multimunicipales | 1 276 | 2,70 | 12,60 | Mediana de 20 convocatorias y 9 municipalidades |

Los nombres son etiquetas descriptivas, no una clasificación validada externamente.

### Territorio

- Cada proveedor cuenta en todas las provincias donde participó: 74 244 apariciones proveedor–provincia (1,57 por proveedor).
- Los porcentajes solo se calculan en las 177 provincias con al menos 100 proveedores. El resto aparece en gris en los mapas.
- Las provincias son las de **participación**, no el domicilio fiscal del proveedor.
- Lima tiene una proporción de postores de una sola vez sin éxito (49,1 %) muy superior a la mediana de las 177 provincias comparables (27,2 %) y una de grandes operadores (9,9 %) muy inferior (20,8 %). Es un contraste descriptivo, no explicado.

## Estructura del repositorio

```
BD_OECE_PROYECTO_DM/
├── Bases de Datos Proyecto DM/              # fuentes CONOSCE/SEACE (Git LFS)
├── HITO_2_Proyecto_DM_OECE_FINAL.ipynb      # integración, calidad y matriz analítica
├── Hito4_OECE_DiamondFinders.ipynb          # modelo final, validación, perfiles y mapas
├── outputs/                                 # generado por el notebook del Hito 4
│   ├── tablas/                              # 14 tablas CSV
│   ├── figuras/                             # 5 figuras PNG
│   └── mapas/mapa_clusters_proveedores.html # mapa interactivo (Plotly)
├── requirements.txt
├── .gitattributes
└── README.md
```

La carpeta `Bases de Datos Proyecto DM` contiene las bases por año (2021–2025): `CONOSCE_POSTOR`, `CONOSCE_ADJUDICACIONES`, `CONOSCE_CONVOCATORIAS` y `CONOSCE_CONSORCIO`, además de `entidades_contratantes.csv` y `penalidades.csv`.

## Notebooks

### Hito 2: integración y matriz analítica

`HITO_2_Proyecto_DM_OECE_FINAL.ipynb` integra las seis fuentes por niveles (ítem de convocatoria, filtro a municipalidades provinciales, colapso a proveedor, consorcios y penalidades), convierte los montos a soles con el tipo de cambio promedio anual del BCRP y construye la matriz analítica. Su resultado es `proveedores_integrado.xlsx`, hoja `dataset_integrado`, publicado en [Matriz_Integrada_DM_OECE](https://github.com/Protmdd/Matriz_Integrada_DM_OECE).

### Hito 4: modelo final

`Hito4_OECE_DiamondFinders.ipynb` parte de la matriz del Hito 2 y ejecuta, de inicio a fin:

| N.° | Sección | Contenido |
|---:|---|---|
| 01 | Entorno y carga | Semilla, rutas, carga y validación de las fuentes |
| 02 | Matriz analítica | 7 variables, `log1p` selectivo, Z-score y correlaciones |
| 03 | Pipeline | Diagrama del pipeline y ejemplo trazable de un proveedor |
| 04 | Selección de k | K-means con k = 2 a 8: silueta, Davies-Bouldin, Calinski-Harabasz, tamaño mínimo y ARI entre semillas |
| 05 | Validación y sensibilidad | Estabilidad, prueba sin `n_items_adjudicados` y renumeración determinista de segmentos |
| 06 | Perfiles | Tabla de perfiles en unidades originales y Z-scores por segmento |
| 07 | Vinculación geográfica | Unión proveedor–provincia y auditoría de cobertura |
| 08 | Mapas | Mapa interactivo HTML y figura estática para el paper |
| 09 | Discusión | Síntesis de evidencia, usos posibles y límites |
| 10 | QA y exportación | Controles de consistencia y escritura de todas las salidas |

Las alternativas exploradas en el Hito 3 (PCA, NMF, DBSCAN y Ward) se resumen en el paper como antecedentes; el notebook final no las recalcula.

## Salidas (`outputs/`)

| Carpeta | Archivos principales |
|---|---|
| `tablas/` | `comparacion_kmeans_k.csv`, `criterios_decision_k.csv`, `sensibilidad_sin_items_adjudicados.csv`, `perfil_clusters.csv`, `revision_nombres_segmentos.csv`, `estadisticas_provinciales.csv`, `auditoria_union_geografica.csv`, `auditoria_penalidades.csv`, `etiquetas_proveedores.csv`, `correlaciones_7_atributos_completa.csv`, `validacion_entrada.csv`, `resumen_interpretacion_segmentos.csv`, `usos_posibles_oece.csv`, `sintesis_para_paper_hito4.csv` |
| `figuras/` | `pipeline.png`, `diagnostico_correlaciones_6_mostradas.png`, `seleccion_k.png`, `perfil_clusters.png`, `mapa_provincias_paper.png` |
| `mapas/` | `mapa_clusters_proveedores.html` |

## Cómo ejecutarlo

### Google Colab

1. Sube `Hito4_OECE_DiamondFinders.ipynb` a Colab.
2. Ejecuta las celdas en orden. No hay que descargar nada a mano: la primera celda clona dos repositorios de apoyo con `git clone`:
   - [Matriz_Integrada_DM_OECE](https://github.com/Protmdd/Matriz_Integrada_DM_OECE) (`proveedores_integrado.xlsx`)
   - [limites-peru-geojson](https://github.com/josedaniel-cb/limites-peru-geojson) (`LIM_PROVINCIAL_PERU.json`)
3. Al final, la sección 10 verifica que se generaron las 20 salidas esperadas.

### Local

```bash
git lfs install
git clone https://github.com/Protmdd/BD_OECE_PROYECTO_DM.git
cd BD_OECE_PROYECTO_DM
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook Hito4_OECE_DiamondFinders.ipynb
```

- Requiere Python 3.10 o superior y conexión a internet para clonar los repositorios de apoyo.
- Si ya tienes los archivos, evita la clonación con variables de entorno: `OECE_EXCEL` (ruta al `proveedores_integrado.xlsx`) y `OECE_GEOJSON` (ruta a `LIM_PROVINCIAL_PERU.json`).
- Reproducibilidad: `SEED = 1`, `MIN_PROVEEDORES = 100`, muestra fija de 5 000 proveedores para la silueta.
- El Hito 2 está pensado para Google Colab: su primera celda instala Git LFS, clona este repositorio e instala `python-calamine`. Fuera de Colab, adapta las líneas de shell (`!apt-get`, `!git lfs`, `!pip`) y las rutas `/content/...`.

## Fuentes de datos

| Fuente | Uso |
|---|---|
| [Portal de Datos Abiertos del SEACE (OECE)](https://bi.seace.gob.pe/pentaho/api/repos/%3Apublic%3Aportal%3Adatosabiertos.html/content) | Postores, adjudicaciones, convocatorias, consorcios, entidades contratantes y penalidades, 2021–2025. Copia en `Bases de Datos Proyecto DM` |
| [BCRP, tipo de cambio promedio del periodo](https://estadisticas.bcrp.gob.pe/estadisticas/series/mensuales/tipo-de-cambio-de-las-principales-monedas-promedio-del-periodo-sol-um-m) | Conversión a soles de los montos en moneda extranjera (cerca de 3 % de las adjudicaciones) |
| [limites-peru-geojson](https://github.com/josedaniel-cb/limites-peru-geojson) | Límites provinciales para los mapas |

## Limitaciones

- La unidad es el identificador de proveedor (RUC y también códigos internos de postor y de consorcio), no el contrato.
- La ubicación es la provincia de participación, no el domicilio fiscal.
- El 53,9 % de los proveedores tiene un perfil idéntico al de otro (21 810 perfiles únicos), lo que infla la silueta; en perfiles únicos baja a 0,597.
- Los segmentos S4 y S6 tienen fronteras difusas (silueta media de 0,27 y 0,25; 17,7 % de siluetas negativas en S6).
- Las penalidades quedan fuera del modelo, y su integración tiene llaves por validar (569 de 1 278 proveedores con penalidad no tienen ítems adjudicados). No se interpretan sanciones ni riesgo.
- Cinco años agregados en una fila: no hay evolución temporal ni causalidad.
- La pertenencia a un segmento no demuestra irregularidades ni predice adjudicaciones futuras.

## Entrega

El paper final (IEEE, LaTeX) y este repositorio se entregan por Blackboard. Los resultados del paper, el repositorio y la exposición coinciden con la ejecución del notebook del Hito 4.
