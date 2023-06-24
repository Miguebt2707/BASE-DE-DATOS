
#  BASE DE DATOS

LO QUE SE RALIZO EN ESTA PRACTICA FUE EL PODER HACER COINCIDIR NUESTRAS LECTURAS DE SENSORES QUE TENEMOS PARA SABER LA DISTANCIA, TEMPERATURA Y HUMEDAD, DONDE PARA MEDIR TEMPERATURA Y HUMEDAD SE UTILIZO UN SENSOR DTH22, PARA LA DISTANCIA SE UTILIZO UN SENSOR ULTTRASONICO (HC-SR-04) 
 
## MATERIALES
- WOKWI
- TARJETA ESP32
- SENSOR DTH22
- SENSOR ULTRASONICO (HC-SR-04)  

A PARTIR DE ESTE PUNTO SE MUESTRAN LAS CONEXIONES QUE SE HICIERON DE NUESTRA PLACA ESP32 A NUESTROS SENSORES QUE ES EL ULTRASONICO Y EL ESP32 ASI COMO TAMBIEN SE MUESTRAN LAS LIBRERIAS QUE SE OCUPARON QUE SON PARTE FUNDAMENTAL.  

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/LIBRERIAS.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/SENS1.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/SENS2.png?raw=true)
EN ESTE PUNTO SE PUEDEN OBSERVAR LAS NUVES QUE SE UTILIZARON PARAPODER HACER LA INTERACCION ENTRE EL WOKWI Y EL NODE RED- ASI COMO EN EL LOCALHOST PARA QUE NUESTROS DATOS SE PUEDAN VER REFLJADOS
![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/NODERED-DIAGRAMA%20GEN.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/FUNCTION5.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/DIPLOMADO1.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/GEEEEN.png?raw=true)

![](https://github.com/Miguebt2707/BASE-DE-DATOS/blob/main/GRAFICOS.png?raw=true)




```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
DHTesp dhtSensor;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "44.195.202.69";
String username_mqtt="Miguelon";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;
const int Trigger = 13;   //Pin digital 2 para el Trigger del sensor
const int Echo = 12;   //Pin digital 3 para el Echo del sensor
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


delay(1000);
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm

TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 3000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    //doc["Anho"] = 2022;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   doc [ "DISTANCIA" ] =d;

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("MigueBT2707", output.c_str());
  }
}
```