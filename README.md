# Segmentación de proveedores en contrataciones de municipalidades provinciales del Perú (2021–2025)

Proyecto de minería de datos que busca identificar segmentos naturales de proveedores que participan en procesos de contratación de municipalidades provinciales del Perú, a partir de sus patrones de participación, adjudicación, escala económica y alcance territorial.

**Grupo Diamond Finders**
- Gianella Silvestre Nina
- Andre de la Fuente
- Luis Atto Pintado

## Descripción

La información de contrataciones públicas registrada en el SEACE está distribuida entre múltiples fuentes (postores, adjudicaciones, convocatorias, consorcios, entidades contratantes y penalidades), lo que dificulta identificar directamente los distintos perfiles de proveedores que operan en el mercado. Este notebook integra esas fuentes a nivel de proveedor —agregando su comportamiento durante 2021–2025—, aplica un diagnóstico de calidad, transforma y escala las variables relevantes, y construye la matriz analítica lista para clustering.

Unidad de análisis: **proveedor postor**. Cada fila del dataset integrado resume el comportamiento agregado de un proveedor (número de convocatorias, ítems adjudicados, tasa de adjudicación, monto total adjudicado, alcance territorial, participación en consorcios y penalidades).

## Contenido del notebook

1. Problema, usuario y objetivo de análisis
2. Fuentes de datos y unidad de análisis
3. Claves y plan de integración (diagrama entidad-relación)
4. Diagnóstico preliminar de calidad
5. Código central
   - Carga de datos
   - Detección de valores faltantes, duplicados, errores de tipo, outliers (IQR) y categorías raras
   - Integración de las fuentes por niveles (ítem de convocatoria → filtro poblacional → colapso a proveedor → consorcios → penalidades)
6. Visualizaciones iniciales y hallazgos (concentración del monto adjudicado, frecuencia × efectividad, frecuencia × alcance territorial)
7. Decisiones de limpieza, transformaciones justificadas y matriz analítica
   - Variables derivadas y reglas de negocio
   - Detección de outliers en la base de trabajo
   - Discretización para interpretabilidad
   - Escalamiento (Z-score vs. Min-Max) y selección de variables
   - Ensamblaje de `base_interpretativa` y `matriz_analitica`
8. Próximos pasos (determinación de K, clustering, perfilamiento y análisis de penalidades)

## Fuentes de datos

Las bases (CONOSCE: Postores, Adjudicaciones, Convocatorias, Consorcios; más Entidades Contratantes y Penalidades) no están incluidas en este repositorio. El notebook las descarga automáticamente al ejecutarse, clonando [BD_OECE_PROYECTO_DM](https://github.com/Protmdd/BD_OECE_PROYECTO_DM) mediante Git LFS en `/content/BD_OECE_PROYECTO_DM`. No es necesario descargarlas a mano.

## Cómo ejecutarlo

Pensado para correr en Google Colab:

1. Abre `HITO_2_Proyecto_DM_OECE_FINAL.ipynb` en Colab.
2. Ejecuta las celdas en orden, de arriba hacia abajo. La primera celda instala Git LFS y clona el repositorio de datos; las siguientes instalan las dependencias que falten (`geopandas`, entre otras).
3. El notebook genera, a partir de `df_proveedores`, los objetos `base_interpretativa` (variables originales y discretizadas, para perfilar los segmentos) y `matriz_analitica` (variables escaladas mediante Z-score, lista como insumo para K-Means).

### Ejecución local

```bash
git clone <url-de-este-repositorio>
cd <carpeta-del-repositorio>
pip install pandas numpy matplotlib seaborn scikit-learn geopandas requests
jupyter notebook HITO_2_Proyecto_DM_OECE_FINAL.ipynb
```

Algunas celdas usan `display()` y comandos de shell (`!apt-get`, `!git lfs`), propios de Colab/Jupyter. Si lo corres en otro entorno, adapta esas líneas e instala Git LFS manualmente antes de clonar el repositorio de datos.

## Salida

El notebook produce dos tablas principales, ambas derivadas de `df_proveedores` (45,754 proveedores, dataset integrado):

| Objeto | Filas | Contenido |
|---|---|---|
| `base_interpretativa` | 45,754 | Variables en sus unidades originales, variables derivadas (alcance territorial, trayectoria, indicadores) y variables discretizadas (`grupo_frecuencia`, `grupo_monto`); no se usa para el modelo, sino para perfilar los clústeres |
| `matriz_analitica` | 45,754 | `ruc_proveedor` (solo trazabilidad) + variables numéricas escaladas por Z-score (`n_convocatorias`, `n_items_adjudicados`, `tasa_adjudicacion`, `monto_total_adjudicado`, `n_municipalidades`, `n_rubros`, `dias_trayectoria`) más `participa_en_consorcio`; es el insumo directo para K-Means |

## Notas

- Los montos se convierten a soles usando el tipo de cambio promedio anual del BCRP; queda documentado como aproximación conocida.
- El análisis se limita a procesos de **municipalidades provinciales**, filtradas a partir de Entidades Contratantes (196 de 3,363 entidades registradas).
- Las penalidades se conservan como variable de caracterización de los segmentos y quedan fuera de la matriz analítica: se cruzarán con los clústeres ya formados, no se usan para formarlos.
- Los valores atípicos (IQR) detectados en montos y cantidades no se eliminan: se consideran proveedores o procesos de gran escala real. Su influencia en el modelo se controla mediante el escalamiento Z-score, no mediante su exclusión.
- `n_departamentos` se calcula y conserva en `base_interpretativa`, pero se excluye de `matriz_analitica` por su alta correlación con `n_municipalidades`.
