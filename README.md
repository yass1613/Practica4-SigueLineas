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

### Código ESP32
El ESP32 se encarga de la comunicación con el servidor MQTT y de la conectividad WiFi. El código configura la red WiFi y se conecta al servidor MQTT para enviar los mensajes JSON necesarios. El robot envía mensajes en diferentes momentos del ciclo, tales como:
- **START_LAP**: Inicia la vuelta al circuito.
- **END_LAP**: Marca el final de la vuelta.
- **OBSTACLE_DETECTED**: Informa cuando se detecta un obstáculo.
- **PING**: Envia un mensaje cada 4 segundos con el estado del robot.
- **LINE_LOST**: Notifica si el robot se ha salido de la línea.

El ESP32 también gestiona la conexión WiFi y verifica que la conexión al servidor MQTT sea exitosa antes de empezar la vuelta.

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

## 4. Inconvenientes y problemas
-

## 5. Conclusión y video
Este proyecto ha cumplido con los objetivos establecidos: el robot sigue la línea de forma autónoma, detecta obstáculos y se comunica mediante MQTT con el servidor.

### Video

## 6. Referencias
- Manual del Kit.
- Documentación de MQTT.
- ChatGPT.
