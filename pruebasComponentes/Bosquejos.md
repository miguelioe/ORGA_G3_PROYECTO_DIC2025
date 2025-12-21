## Prueba básica de código para el servomotor

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

![ServomotorConexionBasica](/img/ServomotorConexionBasica.png)

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
![Conexión basica LCD I2C](../img/lcdi2c.png)

[Accede a este circuito simulado! (haz clic en cualquier sitio azul)](https://wokwi.com/projects/450932545007138817)