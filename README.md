// Código en C++ 
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RTClib.h>

RTC_DS1307 rtc;
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Dirección I2C y tamaño de la pantalla LCD
int sensorPin1 = 2;  // Pin digital utilizado para leer la salida del Sensor PIR 1
int sensorPin2 = 3;  // Pin digital utilizado para leer la salida del Sensor PIR 2
int sensorPinFC28 = A0;  // Pin analógico utilizado para leer la salida del sensor FC-28
int relayPin1 = 5;   
int relayPin2 = 6;   
int relayPinFC28 = 7;   
int relayPin4 = 8;   

bool motionDetected = false;  // Variable para controlar la detección de movimiento

void setup() {
  Wire.begin();
  rtc.begin();
  lcd.begin(16, 2);
  lcd.print("Reloj RTC");

  // Establecer el reloj RTC a las 07:50
  rtc.adjust(DateTime(now.year(), now.month(), now.day(), 7, 50, 0));

  pinMode(sensorPin1, INPUT);
  pinMode(sensorPin2, INPUT);
  pinMode(sensorPinFC28, INPUT);
  pinMode(relayPin1, OUTPUT);
  pinMode(relayPin2, OUTPUT);
  pinMode(relayPinFC28, OUTPUT);
  pinMode(relayPin4, OUTPUT);
  digitalWrite(relayPin1, LOW);    // Apagar el Relé 1 al inicio
  digitalWrite(relayPin2, LOW);    // Apagar el Relé 2 al inicio
  digitalWrite(relayPinFC28, LOW); // Apagar el Relé 3 al inicio
  digitalWrite(relayPin4, LOW);    // Apagar el Relé 4 al inicio
}

void loop() {
  DateTime now = rtc.now();
  lcd.setCursor(0, 1);
  lcd.print(now.hour(), DEC);
  lcd.print(":");
  lcd.print(now.minute(), DEC);
  lcd.print(":");
  lcd.print(now.second(), DEC);

  int sensorValue1 = digitalRead(sensorPin1);
  int sensorValue2 = digitalRead(sensorPin2);
  int sensorValueFC28 = analogRead(sensorPinFC28);
  int humidityPercentage = map(sensorValueFC28, 0, 1023, 0, 100); 

  if (sensorValue1 == HIGH || sensorValue2 == HIGH) {
    motionDetected = true;  
  }

  if (motionDetected) {
    digitalWrite(relayPin1, HIGH); 
    digitalWrite(relayPin2, HIGH);  
  } else {
    digitalWrite(relayPin1, LOW);  
    digitalWrite(relayPin2, LOW); 
  }

  if (now.hour() >= 16 && now.hour() < 22) {
    if (motionDetected) {
      digitalWrite(relayPin4, HIGH);  
    } else {
      digitalWrite(relayPin4, LOW);  
    }
  } else {
    digitalWrite(relayPin4, LOW);   
  }

  motionDetected = false;  // Reiniciar la detección de movimiento

  delay(1000);
}
