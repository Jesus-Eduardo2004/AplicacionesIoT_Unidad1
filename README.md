# AplicacionesIoT_Unidad1

Instrumento de evaluación de la Unidad I de la materia Aplicaciones de IoT

**Nombre:**
- Jesús Eduardo Olvera Hernández

**Número de Control:**
- 1223100424

**Carrera:**
- Desarrollo de Software Multiplataforma

**Institución:**
- Universidad Tecnológica del Norte de Guanajuato

**Asignatura:**
- Aplicaciones de IoT

**Unidad:**
- I. Adquisición y Procesamiento de Datos

## Link a carpeta con vídeos de demostración.
- [Carpeta Completa](https://drive.google.com/drive/folders/1AltVgdPIm4drza0GyoFIB4QJMG3daTU5?usp=drive_link)

## Actividades en pareja.
- [Actividad 1 y 2 parejas]()
### Códigos:
- [Diagrama de conexión](https://drive.google.com/file/d/1Z5yQZsXDXgcedLCH3gsuihzuxUKnMuXe/view?usp=drive_link)

```json
[
    {
        "id": "567153203a11f536",
        "type": "tab",
        "label": "Conexión_Sensores",
        "disabled": false,
        "info": "",
        "env": []
    },
    {
        "id": "ac87398af7c4a476",
        "type": "mqtt in",
        "z": "567153203a11f536",
        "name": "",
        "topic": "utng/sensors",
        "qos": "2",
        "datatype": "auto-detect",
        "broker": "0d16543b2ffdbac5",
        "nl": false,
        "rap": true,
        "rh": 0,
        "inputs": 0,
        "x": 290,
        "y": 200,
        "wires": [
            [
                "1a7eea378e93a6c8"
            ]
        ]
    },
    {
        "id": "1a7eea378e93a6c8",
        "type": "postgresql",
        "z": "567153203a11f536",
        "name": "",
        "query": "INSERT INTO sensor_details (sensor_id, user_id, value) VALUES (1,1,'{{{msg.payload}}}');",
        "postgreSQLConfig": "ef157743cd3ef5d6",
        "split": false,
        "rowsPerMsg": 1,
        "outputs": 1,
        "x": 550,
        "y": 200,
        "wires": [
            [
                "0474a167f419d54e"
            ]
        ]
```

- [Código Python Documentado](https://drive.google.com/file/d/1ZUmbq6Y5RqkkJ_RrJLiuGSuYi21Kj2Ih/view?usp=drive_link)

```python
# Importamos las bibliotecas necesarias para la comunicación en red y el protocolo MQTT
import network
from umqtt.simple import MQTTClient  # Cliente MQTT para interactuar con el broker

# Importamos módulos para el control de hardware
from machine import Pin  # Gestión de pines GPIO
from time import sleep  # Permite generar pausas en la ejecución del código
from hcsr04 import HCSR04  # Librería para el manejo del sensor ultrasónico

# Configuración de conexión con el servidor MQTT
MQTT_BROKER = "broker.emqx.io"  # Dirección del servidor MQTT
MQTT_USER = ""  # Nombre de usuario (si se requiere autenticación)
MQTT_PASSWORD = ""  # Clave de acceso (si se necesita)
MQTT_CLIENT_ID = ""  # Identificación única del cliente MQTT
MQTT_TOPIC = "utng/sensor"  # Tópico donde se publicarán los datos
MQTT_PORT = 1883  # Puerto de conexión MQTT

# Inicialización del sensor ultrasónico con los pines especificados
sensor = HCSR04(trigger_pin=16, echo_pin=4, echo_timeout_us=24000)

# Configuración de los pines GPIO para los LEDs (simulando un semáforo)
led_rojo = Pin(2, Pin.OUT)  # Indicador rojo
led_amarillo = Pin(5, Pin.OUT)  # Indicador amarillo
led_verde = Pin(18, Pin.OUT)  # Indicador verde

# Se aseguran valores iniciales apagados para los LEDs
led_rojo.value(0)
led_amarillo.value(0)
led_verde.value(0)

# Función para establecer la conexión WiFi
def conectar_wifi():
    print("Intentando conexión WiFi...", end="")
    sta_if = network.WLAN(network.STA_IF)  # Se crea una instancia de la interfaz WiFi
    sta_if.active(True)  # Se activa la interfaz
    sta_if.connect('Red-Peter', '12345678')  # Se inicia la conexión a la red
    while not sta_if.isconnected():  # Espera hasta establecer conexión
        print(".", end="")
        sleep(0.3)
    print("¡Conexión establecida!")

# Función para suscribirse al tópico MQTT
def subscribir():
    client = MQTTClient(MQTT_CLIENT_ID,
                        MQTT_BROKER, port=MQTT_PORT,
                        user=MQTT_USER,
                        password=MQTT_PASSWORD,
                        keepalive=0)
    client.set_callback(llegada_mensaje)  # Se asigna la función de callback para manejar mensajes
    client.connect()  # Conexión con el broker MQTT
    client.subscribe(MQTT_TOPIC)  # Suscripción al tópico definido
    print(f"Conectado a {MQTT_BROKER}, suscrito a {MQTT_TOPIC}")
    return client

# Función de callback para el manejo de mensajes entrantes
def llegada_mensaje(topic, msg):
    print("Mensaje recibido:", msg)

# Función para gestionar los LEDs en función de la distancia detectada
def controlar_leds(distancia):
    # Se apagan todos los LEDs antes de encender el correspondiente
    led_rojo.value(0)
    led_amarillo.value(0)
    led_verde.value(0)
    
    # Se determinan los niveles de alerta según la distancia medida
    if distancia < 10:
        led_rojo.value(1)  # Distancia corta (rojo encendido)
    elif 10 <= distancia < 20:
        led_amarillo.value(1)  # Distancia intermedia (amarillo encendido)
    else:
        led_verde.value(1)  # Distancia segura (verde encendido)

# Conectar a la red WiFi antes de iniciar el sistema
conectar_wifi()

# Se establece la conexión con el broker MQTT y se suscribe al tópico
client = subscribir()

# Variable para registrar la última distancia enviada y evitar redundancia
distancia_anterior = 0

# Bucle principal de monitoreo y control
while True:
    client.check_msg()  # Se verifica si hay mensajes nuevos en el tópico MQTT
    distancia = int(sensor.distance_cm())  # Obtención del valor de distancia en cm
    if distancia != distancia_anterior:  # Se envía el dato solo si hay cambios
        print(f"Distancia medida: {distancia} cm")
        client.publish(MQTT_TOPIC, str(distancia))  # Publicación en el tópico MQTT
        controlar_leds(distancia)  # Ajuste de los LEDs según la medición
    distancia_anterior = distancia  # Se actualiza el último valor registrado
    sleep(2)  # Se introduce una espera antes de la próxima medición
```

## Actividades individuales.
- [CRUD en PostgreSQL](https://drive.google.com/file/d/1hrd63iU5XqIcf64Su4R_Zquufry93_7A/view?usp=drive_link)
- [Instalación y Configuraciones Básicas]()
- [LED y Botón con Raspberry Pi](https://drive.google.com/file/d/1x6xWDYppdEP8YjCQMgHCqKXKxpmje_DH/view?usp=drive_link)
- [LED con Raspberry Pi](https://drive.google.com/file/d/13VAt7K7FHOF-ZjDfv2XtxmJ2gM69RNLr/view?usp=drive_link)
- [Conexión MQTT en Node-RED]()

## Placa Fenólica
| Imagen 1 | Imagen 2 |
|----------|----------|
|![Imagen de WhatsApp 2025-02-10 a las 18 18 37_9a71c0b9](https://github.com/user-attachments/assets/56db1118-8622-46d3-a678-0313df70acaa)|![Imagen de WhatsApp 2025-02-10 a las 18 18 38_034d7bcd](https://github.com/user-attachments/assets/22ca3c05-1d7b-430b-b741-67047d1b460a)|



## Calificaciones Curso Fundamentos de Python 2

| Examen | Calificación |
|--------|-------------|
| Examen 1 | ![Screenshot 2025-02-05 195315](https://github.com/user-attachments/assets/7cacb18a-ae0f-4e17-a8cd-0d1af0243749)|
| Examen 2 | ![Screenshot 2025-02-05 200856](https://github.com/user-attachments/assets/f659099a-3a8c-4692-b3c7-bad1dcb2f38b)|
| Examen 3 | ![Screenshot 2025-02-06 130022](https://github.com/user-attachments/assets/3847881c-bdd1-4f87-8c26-edb1b4962fbb)|
| Examen 4 | ![Screenshot 2025-02-06 144047](https://github.com/user-attachments/assets/024fe78c-8aeb-4dc2-a2c4-64556b23b729)|
| Examen Final | ![Screenshot 2025-02-06 173940](https://github.com/user-attachments/assets/73a9aa33-c990-42a6-8139-4a38ee134578)|
