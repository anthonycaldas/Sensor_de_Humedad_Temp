# Sensor_de_Humedad_Temp

#include <WiFi.h>
#include <ThingerESP32.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// --- DATOS DE THINGER.IO ---
#define USER_ID             "nayeli_cruz"     //
#define DEVICE_ID           "ESP32_Planta"    //
#define DEVICE_CREDENTIAL   "Nayeli123"

// --- DATOS WIFI ---
#define WIFI_SSID           "LAB_MEL"
#define WIFI_PASSWORD       "Mel_1981"

ThingerESP32 thing(USER_ID, DEVICE_ID, DEVICE_CREDENTIAL);

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32
#define SENSOR 4   
#define RELAY 26

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// --- VARIABLES DINÁMICAS (ESTAS YA NO SON ESTÁTICAS) ---
float temperatura = 25.0; 
String estado_suelo = "Cargando..."; // Esta cambiará sola

void setup() {
  Serial.begin(115200);
  pinMode(SENSOR, INPUT);
  pinMode(RELAY, OUTPUT);
  digitalWrite(RELAY, HIGH); 

  thing.add_wifi(WIFI_SSID, WIFI_PASSWORD);

  // --- RECURSO DINÁMICO ---
  // Thinger leerá estas variables cada vez que refresque el dashboard
  thing["datos"] >> [](pson& out){
      out["estado"] = estado_suelo; // Envía "SECO" o "HUMEDO" según el sensor
      out["temp"] = temperatura;
  };

  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  display.clearDisplay();
  display.setTextColor(WHITE);
}

void loop() {
  thing.handle();

  // LEER EL SENSOR
  int lectura = digitalRead(SENSOR);
  
  // ACTUALIZAR LA VARIABLE QUE VA A LA NUBE
  if (lectura == LOW) { 
    estado_suelo = "HUMEDO";  // Se actualiza el texto
    digitalWrite(RELAY, LOW); 
  } else {
    estado_suelo = "SECO";    // Se actualiza el texto
    digitalWrite(RELAY, HIGH); 
  }

  // SIMULAR CAMBIO DE TEMPERATURA (Para que el widget Gauge se mueva)
  temperatura = 24.0 + (random(0, 40) / 10.0);

  // ACTUALIZAR PANTALLA OLED
  display.clearDisplay();
  display.setCursor(0,0);
  display.print("Suelo: "); display.println(estado_suelo);
  display.print("Relay: "); display.println(lectura == LOW ? "ON" : "OFF");
  display.display();

  delay(500); 
}
