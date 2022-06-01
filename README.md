# Proyecto-Cerradura con targeta

Con este proyecto queriamos simular una puerta automática

## FUNCIONALIDADES

- Leer targetas
- Encender un led blanco de control
- Compararlas 
- Si la tiene guardada enciende un led verde y genera una pagina web donde aparece quien esta entrando y donde vive
- Sino se enciende un led rojo
- Poder picar a un timbre
- Que el supesto conserge pueda abrir desde dentro

## CODIGO
```
#include <Arduino.h>
// Lector de targetas
#include <SPI.h>
#include <MFRC522.h>
#define RST_PIN 17 
#define SS_PIN 4
 
// Wifi
#include <WiFi.h>
const char* ssid = "iPhone de Sonia";
const char* password = "sonia123";

// Pagina web
#include <WebServer.h>
WebServer server(80);
 
// Zumbador
#include <EasyBuzzer.h>
 
// Variables
int ledblanc = 12;
int ledverd = 13;
int ledvermell = 14;
bool correcte = true;
int zumbador = 2;
MFRC522 mfrc522(SS_PIN, RST_PIN);
 
// Funciones
void PICAR (); // Picar al timbre
void PASA (); // Esta sera la funcion que "habrira la puerta"
void COMPARAR();
void DENEGADO ();
void handle_root (void); // Pagina web

void setup() {
  Serial.begin(115200); //Iniciamos la comunicación serial
  SPI.begin(); //Iniciamos el Bus SPI
  mfrc522.PCD_Init(); // Iniciamos el MFRC522
  Serial.println("Lectura del UID");
  // Leds
  pinMode(ledblanc, OUTPUT);
  pinMode(ledverd, OUTPUT);
  pinMode(ledvermell, OUTPUT);
  pinMode (21, INPUT_PULLUP); // boton para abrir la puerta
  pinMode (15, INPUT_PULLUP); // timbre
  pinMode (2, OUTPUT); // zumbador
  // Zumbador
  EasyBuzzer.setPin(zumbador);
  // Wifi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print(".");
  }
  Serial.print("Got IP: ");
  Serial.println(WiFi.localIP()); //Show ESP32 IP on serial
  // Pagina web
  server.begin();
  server.on("/", handle_root);
}

byte Usuario1[4]= {0x50, 0x60, 0x13, 0x4E} ; //código del usuario 1

void loop() {
  int timbre = digitalRead (15);
  int boto = digitalRead (21);
  // Comprobamos si los botones estan presionados y llamamos a sus respectivas funciones
  if (timbre==LOW){
     PICAR();
  }
  if (boto==LOW){
     PASA();
  }
  // Revisamos si hay nuevas tarjetas
  if ( mfrc522.PICC_IsNewCardPresent())
  {
      //Seleccionamos la tarjeta
      if ( mfrc522.PICC_ReadCardSerial())
      {
        // Encendemos el led de control (blanco)
        digitalWrite(ledblanc, HIGH);
        delay(500);
        digitalWrite(ledblanc, LOW);

        COMPARAR();
        if (correcte){
          PASA();
        }
        else {
          DENEGADO();
        }
      }
  }
  
}
// Esta funcion simula un timbre
void PICAR (){
  EasyBuzzer.beep (1000, 900000); // frequencia en Hz, duracion del pitido en ms
  EasyBuzzer.stopBeep();
}
// Esta función encendera el led verde y la pagina web
void PASA (){
  // encendemos led verde
  digitalWrite(ledverd, HIGH);
  delay(500);               // el deixem ences un moment
  digitalWrite(ledverd, LOW);
  Serial.println("Targeta ACEPTADA");
  server.handleClient(); // pag web
}
void COMPARAR(){
  correcte = true;
  for (byte i = 0; i < mfrc522.uid.size && correcte==true; i++) {
  // comparamos la targeta leida con la de ejemplo
    if (mfrc522.uid.uidByte[i] != Usuario1[i]){
        correcte = false;
    }
  }
}

void DENEGADO (){
// encendemos led roja
  digitalWrite(ledvermell, HIGH);
  delay(500);
  digitalWrite(ledvermell, LOW);
  Serial.println();
  // Terminamos la lectura de la tarjeta actual
  mfrc522.PICC_HaltA();
 Serial.println("Targeta DENEGADA");
}

String HTML ="<!DOCTYPE html>\
<html>\
  <head>\
  <meta charset='utf-8' />\
  <title>PORTAL </title>\
  </head>\
  <body>\
  <center>\
  <h2> <strong> ENTRADA PORTAL EDIFICIO E </strong> </h2>\
  <p> <strong>Acaba de entrar por la porteria el Usuario1</strong> </p>\
  <p> Propietario del 2º 1ª </p>\
  <br>\
  </center>\
  </body>\
</html>";

void handle_root(){
server.send(200, "text/html", HTML);
}
```
## EXPLICACIÓN

Primero de todo hemos tenido que declarar las librerias correspondientes a cada una de las funcionalidades del proyecto. Algunas nos daban error y las hemos tenido que añadir con el PlatformIO.
```
#include <Arduino.h>
// Lector de targetas
#include <SPI.h>
#include <MFRC522.h>
#define RST_PIN 17 
#define SS_PIN 4
 
// Wifi
#include <WiFi.h>
const char* ssid = "iPhone de Sonia";
const char* password = "sonia123";

// Pagina web
#include <WebServer.h>
WebServer server(80);
 
// Zumbador
#include <EasyBuzzer.h>
```
A continuación hemos declarado las variables que posteriormente usaremos en el programa
```
int ledblanc = 12;
int ledverd = 13;
int ledvermell = 14;
bool correcte = true;
int zumbador = 2;
MFRC522 mfrc522(SS_PIN, RST_PIN);
```
Y también hemos declarado las cabeceras de las funciones para que el codigo las reconozca. Al haberlas programado al final, si nos saltaramos este paso daría error en todas las llamadas a funciones
```
void PICAR (); // Picar al timbre
void PASA (); // Esta sera la funcion que "habrira la puerta"
void COMPARAR();
void DENEGADO ();
void handle_root (void); // Pagina web
```
Ahora ya dentro del setup iniciamos la comunicación serial, el Bus SPI, el MFRC522, todos los pines (leds y botones) y el zumbador
```
Serial.begin(115200); //Iniciamos la comunicación serial
SPI.begin(); //Iniciamos el Bus SPI
mfrc522.PCD_Init(); // Iniciamos el MFRC522
Serial.println("Lectura del UID");
pinMode(ledblanc, OUTPUT);
pinMode(ledverd, OUTPUT);
pinMode(ledvermell, OUTPUT);
pinMode (21, INPUT_PULLUP); // boton para abrir la puerta
pinMode (15, INPUT_PULLUP); // timbre
pinMode (2, OUTPUT); // zumbador
EasyBuzzer.setPin(zumbador);
```
También conectamos el Wifi, imprimirá puntos por pantalla hasta que se conecte
```
WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
  delay(1000);
  Serial.print(".");
 }
 ```
 Mostramos la IP del ordenador para poder buscar la pagina web y ponemos los mandos necesarios para que nos funciones
 ```
Serial.print("Got IP: ");
Serial.println(WiFi.localIP()); //Show ESP32 IP on serial
server.begin();
server.on("/", handle_root);
```
Una vez ya fuera del setup creamos la variable Usuario1 para mas tarde poder reconocerlo
```
byte Usuario1[4]= {0x50, 0x60, 0x13, 0x4E} ; //código del usuario 1
```
Ahora ya pasamos al loop
Creamos dos variables para leer los botones
```
int timbre = digitalRead (15);
int boto = digitalRead (21);
```
Y con un if miramos si se han pulsado o no, si es que si llamamos a las respectivas funciones ( las definiremos mas adelante)
```
if (timbre==LOW){
 PICAR();
}
if (boto==LOW){
 PASA();
}
```
Revisamos si hay nuevas tarjetas y si hay encendemos el led de control (blanco) y procedemos a compararlas
```
if ( mfrc522.PICC_IsNewCardPresent())
{
 //Seleccionamos la tarjeta
 if ( mfrc522.PICC_ReadCardSerial())
 {
 // Encendemos el led de control (blanco)
 digitalWrite(ledblanc, HIGH);
 delay(500);
 digitalWrite(ledblanc, LOW);

 COMPARAR();
```
Segun el booleano que nos haya dado llamaremos a la funcion que abre la puerta o a la que no
```
if (correcte){
  PASA();
}
else {
  DENEGADO();
}
```
Cerramos el loop y pasamos a las funciones
Cuando antes hemos picado al timbre estabamos activando esta función, que es la que se encarga de encender el zumbador
```
void PICAR (){
  EasyBuzzer.beep (1000, 900000); // frequencia en Hz, duracion del pitido en ms
  EasyBuzzer.stopBeep();
}
```
La función pasa es la que enciende el led verde, saca por pantalla "Targeta ACEPTADA" y abre la pagina web (el codigo html esta despues de las funciones)
*Podemos ver el monitor del visual studio en las imagenes adjuntas
```
void PASA (){
  // encendemos led verde
  digitalWrite(ledverd, HIGH);
  delay(500);               // el deixem ences un moment
  digitalWrite(ledverd, LOW);
  Serial.println("Targeta ACEPTADA");
  server.handleClient(); // pag web
}
```
Esta simplemente compara la targeta leída con la que hemos guardado anteriormente numero a numero, si algun numero no coincide nos cambia el boolano CORRECTE de valor
```
void COMPARAR(){
  correcte = true;
  for (byte i = 0; i < mfrc522.uid.size && correcte==true; i++) {
  // comparamos la targeta leida con la de ejemplo
    if (mfrc522.uid.uidByte[i] != Usuario1[i]){
        correcte = false;
    }
  }
}
```
La función denegado enciende el led rojo y saca por pantalla "Targeta DENEGADA"
```
void DENEGADO (){
// encendemos led roja
  digitalWrite(ledvermell, HIGH);
  delay(500);
  digitalWrite(ledvermell, LOW);
  Serial.println();
  // Terminamos la lectura de la tarjeta actual
  mfrc522.PICC_HaltA();
 Serial.println("Targeta DENEGADA");
}
```
Y finalmente tenemos el codigo html y la función handle_root. 
*Podemos ver la pagina creada en las imagenes adjuntas
```
String HTML ="<!DOCTYPE html>\
<html>\
  <head>\
  <meta charset='utf-8' />\
  <title>PORTAL </title>\
  </head>\
  <body>\
  <center>\
  <h2> <strong> ENTRADA PORTAL EDIFICIO E </strong> </h2>\
  <p> <strong>Acaba de entrar por la porteria el Usuario1</strong> </p>\
  <p> Propietario del 2º 1ª </p>\
  <br>\
  </center>\
  </body>\
</html>";

void handle_root(){
server.send(200, "text/html", HTML);
}
```
