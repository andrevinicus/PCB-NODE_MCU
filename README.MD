



* ESQUEMATICO
  
![Esquematico](https://github.com/felipeluizcarli/felipeluizcarli-PCB-NodeMCU8266./assets/127907269/57e037a3-2273-499f-9999-c81ca9b1c283)

* PDB
  
![PCB](https://github.com/felipeluizcarli/felipeluizcarli-PCB-NodeMCU8266./assets/127907269/42c922bb-691d-4f6f-b818-c3b6354f5791)

* 3D
  
![3D](https://github.com/felipeluizcarli/felipeluizcarli-PCB-NodeMCU8266./assets/127907269/157629c2-15b8-4fd7-8d05-72b0ccd841b3)
# Controle de Relé com ESP8266 via MQTT

Este projeto utiliza um ESP8266 para controlar um relé via mensagens MQTT. O dispositivo se conecta a uma rede Wi-Fi, assina um tópico MQTT e controla o relé com base nas mensagens recebidas (`ON` ou `OFF`).
## Componentes Necessários

- ESP8266
- Módulo Relé
- Broker MQTT (pode ser local ou baseado em nuvem)
- Fonte de alimentação adequada para o ESP8266

## Bibliotecas Utilizadas

- `ESP8266WiFi.h`: Biblioteca para conexão Wi-Fi com ESP8266.
- `PubSubClient.h`: Biblioteca para comunicação MQTT.

## Instalação

1. Instale o [Arduino IDE](https://www.arduino.cc/en/Main/Software).
2. Adicione o suporte para ESP8266 no Arduino IDE. Siga [este guia](https://github.com/esp8266/Arduino#installing-with-boards-manager).
3. Instale as bibliotecas necessárias:
   - Vá em `Sketch` -> `Include Library` -> `Manage Libraries`.
   - Procure por `ESP8266WiFi` e instale.
   - Procure por `PubSubClient` e instale.

## Configuração

1. **Credenciais Wi-Fi e Endereço do Broker MQTT:**
   - Substitua as seguintes linhas no código com as suas credenciais de Wi-Fi e o endereço do broker MQTT:
     ```cpp
     const char* ssid = "SEU_SSID";
     const char* password = "SUA_SENHA";
     const char* mqtt_server = "ENDERECO_DO_SEU_BROKER";
     ```

2. **Configuração do Pino do Relé:**
   - Certifique-se de que o relé está conectado ao pino D1 do ESP8266:
     ```cpp
     const int Relay = D1;
     ```

## Código

```cpp
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

const char* ssid = "SEU_SSID";
const char* password = "SUA_SENHA";
const char* mqtt_server = "ENDERECO_DO_SEU_BROKER";

WiFiClient espClient;
PubSubClient client(espClient);
const int Relay = D1;

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  if (String(topic) == "ControleRelay") {
    if (message == "ON") {
      digitalWrite(Relay, LOW);
      Serial1.print("OFFFFFFF");
    } else if (message == "OFF") {
      Serial.println("ONNNNNN");
      digitalWrite(Relay, HIGH);
    }
  }
}

void setup() {
  pinMode(Relay, OUTPUT);
  Serial.begin(9600);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP8266Client")) {
      Serial.println("connected");
      client.subscribe("ControleRelay");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}
