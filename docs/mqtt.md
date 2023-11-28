# Using MQTT to communicate the Barduino

Today we will be working on how to connect our Barduinos in a network to communicate between them. To do so we will use the MQTT protocol. 

## What is MQTT?

MQTT is a lightweight, publish-subscribe, machine to machine network protocol for message queue/message queuing service. It is designed for connections with remote locations that have devices with resource constraints or limited network bandwidth, such as in the Internet of Things (IoT).

![mqtt diagram](https://hlassets.paessler.com/common/files/infographics/mqtt-architecture.png)

MQTT is based on topics and subtopics 

![mqtt topics](https://www.cloudmqtt.com/images/blog/mqtt-topics.png)

Let's use it!

## Light up an LED

!!! note
    Pair up, you will control your partner LED :)

!!! warning
    You will need to install the PubSubClient Arduino library.

Let's use this code 

``` c++ linenums="1"
    #include <WiFi.h>
    #include <PubSubClient.h>
    #include <WiFiClientSecure.h>

    //---- WiFi settings
    const char* ssid = "Iaac-Wifi";
    const char* password = "EnterIaac22@";
    //---- MQTT Broker settings
    const char* mqtt_server = "704443975942461f8381dce09a42bf79.s2.eu.hivemq.cloud";  // replace with your broker url
    const char* mqtt_username = "fablabbcn";
    const char* mqtt_password = "fablabbcn";
    const int mqtt_port = 8883;

    bool state = false;

    WiFiClientSecure espClient;
    PubSubClient client(espClient);
    unsigned long lastMsg = 0;

    #define MSG_BUFFER_SIZE (50)
    char msg[MSG_BUFFER_SIZE];

    const char* name_topic = "josep";
    const char* send_topic = "adai";


    static const char* root_ca PROGMEM = R"EOF(
    -----BEGIN CERTIFICATE-----
    MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
    TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
    cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
    WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
    ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
    MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
    h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
    0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
    A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
    T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
    B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
    B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
    KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
    OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
    jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
    qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
    rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
    HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
    hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
    ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
    3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
    NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
    ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
    TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
    jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
    oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
    4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
    mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
    emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
    -----END CERTIFICATE-----
    )EOF";

    void setup() {
    Serial.begin(9600);
    Serial.print("\nConnecting to ");
    Serial.println(ssid);

    pinMode(0, INPUT);
    pinMode(48, OUTPUT);

    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nWiFi connected\nIP address: ");
    Serial.println(WiFi.localIP());

    while (!Serial) delay(1);

    espClient.setCACert(root_ca);
    client.setServer(mqtt_server, mqtt_port);
    client.setCallback(callback);
    }

    void loop() {

    if (!client.connected()) reconnect();
    client.loop();

    if(!digitalRead(0)){
        if (state){
        publishMessage(name_topic, "off", true);
        }
        else{
        publishMessage(name_topic, "on", true);
        }
        state = !state;
        //publishMessage(name_topic, String(1), true);
        delay(500);
    }

    //publishMessage(name_topic, String(1), true);
    //publishMessage(sensor2_topic, String(2), true);
    }

    //=======================================================================Function=================================================================================

    void reconnect() {
    // Loop until we’re reconnected
    while (!client.connected()) {
        Serial.print("Attempting MQTT connection…");
        String clientId = "ESP8266Client -";  // Create a random client ID
        clientId += String(random(0xffff), HEX);
        // Attempt to connect
        if (client.connect(clientId.c_str(), mqtt_username, mqtt_password)) {
        Serial.println("connected");

        client.subscribe(send_topic);  // subscribe the topics here
        //client.subscribe(command2_topic);   // subscribe the topics here
        } else {
        Serial.print("failed, rc=");
        Serial.print(client.state());
        Serial.println(" try again in 5 seconds");  // Wait 5 seconds before retrying
        delay(5000);
        }
    }
    }

    //=======================================
    // This void is called every time we have a message from the broker

    void callback(char* topic, byte* payload, unsigned int length) {
    String incommingMessage = ""
                                "";
    for (int i = 0; i < length; i++) incommingMessage += (char)payload[i];
    Serial.println("Message arrived[" + String(topic) + "]" + incommingMessage);
    if(incommingMessage == "on"){
        digitalWrite(48, HIGH);
    }
    else if(incommingMessage == "off"){
        digitalWrite(48, LOW);
    }

    }

    //======================================= publising as string
    void publishMessage(const char* topic, String payload, boolean retained) {
    if (client.publish(topic, payload.c_str(), true))
        Serial.println("Message publised [" + String(topic) + "]: " + payload);
    }
```

Change the lines

``` c++ linenums="1"
    const char* name_topic = "josep";
    const char* send_topic = "adai";
```

## Let's make some colours!

Now let's control the Neopixel!

### Understanding the Neopixel

Adafruit has a very [nice documentation](https://learn.adafruit.com/adafruit-neopixel-uberguide/arduino-library-use) for theri Neopixel library. 

Today we will focus on HUE color:

![hue](https://cdn-learn.adafruit.com/assets/assets/000/074/094/medium640/leds_hsv-diagram.png?1554489757)

Let's use this code to understand it better: 

``` c++ linenums="1"
#include <Adafruit_NeoPixel.h>
#define PIN 38
#define NUMPIXELS 1

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

int hue = 0;
String name = "";

void setup() {
  pixels.begin();  // INITIALIZE NeoPixel strip object (REQUIRED)
  Serial.begin(115200);
}

void loop() {
  if (Serial.available() > 0) {
    hue = Serial.parseInt();
    name = Serial.readStringUntil('\n');
    Serial.println(hue);
    Serial.println(name);
    pixels.setPixelColor(0, pixels.ColorHSV(hue, 255, 255));
    pixels.show();  // Send the updated pixel colors to the hardware.
  }
}

``` 

### Now with MQTT

Now lets join codes and light up our classmates Neopixels!

``` c++ linenums="1"
#include <WiFi.h>
#include <PubSubClient.h>
#include <WiFiClientSecure.h>

#include <Adafruit_NeoPixel.h>

//---- WiFi settings
const char* ssid = "Iaac-Wifi";
const char* password = "EnterIaac22@";
//---- MQTT Broker settings
const char* mqtt_server = "704443975942461f8381dce09a42bf79.s2.eu.hivemq.cloud";  // replace with your broker url
const char* mqtt_username = "fablabbcn";
const char* mqtt_password = "fablabbcn";
const int mqtt_port = 8883;

bool state = false;

WiFiClientSecure espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;

#define MSG_BUFFER_SIZE (50)
char msg[MSG_BUFFER_SIZE];

const char* class_topic = "mdef/class";
const char* name_topic = "mdef/adai";


static const char* root_ca PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----
)EOF";

#define PIN 38
#define NUMPIXELS 1

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

int hue = 0;
//char* name = "";


void setup() {
  Serial.begin(9600);
  Serial.print("\nConnecting to ");
  Serial.println(ssid);

  pixels.begin();

  pinMode(0, INPUT);
  pinMode(48, OUTPUT);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected\nIP address: ");
  Serial.println(WiFi.localIP());

  while (!Serial) delay(1);

  espClient.setCACert(root_ca);
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
}

void loop() {

  if (!client.connected()){
    reconnect();
  } 
  client.loop();

  if (Serial.available() > 0) {
    hue = Serial.parseInt();
    String name = (Serial.readStringUntil('\n'));
    name = "mdef/"+name;
    //name = nameS.c_str();
    Serial.println(hue);
    Serial.println(name);
    publishMessage(name.c_str(), String(hue), true);
  }
}

//=======================================================================Function=================================================================================

void reconnect() {
  // Loop until we’re reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection…");
    String clientId = "ESP32Client -";  // Create a random client ID
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), mqtt_username, mqtt_password)) {
      Serial.println("connected");

      client.subscribe(class_topic);  // subscribe the topics here
      client.subscribe(name_topic);   // subscribe the topics here
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");  // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

//=======================================
// This void is called every time we have a message from the broker

void callback(char* topic, byte* payload, unsigned int length) {
  String incommingMessage = "";
  for (int i = 0; i < length; i++){
    incommingMessage += (char)payload[i];
  } 
  Serial.println("Message arrived[" + String(topic) + "]" + incommingMessage);
  int hue = incommingMessage.toInt();
  pixels.setPixelColor(0, pixels.ColorHSV(hue, 255, 255));
  pixels.show();  // Send the updated pixel colors to the hardware.
}

//======================================= publising as string
void publishMessage(const char* topic, String payload, boolean retained) {
  if (client.publish(topic, payload.c_str(), true))
    Serial.println("Message publised [" + String(topic) + "]: " + payload);
}
```


## All together now!

Now let's do a class collabaration and let's create some art! 

We will use [p5js](https://p5js.org/), a javascript livrary to create visuals, and MQTT over websockets for our installation. 

Here you have the code: 

``` c++ linenums="1"
#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "Iaac-Wifi";
const char* password = "EnterIaac22@";
WiFiClient wifiClient;

const char* mqttBroker = "mqtt-staging.smartcitizen.me";
const char* mqttClientName = "josep";
const char* mqttUser = "fablabbcn102"; // MQTT User Authentification
const char* mqttPass = ""; // MQTT Password Authentification
const char* draw_topic = "lab/mdef/draw";
const char* class_topic = "lab/mdef/class";
const char* name_topic = "lab/mdef/adai";
PubSubClient mqttClient(wifiClient);

unsigned long values[] = {0,0,0,0};
String direction[] = {"U","R","D","L"};

String dir = "";

unsigned long threshold = 40000;

void mqttConnect() {
  
  while (!mqttClient.connected()) {
  
    Serial.print("Attempting MQTT connection...");
  
    if (mqttClient.connect(mqttClientName, mqttUser, mqttPass)) {
  
      Serial.println("connected");
      mqttClient.publish("hello", mqttClientName);
      
      // Topic(s) subscription
      mqttClient.subscribe(class_topic);
      mqttClient.subscribe(name_topic);
  
    } else {
      
      Serial.print("failed, rc=");
      Serial.print(mqttClient.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    
    }
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  String incommingMessage = "";
  for (int i = 0; i < length; i++){
    incommingMessage += (char)payload[i];
  } 
  Serial.println("Message arrived[" + String(topic) + "]" + incommingMessage);
}

void setup() {
  
  Serial.begin(115200);
  Serial.println("Hello");

  // Connect to wifi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.print("Connected to ");
  Serial.println(ssid);
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
  
  // MQTT setup
  mqttClient.setServer(mqttBroker, 1883);
  mqttClient.setCallback(callback);
}

unsigned long lastMsg = 0;
char msg[50];

void loop() {
  // Check if we are still connected to the MQTT broker
  if (!mqttClient.connected()) {
    mqttConnect();
  }

  // Let PubSubClient library do his magic
  mqttClient.loop();

  // Add your publish code here --------------------
  unsigned long max = 0;
  int max_i;
  for (int i=0; i<4; i++){
    values[i] = touchRead(i+4);
    //Serial.print(values[i]);
    //Serial.print(",");
    if (values[i] > max){
      max = values[i];
      max_i = i;
    }
  }
  //Serial.println("");
  if (max > threshold && dir != direction[max_i]){
    dir = direction[max_i];
    Serial.println(dir);
    mqttClient.publish(draw_topic, dir.c_str());
  }
  else if (max <= threshold){
    dir="";
  }
}
```
