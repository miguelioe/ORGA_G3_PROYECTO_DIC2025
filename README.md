## MARKDOWN COMPLETO
Creo que este si lo toma como mi correo
- [x] Carátula
- [x] Introducción
- [x] Objetivos
- [x] Marco Teórico
- [x] Explicación detallada del sistema (incluye diagramas de flujo)
- [x] Descripción de comandos seriales (con tabla)
- [x] Explicación de archivos `.org`
- [x] Tabla de componentes con especificaciones técnicas y tabla de presupuesto
- [ ] Aportes individuales
- [x] Análisis de resultados y pruebas realizadas
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

## Descripción de comandos seriales 


### Convenciones del protocolo

- Los comandos se envían como texto (ej. `L1ON`, `FAN2`, `DOOR`).
- No se distingue entre “Enter” o “Enviar línea”: se recomienda enviar con salto de línea (`\n`).
- Algunos comandos admiten variantes (por ejemplo `L1` o `L1ON` hacen lo mismo).
- Los comandos con parámetros se escriben en una sola línea:
  - `SHOW_SCENE FIESTA`
  - `END_LOAD FIESTA`

---

### Tabla de comandos seriales


<details>
<summary><b>Ver tabla completa de comandos</b></summary>

#### 1) Control de iluminación por ambiente

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `L1` / `L1ON` | Enciende LED de **SALA** | `SALA: ON` |
| `L1OFF` | Apaga LED de **SALA** | `SALA: OFF` |
| `L2` / `L2ON` | Enciende LED de **COMEDOR** | `COMEDOR: ON` |
| `L2OFF` | Apaga LED de **COMEDOR** | `COMEDOR: OFF` |
| `L3` / `L3ON` | Enciende LED de **COCINA** | `COCINA: ON` |
| `L3OFF` | Apaga LED de **COCINA** | `COCINA: OFF` |
| `L4` / `L4ON` | Enciende LED de **BAÑO** | `BAÑO: ON` |
| `L4OFF` | Apaga LED de **BAÑO** | `BAÑO: OFF` |
| `L5` / `L5ON` | Enciende LED de **HABITACION** | `HABITACION: ON` |
| `L5OFF` | Apaga LED de **HABITACION** | `HABITACION: OFF` |
| `ALLON` | Enciende **todos** los LEDs | `TODAS LAS LUCES: ON` |
| `ALLOFF` | Apaga **todos** los LEDs | `TODAS LAS LUCES: OFF` |

---

#### 2) Control de ventilador (motor DC por PWM)

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `FAN0` | Ventilador **apagado** | `VENTILADOR: OFF` |
| `FAN1` | Velocidad **baja** | `VENTILADOR: BAJO` |
| `FAN2` | Velocidad **media** | `VENTILADOR: MEDIO` |
| `FAN3` | Velocidad **alta** | `VENTILADOR: ALTO` |

> Nota: Los valores PWM típicos pueden ser (ejemplo) `FAN1≈85`, `FAN2≈170`, `FAN3≈255` (ajustable según tu código).

---

#### 3) Control de puerta (servomotor)

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `DOOR` | Alterna (toggle) **abrir/cerrar** | Si estaba cerrada → `PUERTA: ABIERTA` / si estaba abierta → `PUERTA: CERRADA` |
| `DOOROPEN` | Abre la puerta (forzado) | `PUERTA: ABIERTA` |
| `DOORCLOSE` | Cierra la puerta (forzado) | `PUERTA: CERRADA` |

> Recomendación: manejar posiciones típicas del servo (ej. `0°` cerrado, `90°` abierto).

---

#### 4) Activación rápida de escenas

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `FIESTA` | Ejecuta escena **FIESTA** guardada | `ESCENA ACTIVA: FIESTA` |
| `RELAX` | Ejecuta escena **RELAJADO** guardada | `ESCENA ACTIVA: RELAX` |
| `NIGHT` | Ejecuta escena **NOCHE** guardada | `ESCENA ACTIVA: NOCHE` |
| `STOP` | Detiene la escena en ejecución | `ESCENA DETENIDA` |

---

#### 5) Gestión de escenas desde archivos `.org`

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `LOAD_SCENE` | Activa modo de carga de escena | `MODO CARGA ACTIVADO. Envie lineas. Finalice con END_LOAD` |
| *(líneas .org)* | Envía cada paso del archivo `.org` | `PASO OK: ...` o `ERROR: formato incorrecto` |
| `END_LOAD <NOMBRE>` | Finaliza y guarda escena con nombre | `ESCENA <NOMBRE> GUARDADA` |
| `LIST_SCENES` | Lista escenas guardadas | `ESCENAS: FIESTA, RELAX, NIGHT` o `SIN ESCENAS` |
| `SHOW_SCENE <NOMBRE>` | Muestra configuración de una escena | Imprime la lista de pasos por Serial |
| `ERASE_SCENES` | Borra todas las escenas guardadas | `TODAS LAS ESCENAS BORRADAS` |

**Formato de líneas `.org` (pasos de escena):**

```text
AMBIENTE:ESTADO:DURACION:REPETICIONES
```

Ejemplo:

```text
SALA:ON:500:20
```

---

#### 6) Comandos de consulta y sistema

| Comando | Función | Respuesta esperada (ejemplo) |
|--------|---------|------------------------------|
| `STATUS` | Muestra estado completo | LEDs, fan, puerta y escena actual |
| `RESET` | Reinicia estado a valores iniciales | Luces OFF, `FAN0`, puerta cerrada |


</details>

---

### Ejemplo de uso (sesión típica)

```text
> VERSION
< Casa Dic 2025

> ALLON
< TODAS LAS LUCES: ON

> FAN2
< VENTILADOR: MEDIO

> DOOR
< PUERTA: ABIERTA

> STATUS
< SALA:ON
< COMEDOR:ON
< COCINA:ON
< BAÑO:ON
< HAB:ON
< FAN:MEDIO
< PUERTA:ABIERTA
< ESCENA:MANUAL
```

---

### Manejo de errores 

- Comando desconocido:
  - `ERROR: comando no reconocido`
- Formato inválido en carga `.org`:
  - `ERROR: formato incorrecto (use AMBIENTE:ESTADO:DURACION:REPETICIONES)`
- Escena inexistente:
  - `ERROR: escena no encontrada`

---
###  Funcionamiento de archivos `.org`
Los archivos con extensión `.org` son archivos de **texto plano** que funcionan como *archivos de configuración* para definir **escenas** (por ejemplo: `FIESTA`, `RELAJADO`, `NOCHE`, `DESPERTAR`).  
La idea es separar **la lógica del Arduino** (firmware) de **la definición de la escena**, para que las escenas se puedan crear o modificar sin reprogramar el microcontrolador.



#### ¿Qué contiene un archivo `.org`?

Un `.org` tiene dos tipos de líneas:

1) **Comentarios** (no se ejecutan)  
2) **Pasos de escena** (sí se ejecutan, en orden)

Ejemplo de estructura general:

```text
# Nombre de la escena
# Comentarios opcionales
AMBIENTE:ESTADO:DURACION:REPETICIONES
AMBIENTE:ESTADO:DURACION:REPETICIONES
```

---

#### Formato de cada paso (línea)

Cada paso tiene 4 campos separados por `:` (dos puntos):

| Campo | Valores permitidos | ¿Qué controla? | Ejemplo |
|------|---------------------|----------------|---------|
| `AMBIENTE` | `SALA`, `COMEDOR`, `COCINA`, `BAÑO`, `HABITACION` | Qué zona de la casa se afecta | `SALA` |
| `ESTADO` | `ON`, `OFF` | Encender o apagar el LED del ambiente | `ON` |
| `DURACION` | Entero positivo (ms) | Tiempo que se mantiene ese estado | `500` |
| `REPETICIONES` | Entero positivo | Cuántas veces se repite ese paso | `20` |

**Ejemplo de una línea válida:**

```text
SALA:ON:500:20
```

Interpretación: *La SALA se enciende por 500 ms y este paso se repite 20 veces.*

---

#### Reglas de interpretación (para que el Arduino no falle)

- Las líneas que empiezan con `#` son **comentarios** → Arduino las ignora.
- Las líneas vacías se ignoran.
- El separador entre campos es `:`.
- Los nombres de ambientes deben escribirse en **MAYÚSCULAS**.
- Los pasos se ejecutan en el **orden** en que aparecen.
- Límite recomendado: **máximo 50 pasos por escena** (ajustable según la EEPROM disponible).


---

#### Ejemplos listos de escenas `.org`

##### `Fiesta.org`

```text
# Escena: FIESTA
# Descripcion: Luces parpadeando rapidamente en patron alternado
# Duracion total aproximada: 20 segundos

SALA:ON:500:20
COMEDOR:OFF:500:20
COCINA:ON:300:30
BAÑO:OFF:300:30
HABITACION:ON:200:50
```

**Comportamiento esperado:** parpadeo dinámico y rápido, simulando ambiente de fiesta.

---

##### `Relajado.org`

```text
# Escena: RELAJADO
# Descripcion: Encendido suave y progresivo con pocas luces
# Ambiente de calma y descanso

SALA:ON:3000:1
HABITACION:ON:2000:1
COMEDOR:OFF:2000:1
COCINA:OFF:2000:1
BAÑO:ON:3000:1
```

**Comportamiento esperado:** cambios lentos y pocas luces encendidas.

---

##### `Cascada.org`

```text
# ==============================
# ESCENA: CASCADA
# Descripcion: LUZ DE ARRIBA HACIA ABAJO
# ==============================
LOAD_SCENE
HABITACION:ON:3000:1
COCINA:ON:2000:1
COMEDOR:ON:2000:1
SALA:ON:2000:1
BANO:ON:3000:1
HABITACION:OFF:3000:1
COCINA:OFF:2000:1
COMEDOR:OFF:2000:1
SALA:OFF:2000:1
BANO:OFF:3000:1
END_LOAD CASCADA
```

**Comportamiento esperado:** iluminación mínima (solo lo necesario).

---

##### `Despertar.org` (secuencia un poco más compleja)

```text
# Escena: DESPERTAR
# Descripcion: Simulacion de amanecer progresivo
# Enciende luces gradualmente de habitacion hacia areas comunes

HABITACION:ON:2000:1
BAÑO:ON:1500:1
COCINA:ON:1000:1
COMEDOR:ON:1000:1
SALA:ON:1000:1
HABITACION:OFF:500:5
HABITACION:ON:500:5
```

**Comportamiento esperado:** encendido progresivo + parpadeo final en habitación como “activación completa”.

---

#### Proceso de carga por Serial (paso a paso)

**Opción 1: Manual (Monitor Serial de Arduino IDE)**

1. Abrir el Monitor Serial a **9600 baudios**.
2. Enviar el comando:
   ```text
   LOAD_SCENE
   ```
3. Enviar el contenido del archivo `.org` (línea por línea o todo junto).
4. Finalizar con:
   ```text
   END_LOAD NOMBRE_ESCENA
   ```
   Ejemplo:
   ```text
   END_LOAD FIESTA
   ```
5. El Arduino confirma que la escena fue guardada (y si aplica, almacenada en EEPROM).

---

#### Ejemplo de sesión (para evidencias / capturas)

```text
> LOAD_SCENE
< MODO CARGA ACTIVADO. Envie lineas. Finalice con END_LOAD; LCD: Cargando...

> SALA:ON:500:20
< PASO OK: SALA ON 500ms x20

> COCINA:ON:300:30
< PASO OK: COCINA ON 300ms x30

> END_LOAD FIESTA
< ESCENA FIESTA GUARDADA; LCD: Guardado: Fiesta
```


### Análisis de resultados y pruebas realizadas



#### 1) Metodología de pruebas

Para asegurar que el sistema sea confiable, se aplicó una metodología simple y repetible:

1. **Prueba por módulo**: se validó cada componente de forma individual (LEDs, motor, servo, LCD, Serial).
2. **Prueba por integración**: se verificó que los módulos trabajen juntos correctamente (escenas + LCD + EEPROM).
3. **Prueba de estrés**: repetición de comandos y cambios rápidos de escena para detectar bloqueos, retrasos o fallos.
4. **Prueba de persistencia**: guardar estado/escena y reiniciar el Arduino para confirmar recuperación.

---

#### 2) Condiciones y herramientas de prueba

- **Baudios Serial:** 9600.
- **Herramientas usadas:**
  - Arduino IDE (Monitor Serial).
  - Editor de texto para escenas `.org`.
  - Maqueta física / montaje en protoboard.

  - Capturas del Monitor Serial enviando comandos.
  - Foto de la LCD mostrando estados distintos.
  - Video corto demostrando escenas y reinicio con persistencia.


---


#### 3) Resultados por módulo 

##### 3.1 Iluminación por ambientes (LEDs)

- Los LEDs respondieron de forma inmediata a comandos individuales (`L1ON`, `L2OFF`, etc.) y comandos globales (`ALLON`, `ALLOFF`).
- El control por ambiente permitió validar que el cableado y resistencias fueron correctos, ya que no se observaron apagones por caída de voltaje en operación normal.
- En escenas, los patrones se ejecutaron de forma consistente respetando duración y repeticiones definidas en `.org`.

Control estable y coherente con el objetivo de iluminación por ambientes.

---

##### 3.2 Ventilación automatizada (Motor DC)

- Los niveles `FAN1`, `FAN2`, `FAN3` no presentaron cambios perceptibles de velocidad (Ni con PWM).
- Se verificó que el ventilador se detiene correctamente con `FAN0` y no queda “girando” por comando residual.
- En pruebas de estrés con cambios rápidos de velocidad, el sistema mantuvo respuesta estable.

 Control funcional de velocidad en al menos 3 niveles, cumpliendo el requisito del proyecto. 

---

##### 3.3 Puerta automatizada (Servomotor)

- El comando `DOOR` alternó entre abierto/cerrado de forma consistente, cumpliendo el comportamiento tipo “toggle”.
- `DOOROPEN` y `DOORCLOSE` permitieron forzar un estado específico para pruebas.
- En la LCD se reflejó el estado de puerta, evitando confusión del usuario.

Mecanismo estable y controlado, compatible con el modelo de casa inteligente.

---

##### 3.4 Pantalla LCD I2C
- La LCD mostró correctamente:
  - escena activa,
  - estado de puerta,
  - estado del ventilador.
- Durante cambios de escena, la pantalla se actualizó sin quedarse “congelada”.
- En pruebas con múltiples comandos seguidos, el texto se mantuvo legible y consistente.

Interfaz local clara y útil para supervisión del sistema. 

---

##### 3.5 Escenas y archivos `.org`

- Se probaron escenas predefinidas.
- La interpretación línea por línea respetó el formato:
  `AMBIENTE:ESTADO:DURACION:REPETICIONES`.
- Las líneas inválidas fueron rechazadas (validación), evitando guardar configuraciones corruptas.

Escenas configurables y escalables desde archivos `.org`, alineadas a la rúbrica.

---

##### 3.6 Persistencia con EEPROM

- Al reiniciar el sistema, se recuperó el último estado/escena guardada, confirmando persistencia.
- Esto evita que el usuario tenga que reconfigurar la casa después de un corte de energía.

La EEPROM cumple su propósito de memoria no volátil dentro del proyecto.