# Segmentación de Estados Mexicanos por Perfil Delictivo (K-Means Clustering)

**Autor:** Marco Iván Rodríguez Graciano
**Ocupación:** Ingeniero Físico | Data Analyst & Backend Developer (.NET/Python) — experiencia en banca, riesgo crediticio y migración de sistemas core bancarios
**Contacto:** macovanrodriguezgraciano@gmail.com

---

## Objetivo del proyecto

Agrupar los 32 estados de México según su perfil de incidencia delictiva (2015-2025) mediante clustering no supervisado (K-Means), con el fin de identificar patrones de similitud entre entidades — un ejercicio análogo a la segmentación de clientes/cartera que se realiza en análisis de riesgo bancario, aplicado aquí a un dominio distinto para ampliar mi portafolio de Data Science.

## Fuente de datos

[DataMéxico](https://www.economia.gob.mx/datamexico/es) — consulta personalizada sobre el Registro de Crímenes del SESNSP (Secretariado Ejecutivo del Sistema Nacional de Seguridad Pública). El dataset final contiene **166,400 registros** con las siguientes columnas: Entidad federativa, Año, Mes, Tipo de crimen (39 categorías), y Número de crímenes (conteo).

## Stack técnico

- Python (Google Colab)
- pandas, numpy — manipulación de datos
- scikit-learn — StandardScaler, KMeans, PCA, silhouette_score
- matplotlib — visualización

## Metodología

El proyecto sigue un flujo estándar de análisis no supervisado, con un énfasis particular en la **detección y corrección de un sesgo metodológico** que surgió durante el desarrollo (ver sección de Hallazgos y Dificultades).

1. **EDA inicial**: revisión de nulos (ninguno encontrado), tipos de dato, rango temporal (2015-2025), y distribución de la variable `Value` (número de crímenes)
2. **Transformación de formato**: de formato largo (una fila por estado-año-mes-delito) a formato ancho (una fila por estado, una columna por tipo de delito), usando `pivot_table` con suma acumulada del periodo completo
3. **Escalado**: `StandardScaler`, necesario porque K-Means calcula distancias euclidianas y las columnas tenían escalas muy distintas entre sí (ej. "Robo" en cientos de miles vs. "Rapto" en unidades)
4. **Selección de K**: método del codo (Elbow Method) + Silhouette Score, para no elegir el número de clusters de forma arbitraria
5. **Aplicación de K-Means** y interpretación de los centroides en valores reales (no escalados), para que los resultados fueran legibles y explicables

## Resultados — Análisis por volumen total de incidencia delictiva

Con K=3 (silhouette score = 0.397), el modelo segmentó a los estados así:

| Cluster | Estados | Perfil |
|---|---|---|
| **0** | Ciudad de México, Estado de México | Volumen extremadamente alto en prácticamente todos los tipos de delito (ej. ~1.24M casos promedio de Robo) |
| **2** | Baja California, Chihuahua, Jalisco, Guanajuato, Nuevo León, Veracruz | Volumen medio-alto (~318K casos promedio de Robo) — entidades con alta actividad económica/poblacional |
| **1** | Las 24 entidades restantes | Volumen consistentemente bajo (~111K casos promedio de Robo) |

Se validó visualmente con una proyección PCA (40 dimensiones → 2), confirmando la separación entre los dos outliers extremos (Cluster 0), el grupo intermedio (Cluster 2), y la masa compacta del resto de estados (Cluster 1).

## Hallazgos y dificultades (con honestidad)

Este proyecto tuvo un hallazgo importante que decidí documentar en lugar de ocultar, porque refleja el tipo de pensamiento crítico que me parece más valioso que un resultado "perfecto":

**El clustering por volumen total está dominado por el tamaño poblacional/económico del estado, no por un perfil o tipología de crimen distintiva.** Es decir, el modelo no está diciendo "estos estados tienen un tipo de crimen distinto", sino simplemente "estos estados tienen más crimen de todo tipo, porque son más grandes".

Para investigar si existía un perfil delictivo *relativo* (independiente del tamaño), realicé un segundo análisis normalizando cada tipo de delito como proporción del total de delitos de cada estado (en lugar de valores absolutos). El resultado: el silhouette score cayó de 0.40 a un rango de 0.04–0.07 — una separación muy débil. Esto sugiere que, **proporcionalmente, la composición del crimen es bastante homogénea entre estados mexicanos**; lo que realmente los diferencia es la intensidad/volumen total, no el tipo de delito que predomina.

Decidí no forzar una interpretación elaborada sobre un resultado con tan poca separación real — considero que reconocer cuándo los datos no sostienen una conclusión "bonita" es parte importante de un análisis riguroso, y preferí dejar este proyecto enfocado en el análisis que sí tenía una señal clara (el de volumen total), documentando el segundo intento como parte del proceso de investigación.

## Limitaciones conocidas

- La unidad de análisis (32 estados) es pequeña para técnicas de clustering, lo que hace que 1-2 outliers (CDMX, Edomex) tengan un peso desproporcionado en el resultado
- El dataset no permite granularidad a nivel municipal en esta consulta, lo que habría dado más unidades y potencialmente una segmentación más robusta
- No se incluyeron variables de contexto (población, PIB estatal) que habrían permitido normalizar de forma más directa por tasas per cápita

## Cómo correrlo

1. Clona este repositorio
2. Abre `crime_clustering_analysis.ipynb` en Google Colab o Jupyter
3. Asegúrate de que el archivo `data/crimenes_mexico.csv` esté en la misma ruta relativa
4. Ejecuta las celdas en orden

## Próximos pasos

Este proyecto fue un ejercicio de aprendizaje deliberado antes de abordar un proyecto más robusto y directamente aplicable a mi área de interés profesional (banca, detección de fraude y AML), donde planeo trabajar con un dataset transaccional de mayor granularidad y volumen, y extender el análisis hacia una API en ASP.NET Core / C# que sirva los resultados del modelo.

---

*Este proyecto forma parte de mi portafolio como Data Analyst / Backend Developer en transición hacia roles de Data Science aplicada a finanzas.*
