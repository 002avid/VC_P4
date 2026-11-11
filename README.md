# Práctica 4 – Detección y seguimiento de vehículos y matrículas

## 1. Introducción

En esta práctica se desarrolla un sistema de visión por computador orientado al análisis y procesado de vídeos de tráfico. El objetivo principal es detectar y realizar el seguimiento de coches, personas, bicicletas y matrículas presentes en este [vídeo](https://drive.google.com/file/d/13_jrGrxRbnmeE6TLGrElZA5Xu7BUczPr/view?usp=drive_link).

Para ello, se utiliza YOLO como modelo de detección de objetos, tanto mediante un modelo preentrenado como mediante uno entrenado específicamente para localizar matrículas. Además, se incorporan diferentes funcionalidades para analizar el sentido de circulación, contabilizar los objetos detectados y anonimizar determinadas regiones de los vídeos.

Además, el sistema permite trabajar con dos métodos para localizar matrículas: un modelo YOLO entrenado con un conjunto de imágenes anotadas y un método basado en el análisis de contornos mediante OpenCV.

## 2. Detección y seguimiento de objetos

Para la detección de los elementos presentes en el vídeo se utiliza el modelo **YOLO11n**, que permite identificar tres clases de objetos:

- Personas.
- Bicicletas.
- Coches.

A partir de las detecciones, se realiza el seguimiento de los objetos a lo largo de los fotogramas. De esta manera, se mantiene un identificador para cada elemento y se puede seguir su movimiento durante el vídeo.

Esta información también permite evitar que un mismo objeto sea contabilizado varias veces cuando aparece en diferentes fotogramas.

## 3. Análisis del sentido de circulación

Una de las funcionalidades implementadas consiste en determinar el sentido de desplazamiento de los objetos detectados.

Para ello, se establecen líneas de referencia sobre el vídeo y se analiza cómo se desplazan los objetos respecto a ellas. En función de su movimiento, se clasifican como objetos que se acercan o se alejan.

Se mantienen contadores independientes para personas, bicicletas y coches, permitiendo obtener un resumen del tráfico observado.

[![Ver vídeo del análisis del sentido de la circulación](recursos/direccion.png)](https://drive.google.com/file/d/1HXbJKpnTgJLuS4aB-QNzveqdSc3nuR9y/view?usp=drive_link)

## 4. Detección de matrículas

Para localizar las matrículas de los vehículos se han implementado dos métodos diferentes.

### 4.1. Detección mediante YOLO entrenado

Se utiliza un modelo YOLO11n como punto de partida, que posteriormente se entrena con el dataset desarrollado por los compañeros de la asignatura, que contiene un conjunto de imágenes anotadas para la detección de matrículas.

El conjunto de datos se organiza separando las imágenes en tres particiones:

- **Train:** imágenes utilizadas durante el entrenamiento.
- **Validation:** imágenes utilizadas para comprobar el comportamiento del modelo durante el entrenamiento.
- **Test:** imágenes reservadas para evaluar el modelo una vez finalizado el entrenamiento.

El conjunto de datos contiene una única clase, denominada `matricula`.

La configuración utilizada durante el entrenamiento es la siguiente:

| Parámetro | Valor |
|---|---|
| Modelo inicial | YOLO11n |
| Épocas | 100 |
| Paciencia | 20 |
| Tamaño de imagen | 640 × 640 |
| Batch | 8 |
| Dispositivo | GPU, si está disponible; en caso contrario, CPU |

Una vez terminado el entrenamiento, se utiliza el mejor modelo obtenido (`best.pt`) para realizar las detecciones.

[![Ver vídeo de la detección de matrículas por entrenamiento](recursos/normal.png)](https://drive.google.com/file/d/1fr9FyrUlhC3aFWQFUCrxtBt63apLq9d9/view?usp=drive_link)

### 4.2. Detección mediante contornos

Como alternativa al modelo entrenado, se ha implementado un método de detección basado en operaciones de procesamiento de imágenes con OpenCV.

En este caso, primero se localizan los coches mediante YOLO11n y, posteriormente, se analiza individualmente la región de cada vehículo para intentar localizar su matrícula.

El procedimiento aplicado es el siguiente:

1. Se convierte la región del vehículo a escala de grises.
2. Se aplica un desenfoque gaussiano para reducir el ruido.
3. Se utiliza el detector de bordes Canny para localizar los cambios de intensidad.
4. Se aplica una operación morfológica de cierre para unir regiones próximas.
5. Se extraen los contornos y se analizan sus dimensiones y proporciones.

Finalmente, se seleccionan los candidatos que cumplen determinados criterios geométricos, como el ancho, el alto, la relación de aspecto y el área.

[![Ver vídeo de la detección de matrículas por contorno](recursos/contorno.png)](https://drive.google.com/file/d/1MFGXNq1IREiWNRUBbVmPf3CbteWJaglQ/view?usp=drive_link)

Este método permite localizar algunas matrículas, aunque sus resultados son menos consistentes que los obtenidos mediante el modelo entrenado. Esto se debe principalmente a que otras partes del vehículo, como elementos de la carrocería o sombras, pueden presentar características geométricas similares a las de una matrícula.

## 5. Anonimización de matrículas y personas

También se ha incorporado una funcionalidad de anonimización que permite aplicar un desenfoque gaussiano sobre las regiones de las matrículas y las personas detectadas.

A la hora de ejecutar el programa, aparecerá una caja de texto en la que el usuario debe decidir si la detección se hará por contornos o no. Debido a este sistema de preguntas al usuario, es posible tener muchas funcionalidades diferentes en un código compacto.

Para las matrículas se utiliza el modelo entrenado, mientras que las personas se localizan mediante el modelo YOLO11n general. Una vez obtenidas las regiones correspondientes, se aplica un desenfoque gaussiano con un tamaño de kernel de 51 × 51 y una desviación estándar de 30.

En el caso de las matrículas, también se relacionan las detecciones con los vehículos correspondientes. Para ello, se comprueba si el centro de la matrícula se encuentra dentro de la región delimitada por el vehículo.

Cuando se detectan varias matrículas asociadas a un mismo coche, se selecciona la que presenta una mayor confianza.

[![Ver vídeo de anonimización de coches y personas](recursos/blur.png)](https://drive.google.com/file/d/17tH8DFem0tviKMhNdJ4toys-J3yxdseT/view?usp=drive_link)

## 6. Archivo CSV

Cuando se utiliza el método de detección de matrículas mediante YOLO, se genera el archivo `C0142_resultados.csv`.

Este archivo recoge información de las detecciones realizadas durante el procesamiento del vídeo, incluyendo:

- Número de fotograma.
- Tipo de objeto detectado.
- Confianza de la detección.
- Identificador de seguimiento.
- Coordenadas de la región detectada.
- Información asociada a la matrícula localizada.

El archivo permite conservar los resultados del procesamiento para su posterior consulta y análisis, y puede ser consultado [aquí](P4/C0142_resultados.csv).

## 7. P4b – Reconocimiento de caracteres en matrículas

En esta segunda parte de la práctica se amplía el sistema anterior incorporando el reconocimiento de los caracteres de las matrículas detectadas. Para ello, se utiliza el detector YOLO entrenado para localizar las matrículas y, posteriormente, se aplican diferentes modelos OCR para intentar identificar su contenido.

El trabajo se divide en dos partes: primero, se integra EasyOCR en el procesamiento del vídeo de tráfico para mostrar las matrículas reconocidas y almacenar los resultados en un archivo CSV. Después, se realiza una evaluación independiente de EasyOCR y SmolVLM sobre un conjunto de recortes de matrículas con sus correspondientes etiquetas reales, con el objetivo de comparar la precisión y el tiempo de procesamiento de ambos modelos.



### 7.1. Entrenamiento del detector de matrículas

Para detectar las matrículas se entrenó un modelo **YOLO11n** utilizando un conjunto de imágenes anotadas que contiene una única clase: `matricula`.

El entrenamiento se realizó durante **100 épocas**, utilizando imágenes de tamaño 640x640. Una vez finalizado, se seleccionaron los mejores pesos del entrenamiento (`best.pt`), que se utilizaron posteriormente para realizar las detecciones.

#### Evolución del entrenamiento

A continuación, se muestran las principales gráficas obtenidas durante el entrenamiento:

**Pérdida de entrenamiento**

![Pérdida de entrenamiento](recursos/perdida_entrenamiento.png)

Esta gráfica muestra cómo disminuye el error del modelo durante el entrenamiento. Una tendencia descendente indica que el modelo está aprendiendo a localizar mejor las matrículas. También se observa, que al poner mas de 100 épocas se podría seguir bajando el error, ya que este no se estabiliza.

**Pérdida de validacMuestra la evolución del error utilizando imágenes de validación, que no se emplean directamente para entrenar el modelo. Permite comprobar cómo responde el modelo ante imágenes diferentes a las de entrenamiento.

**Precisión**

![Precisión](recursos/precision.png)

Representa la proporción de detecciones realizadas que corresponden realmente a matrículas. Una precisión mayor indica que el modelo comete menos falsas detecciones.

**Calidad de detección**

![Calidad de detección](recursos/calidad_deteccion.png)

Muestra la evolución de la calidad general de las detecciones mediante la métrica mAP@0.5. Cuanto mayor sea su valor, mejor es la capacidad del modelo para detectar las matrículas.

#### Resultados

Una vez finalizado el entrenamiento, se evaluó el modelo sobre el conjunto de validación mediante las siguientes métricas:

| Métrica | Resultado |
|---|---|
| Precisión (Precision) | [Valor obtenido] |
| Exhaustividad (Recall) | [Valor obtenido] |
| mAP@0.5 | [Valor obtenido] |
| mAP@0.5:0.95 | [Valor obtenido] |

En general, estas métricas permiten valorar tanto la fiabilidad de las detecciones como la capacidad del modelo para localizar correctamente las matrículas.

Los pesos finales seleccionados se encuentran en `matriculas/runs/train_val_matriculas/weights/best.pt`.

### 7.2. Reconocimiento de matrículas en vídeo con EasyOCR

Una vez detectadas las matrículas, se incorpora **EasyOCR** para intentar reconocer los caracteres que aparecen en ellas.

El procesamiento mantiene el detector general de objetos y el seguimiento de vehículos de la primera parte. Además, se utiliza el modelo YOLO entrenado para localizar las matrículas en cada fotograma.

Cuando se detecta una matrícula, se recorta la región correspondiente y se aplica un preprocesamiento antes de enviarla a EasyOCR:

1. Conversión del recorte a escala de grises.
2. Ampliación de la imagen cinco veces mediante interpolación cúbica.
3. Mejora del contraste utilizando CLAHE.
4. Reconocimiento del texto con EasyOCR, limitando los caracteres a números y letras mayúsculas.

Cuando el OCR devuelve varias lecturas, se selecciona aquella que presenta una mayor confianza. El texto reconocido se muestra sobre el vídeo junto a la región de la matrícula.

Además, se mantiene la asociación entre las matrículas y los vehículos correspondientes, comprobando si el centro de la matrícula se encuentra dentro de la caja delimitadora del coche.

El resultado del procesamiento se guarda en un vídeo, donde se visualizan las detecciones y las matrículas reconocidas. También se genera el archivo `C0142_resultados_Ocr.csv`, que almacena información de cada objeto detectado, incluyendo el fotograma, la clase, la confianza, el identificador de seguimiento, las coordenadas y, cuando corresponde, el texto reconocido de la matrícula.

### 7.3. Preparación del conjunto de datos para la evaluación OCR

Para comparar los modelos de reconocimiento se prepara un conjunto de imágenes a partir de las anotaciones JSON proporcionadas. De cada imagen original se extrae la región correspondiente a la matrícula y se conserva el texto real asociado a ella.

Durante este proceso se descartan las anotaciones que no contienen una matrícula válida, ya que no permiten evaluar correctamente el reconocimiento.

Finalmente, se obtiene un conjunto de **975 recortes de matrículas**, cada uno acompañado de su correspondiente texto real. Este conjunto se utiliza tanto para EasyOCR como para SmolVLM, de forma que ambos modelos trabajan exactamente con las mismas imágenes y las mismas referencias.

El archivo `matriculas_reales.csv` contiene la información de los recortes y las matrículas reales utilizadas durante la evaluación.

### 7.4. Evaluación de EasyOCR

En primer lugar, se evalúa **EasyOCR**, un sistema de reconocimiento óptico de caracteres que permite extraer texto directamente de las imágenes. Se utiliza el modelo preentrenado, sin realizar ningún entrenamiento adicional.

Para cada uno de los 975 recortes se obtiene una predicción, que posteriormente se compara con la matrícula real. Se registra si la lectura coincide exactamente y cuánto tiempo necesita el modelo para procesar cada imagen.

Los resultados obtenidos son los siguientes:

| Métrica | Resultado |
|---|---:|
| Imágenes evaluadas | 975 |
| Predicciones correctas | 259 |
| Predicciones incorrectas | 716 |
| Predicciones vacías | 118 |
| Exact Match | **26,56 %** |
| CER | **45,77 %** |
| Tiempo total | 30,78 s |
| Tiempo medio por imagen | **31,57 ms** |

El **Exact Match** representa el porcentaje de matrículas reconocidas completamente y sin ningún error. EasyOCR consigue 259 aciertos de las 975 imágenes, lo que supone un 26,56 %. Por tanto, en la mayoría de los casos no consigue reproducir exactamente todos los caracteres de la matrícula.

De las predicciones no exactas, 118 están vacías, lo que significa que el modelo no ha conseguido devolver ningún texto. En el resto de los casos se obtiene una lectura, pero contiene uno o varios caracteres incorrectos.

El **Character Error Rate (CER)** mide la proporción de errores a nivel de carácter respecto al texto real. Tiene en cuenta los caracteres sustituidos, insertados o eliminados, por lo que cuanto menor sea su valor, mejor es el reconocimiento. En esta evaluación se obtiene un CER del 45,77 %, lo que refleja las dificultades del modelo para reconocer correctamente las matrículas.

En cuanto al rendimiento, EasyOCR destaca por su velocidad, ya que procesa las 975 imágenes en aproximadamente 31 segundos, con un tiempo medio de 31,57 milisegundos por recorte.

### 7.5. Evaluación de SmolVLM

Como segundo enfoque se evalúa **SmolVLM-256M-Instruct**, un modelo de lenguaje visual (VLM) con 256 millones de parámetros, capaz de interpretar imágenes y extraer información de ellas.

A diferencia de EasyOCR, SmolVLM recibe cada recorte junto con una instrucción que le solicita identificar los caracteres de la matrícula y devolver el texto reconocido. Se utiliza el modelo preentrenado, sin realizar ningún entrenamiento adicional.

La evaluación se realiza sobre los mismos 975 recortes utilizados anteriormente con EasyOCR. De esta forma, se mantiene el mismo conjunto de imágenes y las mismas matrículas reales para poder comparar ambos enfoques.

Los resultados obtenidos son los siguientes:

| Métrica | Resultado |
|---|---:|
| Modelo | SmolVLM-256M-Instruct |
| Parámetros | 256 millones |
| Imágenes evaluadas | 975 |
| Predicciones correctas | 587 |
| Predicciones incorrectas | 388 |
| Predicciones vacías | 0 |
| Exact Match | **60,21 %** |
| CER | **15,58 %** |
| Tiempo total | 27 min 18 s |
| Tiempo medio por imagen | **1,68 s** |

SmolVLM consigue reconocer correctamente 587 matrículas, alcanzando un Exact Match del 60,21 %. Esto supone una mejora importante respecto a EasyOCR.

El CER obtenido es del 15,58 %, bastante inferior al anterior. Esto indica que las predicciones de SmolVLM contienen menos errores de carácter respecto a las matrículas reales.

Además, el modelo devuelve alguna respuesta para todos los recortes evaluados, por lo que no se producen predicciones vacías. Sin embargo, esto no significa que todas las respuestas sean correctas.

Su principal inconveniente es el tiempo de ejecución. Procesar las 975 imágenes requiere aproximadamente 27 minutos y 18 segundos, con una media de 1,68 segundos por imagen, un tiempo considerablemente superior al de EasyOCR.

### 7.6. Comparación de resultados

Para comparar ambos modelos se utilizan exactamente los mismos 975 recortes de matrículas y sus correspondientes etiquetas reales. De esta forma, las condiciones del experimento son las mismas y las diferencias obtenidas permiten analizar el comportamiento de cada modelo.

Se utilizan tres métricas principales:

- **Exact Match:** porcentaje de matrículas reconocidas completamente y sin errores. Un valor mayor indica un mejor resultado.
- **CER:** proporción de errores cometidos a nivel de carácter respecto al texto real. Un valor menor indica un mejor reconocimiento.
- **Tiempo medio de inferencia:** tiempo que necesita cada modelo para procesar un recorte. Un valor menor indica una mayor velocidad.

#### Comparación de métricas

| Métrica | EasyOCR | SmolVLM |
|---|---:|---:|
| Imágenes evaluadas | 975 | 975 |
| Predicciones correctas | 259 | **587** |
| Predicciones incorrectas | 716 | **388** |
| Predicciones vacías | 118 | **0** |
| Exact Match | 26,56 % | **60,21 %** |
| CER | 45,77 % | **15,58 %** |
| Tiempo total | **30,78 s** | 27 min 18 s |
| Tiempo medio | **31,57 ms** | 1,68 s |

#### Comparación gráfica

Las siguientes gráficas permiten observar las diferencias entre ambos modelos en cuanto a reconocimiento exacto, errores por carácter y velocidad de procesamiento.

![Comparación Exact Match](recursos/comparacion_exact_match.png)

![Comparación CER](recursos/comparacion_cer.png)

![Comparación del tiempo medio de inferencia](recursos/comparacion_tiempo.png)

#### Análisis de los resultados

En cuanto al reconocimiento completo de las matrículas, SmolVLM obtiene un Exact Match del 60,21 %, frente al 26,56 % de EasyOCR. Esto supone una diferencia de **33,65 puntos porcentuales** y 328 matrículas reconocidas correctamente más sobre el mismo conjunto de imágenes.

También se observa una diferencia importante en el CER. SmolVLM obtiene un 15,58 %, mientras que EasyOCR alcanza un 45,77 %. Por tanto, SmolVLM comete menos errores a nivel de carácter y sus predicciones se aproximan más al texto real.

Sin embargo, esta mejora en el reconocimiento tiene un coste importante en tiempo de ejecución. EasyOCR necesita una media de 31,57 milisegundos por imagen, mientras que SmolVLM necesita 1,68 segundos. Esto significa que SmolVLM tarda aproximadamente **53 veces más** en procesar cada recorte.

Por tanto, los resultados muestran que EasyOCR destaca por su velocidad, pero presenta mayores dificultades para reconocer correctamente las matrículas. SmolVLM, por su parte, consigue una precisión considerablemente superior, aunque necesita mucho más tiempo de procesamiento.

#### Especificaciones del equipo

Las pruebas se han realizado utilizando el siguiente equipo:

| Componente | Especificación |
|---|---|
| GPU | NVIDIA GeForce RTX 2060 |
| Memoria de GPU | 6 GB |
| RAM | 32 GB |
| PyTorch | 2.4.1 |
| CUDA | 12.1 |

Los tiempos de inferencia dependen del hardware y de la configuración utilizada, por lo que los resultados temporales corresponden al equipo en el que se han realizado estas pruebas.
