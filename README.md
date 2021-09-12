```C

// Librerias del sensor ESP 8266
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include "FirebaseESP8266.h"
#include <DHT.h>

// Pines de conexión
#define DHT11PIN 5
#define SENSORDHT11 4

// El tipo de sensor
#define DHTTYPE DHT11
DHT dht(SENSORDHT11, DHTTYPE);

// Direccion del host servidor
#define FIREBASE_HOST "https://aldebaran-a6ff8-default-rtdb.firebaseio.com/"

// Secretos de la base de datos
#define FIREBASE_AUTH "KMzKw1WnpafjxCTKsGLwIeBxtw94BaqiSrmg8pdm"


// Configuración de la conexión Wifi
#define WIFI_SSID "Developer"
#define WIFI_PASSWORD "12345678"

WiFiClient client; 
FirebaseData firebaseData;


// Fotorresistencia 
int valueLight;

// Sensor de humedad (FC - 28) 
int valueHumidity;
int percentageHumidity;


// Sensor de Temperatura (DH - 11)
int Vo;
float valueTemperature;
float R1 = 300;              
float logR2, 
float R2;
float c1 = 2.108508173e-03;
float c2 = 0.7979204727e-04; 
float c3 = 6.535076315e-07;


void setup() {
  
  // Se inicializa la comunicación serie a 9600 bps  
  Serial.begin(9600);
  
  // Se inicializa comunicación con el módulo ESP 8266
  WiFi.begin (WIFI_SSID, WIFI_PASSWORD);
  
  // Se revisa la conexión con el módulo ESP 8266
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  // Se imprime la IP del módulo ESP 8266
  Serial.println ("");
  Serial.println ("Se conectó al wifi!");
  Serial.println(WiFi.localIP());
    
  // Se inicializa la conexión con la base de Datos en tiempo real (FIREBASE)
  Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH);
  dht.begin();

  //
  pinMode(DHT11PIN, OUTPUT);
  pinMode(SENSORDHT11, INPUT);
}

void loop() {

  // 
  Firebase.getInt(firebaseData,"/state");
  int dato=firebaseData.intData();
  
  //
  analogWrite(DHT11PIN, dato);

  //
  float t = dht.readTemperature();
  int h = dht.readHumidity(); 
  
  //
  Firebase.setString(firebaseData,"temperature", String(t));  
  Firebase.setString(firebaseData,"humidity", String(h));
    
  
  // Fotorresistencia
  
    //Se realiza la lectura analógica A5
  valueLight = analogRead(A5);
    
    //Se realiza la impresión del valor obtenido de A5
  Serial.print("Luz: ");
  Serial.println(valueLight);
  
  
  // Humedad 
  
    //Se realiza la lectura analógica A1
  valueHumidity = analogRead(A1);
    //Convierte el valor obtenido de A1 a porcentaje
  percentageHumidity = map(valueHumidity, 1023, 0, 0, 100);
  
    //Se realiza la impresión del valor obtenido de A1
  Serial.print("Humedad: ");
  Serial.println(valueHumidity);
    //Se realiza la impresión del valor de la humedad ya convertido a porcentaje
  Serial.print("Porcetaje: ");
  Serial.println(percentageHumidity);
  Serial.print( %);
  //Se realiza la toma de valores cada 0.5 segundos      
  delay(500);
  
  
  // Temperatura
    
    //Se realiza la lectura analógica A0
  Vo = analogRead(A0);
  R2 = R1 * ( 1023.0 / ( (float)Vo - 1.0 ) );
  logR2 = log(R2);
    //Se realiza el cálculo del valor de la Temperatura
  valueTemperature = (1.0 / ( c1 + (c2 * logR2) + (c3 * logR2 * logR2 * logR2) ));
    //Se realiza el cálculo para obtener el valor de la Temperatura en Centígrados
  valueTemperature = valueTemperature - 273.15;
    //Se realiza la impresión del valor de la Temperatura en Centígrados
  Serial.print("Temperatura: ");
  Serial.print(valueTemperature);
  Serial.println(" C");
    //Se realiza la toma de valores cada 0.5 segundos 
  delay(500);
}
```
