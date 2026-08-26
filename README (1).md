# Segmentación de proveedores en contrataciones de municipalidades provinciales del Perú (2021–2025)

Proyecto de minería de datos que busca identificar segmentos naturales de proveedores que participan en procesos de contratación de municipalidades provinciales del Perú, a partir de sus patrones de participación, adjudicación, escala económica y alcance territorial.

**Grupo Diamond Finders**
- Gianella Silvestre Nina
- Andre de la Fuente
- Luis Atto Pintado

## Descripción

La información de contrataciones públicas registrada en el SEACE está distribuida entre múltiples fuentes (postores, adjudicaciones, convocatorias, consorcios, entidades contratantes y penalidades), lo que dificulta identificar directamente los distintos perfiles de proveedores que operan en el mercado. Este notebook integra esas fuentes a nivel de proveedor —agregando su comportamiento durante 2021–2025— como paso previo a la segmentación mediante clustering.

Unidad de análisis: **proveedor**. Cada fila del dataset final resume el comportamiento agregado de un proveedor (número de convocatorias, adjudicaciones, tasa de adjudicación, monto total adjudicado, alcance territorial, participación en consorcios y penalidades).

## Contenido del notebook

1. Problema, usuario y objetivo de análisis
2. Fuentes de datos y unidad de análisis
3. Diagnóstico preliminar de calidad de datos
4. Código central: carga, inspección, limpieza e integración de las fuentes
5. Visualizaciones iniciales y hallazgos
6. Próximos pasos (clustering)

## Fuentes de datos

Las bases (CONOSCE: Postores, Adjudicaciones, Convocatorias, Consorcios; más Entidades Contratantes y Penalidades) no están incluidas en este repositorio. El notebook las descarga automáticamente al ejecutarse, clonando [BD_OECE_PROYECTO_DM](https://github.com/Protmdd/BD_OECE_PROYECTO_DM) en `/content/BD_OECE_PROYECTO_DM`. No es necesario descargarlas a mano.

## Cómo ejecutarlo

Pensado para correr en Google Colab:

1. Abre `ENTREGA_3_Proyecto_DM_OECE_FINAL.ipynb` en Colab.
2. Ejecuta las celdas en orden, de arriba hacia abajo. La primera celda clona el repositorio de datos; las siguientes instalan las dependencias que falten (`python-calamine`, `geopandas`).
3. El notebook genera al final `proveedores_integrado_analisis_FINAL.xlsx` en `/content`.

### Ejecución local

```bash
git clone <url-de-este-repositorio>
cd <carpeta-del-repositorio>
pip install pandas numpy matplotlib seaborn scikit-learn geopandas python-calamine requests
jupyter notebook ENTREGA_3_Proyecto_DM_OECE_FINAL.ipynb
```

Algunas celdas usan `display()` y `!pip install`, propios de Colab/Jupyter. Si lo corres en otro entorno, adapta esas líneas y cambia las rutas `BASE`, `CACHE` y `OUTPUT_DIR` (definidas al inicio del notebook, actualmente apuntan a `/content/...`).

## Salida

`proveedores_integrado_analisis_FINAL.xlsx`, con las hojas:

| Hoja | Contenido |
|---|---|
| `dataset_integrado` | dataset final, una fila por proveedor |
| `calidad_por_columna` | nulos y valores únicos por columna |
| `describe_numericas` | estadísticos descriptivos |
| `outliers_iqr` | conteo de valores atípicos por variable |
| `correlaciones` | matriz de correlaciones entre variables numéricas |

## Notas

- Los montos se convierten a soles usando el tipo de cambio promedio anual del BCRP; queda documentado como aproximación conocida.
- El análisis se limita a procesos de **municipalidades provinciales**, filtradas a partir de Entidades Contratantes.
- Las penalidades se conservan como variable de caracterización de los segmentos, no se usan para formar los clusters.
