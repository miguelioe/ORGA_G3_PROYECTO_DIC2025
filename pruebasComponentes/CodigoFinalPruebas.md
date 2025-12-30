## CODIGO FINAL

Murcia

---

### CODIGO BASE DEL LCD I2C

Mario geidá

---

### CODIGO BASE DEL SERVOMOTOR

```Arduino
#include <Servo.h>
#include <EEPROM.h>

#define PIN_SERVO 10
#define PIN_BOTON 7
#define DIR_DOOR_STATE 0   // EEPROM

Servo puerta;
bool puertaAbierta = false;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_BOTON, INPUT);
  puerta.attach(PIN_SERVO);

  // Recuperar estado guardado
  puertaAbierta = EEPROM.read(DIR_DOOR_STATE) == 1;
  puerta.write(puertaAbierta ? 90 : 0);
}

void loop() {
  if (digitalRead(PIN_BOTON) == HIGH) {
    delay(200); // antirrebote simple
    puertaAbierta = !puertaAbierta;

    puerta.write(puertaAbierta ? 90 : 0);
    EEPROM.update(DIR_DOOR_STATE, puertaAbierta ? 1 : 0);

    Serial.println(puertaAbierta ? "Puerta abierta" : "Puerta cerrada");
  }
}
```
---

### CODIGO BASE DE LOS LED

```Arduino
#define LED_SALA 2
#define LED_COMEDOR 3

void setup() {
  Serial.begin(9600);
  pinMode(LED_SALA, OUTPUT);
  pinMode(LED_COMEDOR, OUTPUT);
}

void loop() {
  if (Serial.available()) {
    char c = Serial.read();

    if (c == '1') digitalWrite(LED_SALA, HIGH);
    if (c == '2') digitalWrite(LED_SALA, LOW);
    if (c == '3') digitalWrite(LED_COMEDOR, HIGH);
    if (c == '4') digitalWrite(LED_COMEDOR, LOW);
  }
}
```

---

### CODIGO BASE DEL MOTOR DC CON RELAY

Soko 

---

### CODIGO BASE DE EEPROM

Mario2