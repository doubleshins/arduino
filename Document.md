## Blink — programming an Arduino sketch
```bash
void setup() {
    Serial.begin(115200);  // open serial connection to USB Serial port (connected to your computer)
    Serial1.begin(57600);  // open internal serial connection to MT7688AN
                           // in MT7688AN, this maps to device
    pinMode(13, OUTPUT);
    }
 
void loop() {
    int c = Serial1.read();      // read from MT7688AN
    if (c != -1) {
        switch(c) {
          case '0':                // turn off D13 when receiving "0"
             digitalWrite(13, 0);
            break;
          case '1':                // turn off D13 when receiving "1"
             digitalWrite(13, 1);
            break;
        }
    }
} 
```

## v1
```bash
#include "DHT.h"
#include <Bridge.h>
#include <BridgeClient.h>
#include <MQTT.h>

BridgeClient net;
MQTTClient client;

unsigned long lastMillis = 0;

#define DHTPIN 18    
#define DHTTYPE DHT22   

DHT dht(DHTPIN, DHTTYPE);

int pin = 4;
int water = 5;
boolean state = false;

void setup() {

  pinMode(pin, OUTPUT); 
  pinMode(water, OUTPUT);
  digitalWrite(pin, !state); 
  digitalWrite(water, !state);
  
  Serial.begin(9600);
  Serial.println("DHTxx test!");

  dht.begin();

  Bridge.begin();

  client.begin("114.33.24.28", net);
  client.onMessage(messageReceived);

  connect();
}


void connect() {
  Serial.print("connecting...");
  while (!client.connect("arduino", "dic", "dic2050014")) {
    Serial.print(".");
    delay(1000);
  }

  Serial.println("\nconnected!");

  client.subscribe("/control");
  // client.unsubscribe("/hello");
}

void messageReceived(String &topic, String &payload) {
  delay(200); //Debounce
  
  if (payload == "lon"){
    digitalWrite(pin, state); //開
  }
  else if(payload == "loff") {
    digitalWrite(pin, !state); //關
  }
  else if(payload == "won") {
    digitalWrite(water, state); //關
  }
  else if(payload == "woff") {
    digitalWrite(water, !state); //關
  }
  else {
    Serial.println("incoming: " + topic + " - " + payload);
  }

  //Serial.println("incoming: " + topic + " - " + payload);
}



void loop() {
  client.loop();
  
  if (!client.connected()) {
    connect();
  }


  
  
  delay(2000);
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  float f = dht.readTemperature(true);

  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }



  
  float hif = dht.computeHeatIndex(f, h);
  float hic = dht.computeHeatIndex(t, h, false);


  
  // publish a message roughly every second.
  if (millis() - lastMillis > 5000) {
    lastMillis = millis();
    String msg = "{\"Humidity\":" + String(h) + " ,\"Temperature\":" + String(t) + ",\"Heatindex\":" + String(hic) + "}";
    client.publish("/sensor", msg);
  }



}
```



## 參考資料
- MPU-MCU UART Connection : https://docs.labs.mediatek.com/resource/linkit-smart-7688/en/tutorials/linkit-smart-7688-duo/mpu-mcu-uart-connection
