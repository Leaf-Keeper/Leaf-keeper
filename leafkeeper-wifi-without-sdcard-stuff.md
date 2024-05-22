```
#include "DHT.h"
#include <WiFi.h> // Include the Wi-Fi library for ESP32
#include <Wire.h>
#include "RTClib.h"

#define DHTTYPE DHT22
#define LOG_INTERVAL 2000 // 2 seconds

const int soilTempPin = A0;
const int soilMoisturePin = A1;
const int sunlightPin = A2;
const int dhtPin = 2;
const int LEDPinGreen = 6;
const int LEDPinRed = 7;
const int solenoidPin = 3;
const int wateringTime = 600000; //Set the watering time (10 min for a start)
const float wateringThreshold = 15; //Value below which the garden gets watered

DHT dht(dhtPin, DHTTYPE);
RTC_DS1307 rtc;

float soilTemp = 0; //Scaled value of soil temp (degrees F)
float soilMoistureRaw = 0; //Raw analog input of soil moisture sensor (volts)
float soilMoisture = 0; //Scaled value of volumetric water content in soil (percent)
float humidity = 0; //Relative humidity (%)
float airTemp = 0; //Air temp (degrees F)
float heatIndex = 0; //Heat index (degrees F)
float sunlight = 0; //Sunlight illumination in lux
bool watering = false;
bool wateredToday = false;
DateTime now;

// Wi-Fi credentials
const char *ssid = "YourSSID";
const char *password = "YourPassword";

WiFiServer server(80);

void setup() {
  Serial.begin(9600);
  Serial.println("Connecting to WiFi...");
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  
  Serial.println("Connected to WiFi");
  
  server.begin();
  Serial.println("HTTP server started");
  
  pinMode(LEDPinGreen, OUTPUT);
  pinMode(solenoidPin, OUTPUT);
  digitalWrite(solenoidPin, LOW);
  
  dht.begin();
  
  Wire.begin();
  if (!rtc.begin()) {
    Serial.println("RTC failed");
  }
  if (! rtc.isrunning()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
  }
}

void loop() {
  WiFiClient client = server.available();
  
  if (client) {
    while (client.connected()) {
      if (client.available()) {
        String request = client.readStringUntil('\r');
        client.flush();
        
        if (request.indexOf("/moisture") != -1) {
          client.println("HTTP/1.1 200 OK");
          client.println("Content-Type: text/plain");
          client.println("Connection: close");
          client.println();
          client.print("Moisture: ");
          client.println(soilMoisture);
          break;
        }
      }
    }
    client.stop();
  }
  
  // Your existing sensor reading code here
  // Ensure to update the soilMoisture value
  
  delay(LOG_INTERVAL);
}

// Sensor reading functions and other helper functions can be kept as they are
```
