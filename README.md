MARKDOWN COMPLETO:
[x] CARATULA 
[x] INTRODUCCION 
[x] OBJETIVOS
[x] MARCO TEORICO
[] EXPLICACION DETALLADA A SU VEZ LLEVA DIAGRAMAS DE FLUJO
[] DESCRIPCION DE COMANDOS SERIALES CON SU TABLA
[] EXPLICACION DE ARCHIVOS .org
[] TABLA DE COMPONENTES CON EXPECIFICACIONES TECNICAS Y TABLA DE PRESUPUESTO
[] APORTES INDIVIDUALES
[] ANALISIS DE RESULTADOS Y PRUEBAS REALIZADAS
[x] CONCLUSIONES
[] BIBLIOGRAFIA

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

## Conclusiones

- Se logró desarrollar una maqueta funcional de casa inteligente con control independiente de iluminación por ambientes, cumpliendo con el objetivo de automatización residencial.
- El sistema de ventilación mediante motor DC y la apertura de la puerta con servomotor funcionaron correctamente, demostrando la integración efectiva de actuadores.
- La implementación de estados secuenciales permitió una gestión ordenada y lógica de los distintos modos de la casa.
- El uso de la EEPROM garantizó la persistencia del estado del sistema ante reinicios.
- La pantalla LCD I2C proporcionó una retroalimentación clara y eficiente del funcionamiento del sistema.
- La comunicación serial permitió un control confiable del sistema desde la computadora, cumpliendo con los objetivos planteados.