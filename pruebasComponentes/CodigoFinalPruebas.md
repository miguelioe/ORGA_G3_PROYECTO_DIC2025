## CODIGO FINAL

Papa Ema

---

### CODIGO BASE DEL LCD I2C

Mario geidá

---

### CODIGO BASE DEL SERVOMOTOR

Chepe

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