## CODIGO FINAL

```Arduino
/*
 PROYECTO CASA DOMOTIZADA
 Control de luces, ventilador con relay, puerta con servo y escenas.
 Se controla por Serial y guarda estados en EEPROM.
*/

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>
#include <EEPROM.h>

// Pines de luces
const int PIN_SALA = 2;
const int PIN_COMEDOR = 3;
const int PIN_COCINA = 4;
const int PIN_BANO = 5;
const int PIN_HAB = 6;

// Pines generales
const int PIN_BOTON = 7;   // Botón físico de la puerta
const int PIN_FAN = 9;     // Relay del ventilador
const int PIN_SERVO = 10;  // Servo de la puerta

// Variables para el antirrebote del botón
int estadoBotonActual = LOW;
int ultimoEstadoBoton = LOW;
unsigned long ultimoTiempoRebote = 0;
const unsigned long delayRebote = 50;

// Objetos principales
LiquidCrystal_I2C lcd(0x27, 16, 2); // LCD por I2C
Servo puertaServo;                 // Servo de la puerta

// Variables generales del sistema
int estadoFan = 0;                  // 0 apagado, >0 encendido
bool puertaAbierta = false;          // Estado actual de la puerta
bool ejecutandoEscena = false;       // Indica si hay escena corriendo
String nombreEscenaActual = "Manual";

// Direcciones usadas en EEPROM
const int DIR_FAN_STATE = 0;
const int DIR_DOOR_STATE = 1;
const int DIR_LAST_SCENE_ACTIVE = 2;
const int DIR_LAST_SCENE_NAME = 11;
const int DIR_NUM_ESCENAS = 50;

// Configuración de escenas
const int MAX_ESCENAS = 3;
const int BYTES_POR_ESCENA = 250;
const int DIR_BASE_ESCENAS = 100;

/*
 ESTRUCTURA DE UN PASO DE ESCENA
 Cada paso indica qué pin se enciende o apaga y por cuánto tiempo
*/
struct PasoEscena {
  byte pin;              // Pin que se controla
  bool estado;           // HIGH o LOW
  unsigned int duracion; // Tiempo del paso
  byte repeticiones;     // Guardado aunque no se use
};

// Buffer temporal para ejecutar escenas
PasoEscena bufferEscena[50];
int pasosBufferCount = 0;

// Variables para controlar el tiempo de escenas
int pasoIndex = 0;
unsigned long tiempoInicioPaso = 0;
bool modoCarga = false;

// SETUP
void setup() {
  Serial.begin(9600);

  // Configuramos pines de salida
  pinMode(PIN_SALA, OUTPUT);
  pinMode(PIN_COMEDOR, OUTPUT);
  pinMode(PIN_COCINA, OUTPUT);
  pinMode(PIN_BANO, OUTPUT);
  pinMode(PIN_HAB, OUTPUT);
  pinMode(PIN_FAN, OUTPUT);
  pinMode(PIN_BOTON, INPUT);

  // Inicializamos servo y LCD
  puertaServo.attach(PIN_SERVO);
  lcd.init();
  lcd.backlight();

  // Mensaje de inicio
  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");
  delay(1000);

  // Recuperamos lo guardado en EEPROM
  recuperarEstadoSistema();

  Serial.println("Sistema listo");
}

// LOOP PRINCIPAL
void loop() {
  // Revisamos el botón físico
  leerBotonFisico();

  // Revisamos si llegó un comando por serial
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    cmd.toUpperCase();

    if (modoCarga)
      procesarLineaArchivoOrg(cmd); // Estamos cargando una escena
    else
      procesarComando(cmd);         // Comando normal
  }

  // Si hay una escena activa, la ejecutamos
  if (ejecutandoEscena)
    manejarLogicaEscena();
}

// LECTURA DEL BOTÓN DE LA PUERTA
void leerBotonFisico() {
  int lectura = digitalRead(PIN_BOTON);

  // Si el estado cambia, reiniciamos el conteo
  if (lectura != ultimoEstadoBoton) {
    ultimoTiempoRebote = millis();
  }

  // Si ya pasó el tiempo de rebote, aceptamos la lectura
  if ((millis() - ultimoTiempoRebote) > delayRebote) {
    if (lectura != estadoBotonActual) {
      estadoBotonActual = lectura;

      // Si se presionó el botón, cambiamos la puerta
      if (estadoBotonActual == HIGH) {
        togglePuerta();
      }
    }
  }

  ultimoEstadoBoton = lectura;
}

// ABRIR O CERRAR LA PUERTA
void togglePuerta() {
  if (puertaAbierta) {
    puertaServo.write(0);          // Cerramos
    puertaAbierta = false;
    Serial.println("Puerta cerrada");
  } else {
    puertaServo.write(90);         // Abrimos
    puertaAbierta = true;
    Serial.println("Puerta abierta");
  }

  // Guardamos el estado en EEPROM
  EEPROM.update(DIR_DOOR_STATE, puertaAbierta);
}

// RECUPERAR ESTADOS DESDE EEPROM
void recuperarEstadoSistema() {
  estadoFan = EEPROM.read(DIR_FAN_STATE);
  puertaAbierta = EEPROM.read(DIR_DOOR_STATE);

  digitalWrite(PIN_FAN, estadoFan > 0 ? HIGH : LOW);
  puertaServo.write(puertaAbierta ? 90 : 0);
}
```

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