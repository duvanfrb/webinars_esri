# Webinar Esri: monitoreo y análisis territorial para entender el fenómeno de El Niño

Material del bloque del caso Barranquilla del webinar *Monitoreo y análisis territorial: herramientas prácticas para entender el fenómeno de El Niño*, organizado desde Esri Colombia. El repositorio contiene un cuaderno de ArcGIS Pro que se ejecuta celda por celda durante la sesión y que el público puede replicar con sus propios datos.

El flujo reproduce la metodología del trabajo de grado *Modelo espacial multicriterio para la identificación de zonas idóneas para la incorporación de infraestructura verde como mitigadora de islas de calor urbanas en el marco del POT de Barranquilla* (Robayo Puerto y Ruiz Buitrago, Universidad Antonio Nariño, 2026).

## Alcance y advertencia

Es un escenario de análisis sencillo, con fines didácticos y metodológicos. Cada análisis territorial tiene sus particularidades (zona, sensor, fechas, datos de entrada, normativa local), y los parámetros usados deben revisarse y justificarse en cada caso. Para un estudio más robusto hay que tener en cuenta, con precaución:

- Una o pocas fechas no representan el comportamiento térmico de un territorio: se recomiendan varias escenas por época y series de tiempo.
- La temperatura de superficie (LST) no es la temperatura del aire.
- Al comparar fechas se requiere el mismo sensor, la misma época del año, nubosidad baja y el mismo procesamiento. Diferencias de varios grados suelen indicar escenas no comparables y no un efecto climático.
- Una comparación no demuestra causalidad: la expansión urbana, los cambios de cobertura y el clima del día también cambian la LST.
- Los pesos del proceso analítico jerárquico son subjetivos y conviene validarlos y hacer un análisis de sensibilidad.
- Los resultados deben contrastarse con datos de campo o estaciones, y revisar el tratamiento de cuerpos de agua, vías y nubes delgadas.

Los productos de este análisis apoyan la decisión y no reemplazan estudios técnicos detallados.

## Contenido

| Archivo | Descripción |
| --- | --- |
| `Webinar_Fenomeno_ElNino/Analisis_Fenomeno_ElNino_.ipynb` | Cuaderno de ArcGIS Pro con las seis secciones del análisis y su documentación. |

Las rutas del cuaderno están reemplazadas por `D:\RUTA\...` y deben ajustarse antes de ejecutarlo.

## Requisitos

- ArcGIS Pro 3.x con la extensión Spatial Analyst.
- Un proyecto de ArcGIS Pro abierto, porque el cuaderno agrega capas al mapa activo con `arcpy.mp`.
- Para el análisis de puntos calientes, las herramientas del conjunto Spatial Statistics.

## Datos de entrada

| Dato | Descripción |
| --- | --- |
| Escenas de superficie | Reflectancia y temperatura de superficie con corrección atmosférica, una carpeta por escena y sin mezclar escenas en la misma carpeta. Se usan las bandas roja (`SR_B4`), infrarroja cercana (`SR_B5`), térmica (`ST_B10`) y la banda de calidad (`QA_PIXEL`). Si la ciudad queda repartida en dos escenas, se indican las dos y se promedian. |
| `AreasActividad_POT` | Áreas de actividad del POT con el campo de uso principal. |
| `Limite_UrbanoExpansion` | Límite del suelo urbano y de expansión, que sirve de máscara del análisis y de fuente del suelo de expansión sin uso definido. |

Las dos capas vectoriales deben estar dentro de la geodatabase definida en el cuaderno y en MAGNA-SIRGAS Origen Nacional (EPSG:9377).

## Parámetros que se deben ajustar

En la primera celda de código:

| Variable | Qué indicar |
| --- | --- |
| `gdb` | Ruta de la geodatabase de resultados. Si no existe, se crea. |
| `escenas` | Lista con la carpeta de cada escena de estudio. |
| `fc_areas_actividad` | Clase de entidad de áreas de actividad del POT. |
| `fc_expansion` y `fc_limite` | Límite urbano y de expansión. |
| `campo_uso` | Campo con el uso principal. Si es un campo con dominio de valores codificados, el cuaderno lo traduce a texto. |

En la sección 6 se ajusta además `ruta_escenas` y el diccionario `referencias`, con una carpeta por año de referencia.

## Flujo del análisis

1. **NDVI y temperatura superficial.** Se aplican los factores de escala de cada producto, se enmascaran píxeles con relleno, nube, sombra y cirros (bits 0 a 4 de `QA_PIXEL`) y reflectancias fuera del rango de 0 a 1, y se calculan NDVI y LST en grados Celsius. Con varias escenas se promedian los píxeles válidos.
2. **Reclasificación y combinación ponderada.** LST y NDVI se llevan a cinco niveles (LST por cortes naturales, NDVI por quintiles de la imagen y con orden invertido). Se combinan con pesos del método AHP de Saaty, 0,60 para LST y 0,40 para NDVI, y el índice continuo se divide en cinco niveles de igual amplitud.
3. **Cruce con el POT y matriz de decisión.** El suelo de expansión sin uso se anexa a las áreas de actividad y se cruza con el nivel de prioridad. Cada polígono recibe una categoría de intervención según el nivel y la exposición de la población asociada al uso.
4. **Puntos calientes.** El índice se agrega a 90 m, se convierte en puntos y se aplica Optimized Hot Spot Analysis (Getis-Ord Gi*) con corrección FDR. Se conservan los puntos con `Gi_Bin = 3` (confianza de 99 %).
5. **Núcleos de intervención.** Las áreas prioritarias en puntos calientes se agrupan a 20 m y se descartan las menores a 1 ha. Cada núcleo recibe un nivel de atención: Nivel 1 (prioridad muy alta) o Nivel 2 (prioridad alta). Se imprime el área en hectáreas de cada etapa del filtro.
6. **Comparación de escenarios.** Se calcula la LST de escenas de referencia de la misma época en otros años y se resta de la escena de estudio. La diferencia se resume por núcleo con estadística zonal.

### Matriz de decisión

Residencial, Mixto, Institucional y Dotacional son de exposición alta. Comercial, Servicios e Industrial son de exposición intermedia.

| Nivel de prioridad | Exposición alta | Exposición intermedia | Uso Otro | Sin uso definido |
| --- | --- | --- | --- | --- |
| 1 Muy bajo | Sin intervención | Sin intervención | Sin intervención | Condicionada |
| 2 Bajo | Complementaria | Complementaria | Condicionada | Condicionada |
| 3 Moderado | Recomendable | Recomendable | Condicionada | Condicionada |
| 4 Alto | Prioritaria | Recomendable | Condicionada | Condicionada |
| 5 Muy alto | Prioritaria | Prioritaria | Condicionada | Condicionada |

## Capas que se muestran en el mapa

El cuaderno desactiva la adición automática de resultados al mapa y solo agrega las capas que sirven para explicar el flujo. Los resultados intermedios quedan guardados en la geodatabase.

| Capa | Sección | Contenido |
| --- | --- | --- |
| `lst_base` | 1 | Temperatura de superficie en °C. |
| `ndvi_base` | 1 | Índice de vegetación. |
| `prioridad_continua` | 2 | Combinación ponderada, con sus valores completos. |
| `nivel_uso` | 3 | Nivel de prioridad y categoría de intervención por polígono. |
| `puntos_hotspot` | 4 | Resultado Gi* con todas las clases de `Gi_Bin`. |
| `nucleos_intervencion` | 5 | Núcleos de al menos 1 ha por nivel de atención. |
| `lst_dif_<año>` | 6 | Diferencia de LST frente a cada año de referencia. |

Otras salidas guardadas en la geodatabase: `prioridad_nivel`, `puntos_calientes_99`, `prioritaria_hotspot`, `nucleos_prioritarios`, `resumen_categoria` y `dif_nucleo_<año>`.

## Cómo usarlo

1. Abrir un proyecto de ArcGIS Pro y agregar un cuaderno a partir de `Webinar_Fenomeno_ElNino/Analisis_Fenomeno_ElNino_.ipynb`.
2. Ajustar las rutas y variables de la primera celda.
3. Ejecutar las celdas en orden. Las más lentas son el cálculo de las escenas, el análisis de puntos calientes y la agregación de polígonos.
4. Si una celda falla con el error 010240 al guardar un ráster, la capa suele estar abierta en un mapa. Se quita del mapa o se elimina el ráster de la geodatabase y se ejecuta de nuevo.

## Referencias

- Robayo Puerto, M. F. y Ruiz Buitrago, M. Y. (2026). Modelo espacial multicriterio para la identificación de zonas idóneas para la incorporación de infraestructura verde como mitigadora de islas de calor urbanas en el marco del POT de Barranquilla. Universidad Antonio Nariño.
- USGS. Landsat 8-9 Collection 2 Level 2 Science Product Guide (LSDS-1619).
- Saaty, T. L. (1977). A scaling method for priorities in hierarchical structures. Journal of Mathematical Psychology, 15(3), 234-281.
- Getis, A. y Ord, J. K. (1992). The analysis of spatial association by use of distance statistics. Geographical Analysis, 24(3), 189-206.
- Ord, J. K. y Getis, A. (1995). Local spatial autocorrelation statistics: distributional issues and an application. Geographical Analysis, 27(4), 286-306.
