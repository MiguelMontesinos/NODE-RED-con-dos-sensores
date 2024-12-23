# NODE-RED-con-dos-sensores

## Practica con un sensor de temperatura y humedad como un sensor ultrasonico, en una placa de desarrollo ESP32 y comunicado en Node-RED. 
Este repositorio muestra como podemos programar una ESP32 con los sensores DHT22 HC-SR04 y mostrar los datos optenidos en Node-RED. 

## Descripción:

La ESP32 la utilizamos en un entorno de adquision de datos, en esta practica ocuparemos un sensor ultrasonico para medir la distancia, como tambien un sensor de temperatura y humedad los datos optenidos seran visualizados en forma de grafica como indicador en Node-RED.

## Material Necesario:

Para realizar esta practica necesitas lo siguiente:
- WOKWI
- Tarjeta ESP32
- Sensor ultrasonico HC-SR04
- Sensor de temperatura y humedad DHT22
- Node-RED

## Instrucciones de operación: 

1. Iniciar Simulador
2. Visualizar los datos en el apartado de Dashboard del node-RED
3. Subir y Bajar la distancia dando doble click al sensor ultrasonico o al dht 22

## Librerías:

1. Arduino Json
2. DHT sensor library for ESPx
3. PubSubClient

## Programación:
```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 0;   
const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.76.79.164";
String username_mqtt="Miguel";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {

 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms
delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"]= String(d);
     doc["nombre"] = "Miguel Montesinos";
   

    String output;
     
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("PRACTICA7", output.c_str());
  }
}
 ```

## Conexión:

![image](https://github.com/MiguelMontesinos/NODE-RED-con-dos-sensores/blob/main/Captura%20de%20pantalla%202024-12-17%20220010.png?raw=true)
![image](https://github.com/MiguelMontesinos/NODE-RED-con-dos-sensores/blob/main/Captura%20de%20pantalla%202024-12-21%20085112.png?raw=true)


## Resultados:

Cuando funcione y Corra los valores serán mostrados en la pantalla de DashBoard, cada 2 segundos se actualizará, se muestran 3 gráficas tipo gauge (indicadores) y 3 de tipo chart (graficas)

![image](https://github.com/MiguelMontesinos/NODE-RED-con-dos-sensores/blob/main/Captura%20de%20pantalla%202024-12-21%20091103.png?raw=true)

## Desarrollado por 

**Ing. Miguel De Jesus Montesinos Molina** 

[GitHub](https://github.com/MiguelMontesinos).



