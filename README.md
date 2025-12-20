# Practica4-SigueLineas
---
Alumnos:
* Yassir El Kasmi
* Mateo Ferreira
---
## 1. Introducción
Este proyecto consiste en la creación de un robot autónomo capaz de seguir una línea en un circuito, detectar obstáculos, y enviar mensajes a través de MQTT para la comunicación IoT. El robot se controla mediante un **Arduino UNO** para los sensores y motores, y un **ESP32 CAM** para la conectividad WiFi y la comunicación con un servidor MQTT.

### Objetivos:
- Seguir la línea sin salirse. (Y si llega salirse que intente recuperar la linea).
- Detectar obstáculos y detenerse de manera correcta (entre 5 y 8 cm del obstáculo). 
- Implementar comunicación IoT a través de MQTT.

## 2. Programación

### Código Arduino
El código de Arduino maneja el control de los motores, la detección de la línea y los obstáculos. A continuación, se describen las principales secciones del código:

1. **Control de Motores**: Utilizamos los pines del Arduino para controlar los motores DC. El código ajusta la velocidad de los motores para seguir la línea y detenerse ante  obstáculos.
2. **Detección de Línea**: Se usan tres sensores infrarrojos (izquierda, medio, derecha) que se encuentran en la parte delantera del coche y estan conectados a los pines analógicos A0, A1 y A2. Estos sensores leen valores continuos para detectar la línea, no como un sensor digital.
3. **Detección de Obstáculos**: Se utiliza un sensor ultrasónico conectado a los pines 12 y 13 del Arduino. El sensor mide la distancia y permite detener el robot cuando detecta un obstáculo dentro de el rango permitido.

### Tareas:
- *Tarea 1: Detección de linea.*
  La tarea más prioritaria con una frecuancia de **20 ms**, actualiza los valores y detecta cuando la linea se ha perdido y encontrado, lo que suma latencia a esta tarea, pero asegura no perder ningún mensaje.
- *Tarea 2: Parpadeo del LED*
  Es la tarea menos prioritaria con una frecuencia de **300 ms**, se encarga de que el LED se encienda cuando se pierda la linea.
- *Tarea 3 : Ultrasonido*
  Es la segunda tarea menos prioritaria con una frecuencia de **60 ms**, deadline corto para evitar choques de obstaculo.
- *Tarea 4: Motores*
  Es la segunda más prioritaria con una frecuencia **30 ms**, controla los motores dependiendo de los valores de los infrarojos de la tarea 1.

### Código ESP32
El ESP32 se encarga de la comunicación con el servidor MQTT y de la conectividad WiFi. El código configura la red WiFi y se conecta al servidor MQTT para enviar los mensajes JSON necesarios. El robot envía mensajes en diferentes momentos del ciclo, tales como:
- **START_LAP**: Inicia la vuelta al circuito.
- **END_LAP**: Marca el final de la vuelta.
- **OBSTACLE_DETECTED**: Informa cuando se detecta un obstáculo.
- **PING**: Envia un mensaje cada 4 segundos con el estado del robot.
- **LINE_LOST**: Notifica si el robot se ha salido de la línea.
- **INIT_LINE_SEARCH**: Notifica el comienzo de busqueda de linea nada más perderla.
- **STOP_LINE_SEARCH**: Notifica el fin de busqueda de linea nada más detectarla.
- **LINE_FOUND**: Notifica que ha encontrado la linea estando previamente perdida.
- **VISIBLE_LINE**: Muestra el procentaje de linea vista durante el camino.

Antes de empezar la vuelta, nuestro robot se asegura de haber conectado con el ESP mediante la comunicación serie. Para ello, en el setup del robot comprobamos constantemente si nos ha llegado el mensaje *{OK}*, que es el mensaje que va a mandar el ESP por el puerto serie cuando haya conseguido conectarse a la WiFi. 

Cuando las tareas quieran mandar mensajes al puerto serie, en vez de mandar el json entero en el robot, para disminuir la latencia del envío, hemos decidido mandar los mensajes en formato: *{NUMERO_ACCION:ARGUMENTO}*, siendo el argumento opcional, solo en acciones que se necesite. Así el ESP va a estar constantemente recibiendo estas acciones, y dependiendo del numero, enviará los json correspondientes, como se verá a continuación.

## 3. Comunicación IoT

### Protocolo MQTT
Envio de los mensajes json del esp.

 
    switch(action) {
    
    case 0: // START_LAP
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"START_LAP\"}";
      break;

    case 1: {// END_LAP
      unsigned long time = arg.toInt();
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"END_LAP\",\"time\":" 
        + String(time) + "}";
      break;
    }

    case 2: {// OBSTACLE_DETECTED
      float dist = arg.toFloat();
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"OBSTACLE_DETECTED\",\"distance\":" 
        + String(dist, 2) + "}";
      break;
    }

    case 3: // LINE_LOST
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"LINE_LOST\"}";
      break;

    case 4: {// PING
      unsigned long time = arg.toInt();
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"PING\",\"time\":" 
        + String(time) + "}";
      break;
    }

    case 5: // INIT_LINE_SEARCH
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"INIT_LINE_SEARCH\"}";
      break;

    case 6: // STOP_LINE_SEARCH
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"STOP_LINE_SEARCH\"}";
      break;

    case 7: // LINE_FOUND
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"LINE_FOUND\"}";
      break;

    case 8: {// VISIBLE_LINE
      float stat = arg.toFloat();
      json = "{\"team_name\":\"Maradona\",\"id\":\"4\",\"action\":\"VISIBLE_LINE\",\"value\":" 
        + String(stat, 2) + "}";
      break;
    }
    
### Detalles de la Conexión:
- **Servidor MQTT**: `193.147.79.118` (teachinghub.eif.urjc.es)
- **Puerto**: `21883`
- **Topic MQTT**: `/SETR/2025/4/`

## 4. Inconvenientes, problemas y soluciones
Un problema fue la batería, no la cargamos hasta el último día, lo que nos limitó bastante en clase. 

Quitando los errores personales, también tuvimos dificultades haciendo el PD encargado de asignar velocidades (todo lo demás fue bastante trivial mirando los códigos de ayuda proporcionados). Tras muchos intentos fallidos, haciendo el PD, decidimos volver al principio: un control puramente reactivo ejecutado como tarea. 
Esto nos restó bastante suavidad en el movimiento, pero también hace que la tarea encargada de mover los motores sea mucho más simple, y esto es importante no solo para simplificar la comprensión, sino para disminuir el tiempo de ejecución (ya que trabajamos en RT).

Como auto mejora, las velocidades deberían ser unas constantes para mayor comprensión, pero nosotros las harcodeamos en la tarea.

Otro problema, fue la detección de la línea. Con un código de prueba, veíamos los valores del infrarrojos con y sin linea debajo. Aproximadamente, cuando veíamos una linea, el infrarrojos marcaba 950, y cuando no había, unos 750.
Con la cinta de prueba, un umbral de detección de 850 no nos dio problemas, pero en el día del examen, probamos el código, y al estar muy pisada la linea, había puntos donde no la detectaba.

Para solucionar esto, bajamos en la segunda prueba el umbral a 800 y ya no nos dio problemas.

También tuvimos problemas en la exactitud del ping. Primeramente hicimos una tarea que se ejecutaba cada 4 segundos y mandaba  el ping, pero la latencia del FREERTOS hacía que se mandase cada 4 segundos y 100 ms, aproximadamente.
Para tener la exactitud deseada (4 segundos exactos), decidimos meter la funcionalidad del ping en el loop del robot:

``
  if (millis() - lastPing >= PING) {
    lastPing = millis();
    sendMsg(4, String(lastPing - startTime));
  }
``

Por último, vamos a hablar de que hemos elegido FREERTOS desde el principio, ya que estaba muy bien explicado en el código de ejemplo visto en clase. Como unica objeción, tuvimos que tener mucha precisión en los deadlines, ya que a veces la tarea del LED no se ejecutaba, pero quitando esos pequeños problemas puntuales, ha sido un acierto.

## 5. Conclusión y video
Este proyecto ha cumplido con los objetivos establecidos: el robot sigue la línea de forma autónoma, detecta obstáculos y se comunica mediante MQTT con el servidor.

### Video


https://github.com/user-attachments/assets/a25eb548-997f-4052-b2d2-b9d8c94fc5bb


## 6. Referencias
- Manual del Kit.
- Documentación de MQTT.
- ChatGPT.
