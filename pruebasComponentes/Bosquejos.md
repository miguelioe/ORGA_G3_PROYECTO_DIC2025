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

'''Union de Componentes, Primera Fase
    
    #include <Servo.h>

    #define pinServo 2
    #define pinVelocidad 3
    #define pinDireccion 4
    #define pinPulsador 5

    // Variables para Ventilador
    int valorPot = 0;
    int velocidadVentilador = 0;
    // Variables para Puerta
    Servo servoPuerta;
    bool abrirPuerta = false; // Angulo 0 false puerta cerrada
    int servoAngulo = 0;
    unsigned long ultimoTiempo  = 0;
    const unsigned long tiempoEspera  = 200; //Evitar rebotes mecanicos



    void setup(){
      Serial.begin(9600);
      pinMode(pinVelocidad, OUTPUT);
      pinMode(pinDireccion, OUTPUT);
      pinMode(pinPulsador, INPUT);
      servoPuerta.attach(pinServo);
      servoPuerta.write(0);
      delay(1000);
        }

      void loop(){
      //Control Puerta
      int estadoPulsador = digitalRead( pinPulsador );// Leer estado actual del pulsador
      if ( millis() - ultimoTiempo > tiempoEspera ) {
      if( estadoPulsador == HIGH && !abrirPuerta ){
      ultimoTiempo = millis();  // Actualizar tiempo
      
      Serial.println( "Abriendo puerta..." );
      servoAngulo = 90;
      servoPuerta.write( servoAngulo );
      abrirPuerta = true;
      delay(300);   
    } else if( estadoPulsador == HIGH && abrirPuerta ){
      ultimoTiempo = millis();  // Actualizar tiempo
      
      Serial.println("Cerrando puerta...");
      servoAngulo = 0;
      servoPuerta.write( servoAngulo );
      abrirPuerta = false;
      delay(500);
    }
    }
  
    //Control Velocidad Ventilador
    valorPot = analogRead(A0);
    valorPot = map(valorPot, 0, 1023, 0, 255);
  
    if (valorPot == 0) { velocidadVentilador = 0; } 
    else if (valorPot < 185) { velocidadVentilador = 150; } 
    else { velocidadVentilador = 255; }
    digitalWrite( pinDireccion, HIGH);
    analogWrite( pinVelocidad, velocidadVentilador );
    }
'''<img width="1494" height="608" alt="Union de Componentes" src="https://github.com/user-attachments/assets/0d56795e-ff41-4459-a09a-a5749ab02e82" />
https://www.tinkercad.com/things/hlgouVLsfX1-union-de-componentes

