<h1 align="center">
  Procesamiento de Alto Volumen de Datos <br>
  Pontificia Universidad Javeriana <br>
  Facultad de Ingeniería de Sistemas <br>
  <img
    src="https://upload.wikimedia.org/wikipedia/commons/6/6c/Javeriana.svg"
    width="200"
    style="display:block; margin:auto; margin-top:12px;"
    alt="Logo Pontificia Universidad Javeriana">
</h1>

<h2 align="center">
  Dana Katalina Diaz Diaz <br>
  00020523642 | Ciencia de Datos
</h2>

---

<h1>Contenido del Taller</h1>

<ul>

<li>
<b>1. Preparación del entorno</b><br>
Se importan las librerías necesarias para el proyecto y se levanta la sesión de Spark.
Este paso es obligatorio antes de cualquier operación, ya que sin las librerías cargadas
y la sesión activa no es posible ejecutar ningún procesamiento de datos.
<ul>
<li><i>Importación de bibliotecas:</i> Se cargan las librerías de Python clásico (pandas, numpy, matplotlib, seaborn) para manipulación y visualización de datos, y las de PySpark para habilitar el procesamiento distribuido sobre el clúster.</li>
<li><i>Levantamiento de sesión SPARK:</i> Se crea y configura la sesión de Spark con el nombre "Calidad_Agua_Diaz". Es el punto de entrada obligatorio para leer datos desde el HDFS, ejecutar consultas SQL distribuidas y aplicar las transformaciones del pipeline.</li>
</ul>
</li>

<br>

<li>
<b>2. Carga de Datos desde el HADOOP HDFS</b><br>
Se cargan los datos del dataset de calidad del agua directamente desde el sistema de
archivos distribuido Hadoop (HDFS). Se usa este sistema porque el volumen de datos
requiere almacenamiento y lectura distribuida, que es precisamente el entorno para el
que está diseñado PySpark.
</li>

<br>

<li>
<b>3. Análisis y Preparación de Datos</b><br>
Antes de construir cualquier modelo es indispensable entender el estado de los datos:
qué contienen, si tienen errores, qué tipos tienen y cómo están distribuidos. Este
apartado cubre todo el proceso de inspección, limpieza e imputación que garantiza que
los datos sean confiables para el modelado.
<ul>
<li><i>Inspección de columnas:</i> Se identifican y describen las variables del dataset para entender qué representa cada una físicamente. Conocer el significado de cada parámetro es necesario para tomar decisiones correctas de limpieza y modelado.</li>
<li><i>Estadísticos descriptivos:</i> Se calculan media, desviación estándar, mínimo y máximo por columna para detectar anomalías como valores imposibles o fuera de rango. Aquí se identifica el problema de los valores "NA" cargados como texto.</li>
<li><i>Visualización con Boxplots:</i> Se grafican los boxplots de los parámetros fisicoquímicos para visualizar la distribución, detectar outliers y confirmar la asimetría de los datos. Esta visualización respalda las decisiones de imputación que se toman más adelante.</li>
<li><i>Tratamiento de tipos de datos:</i> Se convierte cada columna numérica a FloatType de Spark, ya que el dataset fue cargado con todos los tipos como String. Sin este paso, los valores "NA" no se convertirán en nulos reales y las funciones matemáticas de Spark no podrán operar sobre las columnas.</li>
<li><i>Visualización de nulos tras el casteo:</i> Se verifica cuántos nulos reales quedaron después del casteo. Es importante cuantificarlos antes de imputar para saber exactamente qué columnas requieren tratamiento y en qué magnitud.</li>
<li><i>Cálculo del Coeficiente de Variación:</i> Se calcula el CV de cada columna para decidir con criterio estadístico si imputar con la media o la mediana. Usar la media en datos muy dispersos introduciría un sesgo, por lo que es necesario evaluar la dispersión relativa antes de imputar.</li>
<li><i>Imputación de valores nulos:</i> Se reemplazan los nulos con el estadístico adecuado según el CV calculado: media para columnas poco dispersas (CV &lt; 30%) y mediana para columnas con alta dispersión (CV &gt;= 30%). Esto preserva la representatividad de los datos sin eliminar registros.</li>
<li><i>Filtrado final y creación de vista SQL:</i> Se crea una vista SQL temporal de los datos limpios para facilitar las consultas en los pasos siguientes. El filtro adicional garantiza que ningún nulo residual llegue al proceso de modelado.</li>
</ul>
</li>

<br>

<li>
<b>4. Visualización de Datos</b><br>
Se grafican los parámetros fisicoquímicos y se calcula la matriz de correlación para
entender el comportamiento individual de cada variable y las relaciones entre ellas
antes de construir el modelo. Este análisis permite detectar redundancias y confirmar
si las variables aportan información independiente al modelo.
<ul>
<li><i>Extracción de parámetros como listas de Python:</i> Se convierten las columnas del DataFrame de Spark a listas de Python, formato requerido por matplotlib para graficar. PySpark no es compatible directamente con las funciones de visualización de Python.</li>
<li><i>Gráficas de parámetros:</i> Se grafican los parámetros en pares para comparar su comportamiento a lo largo de todas las mediciones e identificar tendencias, picos o valores anómalos que no son visibles en los estadísticos descriptivos.</li>
<li><i>Matriz de Correlación:</i> Se calcula y visualiza la correlación de Pearson entre todos los parámetros para identificar si algún par de variables está tan relacionado que podría generar redundancia en el modelo, lo que afectaría su capacidad de generalización.</li>
</ul>
</li>

<br>

<li>
<b>5. Cálculo del Índice de Calidad del Agua (WQI)</b><br>
Se construye el índice WQI siguiendo la metodología de la referencia bibliográfica,
que asigna un puntaje ponderado a cada parámetro según su importancia relativa en la
calidad del agua. Este índice es la variable objetivo que el modelo aprenderá a predecir.
<ul>
<li><i>Rangos de calidad por parámetro:</i> Se clasifica cada parámetro en rangos discretos (100, 80, 60, 40, 0) según los umbrales definidos en la referencia. Esto estandariza todos los parámetros en una escala común antes de combinarlos.</li>
<li><i>Ponderación de parámetros:</i> Se multiplica cada rango por su peso bibliográfico. Los pesos reflejan la importancia relativa de cada parámetro en la calidad total del agua según la literatura científica.</li>
<li><i>Cálculo y clasificación del WQI:</i> Se suman los valores ponderados para obtener el WQI final y se clasifica en categorías (Excelente, Buena, Baja, Muy_Baja, Inadecuada) para facilitar la interpretación de los resultados.</li>
</ul>
</li>

<br>

<li>
<b>6. Visualización Geográfica</b><br>
Se mapea el índice WQI sobre el territorio de India para analizar la distribución
geográfica de la calidad del agua por estado. Este paso permite evaluar la hipótesis
de que la dispersión de variables como FECAL_COLIFORM o CONDUCTIVITY responde a
diferencias regionales.
<ul>
<li><i>Normalización de nombres de estados:</i> Se estandarizan los nombres de los estados entre el shapefile del mapa y el dataset de calidad del agua para que el merge entre ambas fuentes de datos sea exitoso.</li>
<li><i>Mapa con WQI e histograma por estado:</i> Se pinta el mapa coloreado por WQI y se grafica el histograma por estado para identificar qué regiones tienen mejor o peor calidad del agua y validar o rechazar la hipótesis planteada en el apartado de EDA.</li>
</ul>
</li>

<br>

<li>
<b>7. Creación y Entrenamiento del Modelo</b><br>
Se construye una red neuronal densa con Keras para aprender la relación entre los
rangos de calidad de los parámetros fisicoquímicos y el WQI. Se usa una red neuronal
porque la relación entre los parámetros y el WQI puede no ser lineal, y este tipo de
modelo tiene la capacidad de capturar patrones complejos en los datos.
<ul>
<li><i>Preparación de features y variable objetivo:</i> Se separan las columnas de entrada de la variable a predecir (WQI). Esta separación es el paso previo obligatorio antes de cualquier entrenamiento supervisado.</li>
<li><i>División en entrenamiento y prueba:</i> Se divide el dataset en 80% para entrenamiento y 20% para prueba con una semilla fija. La partición de prueba es un conjunto de datos que el modelo nunca verá durante el entrenamiento, lo que permite evaluar su capacidad de generalización de forma objetiva.</li>
<li><i>Arquitectura, compilación y entrenamiento:</i> Se define una red de 3 capas ocultas con 350 neuronas cada una y activación ReLU, compilada con el optimizador Adam y pérdida MSE. Esta arquitectura es un punto de partida estándar para regresión con redes densas. El entrenamiento se ejecuta por 200 épocas con un tamaño de lote de 81.</li>
</ul>
</li>

<br>

<li>
<b>8. Evaluación del Modelo</b><br>
Se mide el desempeño del modelo sobre los datos de entrenamiento y prueba usando MAE,
MSE, RMSE y R². Evaluar en ambos conjuntos es fundamental para detectar overfitting:
si el modelo funciona bien en entrenamiento pero mal en prueba, aprendió los datos de
memoria en lugar de generalizar el patrón.
</li>
<h1>Referencias</h1>

<ul>

<li>
Tyagi, S., Sharma, B., Singh, P., & Dobhal, R. (2013).
<i>Water Quality Assessment in Terms of Water Quality Index.</i>
American Journal of Water Resources, 1(3), 34-38.
Recuperado de: <a href="https://www.intechopen.com/chapters/69568">https://www.intechopen.com/chapters/69568</a>
</li>

<br>

<li>
Apache Software Foundation. (2024).
<i>Apache Spark Documentation.</i>
Recuperado de: <a href="https://spark.apache.org/docs/latest/">https://spark.apache.org/docs/latest/</a>
</li>

<br>

<li>
Apache Software Foundation. (2024).
<i>Apache Hadoop Documentation.</i>
Recuperado de: <a href="https://hadoop.apache.org/docs/stable/">https://hadoop.apache.org/docs/stable/</a>
</li>

<br>

<li>
Chollet, F. (2024).
<i>Keras Documentation.</i>
Recuperado de: <a href="https://keras.io/">https://keras.io/</a>
</li>

<br>

<li>
Pedregosa, F., et al. (2011).
<i>Scikit-learn: Machine Learning in Python.</i>
Journal of Machine Learning Research, 12, 2825-2830.
Recuperado de: <a href="https://scikit-learn.org/">https://scikit-learn.org/</a>
</li>

<br>

<li>
Central Pollution Control Board, India.
<i>River Water Quality Data — India.</i>
Recuperado de: <a href="https://cpcb.nic.in/">https://cpcb.nic.in/</a>
</li>

<br>

<li>
APHA (2017). Métodos estándar para el examen de agua y aguas residuales (23.ª ed.). Washington D.C.: Asociación Estadounidense de Salud Pública
</li>
</ul>
</ul>
