## MARKDOWN COMPLETO

- [x] Carátula
- [x] Introducción
- [x] Objetivos
- [x] Marco Teórico
- [ ] Explicación detallada del sistema (incluye diagramas de flujo)
- [ ] Descripción de comandos seriales (con tabla)
- [ ] Explicación de archivos `.org`
- [x] Tabla de componentes con especificaciones técnicas y tabla de presupuesto
- [x] Aportes individuales
- [ ] Análisis de resultados y pruebas realizadas
- [x] Conclusiones
- [x] Bibliografía

---

Universidad San Carlos de Guatemala

Facultad de Ingeniería

Ingeniería en Ciencias y Sistemas

Organización Computacional

Escuela de vacaciones diciembre 2025

Ing. Carlos Alberto Arias López

Aux. Dilan Conaher Suy Miranda

**Proyecto final:**
**Casa automatizada**

Integrantes:

- Jose Brayan Arnoldo Murcia López - 3374570262004
- Abner Emanuel Garcia Sandoval - 2791049521905
- Mario Miguel López Sagastume - 3388185962011
- Alejandro José Salazar Ramirez - 3506308361904
- Julio René Morales Posadas - 2977547932008

Guatemala, 30 de diciembre del 2025

---

## Introducción

La domótica es una disciplina orientada a la automatización y control inteligente de espacios habitables, integrando tecnología electrónica, sensores, actuadores y sistemas de comunicación. En la actualidad, su aplicación permite mejorar la comodidad, eficiencia energética y seguridad en viviendas.

El presente proyecto consiste en el diseño y simulación de una casa inteligente mediante una maqueta de tres niveles, controlada por una placa Arduino. A través de comunicación serial con una computadora, se gestionan distintos estados de funcionamiento que representan escenarios cotidianos, integrando iluminación por ambientes, control de ventilación, apertura de puertas y visualización de estados mediante una pantalla LCD.

---

## Objetivo General

Implementar un sistema domótico interactivo utilizando Arduino, integrando actuadores como LEDs, motor DC y servomotor, junto con comunicación serial desde una computadora, aplicando principios de automatización residencial, programación estructurada y manejo de memoria no volátil para el control de una maqueta de casa inteligente.

---

## Objetivos Específicos

- Desarrollar una maqueta funcional de casa inteligente con tres niveles y control de iluminación independiente por cada ambiente.
- Implementar un sistema de ventilación en la sala mediante un motor DC.
- Automatizar la apertura y cierre de la puerta principal usando un servomotor.
- Gestionar escenas o estados predefinidos de la casa de forma secuencial mediante comandos enviados por el puerto serial.
- Almacenar y recuperar el estado del sistema utilizando memoria EEPROM.
- Proporcionar retroalimentación visual del estado actual del sistema mediante una pantalla LCD 16x2 con interfaz I2C.

---

## Marco Teórico

### Domótica

La domótica se refiere al conjunto de tecnologías aplicadas al control y automatización inteligente de una vivienda. Permite integrar sistemas de iluminación, climatización, seguridad y comunicación, mejorando la eficiencia y el confort del usuario. En este proyecto, la domótica se implementa mediante la automatización de luces, ventilación y accesos, controlados desde una computadora.

### Comunicación Serial

La comunicación serial es un método de transmisión de datos bit a bit entre dispositivos. Arduino utiliza este tipo de comunicación para interactuar con una computadora a través del puerto USB. En el sistema desarrollado, la comunicación serial permite enviar comandos desde la computadora para cambiar los modos de funcionamiento de la casa.

### EEPROM

La EEPROM (Electrically Erasable Programmable Read-Only Memory) es una memoria no volátil que permite almacenar datos incluso cuando el sistema se apaga. En Arduino, se utiliza para guardar información persistente como el estado actual del sistema, garantizando que al reiniciar la maqueta se conserve el último modo activo.

### Actuadores

Los actuadores son dispositivos que convierten señales eléctricas en acciones físicas. En este proyecto se emplean:
- **LEDs** para representar la iluminación de cada ambiente.
- **Motor DC** para simular el funcionamiento de un ventilador.
- **Servomotor** para controlar la apertura y cierre de la puerta principal.

### Pantallas LCD I2C

Las pantallas LCD 16x2 permiten mostrar información textual. El uso del protocolo I2C reduce la cantidad de pines necesarios para la comunicación con Arduino. En la maqueta, la pantalla LCD muestra el estado actual de la casa, facilitando la supervisión del sistema.

---

## Funcionamiento del Sistema

La maqueta representa una casa de tres niveles:
- **Primer nivel:** sala y baño.
- **Segundo nivel:** comedor y cocina.
- **Tercer nivel:** dormitorio.

Cada área cuenta con su iluminación específica mediante LEDs. En la sala se integra un motor DC que simula un ventilador, y en la entrada principal se utiliza un servomotor para la apertura de la puerta.

El sistema opera mediante estados secuenciales predefinidos:
- **FIESTA**
- **RELAJADO**
- **NOCHE**
- **DESPERTAR**

Los estados solo pueden activarse en orden, evitando la activación simultánea o desordenada de modos. Los cambios de estado se realizan enviando comandos desde la computadora a través del puerto serial. Cada cambio actualiza la iluminación, los actuadores correspondientes y se guarda el estado en la EEPROM.

La pantalla LCD 16x2 muestra el modo activo en tiempo real, permitiendo al usuario conocer el estado actual del sistema.

---

## Componentes y presupuesto


### Arduino MEGA 2560 (Genérico Steren)

El Arduino MEGA 2560 es una placa de desarrollo basada en el microcontrolador ATmega2560, diseñada para proyectos de mayor complejidad que requieren una gran cantidad de pines de entrada/salida y mayor capacidad de memoria. Es ideal para aplicaciones domóticas donde se integran múltiples actuadores y dispositivos de visualización.

**Especificaciones técnicas:**
- Microcontrolador: ATmega2560  
- Voltaje de operación: 5 V  
- Voltaje de alimentación recomendado: 7–12 V  
- Pines digitales: 54 (15 con PWM)  
- Entradas analógicas: 16  
- Memoria Flash: 256 KB  
- SRAM: 8 KB  
- EEPROM: 4 KB  
- Frecuencia de reloj: 16 MHz  

En el proyecto, el Arduino MEGA actúa como el núcleo del sistema, coordinando la comunicación serial, el control de luces, motores, servomotor, pantalla LCD y el almacenamiento de estados en memoria EEPROM.

---

### Protoboard de 830 Pines

La protoboard es un dispositivo de prototipado que permite realizar conexiones eléctricas sin necesidad de soldadura. Facilita el montaje, modificación y prueba de circuitos electrónicos de forma rápida y segura.

**Características técnicas:**
- Total de conexiones: 830 puntos  
- Conexiones centrales: 630 puntos para inserción de componentes  
- Líneas de alimentación:  
  - 100 pines de positivo  
  - 100 pines de negativo  

En este proyecto, la protoboard se utiliza para interconectar el Arduino con los LEDs, resistencias, relé, motor DC y otros componentes, permitiendo una organización clara del circuito.

---

### Servomotor SG90

El servomotor SG90 es un actuador de posicionamiento preciso ampliamente utilizado en proyectos de automatización y robótica. Permite controlar ángulos específicos mediante señales PWM generadas por el Arduino.

**Especificaciones técnicas:**
- Voltaje de operación: 4.8 V – 6 V  
- Rango de giro: 0° a 180°  
- Torque: aproximadamente 1.8 kg·cm a 4.8 V  
- Velocidad: 0.1 s / 60°  
- Peso: 9 g  
- Tipo de control: señal PWM  

En la maqueta, el SG90 se emplea para controlar la apertura y cierre de la puerta principal de la casa.

---

### Motor DC

El motor de corriente continua (DC) es un actuador que transforma energía eléctrica en movimiento rotatorio continuo. Su velocidad depende del voltaje aplicado y no permite control de posición, únicamente de giro.

**Características técnicas generales:**
- Voltaje de operación común: 3 V – 12 V  
- Tipo de movimiento: rotación continua  
- Control: encendido/apagado o regulación por PWM  

En el sistema desarrollado, el motor DC representa un ventilador ubicado en la sala del primer nivel de la maqueta.

---

### Pantalla LCD 16×2 con Interfaz I2C

La pantalla LCD 16×2 permite mostrar información textual de forma clara. Al utilizar un adaptador I2C, se reduce significativamente la cantidad de pines necesarios para la comunicación con Arduino.

**Especificaciones técnicas:**
- Resolución: 16 caracteres × 2 líneas  
- Interfaz: I2C (SDA y SCL)  
- Voltaje de operación: 5 V  
- Iluminación: retroiluminación LED  

En el proyecto, la pantalla LCD muestra el estado actual de la casa (FIESTA, RELAJADO, NOCHE o DESPERTAR), brindando retroalimentación visual al usuario.

---

### Relé 5 V de 5 Pines (Low Trigger)

El relé es un dispositivo electromecánico que permite controlar cargas de mayor voltaje o corriente utilizando una señal de bajo voltaje proveniente del Arduino.

**Características técnicas:**
- Voltaje de activación: 5 V DC  
- Tipo de activación: Low Trigger (activo con nivel bajo)  
- Terminales de carga: COM, NO (Normally Open), NC (Normally Closed)  

Este componente permite aislar el circuito de control del Arduino de los dispositivos que requieren mayor potencia.

---

### Cables para Protoboard

Los cables de conexión tipo Dupont se utilizan para enlazar eléctricamente el Arduino, la protoboard y los distintos módulos del sistema.

**Características:**
- Tipo: macho–macho, macho–hembra o hembra–hembra  
- Paso estándar: 2.54 mm  
- Uso: interconexión rápida y segura de componentes  

---

### LEDs

Los diodos emisores de luz (LEDs) son componentes electrónicos que emiten luz al ser polarizados correctamente. En proyectos de automatización se emplean como indicadores visuales.

**Características técnicas:**
- Voltaje típico: 2 V – 3.3 V (según color)  
- Corriente recomendada: 10–20 mA  

En la maqueta, los LEDs representan la iluminación de cada ambiente de la casa.

---

### Resistencias

Las resistencias son componentes pasivos que limitan el flujo de corriente dentro de un circuito, protegiendo dispositivos sensibles como los LEDs.

**Valores comunes utilizados:**
- 220 Ω para protección de LEDs  
- 1 kΩ y 10 kΩ para divisores de voltaje o resistencias pull-up  

Su uso es fundamental para garantizar el correcto funcionamiento y la seguridad del sistema electrónico.

---

### Presupuesto

| Nombre                          | Precio individual | Cantidad | Total       |
|---------------------------------|-------------------|----------|-------------|
| Arduino MEGA (genérico Steren)  | Q200.00           | 1        | Q200.00     |
| Protoboard 830 pines            | Q79.00            | 1        | Q79.00      |
| Servo SG90                      | Q30.00            | 1        | Q30.00      |
| Motor DC (5V pequeño)           | Q10.00            | 1        | Q10.00      |
| LCD 16×2 I2C Display            | Q50.00            | 1        | Q50.00      |
| Relay 5V 5PIN Low Trigger       | Q20.00            | 1        | Q20.00      |
| Cables protoboard (por metro)   | Q7.00             | 4        | Q28.00      |
| LEDs                            | Q1.00             | 5        | Q5.00       |
| Resistencias                    | Q1.50             | 5        | Q7.50       |
| Materiales de maqueta           | Q60.50            | 1        | Q60.50      |
| **TOTAL FINAL**                 |                   |          | Q490.00     |

---

## APORTES INDIVIDUALES DEL PROYECTO

### Mario López
- Manejo del módulo LCD I2C tanto en simulador como en montaje físico.
- Implementación y gestión de la memoria EEPROM.
- Aportes de código programable y soporte técnico al resto del equipo.
- Contribución teórica para la explicación general del funcionamiento del sistema.

### Abner García
- Integración de todos los componentes en el simulador para prevenir daños físicos.
- Armado y realización de pruebas del sistema en físico a partir del diseño simulado.
- Programación e integración de los códigos base aportados por los demás integrantes.
- Ensamblaje del circuito final en físico.

### Bryan Murcia
- Manejo del simulador y del montaje físico del sistema de luces LED.
- Aporte significativo en la construcción de la maqueta.
- Corrección e integración de distintos fragmentos de código para el programa final de Arduino.
- Aporte teórico en el análisis de resultados mediante el uso del monitor serial.
- Elaboración del apartado correspondiente a los archivos `.org`.

### Julio Posadas
- Simulación del motor DC y solución práctica en físico mediante el uso de un relé, ante la falta de un integrado.
- Apoyo en la construcción de la maqueta.
- Introducción teórica y práctica al uso de la memoria EEPROM, facilitando su comprensión al resto del equipo.

### Alejandro Salazar
- Obtención y gestión de los componentes electrónicos necesarios para el proyecto.
- Simulación y verificación del funcionamiento del servomotor en físico.
- Aportes teóricos relacionados con las generalidades y documentación del proyecto.


## Conclusiones

- Se logró desarrollar una maqueta funcional de casa inteligente con control independiente de iluminación por ambientes, cumpliendo con el objetivo de automatización residencial.
- El sistema de ventilación mediante motor DC y la apertura de la puerta con servomotor funcionaron correctamente, demostrando la integración efectiva de actuadores.
- La implementación de estados secuenciales permitió una gestión ordenada y lógica de los distintos modos de la casa.
- El uso de la EEPROM garantizó la persistencia del estado del sistema ante reinicios.
- La pantalla LCD I2C proporcionó una retroalimentación clara y eficiente del funcionamiento del sistema.
- La comunicación serial permitió un control confiable del sistema desde la computadora, cumpliendo con los objetivos planteados.

---

## Bibliografía

- Arduino. (s.f.). *Arduino Mega 2560 Rev3 – Technical Specifications*. Recuperado de https://www.arduino.cc  

- Arduino. (s.f.). *EEPROM Library*. Recuperado de https://www.arduino.cc/reference/en/libraries/eeprom/  

- Monk, S. (2017). *Programming Arduino: Getting Started with Sketches* (2nd ed.). McGraw-Hill Education.

- Banzi, M., & Shiloh, M. (2022). *Getting Started with Arduino* (4th ed.). Maker Media.

- Microchip Technology Inc. (s.f.). *ATmega2560 Datasheet*. Recuperado de https://www.microchip.com  

- TowerPro. (s.f.). *SG90 Micro Servo Motor Datasheet*.

- Hitachi. (s.f.). *HD44780U LCD Controller/Driver Datasheet*.

- Axelson, J. (2015). *Serial Port Complete: Programming and Circuits for RS-232 and RS-485*. Lakeview Research.

- Monk, S. (2015). *Practical Electronics for Inventors*. McGraw-Hill Education.