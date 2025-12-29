## Prueba básica de código para el servomotor

```Arduino
#include <Servo.h>

int servoPin = 2; // El servo necesita de un pin PWM
Servo miServo; // Creacion y nombre del servo
int servoAngulo = 0; // Angulo del incicio para testearlo

void setup() {
  miServo.attach(servoPin); // Carga el pin del servo
  Serial.begin(9600);
}

void loop() {
  servoAngulo = 0; //Siempre debe cargarse su posicion inicial en cero

  miServo.write(servoAngulo); // Posicion inicial de cero grados
  delay(2000);

  servoAngulo += 90; // Aumenta posicion a noventa grados
  miServo.write(servoAngulo);
  delay(2000);

  servoAngulo += 90; // Aumenta posisicion a ciento ochenta grados
  miServo.write(servoAngulo);
  delay(2000);

  servoAngulo -= 90; // Disminuye posicion a noventa grados
  miServo.write(servoAngulo);
  delay(2000);
}
```
![ServomotorConexionBasica](/capturas/ServomotorConexionBasica.png)

Link para ver y probar la simulación: https://wokwi.com/projects/450923035980365825



## Prueba de funcionamiento y conexión LCD I2C
> breve bosquejo de funciones principales para realizar el laboratorio

```Arduino
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

byte N[8] = {
  0b00001110,
  0b00000000,
  0b00010001,
  0b00011001,
  0b00010101,
  0b00010011,
  0b00010001,
  0b00000000
};


String estadoATexto(byte estado){
  if(estado == HIGH){
    return "TODO ENCENDIDO";
  }
  else{
    return "TODO APAGADO";
  }
}


String velocidadATexto(byte velocidad){
  if(velocidad == 0){
    return "OFF ";
  }
  else if(velocidad > 0 && velocidad <= 85){
    return "BAJO ";
  }
  else if(velocidad > 85 && velocidad <= 170){
    return "MEDIO ";
  }
  else{
    return "ALTO";
  }
}


String puertaATexto(byte estadoPuerta){
  if(estadoPuerta==HIGH){
    return "ABIERTA ";
  }
  else{
    return "CERRADA ";
  }
}


void imprimir(String escena, byte estadoPuerta, byte estadoVentilador){
  lcd.clear();
  lcd.setCursor(0,0);
  if(escena.equals("BAÑO")){
    lcd.print("BA");
    lcd.write(0);
    lcd.print("O");
  }else{
    lcd.print(escena);
  }
  lcd.print(" PUERTA:");
  lcd.print(puertaATexto(estadoPuerta));
  lcd.print("VENTILADOR:");
  lcd.print(velocidadATexto(estadoVentilador));
}


void imprimir(String escena, byte estadoPuerta, byte estadoVentilador, byte estadoLeds){
  lcd.clear();
  lcd.setCursor(0,0);
  if(escena.equals("BAÑO")){
    lcd.print("BA");
    lcd.write(0);
    lcd.print("O");
  }else{
    lcd.print(escena);
  }
  lcd.print(" PUERTA:");
  lcd.print(puertaATexto(estadoPuerta));
  lcd.print("VENTILADOR:");
  lcd.print(velocidadATexto(estadoVentilador));
  lcd.setCursor(0,1);
  lcd.print(estadoATexto(estadoLeds));
}


void setup() {
  // put your setup code here, to run once:
  lcd.init();
  lcd.createChar(0, N);
  lcd.backlight();
  lcd.setCursor(0,0);
  imprimir("FIESTA", HIGH, 110, HIGH);
}


void loop() {
  // put your main code here, to run repeatedly:
  delay(500);
  lcd.scrollDisplayLeft();
}
```
![Conexión basica LCD I2C](../capturas/lcdi2c.png)

[Accede a este circuito simulado! (haz clic en cualquier sitio azul)](https://wokwi.com/projects/450932545007138817)

---
## Prueba de funcionamiento del Motor DC con L293D

>Comandos para la EEPROM:  
>FAN0= Apagar  
>FAN1= Velocidad baja  
>FAN2= Velocidad media  
>FAN3= Velocidad alta  

```Arduino
#include <EEPROM.h>

#define PIN_RELE 9

// Variables 
byte velocidadApagada = 0; 
byte velocidadBaja = 85;
byte velocidadMedia = 170;
byte velocidadAlta = 255;

int estadoMotor = 0;

void setup() {
  Serial.begin(9600);
  pinMode(PIN_RELE, OUTPUT);

  estadoMotor = EEPROM.read(0);

  if (estadoMotor > 3) estadoMotor = 0;

  aplicarMotor();
}

void loop() {
  if (Serial.available()) {
    String termi = Serial.readStringUntil('\n'); //Guardar valor de terminal
    termi.trim();
    termi.toUpperCase();

    if (termi == "FAN0") cambiarEstado(0);
    else if (termi == "FAN1") cambiarEstado(1);
    else if (termi == "FAN2") cambiarEstado(2);
    else if (termi == "FAN3") cambiarEstado(3);
    else cambiarEstado(0);
  }
  delay(10);
}

void cambiarEstado(int nuevo) {
  estadoMotor = nuevo;
  EEPROM.update(0, estadoMotor);  // Guardar 
  aplicarMotor();
  
  Serial.print("FAN");
  Serial.print(estadoMotor);
  Serial.println(" guardado en EEPROM");
}

void aplicarMotor() {
  switch (estadoMotor) {
    case 0:  // FAN0
      digitalWrite(PIN_RELE, HIGH);
      Serial.println("MOTOR: APAGADO");
      break;
    case 1:  // FAN1
      digitalWrite(PIN_RELE, LOW);
      Serial.println("MOTOR: ENCENDIDO BAJO");
      break;
    case 2:  // FAN2
      digitalWrite(PIN_RELE, LOW);
      Serial.println("MOTOR: ENCENDIDO MEDIO");
      break;
    case 3:  // FAN3
      digitalWrite(PIN_RELE, LOW);
      Serial.println("MOTOR: ENCENDIDO ALTO");
      break;
  }
}


'''Union de Componentes, Segunda Fase

> Estas pruebas usan botones para simular la EEPROM  


    
    #include <Servo.h>
    #include <Wire.h> 
    #include <LiquidCrystal_I2C.h>

    #define pinServo 2
    #define pinMotor 3
    #define pinInterruptorMotor 4
    #define pinInterruptorServo 5
    #define pinL1 8
    #define pinL2 9
    #define pinL3 10
    #define pinL4 11
    #define pinL5 12

    int cont = 0;
    int contVentilador = 0;
    int contPuerta = 0;
    // Variables para Ventilador
    int valorPot = 0;
    int velocidadVentilador = 0;
    // Variables para Puerta
    Servo servoPuerta;
    bool abrirPuerta = false; // Angulo 0 false puerta cerrada
    int servoAngulo = 0;
    // Variable para LCD
    LiquidCrystal_I2C lcd(0x20, 16, 2);


    void setup(){
      Serial.begin(9600);
      pinMode(pinMotor, OUTPUT);
      pinMode(pinInterruptorMotor, INPUT);
      pinMode(pinInterruptorServo, INPUT);
      servoPuerta.attach(pinServo);
      servoPuerta.write(0);
      delay(1000);

    digitalWrite(pinL1, HIGH);
    digitalWrite(pinL2, HIGH);
    digitalWrite(pinL3, HIGH);
    digitalWrite(pinL4, HIGH);
    digitalWrite(pinL5, HIGH);
      
    lcd.init();
    lcd.clear();
    lcd.backlight();
    
    lcd.setCursor(0,0);
    lcd.print(" Bienvenido");
    delay(100);
    lcd.setCursor(0,1);
    lcd.print( " Listo para Iniciar" );
    }

    void loop(){
      if(cont <= 20){
        lcd.scrollDisplayLeft();
        delay(100);
        cont++;
      }
    
    
    //Control Puerta
    int estadoServo = digitalRead( pinInterruptorServo );// Leer estado actual del interruptor
      if( estadoServo == HIGH && !abrirPuerta ){
        
        Serial.println( "Abriendo puerta..." );
        if(contPuerta != 1){
          lcd.clear();
          lcd.print( "Abriendo Puerta" );
          contPuerta++;
        }
        servoAngulo = 90;
        servoPuerta.write( servoAngulo );
        delay(300);
        abrirPuerta = true;   
      } else if( estadoServo == LOW && abrirPuerta ){
        Serial.println( "Cerrando puerta..." );
        if(contPuerta != 0){
        lcd.clear();
        lcd.print( "Cerrando Puerta" );
        delay(100);
        contPuerta = 0;
        }
        servoAngulo = 0;
        servoPuerta.write( servoAngulo );
        delay(500);
        abrirPuerta = false;
      }
    
    
    //Control Ventilador
    int estadoMotor = digitalRead( pinInterruptorMotor );
      Serial.println(estadoMotor);
        if( estadoMotor == HIGH ){
          digitalWrite(pinMotor, HIGH);
          if(contVentilador != 1){
            lcd.clear();
            lcd.print( "Ventilador Encendido" );
            contVentilador++;
          }
          delay(100);
        }else{
          digitalWrite(pinMotor, LOW);
          if(contVentilador != 0){
          lcd.clear();
          lcd.print( "Ventilador Apagado" );
          delay(100);
          contVentilador = 0;
          }
        }
'''<img width="1494" height="608" alt="Union de Componentes" src="https://github.com/user-attachments/assets/0d56795e-ff41-4459-a09a-a5749ab02e82" />
https://www.tinkercad.com/things/hlgouVLsfX1-union-de-componentes

