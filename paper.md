# Anticipar el drop: paper de uso y guía de la idea

## Resumen

Este documento acompaña a **Drop Anticipator**, un prototipo de la idea "sincronización lumínico-musical inmersiva". La idea parte de una observación: las luces automáticas de una fiesta **reaccionan** a la música, y por eso siempre llegan un instante tarde. La herramienta propone luces que **anticipan**: escuchan un tema, siguen el pulso, reconocen el **build-up** y predicen en qué pulso cae el **drop**, para que la explosión de luz caiga justo en el golpe. Muestra lado a lado unas luces que anticipan y otras que solo reaccionan, e incluye un **anotador de drops** para construir un conjunto de datos y medir la predicción contra temas reales.

La primera parte es un paper de uso: cómo cargar audio, qué muestran los paneles, un ejemplo resuelto con el tema sintético y sus límites. La segunda es una guía de la idea: la psicología de la expectativa musical según Leonard Meyer y David Huron, la neurociencia del placer anticipatorio de Salimpoor y Zatorre, la estructura de la música electrónica de baile según Mark Butler y Rune Solberg, el procesamiento de señales que hace posible el detector (la FFT de Cooley y Tukey, las ventanas de Harris, el flujo espectral y la detección de ataques de Bello, el seguimiento del pulso de Scheirer, Dixon y Ellis), la percepción de la sincronía audiovisual y la historia de los instrumentos de luz, desde el clave ocular de Castel hasta el protocolo DMX512. La tesis es que, en un género con una estructura tan regular como la música electrónica de baile, anticipar es posible, y que la diferencia entre anticipar y reaccionar es perceptible.

## Parte I. Paper de uso

### 1. Qué es la herramienta

Drop Anticipator es una página estática sin dependencias que se abre desde la [demo en vivo](https://loiaconobruno.github.io/drop-anticipator/) o con doble clic en `index.html`. Todo el análisis ocurre en el navegador: el audio no sale de la computadora.

La página tiene cuatro zonas:

1. **Audio:** generar un tema de prueba o abrir un audio propio; el deslizador "Desde" para elegir dónde empezar a reproducir; Reproducir; Marcar drop (tecla D); y los botones para exportar y borrar anotaciones.
2. **Análisis:** el tempo, cuántos build-ups, drops predichos y falsos drops encontró el detector, y una tabla que compara predicciones con anotaciones.
3. **Anticipa / Reacciona:** dos escenarios de luces lado a lado. El de la izquierda usa la predicción; el de la derecha reacciona a los graves.
4. **Lo que oye el detector:** un gráfico con la energía de graves, medios y agudos en el tiempo y las marcas de pulso, build-up y drop.

La lógica está en `src/audio.js`, sin acceso al DOM. Los tests se corren con `npm test` (Node 18 o más nuevo).

### 2. Instalación

```bash
git clone https://github.com/LoiaconoBruno/drop-anticipator.git
cd drop-anticipator
npm test
```

Después se abre `index.html` con doble clic.

### 3. Cargar audio

- **Generar tema de prueba:** un tema sintético de un minuto a 128 BPM (pulsos por minuto), con intro, build-up y drop a los 0:45. Sirve para ver el funcionamiento sin depender de un archivo.
- **Abrir audio propio:** acepta cualquier formato que el navegador sepa decodificar (mp3, wav, ogg, m4a). El archivo se decodifica y analiza en la computadora del usuario.

### 4. Los resultados del análisis

| Dato | Qué significa |
|---|---|
| Tempo | Pulsos por minuto estimados |
| Build-ups | Cuántas veces el detector reconoció el patrón de build-up |
| Drops predichos | Cuántas predicciones hizo |
| Falsos | Cuántas predicciones descartó porque los graves no volvieron |

La tabla compara cada drop real (en el tema sintético) o anotado (por el usuario) con la predicción más cercana: momento predicho, error y **anticipación** (cuántos segundos antes del drop se hizo la predicción).

### 5. Las luces

- **Anticipa:** en el pulso en que el detector reconoce el build-up, entra en un modo de tensión ("el drop cae en 3,8 s…") con un estrobo que se acelera, y explota en el pulso predicho.
- **Reacciona:** sigue la energía de los graves, como un sistema automático común. Explota cuando vuelven los bajos, es decir, después del drop.

### 6. El anotador

Mientras suena el tema, apretar **D** (o el botón "Marcar drop") registra un drop en ese instante. La anotación aparece en la tabla con el error de la predicción. "Exportar anotaciones (JSON)" guarda un archivo con el nombre del tema, el tempo, los drops anotados y las predicciones. Es la base del conjunto de 20 temas anotados que la idea pedía.

### 7. Ejemplo resuelto

1. **Generar tema de prueba.** El análisis da **128,0 BPM**, **1** build-up, **1** drop predicho y **0** falsos. La tabla compara con el drop real: predicción en **0:45.00**, error **+0,00 s** y **13,1 s** de anticipación. El detector supo dónde iba a caer el drop trece segundos antes.
2. **Desde.** El deslizador queda unos segundos antes del build-up. Llevarlo a **0:41** y apretar **▶ Reproducir**.
3. **Las luces.** Las de la izquierda entran en modo build-up ("el drop cae en 3,8 s…"), con un estrobo que se acelera, y explotan **a los 0:45.0**. Las de la derecha siguen "reaccionando a los graves" y recién explotan alrededor de **0:45.5**, cuando vuelven los bajos: medio segundo tarde, que es casi un pulso entero a 128 BPM (un pulso dura 60/128 ≈ 0,47 s).
4. **Anotar.** Apretar **D** al sentir el drop. La anotación aparece con su error respecto de la predicción. Es una buena manera de medir la propia latencia: la mayoría de las personas marca unos cientos de milisegundos tarde si reacciona, y casi en el golpe si anticipa.

### 8. Cómo leer los resultados

- **Error** es la diferencia entre la predicción y el drop real o anotado. El detector cuenta un acierto si el error es menor a 0,5 s y la predicción se hizo antes del drop.
- **Anticipación** es cuánto antes del drop se hizo la predicción. Es lo que hace útil al sistema: con 13 s de anticipación, un sistema de luces tiene tiempo de sobra para preparar la explosión.
- **Falsos** son predicciones que el detector descartó porque, dos compases después de la fecha predicha, los graves no habían vuelto. No son errores visibles para el público: el sistema las corrige antes de explotar.

#### 8.1 Preguntas frecuentes

**¿Por qué necesita 20 segundos al principio?** Porque usa los primeros 20 s para estimar el tempo con autocorrelación. Un build-up que empiece antes no se detecta.

**¿Funciona con cualquier música?** No. Busca el patrón típico de la música electrónica de baile: los graves desaparecen durante el build-up y vuelven en el drop. Un build-up que mantiene el bajo, o géneros sin drops (rock, jazz), no entran.

**¿Si el análisis se hace sobre todo el archivo, no está "haciendo trampa"?** El análisis se calcula de antemano sobre el archivo por comodidad, pero el detector es **causal**: en cada pulso usa solo lo que ya sonó hasta ese pulso. La predicción de las 0:45 se hace con la información disponible a las 0:31,9.

**¿Las luces de la página son reales?** Son una simulación en pantalla. El prototipo no manda señales a equipos de iluminación (DMX o Art-Net); está entre los próximos pasos.

#### 8.2 Recorrido guiado

1. **Generá el tema de prueba** y mirá el gráfico "Lo que oye el detector". *¿Dónde desaparecen los graves?* Alrededor de 0:30. *¿Qué pasa con los agudos en ese tramo?* Siguen: es la firma del build-up.
2. **Contá pulsos.** A 128 BPM, un compás dura 1,875 s y una frase de 8 compases, 15 s. Las frases empiezan en 0:00, 0:15, 0:30 y 0:45. *¿En qué comienzo de frase esperarías el drop si el build-up empezó en 0:30?* El detector exige al menos 4 compases (7,5 s) de build-up, así que 0:30 queda descartado por estar demasiado cerca y elige 0:45.
3. **Reproducí desde 0:41** mirando solo las luces de la derecha. *¿Se nota el retraso?* Después, mirando solo las de la izquierda.
4. **Anotá reaccionando.** Volvé a 0:41, cerrá los ojos y apretá D apenas sientas el drop. Mirá el error en la tabla.
5. **Anotá anticipando.** Volvé a 0:41 y contá los pulsos en voz baja ("5, 6, 7, 8") hasta el comienzo de frase. Apretá D en el "uno". Compará el error con el anterior.
6. **Exportá** las anotaciones y abrí el JSON en un editor de texto para ver qué se guarda.

**¿Qué pasa si abro un tema que no es de música electrónica?** Lo más probable es que el detector no encuentre ningún build-up (los graves no desaparecen de esa manera) y no haga predicciones. Si encuentra uno, probablemente sea falso: un tramo sin bajo en una canción no anuncia un drop.

**¿Puedo confiar en el tempo que muestra?** En temas de tempo constante, sí, con mucha precisión. Si el tema tiene cambios de tempo, rubatos o una intro sin percusión de más de 20 s, el tempo puede salir mal o al doble o la mitad.

### 9. Limitaciones

- Los resultados de los tests son sobre audio sintético, armado con las señales que el detector busca. Con temas reales va a fallar más: esa es la medición que falta, con el conjunto de datos anotado.
- Supone tempo constante y frases de 8 compases que arrancan con el tema.
- Busca el patrón "los graves desaparecen".
- El análisis se hace de antemano sobre el archivo, aunque en cada instante use solo el pasado. No escucha el micrófono en vivo.
- La latencia real de un sistema en vivo (audio, análisis, luces) no está medida.

## Parte II. Guía de la idea

### 10. Reaccionar llega tarde

Una luz que reacciona al sonido tiene que esperar a que el sonido llegue, analizarlo y después prenderse. Cada paso suma demora: el buffer de audio (decenas de milisegundos), el análisis, la transmisión a las luces y el tiempo de encendido de la lámpara. Pero el problema más grande no es técnico sino conceptual: el **drop** de un tema de música electrónica es, por definición, el momento en que vuelven los graves después de un build-up en el que desaparecieron. Un sistema que espera a que vuelvan los graves para explotar llega, como mínimo, un análisis tarde. En el ejemplo de la herramienta, medio segundo.

Medio segundo parece poco, pero a 128 BPM es casi un pulso entero. Y la percepción humana es muy sensible a la sincronía entre lo que se ve y lo que se oye, sobre todo en un evento esperado. La idea propone dar vuelta el problema: si el drop es predecible, las luces pueden saber de antemano cuándo va a llegar y explotar justo en el golpe.

### 11. La música como máquina de expectativas

#### 11.1 Meyer, 1956

Leonard Meyer, musicólogo estadounidense, publicó en 1956 *Emotion and Meaning in Music*. Su tesis central es que la emoción musical surge de las **expectativas**: la música crea tendencias (una melodía que sube "pide" seguir subiendo, una cadencia "pide" resolver), y la emoción aparece cuando esas tendencias se cumplen, se demoran o se frustran. Meyer se apoyó en la psicología de la Gestalt y en la teoría de la información incipiente. Su idea, que en su momento fue especulativa, se convirtió en un programa de investigación.

#### 11.2 Huron y la teoría ITPRA

David Huron, en *Sweet Anticipation: Music and the Psychology of Expectation* (2006), retomó a Meyer con los métodos de la psicología cognitiva y la estadística. Propuso que la respuesta a un evento esperado tiene cinco fases, que resumió con la sigla **ITPRA**: imaginación (antes del evento, imaginamos qué va a pasar y cómo nos sentiríamos), tensión (justo antes, el cuerpo se prepara), predicción (al ocurrir, una recompensa o castigo según si acertamos), reacción (una respuesta rápida y automática) y evaluación (una valoración más lenta y consciente).

El build-up y el drop de la música electrónica son casi un ejemplo de manual de ITPRA. El build-up construye **tensión**: la música anuncia que algo va a pasar y deja que el oyente prediga cuándo. El drop es la **predicción** cumplida, y Huron muestra que acertar una predicción produce placer por sí mismo, independientemente de qué tan "lindo" sea el sonido. Una luz que explota en el drop refuerza esa recompensa; una que llega tarde, la diluye.

#### 11.3 La dopamina de la anticipación

En 2011, Valorie Salimpoor, Robert Zatorre y colaboradores midieron con tomografía por emisión de positrones la liberación de dopamina mientras personas escuchaban música que les producía escalofríos (*Nature Neuroscience*, 14, 257–262). Encontraron dos picos en regiones distintas: uno en el núcleo caudado **durante la anticipación**, en los segundos previos al momento de máxima emoción, y otro en el núcleo accumbens **durante** ese momento. El placer musical tiene una fase anticipatoria con sustrato neuroquímico propio.

Esto es importante para la idea: el build-up no es solo un preámbulo del drop; es una fase de placer en sí misma, y las luces pueden acompañarla. Por eso las luces que anticipan de la herramienta no solo explotan a tiempo: entran en un modo de tensión durante el build-up, con un estrobo que se acelera.

#### 11.4 El cerebro predictivo

Desde los años 2000, una corriente de la neurociencia, el **procesamiento predictivo**, sostiene que el cerebro es fundamentalmente una máquina de predecir: genera continuamente expectativas sobre lo que va a percibir y procesa sobre todo los **errores de predicción**. Karl Friston la formuló en términos de minimización de energía libre (*Nature Reviews Neuroscience*, 11, 127–138, 2010), y Andy Clark la presentó para un público amplio en *Whatever next? Predictive brains, situated agents, and the future of cognitive science* (*Behavioral and Brain Sciences*, 36, 181–204, 2013). Peter Vuust y Maria Witek aplicaron el marco al ritmo (*Frontiers in Psychology*, 5, 1111, 2014): el **groove**, las ganas de moverse con la música, aparece cuando el ritmo tiene la cantidad justa de sorpresa frente a un pulso predecible.

Desde este punto de vista, un sistema de luces que anticipa hace lo mismo que el cerebro del público: predice. Uno que reacciona hace algo que el cerebro no hace: espera.

#### 11.5 El escalofrío

Hay una respuesta física que muchos oyentes reconocen en el drop: el **escalofrío** o la piel de gallina, que la literatura llama *chills* o *frisson*. Jaak Panksepp lo estudió en 1995 (*Music Perception*, 13, 171–207) y encontró que aparece sobre todo en momentos de cambio dramático: una entrada de volumen, una voz que se eleva, una resolución largamente esperada. Anne Blood y Robert Zatorre midieron en 2001 la actividad cerebral durante esos escalofríos (*Proceedings of the National Academy of Sciences*, 98, 11818–11823) y encontraron activación en los circuitos de recompensa, los mismos que responden a la comida o al sexo.

Solberg (sección 12.2) mostró que en la música electrónica los escalofríos se concentran justamente en el drop. Una luz que llega medio segundo después del escalofrío no lo acompaña: lo comenta. Una que llega en el mismo instante puede reforzarlo, porque el cerebro integra lo que ve y lo que oye en un solo evento cuando ocurren dentro de la ventana de sincronía.

### 12. La forma de la música electrónica de baile

#### 12.1 Frases de ocho compases

La música electrónica de baile (EDM, por su sigla en inglés, aunque el término abarca house, techno, trance, dubstep y muchos otros) tiene una estructura métrica muy regular. Casi siempre está en compás de cuatro tiempos, con tempo constante, y se organiza en **frases** de 8, 16 o 32 compases. Mark Butler estudió esta estructura en *Unlocking the Groove: Rhythm, Meter, and Musical Design in Electronic Dance Music* (2006), con análisis de temas y entrevistas a DJs. Butler mostró que la regularidad no es pobreza sino una condición para que el DJ pueda mezclar temas y para que el público pueda anticipar los cambios.

La herramienta se apoya en esa regularidad: supone frases de 8 compases (32 pulsos) que arrancan con el tema, y predice que el drop caerá en el próximo comienzo de frase después del build-up.

#### 12.2 Build-up y drop

Rune Solberg estudió en 2014 los recursos de producción del build-up y del drop y su relación con las experiencias emocionales intensas de los oyentes, en un artículo titulado *Waiting for the bass to drop* (*Dancecult*, 6(1), 61–82). Identificó los recursos típicos del build-up: **eliminación de los graves** (el bombo y el bajo desaparecen), **aceleración de la percusión** (los redobles pasan de negras a corcheas, semicorcheas y fusas), **barridos de filtro y alturas que suben** (*risers*) y, a veces, un silencio breve justo antes del drop. Encontró que los oyentes reportan las experiencias más intensas precisamente en el drop y en los segundos previos.

El detector de la herramienta usa la señal más robusta de esa lista: los graves que desaparecen mientras los agudos siguen. Los redobles acelerados y los barridos son visibles en el gráfico "Lo que oye el detector" pero no se usan para la decisión, para mantener el detector simple.

#### 12.3 Un drop, un evento social

Karthik Yadati, Martha Larson, Cynthia Liem y Alan Hanjalic presentaron en 2014, en la conferencia de la Sociedad Internacional de Recuperación de Información Musical (ISMIR), un trabajo titulado *Detecting drops in electronic dance music: Content based approaches to a socially significant music event*. Usaron comentarios con marca de tiempo en SoundCloud (donde los oyentes escriben "¡acá viene el drop!") como anotaciones, y entrenaron detectores de drops a partir del contenido del audio. Es el antecedente académico más directo de la herramienta, con una diferencia importante: su objetivo era **encontrar** drops en un tema completo, no **predecirlos** de manera causal antes de que ocurran.

#### 12.4 Una pista que se mueve junta

El drop no es una experiencia individual: en una pista de baile, cientos de personas lo esperan y lo reciben al mismo tiempo. Bronwyn Tarr, Jacques Launay y Robin Dunbar revisaron en 2014 la evidencia de que moverse en sincronía con otros, siguiendo música, produce sensación de vínculo social y libera endorfinas (*Frontiers in Psychology*, 5, 1096). En experimentos posteriores, personas que bailaron sincronizadas toleraron más dolor (un marcador indirecto de liberación de endorfinas) y reportaron sentirse más unidas que las que bailaron desincronizadas.

Las luces son una de las señales que coordinan a la pista. Si explotan en el drop, refuerzan el momento compartido: todos ven y oyen lo mismo a la vez. Si llegan tarde, introducen un pequeño desacuerdo entre el oído y la vista que, multiplicado por cientos de personas, diluye la sincronía colectiva. La idea de anticipar el drop es, en este sentido, una idea sobre la experiencia social de la música, no solo sobre la técnica de iluminación.

### 13. Cómo escucha una computadora

#### 13.1 La transformada de Fourier rápida

Para saber cuánta energía hay en los graves, los medios y los agudos, la herramienta descompone el audio en frecuencias con una **transformada de Fourier**. La versión discreta sobre N muestras tiene un costo de N² operaciones si se calcula directamente. En 1965, James Cooley y John Tukey publicaron un algoritmo que la calcula en N log N operaciones (*Mathematics of Computation*, 19, 297–301): la **FFT**. Con N = 2048, eso es unas 200 veces más rápido. Se supo después que Gauss había usado una idea equivalente en 1805, en un manuscrito no publicado en vida. La FFT es uno de los algoritmos más importantes del siglo XX y la base de todo el audio digital.

La herramienta implementa su propia FFT, sin bibliotecas, y uno de los tests verifica que un seno puro cae en el casillero de frecuencia correcto.

#### 13.2 Ventanas

El audio se analiza en **cuadros** de 1024 o 2048 muestras (unos 20 a 45 milisegundos a 44,1 kHz). Cortar la señal abruptamente en los bordes de cada cuadro introduce frecuencias falsas, un fenómeno llamado *leakage* espectral. Para evitarlo, cada cuadro se multiplica por una **ventana** que lo hace llegar suavemente a cero en los bordes. La herramienta usa la **ventana de Hann**, nombrada por Julius von Hann. Fredric Harris publicó en 1978 el estudio de referencia sobre ventanas (*Proceedings of the IEEE*, 66, 51–83), que compara sus propiedades y sigue siendo citado en todo texto de procesamiento de señales.

#### 13.3 Bandas

Con el espectro de cada cuadro, la herramienta suma la energía en tres bandas:

| Banda | Rango | Qué contiene en EDM |
|---|---|---|
| Graves | 20–150 Hz | Bombo y bajo |
| Medios | 150 Hz–2 kHz | Voces, sintetizadores, cuerpo de los redobles |
| Agudos | más de 2 kHz | Platillos, hi-hats, *risers*, brillo |

Un build-up típico muestra graves que caen casi a cero mientras los agudos se mantienen o suben.

#### 13.4 Flujo espectral y ataques

Además de la energía, la herramienta calcula el **flujo espectral**: cuánto aparece de nuevo en el espectro de un cuadro respecto del anterior (solo los aumentos). Es la medida estándar para detectar **ataques** (*onsets*): el comienzo de una nota o de un golpe de percusión. Juan Pablo Bello y colaboradores publicaron en 2005 un tutorial sobre detección de ataques que comparó los métodos principales (*IEEE Transactions on Speech and Audio Processing*, 13, 1035–1047), y Simon Dixon mostró en 2006 que el flujo espectral, con algunos ajustes, está entre los más robustos. La herramienta calcula el flujo total y el flujo solo en graves; este último marca los golpes del bombo.

### 14. Encontrar el pulso

#### 14.1 Autocorrelación

Si el bombo golpea a intervalos regulares, la serie del flujo en graves se parece a sí misma desplazada un pulso, dos pulsos, cuatro pulsos. La **autocorrelación** mide cuánto se parece una señal a sí misma con cada desplazamiento posible; el desplazamiento con mayor autocorrelación es el candidato a período del pulso. Eric Scheirer propuso en 1998 un método clásico de análisis de tempo con bancos de filtros y resonadores (*Journal of the Acoustical Society of America*, 103, 588–601), y Simon Dixon en 2001 un sistema de seguimiento del pulso en interpretaciones expresivas (*Journal of New Music Research*, 30, 39–58).

La autocorrelación tiene un problema conocido: no distingue bien entre un tempo y su doble o su mitad. Un tema a 128 BPM también "se parece a sí mismo" cada dos pulsos (64 BPM) y cada medio pulso (256 BPM). La herramienta resuelve la ambigüedad con una **preferencia suave por los tempos de baile**, alrededor de 120 a 130 BPM, como hacen muchos sistemas.

#### 14.2 Ajuste fino

Con el tempo aproximado, la herramienta detecta los golpes individuales y ajusta por **mínimos cuadrados** una recta que pasa lo más cerca posible de todos ellos: la pendiente da el período exacto y la ordenada al origen la fase (dónde cae el primer pulso). La posición de cada golpe se estima con precisión **sub-cuadro**, interpolando entre cuadros. Así, los tests encuentran el tempo con un error menor a 0,05 BPM en temas sintéticos de 100 a 174 BPM.

Daniel Ellis propuso en 2007 un método muy usado de seguimiento del pulso por **programación dinámica** (*Journal of New Music Research*, 36, 51–60), que busca la secuencia de pulsos que mejor combina regularidad y coincidencia con los ataques. La herramienta usa un método más simple, válido para tempo constante; Ellis permite tempos que varían. Meinard Müller recopiló todas estas técnicas en *Fundamentals of Music Processing* (2015), el manual de referencia del área.

### 15. El detector causal

#### 15.1 Qué significa causal

Un detector es **causal** si, para decidir algo en el instante t, usa solo información de antes de t. Es la condición para que funcione en vivo. Muchos sistemas de análisis musical no son causales: miran el tema completo y deciden después. Encontrar un drop mirando el tema completo es fácil; **predecirlo** antes de que ocurra, no.

El detector de la herramienta avanza pulso a pulso y, en cada pulso, mira solo lo que ya sonó.

#### 15.2 Las reglas

1. **Build-up:** si los graves caen por debajo del 30 % de su nivel habitual (la mediana de los graves hasta ese momento) durante 4 pulsos seguidos y los agudos no bajan, se declara un build-up.
2. **Predicción:** el drop caerá en el próximo comienzo de frase (cada 8 compases, 32 pulsos), al menos 4 compases (16 pulsos) después de que empezó el build-up.
3. **Reactivo:** en paralelo, se registra el pulso en que vuelven los graves: lo que notaría un sistema que solo reacciona.
4. **Falso drop:** si pasan 2 compases de la fecha predicha y los graves no volvieron, la predicción se descarta y se vuelve a buscar.

En el tema de prueba, el build-up empieza alrededor de 0:30, el detector lo reconoce a los 0:31,9 (después de 4 pulsos sin graves) y predice el drop para el comienzo de frase siguiente que esté al menos 16 pulsos más adelante: 0:45.

#### 15.3 Por qué funciona con tan poco

El detector es casi trivial: tres umbrales y una regla de frases. Funciona en el tema sintético porque el tema tiene exactamente las señales que busca. En temas reales, la pregunta es cuán seguido los productores respetan las convenciones. La literatura de Butler y Solberg sugiere que bastante: la estructura de frases y la eliminación de graves son convenciones muy fuertes del género, porque los DJs las necesitan para mezclar y el público para anticipar. Pero hay muchas excepciones: build-ups de 16 compases, drops a mitad de frase, falsos drops deliberados (el productor amaga un drop y lo demora), build-ups que mantienen el bajo. Eso es lo que el conjunto de datos anotado tiene que medir.

#### 15.4 Los falsos drops

El **falso drop** es un recurso de producción: el tema construye un build-up, llega al momento esperado y, en vez de soltar los graves, sigue con otro build-up o con un silencio. Juega con la expectativa del oyente, exactamente como describen Meyer y Huron: la demora aumenta la tensión. Para un sistema de luces, un falso drop es un riesgo: si explota en el momento predicho y el drop no llega, el efecto es peor que llegar tarde. La regla 4 del detector lo maneja descartando la predicción y volviendo a buscar; uno de los tests verifica que un build-up más largo que una frase se marca como falso y se vuelve a predecir. Pero la luz de la izquierda ya habría explotado en la fecha falsa: un sistema real necesitaría una estrategia más prudente (por ejemplo, explotar a medias en la primera fecha y del todo cuando los graves vuelvan).

#### 15.5 Qué cambiaría con aprendizaje automático

Un camino natural para mejorar el detector sería reemplazar las reglas por un modelo entrenado con temas anotados: una red neuronal que reciba el espectro de los últimos segundos y devuelva la probabilidad de que el drop caiga en cada uno de los próximos pulsos. Es lo que hicieron Yadati y colaboradores en 2014 con métodos de clasificación, y lo que hace hoy la mayoría de los sistemas de análisis musical: los detectores de pulso basados en redes recurrentes de Sebastian Böck y colaboradores, por ejemplo, superan a los métodos de reglas en las evaluaciones de MIREX.

Pero el aprendizaje automático necesita justamente lo que falta: datos anotados, en cantidad. Con 20 temas alcanza para medir un detector de reglas, no para entrenar uno aprendido. Por eso el orden que propone el proyecto es razonable: primero un detector simple y explicable, después el anotador, después la medición, y solo entonces, si las reglas fallan de maneras que un modelo podría aprender, un modelo. Un detector de reglas tiene además una ventaja para un show en vivo: cuando falla, se entiende por qué.

### 16. Cuánto tarde es tarde

#### 16.1 Sincronía audiovisual

¿A partir de qué desfase se nota que la luz no está sincronizada con el sonido? La Unión Internacional de Telecomunicaciones fijó en la recomendación ITU-R BT.1359 (1998) que, en televisión, el desfase entre audio y video se detecta cuando el sonido se adelanta más de unos 45 ms o se atrasa más de unos 125 ms respecto de la imagen. La asimetría tiene una explicación física: la luz llega antes que el sonido, y el cerebro está acostumbrado a que el sonido de algo lejano llegue un poco tarde.

Jean Vroomen y Mirjam Keetels revisaron la percepción de la sincronía entre sentidos en 2010 (*Attention, Perception, & Psychophysics*, 72, 871–884). La **ventana de integración** audiovisual, dentro de la cual dos estímulos se perciben como simultáneos, mide entre 100 y 200 ms según la tarea, y es más estrecha para eventos con un ataque nítido, como un golpe de bombo.

Una luz que reacciona medio segundo tarde está muy fuera de esa ventana. Una que reacciona en 50 ms estaría dentro, pero eso exige una cadena audio-análisis-luces muy rápida, y no resuelve el problema de fondo: en el drop, la señal que dispara la reacción (la vuelta de los graves) es el propio drop.

#### 16.2 Sincronizarse con un pulso

Cuando una persona aplaude o golpea con el pie siguiendo un pulso, no reacciona a cada golpe: lo anticipa. Bruno Repp revisó un siglo de investigación sobre **sincronización sensoriomotora** (*Psychonomic Bulletin & Review*, 12, 969–992, 2005; con Yi-Huang Su, 20, 403–452, 2013). Uno de los hallazgos más robustos es la **asincronía media negativa**: las personas tienden a golpear unos 20 a 80 ms **antes** del pulso, no después. Sincronizarse con la música es, por definición, anticipar.

La herramienta lleva esa idea a las luces: en vez de reaccionar al sonido, se sincronizan con el pulso y con la estructura, como lo haría un iluminador humano que conoce el tema.

### 17. El presupuesto de latencia de un sistema en vivo

La herramienta analiza un archivo; un sistema en vivo escucharía un micrófono o la salida de la consola del DJ. Vale la pena estimar cuánto tardaría cada eslabón, porque eso define cuánta anticipación hace falta como mínimo.

| Eslabón | Demora típica | Comentario |
|---|---|---|
| Captura de audio (buffer de 512 muestras a 48 kHz) | unos 11 ms | Configurable; buffers más chicos aumentan el riesgo de cortes |
| Cuadro de análisis (2048 muestras a 48 kHz) | unos 43 ms | Para detectar graves hace falta un cuadro largo: una onda de 50 Hz dura 20 ms |
| Cálculo (FFT y detector) | menos de 1 ms | Despreciable en una computadora actual |
| Transmisión DMX512 (universo completo) | unos 23 ms | 250 kbit/s, 513 ranuras de 11 bits más el *break*: unas 44 actualizaciones por segundo |
| Red Art-Net | 1 a 5 ms | En una red local |
| Respuesta de la luz | menos de 1 ms (LED) a 100 ms o más (lámparas incandescentes) | Los LED son casi instantáneos |

Sumando, un sistema reactivo bien hecho con LED tendría entre 80 y 100 ms de latencia de punta a punta. Eso está dentro del umbral de la ITU para un atraso del sonido respecto de la imagen, pero **fuera** de él en la dirección que importa acá: la luz llega después del sonido, que es como si el sonido se adelantara. Y eso sin contar el problema conceptual de que la señal que dispara la reacción es el propio drop.

Con anticipación, la cuenta cambia. Si el sistema sabe con 13 s de antelación cuándo cae el drop, puede mandar la orden a las luces descontando la latencia conocida de la cadena (unos 25 ms de DMX más la respuesta de la lámpara). El margen es enorme. Por eso la anticipación no solo mejora el efecto: hace que la latencia deje de ser un problema.

### 18. Diseñar las luces de la anticipación

Saber cuándo cae el drop abre preguntas de diseño que un sistema reactivo nunca se hace: ¿qué hacer durante los segundos de anticipación?

**La curva de tensión.** El build-up musical sube la tensión de manera progresiva: los redobles se aceleran, los filtros se abren, las alturas suben. Las luces pueden acompañar esa curva. La herramienta usa un estrobo que se acelera; otras opciones son subir el brillo de a poco, estrechar la paleta de colores o concentrar los haces en un punto. Huron llama a esta fase "tensión" y muestra que su función es preparar al cuerpo para el evento: la luz puede ayudar a esa preparación.

**El silencio antes del golpe.** Muchos productores ponen un silencio breve (un pulso o medio) justo antes del drop. El contraste hace más intenso el golpe. Las luces pueden imitarlo: un apagón total de una fracción de segundo antes de la explosión. Solo es posible si se sabe cuándo viene el drop.

**La explosión.** En el drop, la energía musical llega a su máximo. El efecto lumínico debería coincidir con el primer golpe de bombo, con una precisión de pocas decenas de milisegundos para quedar dentro de la ventana de integración audiovisual. Después del golpe, las luces pueden volver a seguir el ritmo de manera reactiva, porque una vez que el bombo está sonando, reaccionar a cada golpe con 50 ms de demora no se nota tanto como fallar el primero.

**La prudencia ante el falso drop.** Si la predicción falla, una explosión en el vacío es peor que no hacer nada. Una estrategia posible es graduar la respuesta según la confianza: explotar a medias en la fecha predicha y completar cuando los graves vuelvan, o reservar la explosión total para drops que coinciden con varias señales a la vez (fin de frase, silencio previo, graves que vuelven).

Estas decisiones no son técnicas sino estéticas, y son las que en los shows profesionales toman los iluminadores. La herramienta no reemplaza ese criterio: lo hace posible en situaciones en las que nadie programó el tema de antemano.

### 19. Luz y música: una historia corta

#### 19.1 El clave ocular

La idea de un instrumento que toque luz junto con la música es antigua. En 1725, el jesuita francés Louis-Bertrand Castel propuso el **clave ocular**: un clavecín en el que cada tecla, además de una nota, descubriera un color. Castel asoció los siete colores de Newton a las siete notas de la escala. No está claro que haya llegado a construir una versión funcional, pero la idea circuló por toda Europa e inspiró a Telemann, que escribió sobre ella.

#### 19.2 Scriabin y Wilfred

En 1910, Alexander Scriabin compuso *Prometeo: el poema del fuego*, con una parte para *clavier à lumières*, un teclado de luz que debía bañar la sala de colores. Scriabin, que probablemente tenía alguna forma de sinestesia, asoció tonalidades a colores. La primera interpretación con luces se hizo en Nueva York en 1915, con un aparato rudimentario. En los años veinte, Thomas Wilfred construyó el **Clavilux** y llamó a su arte *Lumia*: composiciones de luz en movimiento, a veces sin música. Varias obras de Wilfred están en la colección del Museo de Arte Moderno de Nueva York.

#### 19.3 Del órgano de luces al DMX

En los años sesenta y setenta, los **órganos de luces** de las discotecas dividían el sonido en tres bandas (graves, medios, agudos) con filtros analógicos y prendían una lámpara por banda: el antecesor directo del modo "reacciona" de la herramienta. En 1986, el Instituto de Tecnología del Espectáculo de Estados Unidos (USITT) publicó el protocolo **DMX512**, que permite controlar hasta 512 canales de luces por un solo cable y que, con revisiones, sigue siendo el estándar de la iluminación escénica. **Art-Net**, desarrollado por la empresa Artistic Licence en los años noventa, transporta DMX por redes Ethernet. Mandar las decisiones de la herramienta a equipos reales por Art-Net está entre los próximos pasos.

#### 19.4 Lo que hacen los profesionales

En los grandes shows, la iluminación se **programa** de antemano, a mano, sobre una línea de tiempo sincronizada con el tema (por código de tiempo SMPTE o por la grilla de pulsos). Así se logra la precisión que la idea busca, pero con horas de trabajo por tema. Programas para DJs como SoundSwitch o el modo de iluminación de rekordbox analizan la estructura de los temas de antemano y generan escenas automáticamente. La idea propone algo intermedio: una predicción automática y causal que no requiera programación previa. Comparar con esos programas está en los próximos pasos del proyecto.

### 20. Medir: el conjunto de datos de 20 temas

#### 20.1 Por qué hace falta

Los tests de la herramienta son sobre audio sintético, armado con las señales que el detector busca. Eso prueba que el código hace lo que se diseñó, no que el diseño funciona con música real. Para saberlo hace falta un **conjunto de datos anotado**: temas reales con los drops marcados a mano por personas, contra los que se compara la predicción.

#### 20.2 Cómo anotar

El anotador de la herramienta está pensado para eso. Algunas recomendaciones de la literatura de recuperación de información musical:

- **Más de un anotador.** Distintas personas no coinciden exactamente en dónde está el drop, y el desacuerdo entre anotadores pone un límite a la precisión que tiene sentido exigir. Se mide con medidas de acuerdo, como el kappa de Jacob Cohen (1960), o directamente con la diferencia en segundos entre anotaciones.
- **Escuchar sin mirar el detector.** Para no sesgar las anotaciones.
- **Corregir la latencia propia.** Apretar una tecla tiene una demora; si es sistemática, se puede medir con el tema sintético (donde se conoce el drop real) y descontar.
- **Elegir temas variados.** Distintos subgéneros, tempos y productores, incluidos temas con falsos drops y con build-ups atípicos.

#### 20.3 Qué medir

Con las anotaciones, se pueden calcular las medidas estándar de detección de eventos:

- **Precisión:** de las predicciones, cuántas acertaron (error menor a 0,5 s).
- **Exhaustividad (recall):** de los drops anotados, cuántos fueron predichos.
- **Medida F:** la media armónica de las dos.
- **Anticipación media:** cuántos segundos antes se predijo, en los aciertos.
- **Falsos drops:** cuántas predicciones se descartaron, y cuántas explotaron en falso antes de descartarse.

La campaña anual MIREX (Music Information Retrieval Evaluation eXchange), organizada por J. Stephen Downie desde 2005, usa medidas de este tipo para comparar algoritmos de detección de ataques, seguimiento del pulso y segmentación. La tolerancia de 0,5 s de la herramienta es generosa comparada con las de MIREX para ataques (unos 50 ms), pero razonable para un evento estructural como el drop.

### 21. Qué color tiene un sonido

Sincronizar luces con música no es solo cuestión de tiempo: también hay que decidir **qué** luz corresponde a **qué** sonido. La investigación sobre las correspondencias entre los sentidos da pistas para esa decisión.

#### 21.1 Sinestesia

Algunas personas experimentan, de manera involuntaria y estable, sensaciones en un sentido provocadas por estímulos de otro: ven colores al escuchar notas, o asocian cada letra a un color. Es la **sinestesia**. Durante mucho tiempo se la consideró una curiosidad o una metáfora; Richard Cytowic la volvió objeto de estudio clínico en los años ochenta (*Synesthesia: A Union of the Senses*, 1989), y Vilayanur Ramachandran y Edward Hubbard mostraron en 2001 (*Journal of Consciousness Studies*, 8(12), 3–34), con pruebas perceptivas, que es un fenómeno sensorial real y no una asociación aprendida. Se atribuye a Scriabin (sección 19.2) una sinestesia de sonido y color, aunque los historiadores discuten si era sensorial o conceptual.

#### 21.2 Correspondencias que todos compartimos

Lo más útil para el diseño de luces no es la sinestesia, que es rara e individual, sino las **correspondencias intermodales** que casi todas las personas comparten sin ser sinestésicas. Charles Spence las revisó en 2011 (*Attention, Perception, & Psychophysics*, 73(4), 971–995). Entre las más robustas:

- **Altura y brillo:** los sonidos agudos se asocian con colores claros y luces brillantes; los graves, con oscuros.
- **Altura y posición:** los agudos se asocian con "arriba" y los graves con "abajo", en muchas culturas.
- **Intensidad y tamaño:** los sonidos fuertes se asocian con objetos grandes.
- **Timbre y forma:** sonidos ásperos con formas angulosas, suaves con formas redondeadas (el célebre efecto "bouba/kiki").

Jamie Ward, Brett Huckstep y Elias Tsakanikos mostraron en 2006 (*Cortex*, 42(2), 264–280) que los sinestésicos de sonido y color siguen, en promedio, las mismas tendencias que los no sinestésicos (agudo con claro, por ejemplo), solo que de manera más intensa y consistente. Las correspondencias parecen ser una base compartida.

#### 21.3 Aplicación a las luces

Estas correspondencias sugieren reglas de diseño para las luces de la anticipación que van más allá del momento de la explosión:

- **Durante el build-up**, cuando los graves desaparecen y los agudos suben (los *risers*), las luces pueden subir de brillo y moverse hacia arriba: la correspondencia altura-brillo-posición acompaña la tensión.
- **En el drop**, con la vuelta de los graves, las luces pueden bajar y expandirse, ganar tamaño e intensidad: el sonido grave y fuerte se corresponde con algo grande y profundo.
- **El color** puede seguir el balance entre bandas que la herramienta ya calcula: más energía en agudos, colores más claros; más en graves, más saturados u oscuros.

Son hipótesis de diseño, no reglas validadas para shows. Pero tienen la ventaja de apoyarse en asociaciones que el público comparte, y por eso es más probable que una luz diseñada así se sienta "correcta" para la mayoría.

### 22. El pulso en vivo: del archivo al micrófono

El próximo paso más ambicioso del proyecto es un modo en vivo con el micrófono. Eso cambia el problema del seguimiento del pulso de una manera importante.

#### 22.1 Lo que cambia

Con un archivo, la herramienta estima el tempo con los primeros 20 segundos y después lo da por fijo. En vivo, el DJ cambia de tema, ajusta el tempo para mezclar y a veces lo modifica gradualmente durante una transición. El seguidor tiene que **actualizar** su estimación continuamente y decidir, en cada instante, si un golpe que llegó un poco antes de lo esperado es un cambio de tempo o un error de detección.

#### 22.2 Modelos de estado

La manera estándar de hacerlo es con un **modelo de espacio de estados**: el sistema mantiene, en cada instante, una distribución de probabilidad sobre el tempo y la posición dentro del compás, y la actualiza con cada cuadro de audio según cuán bien explica lo que se oye. Florian Krebs, Sebastian Böck y Gerhard Widmer propusieron en 2015 un modelo eficiente de este tipo para seguir conjuntamente el tempo y el compás (*Proceedings of ISMIR*, 72–78). Böck, Krebs y Widmer combinaron en 2016 redes neuronales recurrentes, que estiman en cada cuadro la probabilidad de que haya un pulso o un tiempo fuerte, con ese tipo de modelo (*Proceedings of ISMIR*, 255–261). El grupo publicó además una biblioteca abierta, madmom (*Proceedings of the 24th ACM International Conference on Multimedia*, 1174–1178, 2016), que implementa estos métodos y funciona en tiempo real.

#### 22.3 El compás y la frase

Para anticipar el drop no alcanza con el pulso: hace falta saber dónde empieza cada compás (el **tiempo fuerte**) y cada frase. En un archivo, la herramienta supone que la primera frase empieza con el tema. En vivo, eso no se sabe: el sistema empieza a escuchar en cualquier momento. Los seguidores de tiempos fuertes como el de Böck y colaboradores estiman la posición dentro del compás; para las frases de ocho compases, harían falta indicios adicionales, como los cambios de textura que suelen marcar el comienzo de una frase (entra o sale un instrumento, cambia el patrón de batería).

Una alternativa práctica, que usan algunos sistemas comerciales, es aprovechar que los programas de DJ ya analizaron cada tema antes de reproducirlo y conocen su grilla de pulsos y frases. Si el programa comparte esa información con el sistema de luces (por red, con protocolos propios de cada fabricante), el problema del análisis en vivo desaparece y queda solo el de la latencia. Es menos elegante que escuchar el micrófono, pero mucho más confiable en un show.

#### 22.4 Medir la latencia de punta a punta

Para el modo en vivo, la tabla de latencias de la sección 17 tendría que convertirse en una medición. Una manera simple: reproducir un sonido de prueba con un golpe nítido, filmar a alta velocidad (muchos teléfonos filman a 240 cuadros por segundo, unos 4 milisegundos por cuadro) el parlante y la luz a la vez, y contar los cuadros entre el golpe visible en la membrana del parlante (o un indicador sincronizado) y el encendido de la luz. Es un experimento casero que da el número que todo el resto del diseño necesita.

### 23. Del modelo al código

| Concepto | Función en `src/audio.js` | Qué produce |
|---|---|---|
| Generador pseudoaleatorio | `rng` | Ruido reproducible en el tema sintético |
| Tema de prueba | `sintetizar` | Un minuto a 128 BPM con build-up y drop a los 0:45 |
| FFT con ventana de Hann | `fft` | Espectro por cuadro |
| Bandas y flujo | `analizar` | Graves, medios, agudos y flujo espectral por cuadro |
| Tempo y fase | `pulso` | BPM y grilla de pulsos |
| Detector causal | `detectar` | Build-ups, predicciones, reactivos y falsos drops |
| Comparación con anotaciones | `evaluar` | Error, anticipación, aciertos |
| Todo junto | `procesar` | El análisis completo de un audio |
| Mediana | `mediana` | Nivel habitual de los graves |

Los tests (`test/audio.test.js`) verifican: que un seno cae en su casillero de la FFT; que el pulso del tema de prueba sale con el tempo y la fase correctos; que la predicción se hace antes del drop y con error chico; que un sistema que solo reacciona llega tarde; que sin build-up no hay predicciones; que un build-up más largo que una frase se marca como falso drop y se vuelve a predecir, y que la evaluación compara contra drops anotados a mano.

### 24. Ejercicios

1. A 128 BPM, ¿cuánto dura un compás de cuatro pulsos? ¿Y una frase de 8 compases? ¿Cuántas frases caben en un tema de 6 minutos?
2. Si un sistema reactivo tarda 80 ms en total (buffer, análisis, transmisión), ¿está dentro de la ventana de sincronía de la ITU para un sonido que se atrasa respecto de la imagen? ¿Y si el problema es que el disparo depende del propio drop?
3. Con el tema de prueba, anotá el drop diez veces reaccionando y diez veces anticipando (contando los pulsos del build-up). Compará el error medio y la dispersión.
4. ¿Por qué la autocorrelación confunde 128 BPM con 64 BPM? Dibujá la autocorrelación de una serie de impulsos regulares.
5. Elegí un tema real que conozcas bien y predecí a mano, contando compases desde el principio, dónde cae el drop. Después cargalo en la herramienta. ¿Coinciden?
6. Diseñá una estrategia para que las luces no exploten en un falso drop sin perder la anticipación. ¿Qué información del build-up podría distinguir un drop verdadero de uno falso?
7. Calculá la precisión, la exhaustividad y la medida F de un detector que predijo 18 drops en 20 temas con 22 drops anotados, de los cuales acertó 15.

### 25. Glosario

- **BPM:** pulsos por minuto; el tempo.
- **Compás:** grupo de pulsos (en EDM, casi siempre cuatro).
- **Frase:** grupo de compases (en EDM, típicamente 8, 16 o 32).
- **Build-up:** sección que construye tensión antes del drop, típicamente sin graves y con percusión que se acelera.
- **Drop:** momento en que vuelven los graves y la energía del tema llega a su máximo.
- **Falso drop:** drop esperado que no llega en el momento previsto.
- **FFT:** algoritmo rápido para calcular la transformada de Fourier discreta.
- **Ventana de Hann:** función que suaviza los bordes de un cuadro de audio antes de la FFT.
- **Flujo espectral:** medida de cuánto contenido nuevo aparece en el espectro de un cuadro al siguiente.
- **Ataque (onset):** comienzo de un evento sonoro.
- **Causal:** que usa solo información del pasado.
- **Asincronía media negativa:** tendencia a golpear antes del pulso al sincronizarse con él.
- **DMX512 / Art-Net:** protocolos de control de iluminación escénica.

### 26. Conclusión

La idea de luces que anticipan el drop en vez de reaccionar a él tiene fundamentos sólidos en tres campos. La psicología de la música, de Meyer a Huron, y la neurociencia de Salimpoor y Zatorre muestran que la anticipación es una parte central del placer musical, con su propio pico de dopamina, y que acertar una predicción es placentero en sí mismo. La musicología de la música electrónica, de Butler a Solberg, muestra que el género está construido para ser anticipado: frases regulares, build-ups con recursos convencionales, drops en comienzos de frase. Y el procesamiento de señales musicales, de la FFT de Cooley y Tukey a los detectores de ataques de Bello y Dixon y los seguidores del pulso de Scheirer y Ellis, da las herramientas para escuchar esas convenciones de manera automática y causal.

El prototipo funciona perfectamente sobre audio sintético: predice el drop con error cero y trece segundos de anticipación, mientras un sistema reactivo llega medio segundo tarde, muy fuera de la ventana de sincronía audiovisual. Lo que falta es lo más importante: medir con música real. El anotador está listo para construir el conjunto de 20 temas que la idea pedía, y los próximos pasos (modo en vivo con micrófono, salida Art-Net y comparación con los programas comerciales) dependen de lo que esa medición muestre.

## Referencias

- Bello, J. P., Daudet, L., Abdallah, S., Duxbury, C., Davies, M., & Sandler, M. B. (2005). A tutorial on onset detection in music signals. *IEEE Transactions on Speech and Audio Processing*, 13(5), 1035–1047. https://doi.org/10.1109/TSA.2005.851998
- Blood, A. J., & Zatorre, R. J. (2001). Intensely pleasurable responses to music correlate with activity in brain regions implicated in reward and emotion. *Proceedings of the National Academy of Sciences*, 98(20), 11818–11823. https://doi.org/10.1073/pnas.191355898
- Butler, M. J. (2006). *Unlocking the Groove: Rhythm, Meter, and Musical Design in Electronic Dance Music*. Indiana University Press. ISBN 978-0-253-34662-9.
- Böck, S., Korzeniowski, F., Schlüter, J., Krebs, F., & Widmer, G. (2016). madmom: A new Python audio and music signal processing library. En *Proceedings of the 24th ACM International Conference on Multimedia* (pp. 1174–1178). https://doi.org/10.1145/2964284.2973795
- Böck, S., Krebs, F., & Widmer, G. (2016). Joint beat and downbeat tracking with recurrent neural networks. En *Proceedings of the 17th International Society for Music Information Retrieval Conference* (pp. 255–261).
- Clark, A. (2013). Whatever next? Predictive brains, situated agents, and the future of cognitive science. *Behavioral and Brain Sciences*, 36(3), 181–204. https://doi.org/10.1017/S0140525X12000477
- Cohen, J. (1960). A coefficient of agreement for nominal scales. *Educational and Psychological Measurement*, 20(1), 37–46. https://doi.org/10.1177/001316446002000104
- Cooley, J. W., & Tukey, J. W. (1965). An algorithm for the machine calculation of complex Fourier series. *Mathematics of Computation*, 19(90), 297–301. https://doi.org/10.1090/S0025-5718-1965-0178586-1
- Cytowic, R. E. (1989). *Synesthesia: A Union of the Senses*. Springer.
- Dixon, S. (2001). Automatic extraction of tempo and beat from expressive performances. *Journal of New Music Research*, 30(1), 39–58. https://doi.org/10.1076/jnmr.30.1.39.7119
- Dixon, S. (2006). Onset detection revisited. En *Proceedings of the 9th International Conference on Digital Audio Effects (DAFx-06)* (pp. 133–137). Montreal.
- Ellis, D. P. W. (2007). Beat tracking by dynamic programming. *Journal of New Music Research*, 36(1), 51–60. https://doi.org/10.1080/09298210701653344
- Friston, K. (2010). The free-energy principle: A unified brain theory? *Nature Reviews Neuroscience*, 11(2), 127–138. https://doi.org/10.1038/nrn2787
- Harris, F. J. (1978). On the use of windows for harmonic analysis with the discrete Fourier transform. *Proceedings of the IEEE*, 66(1), 51–83. https://doi.org/10.1109/PROC.1978.10837
- Huron, D. (2006). *Sweet Anticipation: Music and the Psychology of Expectation*. MIT Press. ISBN 978-0-262-08345-4.
- ITU-R. (1998). *Recommendation BT.1359-1: Relative timing of sound and vision for broadcasting*. International Telecommunication Union.
- Krebs, F., Böck, S., & Widmer, G. (2015). An efficient state-space model for joint tempo and meter tracking. En *Proceedings of the 16th International Society for Music Information Retrieval Conference* (pp. 72–78).
- Meyer, L. B. (1956). *Emotion and Meaning in Music*. University of Chicago Press.
- Müller, M. (2015). *Fundamentals of Music Processing: Audio, Analysis, Algorithms, Applications*. Springer. ISBN 978-3-319-21944-8.
- Panksepp, J. (1995). The emotional sources of "chills" induced by music. *Music Perception*, 13(2), 171–207. https://doi.org/10.2307/40285693
- Ramachandran, V. S., & Hubbard, E. M. (2001). Synaesthesia: A window into perception, thought and language. *Journal of Consciousness Studies*, 8(12), 3–34.
- Repp, B. H. (2005). Sensorimotor synchronization: A review of the tapping literature. *Psychonomic Bulletin & Review*, 12(6), 969–992. https://doi.org/10.3758/BF03206433
- Repp, B. H., & Su, Y.-H. (2013). Sensorimotor synchronization: A review of recent research (2006–2012). *Psychonomic Bulletin & Review*, 20(3), 403–452. https://doi.org/10.3758/s13423-012-0371-2
- Salimpoor, V. N., Benovoy, M., Larcher, K., Dagher, A., & Zatorre, R. J. (2011). Anatomically distinct dopamine release during anticipation and experience of peak emotion to music. *Nature Neuroscience*, 14(2), 257–262. https://doi.org/10.1038/nn.2726
- Scheirer, E. D. (1998). Tempo and beat analysis of acoustic musical signals. *Journal of the Acoustical Society of America*, 103(1), 588–601. https://doi.org/10.1121/1.421129
- Solberg, R. T. (2014). "Waiting for the bass to drop": Correlations between intense emotional experiences and production techniques in build-up and drop sections of electronic dance music. *Dancecult: Journal of Electronic Dance Music Culture*, 6(1), 61–82. https://doi.org/10.12801/1947-5403.2014.06.01.04
- Spence, C. (2011). Crossmodal correspondences: A tutorial review. *Attention, Perception, & Psychophysics*, 73(4), 971–995. https://doi.org/10.3758/s13414-010-0073-7
- Tarr, B., Launay, J., & Dunbar, R. I. M. (2014). Music and social bonding: "Self-other" merging and neurohormonal mechanisms. *Frontiers in Psychology*, 5, 1096. https://doi.org/10.3389/fpsyg.2014.01096
- USITT. (1986). *DMX512: Digital Data Transmission Standard for Dimmers and Controllers*. United States Institute for Theatre Technology.
- Vroomen, J., & Keetels, M. (2010). Perception of intersensory synchrony: A tutorial review. *Attention, Perception, & Psychophysics*, 72(4), 871–884. https://doi.org/10.3758/APP.72.4.871
- Vuust, P., & Witek, M. A. G. (2014). Rhythmic complexity and predictive coding: A novel approach to modeling rhythm and meter perception in music. *Frontiers in Psychology*, 5, 1111. https://doi.org/10.3389/fpsyg.2014.01111
- Ward, J., Huckstep, B., & Tsakanikos, E. (2006). Sound-colour synaesthesia: To what extent does it use cross-modal mechanisms common to us all? *Cortex*, 42(2), 264–280. https://doi.org/10.1016/S0010-9452(08)70352-6
- Yadati, K., Larson, M., Liem, C. C. S., & Hanjalic, A. (2014). Detecting drops in electronic dance music: Content based approaches to a socially significant music event. En *Proceedings of the 15th International Society for Music Information Retrieval Conference (ISMIR)* (pp. 143–148). Taipéi.
