## CODIGO FINAL

```Arduino
/**
 * PROYECTO: Casa Domotizada | PROYECTO DE VAQUEROS ORGA 2025
 * Este es el cerebro de nuestra casa inteligente.
 * Se encarga de controlar luces, ventilador, puerta y ejecutar escenas automáticas.
 * Lo bueno: guarda todo en memoria (EEPROM) así que no se olvida ni después de un apagón.
 * NOTA: Ahora el ventilador usa un relay, no PWM. El motor DC tiene su propia batería.
 */

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>
#include <EEPROM.h>

// ********** DEFINIENDO LOS PINES (nuestros "cables de control") **********
const int PIN_SALA = 2;
const int PIN_COMEDOR = 3;
const int PIN_COCINA = 4;
const int PIN_BANO = 5;
const int PIN_HAB = 6;
const int PIN_BOTON = 7;    // El botón físico para la puerta
const int PIN_FAN = 9;      // Ahora va a un RELAY, no a PWM
const int PIN_SERVO = 10;   // El servomotor que mueve la puerta

// ********** VARIABLES PARA EL BOTÓN (para evitar que haga cosas raras con el rebote) **********
int estadoBotonActual;
int ultimoEstadoBoton = LOW;
unsigned long ultimoTiempoRebote = 0;
unsigned long delayRebote = 50;  // Esperamos 50ms para estar seguros de que el botón está realmente presionado

// ********** NUESTROS "ASISTENTES" (objetos que hacen el trabajo pesado) **********
LiquidCrystal_I2C lcd(0x27, 16, 2);  // La pantallita LCD con conexión I2C
Servo puertaServo;                    // El objeto que controla el servo de la puerta

// ********** VARIABLES GLOBALES (el "estado de ánimo" de la casa) **********
String comandoSerial = "";           // Lo que recibimos por el monitor serial
int estadoFan = 0;                   // 0 = APAGADO | 1, 2 y 3 = ENCENDIDO (bajo, medio, alto)
bool puertaAbierta = false;          // ¿La puerta está abierta? true = sí, false = no
String nombreEscenaActual = "Manual"; // Qué estamos haciendo ahora: "Manual" o el nombre de una escena
bool ejecutandoEscena = false;       // ¿Está corriendo una escena automática?
String firmwareVersion = "Casa Dic 2025 - Relay";  // Para saber qué versión estamos usando

// ********** MAPA DE LA MEMORIA EEPROM (como un archivador de 1000 cajones) **********
// Los primeros cajones (0-10) guardan el estado general del sistema
const int DIR_FAN_STATE = 0;         // Guarda 0-3 (los estados del ventilador)
const int DIR_DOOR_STATE = 1;        // Guarda si la puerta está abierta (1) o cerrada (0)
const int DIR_LAST_SCENE_ACTIVE = 2; // ¿Había una escena activa antes de apagarse? 1=sí, 0=no

// Cajones 11-30: El nombre de la última escena activa (para retomar donde íbamos)
const int DIR_LAST_SCENE_NAME = 11;

// Cajón 50: Cuántas escenas tenemos guardadas
const int DIR_NUM_ESCENAS = 50;

// Cajones 100 en adelante: Aquí guardamos las escenas completas
// Tenemos espacio para 3 escenas máximo, cada una con su propio espacio
const int MAX_ESCENAS = 3;
const int BYTES_POR_ESCENA = 250;    // Cada escena ocupa hasta 250 bytes (50 pasos × 5 bytes)
const int DIR_BASE_ESCENAS = 100;    // Aquí empieza el primer espacio para escenas

// ********** ESTRUCTURA DE UN "PASO" DE ESCENA (como una instrucción en una receta) **********
struct PasoEscena {
  byte pin;               // Qué luz o dispositivo (1 byte)
  bool estado;            // Encendido o apagado (1 byte)
  unsigned int duracion;  // Cuánto tiempo dura este paso, en milisegundos (2 bytes)
  byte repeticiones;      // Cuántas veces repetir este paso (1 byte)
};

// ********** BUFFER PARA TRABAJAR CON ESCENAS (nuestra "mesa de trabajo") **********
PasoEscena bufferEscena[50];  // Aquí cargamos la escena que vamos a ejecutar o guardar
int pasosBufferCount = 0;     // Cuántos pasos tenemos actualmente en el buffer

// ********** VARIABLES PARA CONTROLAR EL TIEMPO (sin usar delay, para no "congelar" el sistema) **********
int pasoIndex = 0;               // Qué paso de la escena estamos ejecutando
unsigned long tiempoInicioPaso = 0;  // Cuándo empezó el paso actual
int repeticionCount = 0;         // Cuántas veces hemos repetido el paso actual
bool modoCarga = false;          // ¿Estamos recibiendo un archivo .org? true = sí

// ********** SETUP (lo que hace la casa al despertarse) **********
void setup() {
  Serial.begin(9600);  // Abrimos la comunicación con la computadora

  // Configuramos todos los pines como salidas (excepto el botón)
  pinMode(PIN_SALA, OUTPUT);
  pinMode(PIN_COMEDOR, OUTPUT);
  pinMode(PIN_COCINA, OUTPUT);
  pinMode(PIN_BANO, OUTPUT);
  pinMode(PIN_HAB, OUTPUT);
  pinMode(PIN_FAN, OUTPUT);  // ¡Importante! Ahora es un relay, pero sigue siendo salida
  pinMode(PIN_BOTON, INPUT); // El botón es entrada
  
  // Inicializamos los periféricos (les damos la mano para que empiecen a trabajar)
  puertaServo.attach(PIN_SERVO);  // "Hola servo, tú controlarás la puerta"
  lcd.init();                     // "Hola LCD, prepárate"
  lcd.backlight();                // Encendemos la luz de fondo de la LCD

  // Mostramos un mensaje de bienvenida en la pantalla
  lcd.setCursor(0, 0);
  lcd.print("Iniciando...");
  delay(1000);  // Una pausa dramática para que se lea el mensaje

  // Recuperamos cómo estaba todo antes de apagarnos (¡la casa tiene memoria!)
  recuperarEstadoSistema();

  // Listo para recibir órdenes
  Serial.println("--- SISTEMA LISTO (Modo Relay) ---");
  Serial.println("Escribe HELP para ver comandos.");
}

// ********** LOOP PRINCIPAL (lo que hace la casa todo el tiempo) **********
void loop() {
  // Primero, revisamos si alguien presionó el botón físico
  leerBotonFisico();
  
  // 1. ¿Nos están enviando comandos por el puerto serial?
  if (Serial.available() > 0) {
    String lectura = Serial.readStringUntil('\n');
    lectura.trim();
    if (lectura.length() > 0) {
      if (modoCarga) {
        // Si estamos en modo carga, procesamos líneas de archivo .org
        procesarLineaArchivoOrg(lectura);
      } else {
        // Si no, procesamos comandos normales
        lectura.toUpperCase();  // Convertimos a mayúsculas para evitar "l1" vs "L1"
        procesarComando(lectura);
      }
    }
  }

  // 2. Si hay una escena en ejecución, avanzamos un paso (si es tiempo)
  if (ejecutandoEscena) {
    manejarLogicaEscena();
  }
}

// ********** FUNCIÓN PARA LEER EL BOTÓN FÍSICO (con protección contra rebotes) **********
void leerBotonFisico() {
  int lectura = digitalRead(PIN_BOTON);

  // Si detectamos un cambio (posible rebote), reiniciamos el temporizador
  if (lectura != ultimoEstadoBoton) {
    ultimoTiempoRebote = millis();
  }

  // Esperamos un poco para asegurarnos de que no sea solo ruido
  if ((millis() - ultimoTiempoRebote) > delayRebote) {
    // Si la lectura es estable y diferente a la anterior
    if (lectura != estadoBotonActual) {
      estadoBotonActual = lectura;

      // Solo actuamos cuando se PRESIONA el botón (HIGH)
      if (estadoBotonActual == HIGH) {
        Serial.println("--> Boton: Alternando Puerta");
        togglePuerta();  // Abrimos o cerramos la puerta, y actualizamos todo
      }
    }
  }

  // Guardamos el último estado para la próxima vez
  ultimoEstadoBoton = lectura;
}

// ********** PROCESAMIENTO DE COMANDOS (el "traductor" de lo que escribimos) **********
void procesarComando(String cmd) {
  // --- Ayuda: Mostramos todos los comandos disponibles ---
  if (cmd == "HELP") {
    imprimirAyuda();
  }
  // --- Control Manual de Luces: Encendemos/apagamos ambientes específicos ---
  else if (cmd == "L1" || cmd == "L1ON")
    setLuz(PIN_SALA, true, "SALA");
  else if (cmd == "L1OFF") setLuz(PIN_SALA, false, "SALA");
  else if (cmd == "L2" || cmd == "L2ON") setLuz(PIN_COMEDOR, true, "COMEDOR");
  else if (cmd == "L2OFF") setLuz(PIN_COMEDOR, false, "COMEDOR");
  else if (cmd == "L3" || cmd == "L3ON") setLuz(PIN_COCINA, true, "COCINA");
  else if (cmd == "L3OFF") setLuz(PIN_COCINA, false, "COCINA");
  else if (cmd == "L4" || cmd == "L4ON") setLuz(PIN_BANO, true, "BANO");
  else if (cmd == "L4OFF") setLuz(PIN_BANO, false, "BANO");
  else if (cmd == "L5" || cmd == "L5ON") setLuz(PIN_HAB, true, "HAB");
  else if (cmd == "L5OFF") setLuz(PIN_HAB, false, "HAB");
  else if (cmd == "ALLON") {
    toggleTodas(true);
    Serial.println("TODAS: ON");
  } else if (cmd == "ALLOFF") {
    toggleTodas(false);
    Serial.println("TODAS: OFF");
  }

  // --- Control del Ventilador (Modo Relay): 0=apagado, 1=bajo, 2=medio, 3=alto ---
  else if (cmd == "FAN0")
    setVentilador(0);
  else if (cmd == "FAN1") setVentilador(1);
  else if (cmd == "FAN2") setVentilador(2);
  else if (cmd == "FAN3") setVentilador(3);

  // --- Control de la Puerta: Abrir, cerrar o alternar ---
  else if (cmd == "DOOR") togglePuerta();
  else if (cmd == "DOOROPEN") setPuerta(true);
  else if (cmd == "DOORCLOSE") setPuerta(false);

  // --- Sistema de Escenas: Cargar, listar, borrar o ejecutar ---
  else if (cmd == "LOAD_SCENE") {
    iniciarModoCarga();
  } else if (cmd == "STOP") {
    detenerEscena();
  } else if (cmd == "LIST_SCENES") {
    listarEscenasGuardadas();
  } else if (cmd == "ERASE_SCENES") {
    borrarEEPROM();
  } else if (cmd == "STATUS") {
    mostrarStatus();
  }
  // --- Si no es un comando conocido, quizás sea el nombre de una escena guardada ---
  else {
    // Buscamos en la memoria si existe una escena con ese nombre
    if (intentarCargarEscenaDeEEPROM(cmd)) {
      // ¡Encontrada! La función ya se encarga de activarla
    } else {
      // Si no la encontramos, avisamos que no entendimos el comando
      Serial.println("COMANDO DESCONOCIDO O ESCENA NO ENCONTRADA.");
    }
  }
}

// ********** GESTIÓN DE ESCENAS (el "director de orquesta" de las secuencias) **********

void manejarLogicaEscena() {
  // Primero, chequeamos si hay pasos para ejecutar
  if (pasosBufferCount == 0) {
    detenerEscena();  // Si no hay pasos, paramos la escena
    return;
  }

  unsigned long tiempoActual = millis();

  // ¿Ya pasó el tiempo que debía durar este paso?
  if (tiempoActual - tiempoInicioPaso >= bufferEscena[pasoIndex].duracion) {
    // ¡Sí! Pasamos al siguiente paso
    
    pasoIndex++;  // Siguiente instrucción en la receta

    // Si llegamos al final de los pasos...
    if (pasoIndex >= pasosBufferCount) {
      // ...volvemos al inicio (las escenas se repiten en bucle)
      pasoIndex = 0;
    }

    // Ejecutamos el nuevo paso inmediatamente
    ejecutarPasoActual();
  }
}

void ejecutarPasoActual() {
  // Tomamos el paso actual del buffer
  PasoEscena p = bufferEscena[pasoIndex];
  
  // Aplicamos la acción: encendemos o apagamos el pin correspondiente
  digitalWrite(p.pin, p.estado ? HIGH : LOW);
  
  // Marcamos el momento en que empezó este paso
  tiempoInicioPaso = millis();
}

void detenerEscena() {
  // 1. Cambiamos los estados lógicos (ya no hay escena ejecutándose)
  ejecutandoEscena = false;
  nombreEscenaActual = "Manual";
  
  // 2. Actualizamos la EEPROM para que sepa que no hay escena activa
  EEPROM.update(DIR_LAST_SCENE_ACTIVE, 0); 

  // 3. Avisamos por serial (útil para depurar)
  Serial.println("--- DETENIENDO ESCENA ---");
  Serial.println("Modo actual: Manual");

  // 4. FINALMENTE actualizamos la pantalla LCD
  actualizarLCD();
}

// ********** CARGA DE ARCHIVOS .ORG (cuando le enviamos una "receta" completa) **********

void iniciarModoCarga() {
  // Activamos el "modo recepción" para archivos .org
  modoCarga = true;
  pasosBufferCount = 0;  // Limpiamos el buffer para empezar de cero
  Serial.println(">>> MODO CARGA ACTIVADO <<<");
  Serial.println("Envie lineas formato: AMBIENTE:ESTADO:DURACION:REPETICIONES");
  Serial.println("Ejemplo: SALA:ON:500:1");
  Serial.println("Finalice con: END_LOAD NOMBRE_ESCENA");
  
  // También mostramos en la LCD que estamos en modo carga
  lcd.clear();
  lcd.print("Modo Carga...");
}

void procesarLineaArchivoOrg(String linea) {
  // Primero: ¿Es la línea que indica el fin del archivo?
  if (linea.startsWith("END_LOAD")) {
    String nombre = linea.substring(8);
    nombre.trim();
    nombre.toUpperCase();  // Normalizamos a mayúsculas
    
    // Verificamos que tengamos un nombre y al menos un paso
    if (nombre.length() > 0 && pasosBufferCount > 0) {
      guardarEscenaEnEEPROM(nombre);  // ¡Guardamos en la memoria permanente!
    } else {
      Serial.println("ERROR: Nombre vacio o sin pasos.");
    }
    
    // Salimos del modo carga
    modoCarga = false;
    actualizarLCD();  // Volvemos a mostrar el estado normal
    return;
  }

  // Si no es el final, procesamos una línea de instrucción
  // Formato: AMBIENTE:ESTADO:DURACION:REPETICIONES
  // Ejemplo: SALA:ON:500:20
  
  // Buscamos los separadores (los dos puntos)
  int p1 = linea.indexOf(':');
  int p2 = linea.indexOf(':', p1 + 1);
  int p3 = linea.indexOf(':', p2 + 1);

  // Si encontramos todos los separadores y tenemos espacio en el buffer...
  if (p1 > 0 && p2 > 0 && p3 > 0 && pasosBufferCount < 50) {
    // Extraemos cada parte de la línea
    String amb = linea.substring(0, p1);
    String est = linea.substring(p1 + 1, p2);
    String dur = linea.substring(p2 + 1, p3);
    String rep = linea.substring(p3 + 1);

    // Convertimos el nombre del ambiente a un número de pin
    int pin = obtenerPinDeNombre(amb);
    if (pin != -1) {
      // ¡Todo bien! Guardamos el paso en el buffer
      bufferEscena[pasosBufferCount].pin = pin;
      bufferEscena[pasosBufferCount].estado = (est == "ON");
      bufferEscena[pasosBufferCount].duracion = dur.toInt();
      bufferEscena[pasosBufferCount].repeticiones = rep.toInt();
      pasosBufferCount++;  // Un paso más en nuestro buffer
      Serial.println("Paso OK: " + amb);
    } else {
      Serial.println("Error: Ambiente desconocido (" + amb + ")");
    }
  } else {
    // Ignoramos comentarios (líneas que empiezan con #) o líneas vacías
    if (!linea.startsWith("#") && linea.length() > 0) {
      Serial.println("Error de formato en linea: " + linea);
    }
  }
}

// ********** MEMORIA EEPROM (la "memoria a largo plazo" de la casa) **********

void guardarEscenaEnEEPROM(String nombre) {
  // Primero, vemos cuántas escenas ya tenemos guardadas
  int numEscenas = EEPROM.read(DIR_NUM_ESCENAS);
  if (numEscenas == 255) numEscenas = 0;  // Si es 255, significa que la EEPROM está virgen

  int slotIndex = -1;  // ¿En qué "cajón" guardaremos esta escena?

  // 1. ¿Ya existe una escena con este nombre? Si sí, la sobrescribimos
  for (int i = 0; i < numEscenas; i++) {
    String n = leerNombreEscena(i);
    if (n == nombre) {
      slotIndex = i;
      break;  // ¡Encontrada! Usaremos este slot
    }
  }

  // 2. Si no existe y tenemos espacio, creamos un nuevo slot
  if (slotIndex == -1) {
    if (numEscenas < MAX_ESCENAS) {
      slotIndex = numEscenas;  // El próximo slot disponible
      numEscenas++;  // Ahora tenemos una escena más
      EEPROM.update(DIR_NUM_ESCENAS, numEscenas);
    } else {
      // ¡Oh no! No hay más espacio
      Serial.println("MEMORIA LLENA (Max 3 escenas). Borre usando ERASE_SCENES.");
      return;
    }
  }

  // ¡Perfecto! Ahora guardamos la escena en su slot correspondiente
  int direccionBase = DIR_BASE_ESCENAS + (slotIndex * BYTES_POR_ESCENA);

  // Guardamos el nombre (primeros 10 bytes del slot)
  for (int i = 0; i < 10; i++) {
    if (i < nombre.length()) EEPROM.update(direccionBase + i, nombre[i]);
    else EEPROM.update(direccionBase + i, 0);  // Rellenamos con ceros si el nombre es corto
  }

  // Guardamos cuántos pasos tiene la escena (byte 11 del slot)
  EEPROM.update(direccionBase + 10, pasosBufferCount);

  // Guardamos todos los pasos (a partir del byte 12)
  int dirDatos = direccionBase + 11;
  for (int i = 0; i < pasosBufferCount; i++) {
    EEPROM.update(dirDatos++, bufferEscena[i].pin);
    EEPROM.update(dirDatos++, bufferEscena[i].estado);
    // La duración son 2 bytes, así que usamos EEPROM.put
    EEPROM.put(dirDatos, bufferEscena[i].duracion);
    dirDatos += 2;
    EEPROM.update(dirDatos++, bufferEscena[i].repeticiones);
  }

  Serial.println("Escena '" + nombre + "' GUARDADA en EEPROM Slot " + String(slotIndex));
}

bool intentarCargarEscenaDeEEPROM(String nombre) {
  // Primero, vemos si hay escenas guardadas
  int num = EEPROM.read(DIR_NUM_ESCENAS);
  if (num == 255) return false;  // 255 significa "no inicializado"

  // Buscamos en todas las escenas guardadas
  for (int i = 0; i < num; i++) {
    // ¿Esta escena tiene el nombre que buscamos?
    if (leerNombreEscena(i) == nombre) {
      // ¡Sí! Cargamos los datos a la RAM
      cargarSlotAMemoria(i);
      
      // Actualizamos las variables globales
      nombreEscenaActual = nombre;
      ejecutandoEscena = true;
      pasoIndex = 0; 
      tiempoInicioPaso = 0;

      // Guardamos en la EEPROM que esta escena está activa (para si se apaga la casa)
      EEPROM.update(DIR_LAST_SCENE_ACTIVE, 1);
      guardarStringEEPROM(DIR_LAST_SCENE_NAME, nombre);
      
      // Avisamos por serial
      Serial.print(">>> ESCENA ACTIVADA: ");
      Serial.println(nombre);

      // Actualizamos la pantalla LCD
      actualizarLCD();
      
      // Ejecutamos el primer paso inmediatamente (sin esperar)
      ejecutarPasoActual(); 
      return true;  // ¡Éxito!
    }
  }
  return false;  // No encontramos ninguna escena con ese nombre
}

void cargarSlotAMemoria(int slot) {
  // Calculamos dónde empieza esta escena en la EEPROM
  int direccionBase = DIR_BASE_ESCENAS + (slot * BYTES_POR_ESCENA);
  
  // Leemos cuántos pasos tiene
  pasosBufferCount = EEPROM.read(direccionBase + 10);

  // Leemos todos los pasos, uno por uno
  int dirDatos = direccionBase + 11;
  for (int i = 0; i < pasosBufferCount; i++) {
    bufferEscena[i].pin = EEPROM.read(dirDatos++);
    bufferEscena[i].estado = EEPROM.read(dirDatos++);
    EEPROM.get(dirDatos, bufferEscena[i].duracion);
    dirDatos += 2;
    bufferEscena[i].repeticiones = EEPROM.read(dirDatos++);
  }
}

String leerNombreEscena(int slot) {
  // Calculamos dónde está guardado el nombre de esta escena
  int direccionBase = DIR_BASE_ESCENAS + (slot * BYTES_POR_ESCENA);
  String nombre = "";
  
  // Leemos caracter por caracter hasta encontrar un cero o llegar a 10 caracteres
  for (int i = 0; i < 10; i++) {
    char c = EEPROM.read(direccionBase + i);
    if (c == 0) break;  // El cero marca el final del nombre
    nombre += c;
  }
  return nombre;
}

void recuperarEstadoSistema() {
  // 1. Ventilador: ¿cómo estaba antes de apagarse?
  estadoFan = EEPROM.read(DIR_FAN_STATE);
  if (estadoFan > 3) estadoFan = 0;  // Si es un valor inválido, lo ponemos en 0 (apagado)
  aplicarMotor();  // Aplicamos el estado al relay

  // 2. Puerta: ¿abierta o cerrada?
  puertaAbierta = (EEPROM.read(DIR_DOOR_STATE) == 1);
  puertaServo.write(puertaAbierta ? 90 : 0);  // 90° = abierta, 0° = cerrada

  // 3. ¿Había una escena activa antes de apagarse?
  if (EEPROM.read(DIR_LAST_SCENE_ACTIVE) == 1) {
    // Leemos el nombre de la última escena activa
    String lastScene = "";
    for (int i = 0; i < 10; i++) {
      char c = EEPROM.read(DIR_LAST_SCENE_NAME + i);
      if (c != 0) lastScene += c;
    }
    Serial.println("Recuperando escena: " + lastScene);
    
    // Intentamos reactivar esa escena
    intentarCargarEscenaDeEEPROM(lastScene);
  } else {
    // Si no había escena activa, solo actualizamos la LCD
    actualizarLCD();
  }
}

void borrarEEPROM() {
  // ¡CUIDADO! Esto borra TODO lo guardado en la EEPROM
  for (int i = 0; i < EEPROM.length(); i++) {
    EEPROM.update(i, 255);  // 255 es el valor "de fábrica" (no inicializado)
  }
  
  // Reiniciamos los contadores importantes
  EEPROM.update(DIR_NUM_ESCENAS, 0);
  EEPROM.update(DIR_LAST_SCENE_ACTIVE, 0);
  Serial.println("EEPROM BORRADA COMPLETAMENTE.");
}

// ********** FUNCIONES AUXILIARES (nuestras "herramientas" de trabajo) **********

void setLuz(int pin, bool estado, String nombre) {
  // Cualquier comando manual detiene la escena automática
  detenerEscena();
  
  // Encendemos o apagamos la luz
  digitalWrite(pin, estado ? HIGH : LOW);
  
  // Avisamos por serial qué hicimos
  Serial.println(nombre + ":" + (estado ? "ON" : "OFF"));
}

void toggleTodas(bool estado) {
  // Detenemos cualquier escena primero
  detenerEscena();
  
  // Encendemos o apagamos TODAS las luces a la vez
  digitalWrite(PIN_SALA, estado);
  digitalWrite(PIN_COMEDOR, estado);
  digitalWrite(PIN_COCINA, estado);
  digitalWrite(PIN_BANO, estado);
  digitalWrite(PIN_HAB, estado);
}

// ********** FUNCIONES DE CONTROL DEL RELAY DEL VENTILADOR **********
void setVentilador(int nuevoEstado) {
  // Validamos que el estado esté entre 0 y 3
  if (nuevoEstado < 0 || nuevoEstado > 3) return;
  
  // Actualizamos el estado en memoria
  estadoFan = nuevoEstado;
  EEPROM.update(DIR_FAN_STATE, estadoFan);  // ¡Guardamos inmediatamente en EEPROM!
  aplicarMotor();  // Aplicamos el cambio al relay
  
  Serial.print("FAN");
  Serial.print(estadoFan);
  Serial.println(" guardado en EEPROM");
  
  actualizarLCD();  // Mostramos el cambio en la pantalla
}

void aplicarMotor() {
  // IMPORTANTE: HIGH = relay APAGADO, LOW = relay ENCENDIDO
  // (Depende de cómo esté conectado el relay)
  switch (estadoFan) {
    case 0:  // FAN0 - APAGADO
      digitalWrite(PIN_FAN, HIGH);  // Relay desactivado
      Serial.println("MOTOR: APAGADO");
      break;
    case 1:  // FAN1 - BAJO (Relay encendido)
      digitalWrite(PIN_FAN, LOW);
      Serial.println("MOTOR: ENCENDIDO (BAJO)");
      break;
    case 2:  // FAN2 - MEDIO (Relay encendido)
      digitalWrite(PIN_FAN, LOW);
      Serial.println("MOTOR: ENCENDIDO (MEDIO)");
      break;
    case 3:  // FAN3 - ALTO (Relay encendido)
      digitalWrite(PIN_FAN, LOW);
      Serial.println("MOTOR: ENCENDIDO (ALTO)");
      break;
  }
}

void setPuerta(bool abierta) {
  // Actualizamos el estado de la puerta
  puertaAbierta = abierta;
  
  // Movemos el servo a la posición correspondiente
  puertaServo.write(abierta ? 90 : 0);
  
  // Guardamos el estado en la EEPROM
  EEPROM.update(DIR_DOOR_STATE, abierta ? 1 : 0);
  
  Serial.println(abierta ? "PUERTA: ABIERTA" : "PUERTA: CERRADA");
  
  actualizarLCD();  // Mostramos el cambio en la pantalla
}

void togglePuerta() {
  // Alterna entre abierta y cerrada
  setPuerta(!puertaAbierta);
}

int obtenerPinDeNombre(String nombre) {
  // Convierte un nombre de ambiente en su número de pin correspondiente
  nombre.trim();
  if (nombre == "SALA") return PIN_SALA;
  if (nombre == "COMEDOR") return PIN_COMEDOR;
  if (nombre == "COCINA") return PIN_COCINA;
  if (nombre == "BANO" || nombre == "BAÑO") return PIN_BANO;
  if (nombre == "HABITACION" || nombre == "HAB") return PIN_HAB;
  return -1;  // -1 significa "no encontrado"
}

void actualizarLCD() {
  // Limpiamos la pantalla (para evitar textos "fantasma")
  lcd.clear();
  
  // --- LINEA 1: Mostramos qué escena está activa ---
  lcd.setCursor(0, 0);
  lcd.print("Escena:");
  lcd.print(nombreEscenaActual); 
  
  // --- LINEA 2: Mostramos estado del ventilador y puerta ---
  lcd.setCursor(0, 1);
  
  // Ventilador (F:)
  lcd.print("F:");
  if (estadoFan == 0) lcd.print("OFF");
  else if (estadoFan == 1) lcd.print("LO");
  else if (estadoFan == 2) lcd.print("MED");
  else lcd.print("HI");

  lcd.print(" ");  // Un espacio de separación

  // Puerta (P:)
  lcd.print("P:");
  lcd.print(puertaAbierta ? "ABR" : "CER");
}

void guardarStringEEPROM(int addr, String str) {
  // Guarda un string en la EEPROM, máximo 10 caracteres
  for (int i = 0; i < 10; i++) {
    if (i < str.length()) EEPROM.update(addr + i, str[i]);
    else EEPROM.update(addr + i, 0);  // Rellenamos con ceros
  }
}

void listarEscenasGuardadas() {
  // Lista todas las escenas que tenemos guardadas en la EEPROM
  int n = EEPROM.read(DIR_NUM_ESCENAS);
  if (n == 255 || n == 0) {
    Serial.println("No hay escenas guardadas.");
  } else {
    Serial.println("Escenas en memoria:");
    for (int i = 0; i < n; i++) {
      Serial.print(i + 1);
      Serial.print(". ");
      Serial.println(leerNombreEscena(i));
    }
  }
}

void mostrarStatus() {
  // Muestra un reporte completo del estado del sistema
  Serial.println("--- STATUS (Modo Relay) ---");
  Serial.print("Fan Estado: ");
  Serial.println(estadoFan);
  Serial.print("Fan Estado Texto: ");
  switch(estadoFan) {
    case 0: Serial.println("APAGADO"); break;
    case 1: Serial.println("BAJO"); break;
    case 2: Serial.println("MEDIO"); break;
    case 3: Serial.println("ALTO"); break;
  }
  Serial.print("Puerta: ");
  Serial.println(puertaAbierta ? "Abierta" : "Cerrada");
  Serial.print("Escena Activa: ");
  Serial.println(nombreEscenaActual);
}

void imprimirAyuda() {
  // Muestra todos los comandos disponibles
  Serial.println("AYUDA SISTEMA DE CASA DOMOTICO");
  Serial.println("LUCES: L1, L1OFF, L2, L2OFF, L3, L3OFF, L4, L4OFF, L5, L5OFF");
  Serial.println("       ALLON, ALLOFF");
  Serial.println("VENTILADOR: FAN0 (Apagado), FAN1 (Bajo), FAN2 (Medio), FAN3 (Alto)");
  Serial.println("PUERTA: DOOR (toggle), DOOROPEN, DOORCLOSE");
  Serial.println("ESCENAS: LOAD_SCENE (para cargar), END_LOAD NOMBRE (para finalizar)");
  Serial.println("         [Nombre de escena guardada] para ejecutar (ej: CASCADA)");
  Serial.println("         STOP (detener escena), LIST_SCENES");
  Serial.println("SISTEMA: STATUS, ERASE_SCENES, HELP");
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