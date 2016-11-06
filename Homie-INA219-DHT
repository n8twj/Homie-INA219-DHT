#include <Homie.h>
#include <Wire.h>
#include "DHT.h"
#include <Adafruit_INA219.h>

extern "C" {
  #include "user_interface.h"
}

#define DHTPIN 14
#define DHTTYPE AM2301

Adafruit_INA219 battery(0x40);

float shuntVoltage = 0;
float busVoltage = 0;
float currentMa = 0;
float loadVoltage = 0;

const int INTERVAL = 60;
unsigned long lastSent = 0;

DHT dht(DHTPIN, DHTTYPE);

HomieNode batteryLoadVoltageNode("batteryLoadVoltage", "voltage");
HomieNode batteryCurrentNode("batteryCurrent", "millamps");
HomieNode temperatureNode("temperature", "Fahrenheit");
HomieNode humidityNode("humidity", "percent");
HomieNode heatIndexNode("heatIndex", "Fahrenheit");

void loopHandler() {
  if (millis() - lastSent >= INTERVAL * 1000UL || lastSent == 0) {
    shuntVoltage = battery.getShuntVoltage_mV();
    busVoltage = battery.getBusVoltage_V();
    currentMa = battery.getCurrent_mA();
    loadVoltage = busVoltage + (shuntVoltage / 1000);
    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature(true);
    float heatIndex = dht.computeHeatIndex(temperature, humidity);
    if (Homie.isConnected()) {
      batteryLoadVoltageNode.setProperty("volts").send(String(loadVoltage));
      batteryCurrentNode.setProperty("milliamps").send(String(currentMa));
      temperatureNode.setProperty("Fahrenheit").send(String(temperature));
      humidityNode.setProperty("percent").send(String(humidity));
      heatIndexNode.setProperty("Fahrenheit").send(String(heatIndex));
    }
    lastSent = millis();
  }
}

void setup() {
  Serial.begin(115200);
  battery.begin();
  Homie_setFirmware("ina219-dht", "1.0.0");
  Homie.setLoopFunction(loopHandler);
  Homie.disableLedFeedback();
  Homie.setup();
}

void loop() {
  Homie.loop();
}
