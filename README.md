# Computer-Vision-Final

En este informe, queremos explicar cuál de todos estos modelos es mejor
al momento de detectar vestimenta protectora en los trabajos de
construcción.

Realizando algunos experimentos, y entendiendo también un poco la teoría
con todas sus excepciones, creemos que, el mejor modelo a utilizar,
entre **YOLO-World** y **YOLOv8**, el más confiable y preciso, será
**YOLOv8**, ya que es capaz de aprender las características principales
de los cascos, la iluminación real de una cabeza sin casco y es
entrenado mediante datos del mundo real.

Para que entendamos que es cada modelo, sus funciones y como se
diferencian, vamos a hacer algunas definiciones.

Primero que todo tenemos a **YOLOv8**, este es un modelo de aprendizaje
supervisado profundo (Deep Learning) especializado en la detección de
objetos en tiempo real. Funciona mediante un entrenamiento directo y
exclusivo con un conjunto de datos cerrado (como imágenes de obras de
construcción), donde aprende a reconocer clases predefinidas (por
ejemplo, "casco" y "no_casco") mediante la extracción de características
visuales específicas.

Segundo, tenemos a **YOLO-World,** este es un modelo que une la visión
por computador con el procesamiento de lenguaje natural (NLP). A
diferencia de YOLOv8, no requiere un reentrenamiento con imágenes para
detectar nuevas clases; en su lugar, utiliza descripciones de texto
("prompts") para localizar objetos en tiempo real. Sin embargo, su
precisión depende críticamente de la claridad del texto y puede
presentar dificultades para interpretar variaciones visuales complejas o
términos abstractos sin un entrenamiento visual previo en el entorno
específico.

Por último, tenemos a los **Modelos de Lenguaje y Visión (VLM -
Florence-2 / Moondream2),** son modelos generativos que integran la
comprensión visual con la generación de lenguaje natural a gran escala.
Su principal ventaja es el razonamiento contextual profundo y la
capacidad de responder a preguntas libres sobre la escena (*Visual
Question Answering*), permitiendo capturar detalles semánticos sutiles
que los detectores tradicionales ignoran. No obstante, su alta demanda
computacional y menor velocidad de procesamiento los hacen difíciles de
implementar en sistemas de alerta de seguridad en tiempo real.

Para poder explicar a más detalles la implementación de cada uno de
ellos, con estadísticas empíricas, les vamos a ir mostrando todos los
datos y sus significados, a su vez que les vamos explicando el motivo
detrás de cada uno.

Funcionamiento y datos de **YOLOv8:**

**1. Rendimiento por Clase (AP50-95 y AP@0.5)**

- **Éxito en Clases Activas**: El modelo detecta muy bien el equipo de
  seguridad cuando está presente. Por ejemplo, Hardhat (Casco) tiene un
  **AP@0.5 de 0.8580** y Safety Vest (Chaleco) un **AP50-95 de 0.5082**.
  Esto ocurre porque están diseñados industrialmente para ser muy
  llamativos (alta visibilidad).

- **Dificultad en Clases Pasivas**: Las clases que representan la
  ausencia de equipo, como NO-Hardhat (**0.2702**) y NO-Safety Vest
  (**0.3828**), tienen un rendimiento mucho menor. Esto se debe a la
  enorme variabilidad visual (cabello, gorras, ángulos) y a que factores
  como la iluminación o reflejos pueden engañar al modelo imitando la
  forma de un casco.

**2. Evaluación a Nivel de Evento (Detección de Infractores)**

Evaluamos la capacidad del sistema para detectar si hay **al menos una
persona sin casco** en la imagen:

- **Métricas Obtenidas**:

  - Precision: **0.9500** (El 95% de las alertas son reales, solo 1
    falsa alarma).

  - Recall: **0.7600** (Se detecta el 76% de las infracciones reales).

  - F1-Score: **0.8444** (Balance general).

- **Análisis de Riesgo**: Identificamos que los **6 Falsos Negativos
  (FN)** son el peligro real en la obra, ya que significan trabajadores
  desprotegidos que el sistema no detectó. El **Falso Positivo (FP =
  1)** solo representa una falsa alarma tolerable.

**3. Propuesta de Mejora Técnica**

- Decidimos que para un entorno de alta seguridad es preferible **bajar
  el umbral de confianza** (por ejemplo, de 0.25 a 0.10). Esto obligará
  a la máquina a alertar ante la más mínima sospecha, reduciendo los
  peligrosos Falsos Negativos, aunque aumenten un poco las falsas
  alarmas.

Funcionamiento y datos de **YOLO-World (Zero-Shot):**

**1. Rendimiento y Métricas a Nivel de Evento**

El modelo YOLO-World demostró una alta sensibilidad teórica al registrar
un Recall de 0.8800, logrando identificar correctamente 22 de las
infracciones reales (Verdaderos Positivos). Sin embargo, este
rendimiento conllevó un alto costo operativo: el sistema generó un
elevado número de falsas alarmas, acumulando 24 Falsos Positivos. Como
consecuencia, su precisión se redujo críticamente a un 0.4783,
estableciendo un F1-Score general de 0.6197.

**2. Naturaleza del Modelo y Falsas Alarmas**

A diferencia de YOLOv8, YOLO-World funciona bajo el paradigma de
vocabulario abierto (zero-shot). Al no haber sido entrenado con un
conjunto de datos supervisado y delimitado a imágenes reales de
construcción, el modelo carece del ajuste fino necesario para reconocer
texturas, brillos y geometrías específicas del entorno industrial.
Depende del mapeo de conceptos lingüísticos generales, lo que causa que
confunda con facilidad materiales, herramientas u otros objetos con
cascos de seguridad, elevando la tasa de falsos positivos.

**3. Velocidad de Inferencia**

En términos de rendimiento computacional, el modelo demostró una alta
eficiencia al registrar una latencia promedio de 0.0339 segundos por
frame, lo que equivale a un procesamiento de 29.47 cuadros por segundo
(FPS). Esta velocidad de procesamiento en tiempo real resulta idónea
para sistemas de monitoreo activo, ya que equipara la velocidad de
actualización de la percepción visual humana.

**4. Sensibilidad Semántica y Vulnerabilidad del Prompt**

La prueba de robustez cuantitativa reveló una debilidad crítica en la
estabilidad del sistema. Variaciones semánticas mínimas en las
instrucciones de texto destruyeron la consistencia del detector:

- El uso de la variante sintáctica \['protective headwear', 'unprotected
  head'\] resultó en un F1-Score de 0.0000, provocando una ceguera
  operativa absoluta ante los riesgos de la obra.

- Solo las etiquetas con alta representación en el preentrenamiento del
  modelo (como \['cap', 'person'\]) mantuvieron un desempeño moderado
  (F1-Score de 0.5897).

Esto demuestra que un sistema de vocabulario abierto presenta un riesgo
de seguridad latente en producción, ya que una modificación accidental
en la sintaxis del prompt por parte de un operador podría desactivar por
completo las alarmas de seguridad.

**Conclusión Comparativa**

Mientras que YOLOv8 ofrece un sistema robusto, con alta precisión
(0.9500) y baja tasa de falsas alarmas gracias a su entrenamiento
supervisado, YOLO-World se muestra sumamente inestable debido a su
dependencia semántica y alta tasa de falsos positivos. Para entornos de
alta responsabilidad y riesgo como la construcción, el uso de YOLOv8
resulta significativamente más seguro y viable para proteger la
integridad de los trabajadores.

Funcionamiento y datos de **Florence-2 (VLM de Grounding)**

**1. Métricas a Nivel de Evento**

El modelo Florence-2 exhibió un comportamiento polarizado en la
detección de infracciones. Registró una sensibilidad o Recall perfecto
de 1.0000, identificando de manera efectiva los 25 eventos de riesgo
reales (Verdaderos Positivos) y generando 0 Falsos Negativos. No
obstante, esta sensibilidad absoluta se obtuvo a costa de una tasa
crítica de falsas alarmas, acumulando 57 Falsos Positivos y 0 Verdaderos
Negativos. Como resultado, su precisión descendió a un 0.3049, con un
F1-Score resultante de 0.4673.

**2. Diagnóstico de Alineación Lógica y Robustez**

El análisis de depuración reveló un fallo de alineación semántica en el
modelo. Al evaluar una misma coordenada espacial, Florence-2 predijo
simultáneamente las etiquetas contrapuestas "head" (cabeza) y "bare
head" (cabeza descubierta) sobre el mismo objeto. Esta incapacidad para
discriminar el uso efectivo del casco provoca que el sistema clasifique
cualquier cabeza detectada como una infracción. En un entorno de
producción, este comportamiento se traduce en un estado de alarma
constante e indiscriminado, inutilizando el sistema debido al fenómeno
técnico de fatiga de alarmas.

**3. Rendimiento Computacional**

La velocidad de inferencia promedio del modelo fue de 1.81 fotogramas
por segundo (FPS), correspondiente a una latencia de 0.5522 segundos por
frame. Esta tasa de procesamiento resulta insuficiente para la
prevención de accidentes en tiempo real en entornos dinámicos de
construcción.

Funcionamiento y datos de **Moondream2 (VQA)**

**1. Métricas a Nivel de Evento**

Moondream2 demostró un desempeño más equilibrado en la clasificación de
escenas en comparación con los otros modelos zero-shot. Obtuvo un Recall
de 0.8000 (20 Verdaderos Positivos y 5 Falsos Negativos) y una precisión
de 0.3922, consecuencia de registrar 31 Falsos Positivos y 26 Verdaderos
Negativos. Su balance general se tradujo en un F1-Score de 0.5263.

**2. Operación Basada en Preguntas y Respuestas Visuales**

A diferencia de los detectores de objetos tradicionales, Moondream2
opera bajo un esquema de Visual Question Answering (VQA), procesando la
consulta sintáctica: *"Is everyone in the image wearing a hard hat /
helmet?"*. Aunque el modelo demuestra una capacidad de razonamiento
conceptual superior para evaluar el contexto de seguridad de la imagen
completa, carece de la capacidad nativa de generar coordenadas de
geolocalización (bounding boxes) para múltiples infractores simultáneos
en el mismo frame.

**3. Limitación Crítica de Velocidad**

El principal obstáculo de Moondream2 es su rendimiento temporal. El
modelo registró una latencia promedio de 1.1810 segundos por frame,
procesando únicamente 0.85 FPS. Al ser aproximadamente 30 veces más
lento que YOLOv8, este desfase temporal impide su despliegue como
sistema de seguridad activa, dado que el tiempo de cómputo supera el
margen seguro de reacción ante un accidente en desarrollo.

Después de todos los datos mostrados entenderán que, el modelo que mejor
se encuentra equilibrado hoy en día es YOLOv8, es rápido, preciso y, si
bien es cierto que lanza "falsas alarmas", a diferencia de los demás
modelos, son totalmente admisibles.

Las mejoras técnicas serian realizar una serie de packs de imágenes de
todo tipo de equipo de protección, para que, evitar cualquier confusión
del modelo y así disminuir la cantidad de FP.

Ahora, seguimos con los puntos de Metodologías de Pruebas y Propuestas
de Implementación y Mitigación.

**Metodología de Pruebas (Validación Incremental)**

Esta sección describe el plan de evaluación secuencial para certificar
que el modelo cumple con el estándar de seguridad antes de su puesta en
marcha.

- **Fase 1: Validación de Línea Base (Condiciones Ideales):** Pruebas en
  exteriores con iluminación solar óptima y clima despejado. El objetivo
  es comprobar que el algoritmo base alcanza el 95% de precisión en
  condiciones óptimas.

- **Fase 2: Robustez Ambiental (Variabilidad Climática en Chile):**
  Evaluación bajo escenarios climáticos adversos en exteriores (días
  nublados, atardeceres, lluvia y nieve en zonas de alta montaña o sur
  del país).

- **Fase 3: Entornos Semi-confinados (Interiores de Obra):** Pruebas
  dentro de edificaciones en construcción y estructuras habitacionales,
  donde la luz natural disminuye y aumentan las oclusiones por
  herramientas o muros.

- **Fase 4: Ambientes Críticos (Espacios Confinados):** Evaluación final
  en entornos de visibilidad reducida como túneles, subterráneos y
  bodegas de barcos, caracterizados por iluminación artificial
  deficiente y polvo en suspensión.

**Propuesta de Implementación y Mitigación**

Esta sección detalla la solución tecnológica seleccionada y las acciones
para su puesta en producción.

- **Selección de la Arquitectura:** Elección de YOLOv8 debido a su alta
  velocidad de procesamiento en tiempo real (requisito crítico de
  seguridad activa para evitar accidentes) frente a la latencia inviable
  de modelos multimodales como Moondream2.

- **Estrategia de Ajuste Fino (Fine-Tuning):** Creación de un paquete de
  datos especializado de Equipos de Protección Personal (EPP). Este
  conjunto de datos incluirá variabilidad de cascos (colores, accesorios
  como barbiquejos) y contraejemplos de control (gorros, capuchas) para
  reducir los Falsos Positivos.

- **Criterio de Aceptación (Éxito):** Se establece un umbral mínimo de
  precisión del 95% de forma individual en cada una de las cuatro fases
  de prueba. No se autorizará el despliegue si el modelo promedia 95%
  pero falla en entornos críticos como túneles.

| **Entorno de Trabajo** | **Porcentaje del Dataset** | **Justificación Técnica y de Seguridad**                                                                                                                                           |
|------------------------|----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Intemperie**         | **25%**                    | Permite mantener la precisión básica de detección en condiciones óptimas de visibilidad sin saturar el entrenamiento con escenarios visualmente sencillos.                         |
| **Interiores de Obra** | **45%**                    | Concentra el aprendizaje en las zonas con mayor tasa estadística de accidentabilidad laboral en construcción, donde la luz natural disminuye y aumentan las oclusiones.            |
| **Espacios Cerrados**  | **30%**                    | Prioriza la detección en áreas de alto peligro (túneles, búnkeres o bodegas de barcos), donde el tiempo de respuesta del sistema es crítico para la supervivencia ante un colapso. |

**Análisis de Frameworks, Licencias y el Fenómeno de la Negación**

Para complementar la recomendación de despliegue, se analizaron los
cuatro modelos bajo criterios de propiedad intelectual, licenciamiento y
su capacidad lógica para resolver la pregunta de seguridad fundamental:
la negación (*"¿quién NO lleva casco?"*).

**1. Definición de Frameworks y Licencias**

- **YOLOv8 (Ultralytics):** Detector cerrado afinado (*fine-tuning*)
  sobre las clases específicas de EPP. Ofrece una velocidad de
  procesamiento en tiempo real sumamente alta (\>30 FPS). Opera bajo la
  licencia **AGPL-3.0**. Esta licencia es altamente restrictiva para
  entornos comerciales, ya que obliga a liberar el código fuente de
  cualquier software o servicio web que interactúe con él, lo que debe
  ser considerado en el análisis de costos del proyecto.

- **YOLO-World:** Detector de vocabulario abierto que permite definir
  clases mediante texto en tiempo de inferencia sin necesidad de
  reentrenamiento. Corre en tiempo real bajo la licencia **GPL-3.0**, la
  cual también exige código abierto bajo ciertas condiciones de
  distribución, pero con un alcance menor que AGPL-3.0.

- **Florence-2 (Microsoft):** VLM de *grounding* que procesa texto para
  devolver coordenadas visuales (bounding boxes). Corre bajo la licencia
  **MIT**, la cual es sumamente permisiva y óptima para el uso comercial
  cerrado e integración empresarial sin costo de licenciamiento.

- **Moondream2:** VLM generativo que opera bajo un esquema de preguntas
  y respuestas en lenguaje natural (VQA). Su licencia es **Apache-2.0**,
  permitiendo su modificación y uso comercial privado con gran libertad
  legal.

**2. El Impacto Crítico del "Fenómeno de la Negación" en Seguridad**

El mayor desafío técnico de este proyecto radica en que la pregunta
crítica de seguridad activa es una negación: *“¿Hay alguien sin
casco?”*. Los resultados demuestran que:

- **YOLOv8** resuelve esta negación de forma nativa debido a que fue
  entrenado explícitamente con la clase de infracción NO-Hardhat. Al
  acotar el dominio, el modelo no necesita razonar, sino clasificar
  patrones visuales directos.

- **YOLO-World** y los **VLM (Florence-2 y Moondream2)**, a pesar de su
  flexibilidad teórica, tropiezan severamente con los atributos
  negativos y las negaciones sintácticas. Al evaluar en modo zero-shot,
  tienden a ignorar el modificador "NO" o "sin", detectando erróneamente
  cabezas desprotegidas como si estuvieran equipadas o viceversa. Esto
  destruye la fiabilidad del monitoreo automatizado en una obra real,
  donde un solo Falso Negativo de seguridad puede traducirse en
  accidentes graves o multas severas.

**Cuadro Comparativo Final para la Decisión de Despliegue**

La siguiente matriz unifica las métricas obtenidas experimentalmente en
la GPU T4 de Google Colab y los criterios de propiedad intelectual para
fundamentar la recomendación técnica:

<table>
<colgroup>
<col style="width: 19%" />
<col style="width: 27%" />
<col style="width: 29%" />
<col style="width: 24%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Criterio de Evaluación</strong></th>
<th><strong>Paradigma 1: Cerrado (YOLOv8)</strong></th>
<th><strong>Paradigma 2: Vocabulario Abierto (YOLO-World)</strong></th>
<th><strong>Paradigma 3: VLM de Grounding / VQA (Florence-2 /
Moondream2)</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Precisión Nivel-Evento (</strong><span
class="math inline"><em>F</em><sub>1</sub></span><strong>-Score)</strong></td>
<td><p><strong>0.9500 (Umbral Objetivo)</strong></p>
<p><em>(Excelente tras fine-tuning con clase explícita de
infracción</em> NO-Hardhat<em>)</em></p></td>
<td><p><strong>Bajo</strong></p>
<p><em>(Débil capacidad de discriminación en atributos negativos como
"sin casco" sin reentrenamiento)</em></p></td>
<td><p><strong>Bajo - Moderado (0.4673 - 0.5263)</strong></p>
<p><em>(Sufre de fallos lógicos graves por solapamiento semántico o
altas tasas de falsas alarmas)</em></p></td>
</tr>
<tr class="even">
<td><strong>Velocidad y Latencia (en T4)</strong></td>
<td><p><strong>Muy Alta (&gt; 30 FPS)</strong></p>
<p><em>(Permite el procesamiento en tiempo real para seguridad activa en
obra)</em></p></td>
<td><p><strong>Alta (Tiempo Real)</strong></p>
<p><em>(Velocidad apta para feeds de video fluido de hasta 30
FPS)</em></p></td>
<td><p><strong>Inviable (&lt; 1-2 FPS)</strong></p>
<p><em>(Florence-2: 1.81 FPS; Moondream2: 0.85 FPS. Requiere muestreo
forzado a ~1 FPS)</em></p></td>
</tr>
<tr class="odd">
<td><strong>Flexibilidad (Clases Nuevas)</strong></td>
<td><p><strong>Nula</strong></p>
<p><em>(Imposible detectar nuevas clases como "guantes" o "gafas" sin un
nuevo reentrenamiento)</em></p></td>
<td><p><strong>Alta (Zero-Shot)</strong></p>
<p><em>(Permite agregar o cambiar etiquetas de texto en tiempo de
inferencia instantáneamente)</em></p></td>
<td><p><strong>Muy Alta (Zero-Shot)</strong></p>
<p><em>(Capacidad de resolver consultas complejas por lenguaje natural o
generar coordenadas de texto)</em></p></td>
</tr>
<tr class="even">
<td><strong>Manejo de la Negación</strong></td>
<td><p><strong>Excelente</strong></p>
<p><em>(Resuelve de forma nativa la detección de</em> NO-Hardhat <em>al
entrenarse explícitamente)</em></p></td>
<td><p><strong>Muy Débil</strong></p>
<p><em>(Falla al identificar la ausencia de un objeto o la negación
sintáctica)</em></p></td>
<td><p><strong>Inconsistente / Falla</strong></p>
<p><em>(Incapacidad de separar "cabeza" de "cabeza descubierta",
rompiendo la lógica del monitoreo)</em></p></td>
</tr>
<tr class="odd">
<td><strong>Tipo de Licencia de Software</strong></td>
<td><p><strong>AGPL-3.0</strong></p>
<p><em>(Altamente restrictiva para uso comercial cerrado o
SaaS)</em></p></td>
<td><p><strong>GPL-3.0</strong></p>
<p><em>(Restrictiva con obligaciones de distribución abierta de
modificaciones)</em></p></td>
<td><p><strong>MIT / Apache-2.0</strong></p>
<p><em>(Sumamente permisivas, óptimas para uso comercial privado o
integración corporativa)</em></p></td>
</tr>
</tbody>
</table>

**Recomendación y Justificación de Despliegue (Respuesta a la Pregunta
Final)**

**Decisión Técnica:** Se recomienda el despliegue de **YOLOv8** en el
entorno de producción de la obra en construcción.

**Argumentación Científica:**

1.  **Garantía de Seguridad Activa (Tiempo Real):** En la prevención de
    accidentes laborales, cada milisegundo cuenta. YOLOv8 opera
    cómodamente por sobre los 30 FPS en una GPU T4, lo que permite
    alertar en fracciones de segundo si un operador ingresa a una zona
    peligrosa sin casco. Por el contrario, la latencia extrema de
    Moondream2 (1.18 segundos por frame / 0.85 FPS) inhabilita cualquier
    capacidad de detención temprana de un accidente en desarrollo.

2.  **Robustez ante el Fenómeno de la Negación:** El monitoreo de
    seguridad por video no requiere un modelo que simplemente "entienda"
    lo que ve, sino que detecte con precisión quirúrgica la infracción
    de seguridad (*"¿quién no lleva casco?"*). Los modelos de
    vocabulario abierto y VLMs fallaron en esta tarea lógica fundamental
    durante la experimentación de grounding (clasificando "cabeza" y
    "cabeza descubierta" de forma solapada), arrojando falsos positivos
    prohibitivos (F1-Score \< 0.53) que provocarían fatiga de alarmas.
    YOLOv8 soluciona este comportamiento al mapear directamente patrones
    de NO-Hardhat.

3.  **Viabilidad de Mitigación:** La falta de flexibilidad de YOLOv8
    (imposibilidad de detectar nuevos objetos como guantes sin
    reentrenamiento) se compensa con un robusto plan de ingeniería de
    datos (fine-tuning con el dataset de prueba propuesto de 25%
    intemperie, 45% interiores y 30% espacios críticos). El coste
    operativo de reentrenar YOLOv8 es infinitamente inferior al coste
    humano de desplegar un VLM incapaz de alertar a tiempo y con
    precisión.

4.  **Viabilidad Comercial:** Aunque la licencia AGPL-3.0 de YOLOv8
    obliga a una cuidadosa arquitectura de software (por ejemplo,
    mediante microservicios aislados para no contaminar el código
    propietario de la constructora), su superioridad técnica en
    precisión y velocidad la convierte en la única alternativa
    éticamente aceptable para resguardar la vida de los trabajadores en
    la faena chilena. **Restricciones, Limitaciones y Advertencias del
    Sistema**

Para un despliegue transparente y responsable en entornos industriales,
se deben documentar formalmente las siguientes limitantes técnicas
identificadas durante la investigación:

- **Restricción de Cómputo y Muestreo Necesario:** Debido a las
  limitaciones de hardware al operar sobre una GPU NVIDIA T4 en Google
  Colab, se determinó que la ejecución de VLMs generativos en tiempo
  real (30 FPS) es computacionalmente inviable. Para procesar estos
  modelos de manera sostenible, el sistema requiere un muestreador de
  frames forzado a ~1 FPS o basado en keyframes. Esto implica una
  latencia de detección tolerable para auditorías de seguridad, pero
  inaceptable para sistemas de frenado de emergencia autónomos.

- **Reproducibilidad Imperfecta en Entornos de Cómputo:** Los
  experimentos de inferencia y *fine-tuning* están sujetos al
  no-determinismo propio de las librerías de aceleración CUDA en
  arquitecturas de GPU. A pesar de fijar semillas artificiales (*random
  seeds*) en el código, pequeños desfases en las operaciones de punto
  flotante de los tensores pueden generar variaciones mínimas en las
  métricas de precisión final al replicar el notebook.

- **Sensibilidad Crítica al Prompt:** Los modelos de vocabulario abierto
  y VLMs exhiben una preocupante vulnerabilidad semántica. Pequeños
  cambios de sintaxis en las consultas de texto modifican drásticamente
  el rendimiento visual del detector, introduciendo un riesgo latente si
  el personal de operaciones altera las variables de texto en producción
  sin supervisión de ingeniería.

- **Riesgo Legal y de Propiedad Intelectual:** La arquitectura
  seleccionada (YOLOv8) opera bajo la licencia restrictiva **AGPL-3.0**.
  Esto prohíbe explícitamente integrar el modelo directamente en
  sistemas de software propietario comerciales o plataformas en la nube
  (SaaS) cerradas sin liberar la totalidad del código fuente del
  servicio. La empresa debe mitigar esto aislando el detector en un
  microservicio independiente conectado únicamente por APIs cerradas.
