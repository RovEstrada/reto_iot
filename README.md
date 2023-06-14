
##Conexiones de sensores

##Base de datos

###Configuración
En nuestro proyecto hicimos uso del servicio de Google de Firebase, enseguida se muestra la configuración de la misma.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/e1e461c7-2188-4d43-8a07-793e6d42ba06)
**Figura 1.** Configuración de la base de datos creada

![](https://github.com/RovEstrada/reto_iot/assets/103225320/184cee14-cee5-451a-846e-f7b11f48fed9)
**Figura 1.** Información de configuración de conexión a la base de datos

________
##App de App inventor
###Configuración de elementos

###Bloques de código

![](https://github.com/RovEstrada/reto_iot/assets/103225320/6051f54c-58aa-45d7-895b-7df049eaa700)

Estos bloques de código, describen el comportamiento, mediantes condicionales if,  lo que se traduce en si cambia una condición en específico, relacionada a la database, esta se asignará el valor en la interfaz de usuario, facilitando su observación, y checando condición por condición, la variable que cambia.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/fd4a7b5b-f020-4e51-a140-d2f21f0c13c7)

Igualmente el último bloque de código es el encargado de enviar la información al momento de realizar un click de manera que dependiendo de qué campo cambie el usuario en la aplicación ejecutándose, esta se verá reflejada directamente en la base de datos.



###Diseño de interfaz de usuario

![](https://github.com/RovEstrada/reto_iot/assets/103225320/69f93289-d654-45b5-ae76-93bb7082753f)
**Figura 1.** Interfaz de usuario de la aplicación que muestra los datos existentes de la base de datos en Firebase
_______

##ESP32
Con este programa te conectarás a internet para entrar en tu Firebase. Transmitimos la humedad, el sensor de temperatura DHT11 y el sensor RGB TC3200. Hay un total de 5 valores que transmitimos a la Firebase. 

###Código
####Código ESP32
```cpp
#include <WiFi.h>
#include <PubSubClient.h>
#include <Firebase_ESP_Client.h>
#include <addons/TokenHelper.h>
#include "DHT.h"

// Replace the next variables with your SSID/Password combination
const char* ssid = "Raphael";
const char* password = "12345678";
//___
#define DHTPIN 4   // Digital pin connected to the DHT sensor
// Feather HUZZAH ESP8266 note: use pins 3, 4, 5, 12, 13 or 14 --
// Pin 15 can work but DHT must be disconnected during program upload.
#define DHTTYPE DHT11   // DHT 11
DHT dht(DHTPIN, DHTTYPE);
//__
//Sensor TCS3200 Color 
#define S2 19 // Define S2 Pin Number of ESP32
#define S3 18 // Define S3 Pin Number of ESP32
#define sensorOut 5 // Define Sensor Output Pin Number of ESP32/
//__


#define API_KEY "AIzaSyD4xRdcLCjxIrTh4CTrENuMvq4FzjrFAKY"  //AIzaSyAjjTHMIV0y394tayvijhU-aVVcKdkIZxU//AIzaSyAjjTHMIV0y394tayvijhU-aVVcKdkIZxU

// Insert RTDB URLefine the RTDB URL */
#define DATABASE_URL "https://prueba1-64536-default-rtdb.firebaseio.com"

//Define Firebase Data object
FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;

unsigned long sendDataPrevMillis = 0;
int intValue;
float floatValue;
int entrance=0; //Is for a RoundRobin system 

bool signupOK = false;
void setup(){ 
  pinMode(S2, OUTPUT); // Define S2 Pin as a OUTPUT
  pinMode(S3, OUTPUT); // Define S3 Pin as a OUTPUT
  pinMode(sensorOut, INPUT); // Define Sensor Output Pin as a INPUT
  Serial.begin(115200); // Set the baudrate to 115200
  //Serial.println(F("DHTxx test!"));
  delay(10);
  setup_wifi();
  dht.begin();
}

void setup_wifi() {
  //WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
  //* Assign the api key (required) */
  config.api_key = API_KEY;

  /* Assign the RTDB URL (required) */
  config.database_url = DATABASE_URL;

  /* Sign up */
  if (Firebase.signUp(&config, &auth, "", "")){
    Serial.println("ok");
    signupOK = true;
  }
  else{
    Serial.printf("%s\n", config.signer.signupError.message.c_str());
  }

  /* Assign the callback function for the long running token generation task */
  config.token_status_callback = tokenStatusCallback; //see addons TokenHelper.h

  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
}

//Create function for sensor values
float sensor(bool humidity,bool temperature);
int getRed();
int getGreen();
int getBlue();


void loop() {
  
  if (Firebase.ready() && signupOK && (millis() - sendDataPrevMillis > 15000 || sendDataPrevMillis == 0)) {
    sendDataPrevMillis = millis();
    delay(500);
   // Write an Int number on the database path test/int Value 1
    if (entrance == 0 && Firebase.RTDB.setFloat(&fbdo, "app_inventor/Temperature", sensor(false,true))){
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
      entrance = 1;
    }
    // Write an Float number on the database path test/float Value 2
    if (entrance == 1 && Firebase.RTDB.setFloat(&fbdo, "app_inventor/humidity", sensor(true,false))){
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
      entrance = 2;
    }
    // Sensor 3 for Value 3
    if (entrance == 2 &&Firebase.RTDB.setFloat(&fbdo, "app_inventor/Color_Red", getRed())){
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
      entrance = 3;
    }
    // Sensor 4 for Value 4
    if (entrance == 3 &&Firebase.RTDB.setFloat(&fbdo, "app_inventor/Color_Green", getGreen())){
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
      entrance = 4;
    }
    // Sensor 5 for Value 5
    if (entrance == 4 && Firebase.RTDB.setFloat(&fbdo, "app_inventor/Color_Blue", getBlue())){
      Serial.println("PASSED");
      Serial.println("PATH: " + fbdo.dataPath());
      Serial.println("TYPE: " + fbdo.dataType());
      entrance = 0;
    }
  }
}



// DHT SENSOR
float sensor(bool humidity,bool temperature){
 // Wait a few seconds between measurements.

  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
  float h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  float t = dht.readTemperature();
  // Read temperature as Fahrenheit (isFahrenheit = true)
  float f = dht.readTemperature(true);

  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t) || isnan(f)) 
  {
    Serial.println(F("Failed to read from DHT sensor!"));
    return 0;
  }

  // Compute heat index in Fahrenheit (the default)
  float hif = dht.computeHeatIndex(f, h);
  // Compute heat index in Celsius (isFahreheit = false)
  float hic = dht.computeHeatIndex(t, h, false);
 
  if(humidity==true)return h;
  else return t;
}



// RGB SENSOR TC3200
int getRed() {
  digitalWrite(S2,LOW);
  digitalWrite(S3,LOW);
  int Frequency = pulseIn(sensorOut, LOW); // Get the Red Color Frequency
  return Frequency;
}

int getGreen() {
  digitalWrite(S2,HIGH);
  digitalWrite(S3,HIGH);
  int Frequency = pulseIn(sensorOut, LOW); //Get the Green Color Frequency
  return Frequency;
}

int getBlue() {
  digitalWrite(S2,LOW);
  digitalWrite(S3,HIGH);
  int Frequency = pulseIn(sensorOut, LOW); // Get the Blue Color Frequency
  return Frequency;
}
}
```


![](https://github.com/RovEstrada/reto_iot/assets/103225320/02aad226-effe-4c4d-8c21-7bf067b62457)

####Configuración
En esta ilustración, establecemos los pines ocupados para los sensores, así como los datos de conexión para la base de datos y la conexión WLan.


![](https://github.com/RovEstrada/reto_iot/assets/103225320/38f9ddae-17da-4044-8c1c-7468f4b0d3cc)


####void setup()
En el void setup() tenemos que establecer los PINS como entradas o salidas, dependiendo de cómo conectemos el sensor. Además, llamamos a las funciones para la conexión Wifi y a la función para el sensor DHT.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/4e273fc9-2c3d-4fde-9f88-c45c5f4992b5)

####void setup_wifi()
En la función void setup_wifi()
Se llama al SSID y a la contraseña previamente determinados y se indica si se ha podido establecer una conexión.
Además, se recuperan el token y la URL de Firebase y se marca si se puede establecer una conexión.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/c52ff02f-149e-4c2f-8b4a-d6d1c13d93c0)

####void loop()
Es la función principal. Aquí llamamos a las sub-funciones(sensores) para poder usar sus valores en el void loop(). Tan pronto como la Firebase esté disponible y estemos conectados, podemos dar salida a los valores.  Para ello, especificamos la dirección a la que queremos escribir en la Firebase y el tipo de datos. A través de la variable "entrada" damos salida al estado actual y no se puede saltar ningún valor.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/eabbc040-76ed-4831-ba48-e7f2728a95a7)

####float sensor(...)
En esta función se leen los datos del sensor DHT11 y dependiendo de si "bool humedad" es igual a "TRUE" o "bool temperatura" es igual a "TRUE", se pasa este valor como valor de retorno.

![](https://github.com/RovEstrada/reto_iot/assets/103225320/7a46222d-e9d1-4239-8081-dead2ab034b8)

####int getColor()
Utilizamos el sensor TC3200 para la salida de la frecuencia del componente de color respectivo. A través de las salidas "S2,S3" podemos ajustar qué componente de color se pasa como retorno.
_________
Eso es todo, diviértete con el programa que se proporciona de forma gratuita.

Realizado por
 
Roberto Ivan
Jeffry Johnson
Rafael Mayer
