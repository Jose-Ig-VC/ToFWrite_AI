# ToFWrite AI — Plataforma de Reconocimiento Gestual con Sensores ToF y MLP Embebido

Plataforma educativa e interactiva de bajo costo para capturar, visualizar y clasificar cuatro trazos bidimensionales (**«C»**, **círculo**, **«1»** y **«2»**) mediante dos sensores láser *Time-of-Flight* (**VL53L0X**), un microcontrolador **ESP32-S3** (programado en **Arduino IDE / C++**), una interfaz en **Python (PyQt5)** y un **Perceptrón Multicapa (MLP)** desplegado de forma embebida.

---

## Descripción General

El proyecto aborda la visualización del reconocimiento gestual y el flujo de inferencia en tiempo real sobre una pizarra de **140 × 140 mm** con dos paredes perpendiculares y un área efectiva de trazado de **100 × 100 mm** (aplicando un desplazamiento geométrico $d_0 = 42.5\text{ mm}$)

* **Adquisición y FSM (ESP32-S3 en Arduino IDE):** Lee ambos sensores VL53L0X en buses $\text{I}^2\text{C}$ independientes cada **20 ms** (50 Hz), valida coordenadas dentro de $[0, 100]\text{ mm}$ y gestiona el flujo mediante una máquina de estados finitos de 3 estados:
  * `ESPERA`: LED verde fijo; el botón 1 limpia el buffer, envía `START` y pasa a `CAPTURA`.
  * `CAPTURA`: LED rojo parpadeante; muestreo cada 20 ms, emite `POINT:x,y` (dentro de rango) o `OUT_OF_BOUNDS` (fuera de rango); el botón 2 pasa a `PROCESAR`.
  * `PROCESAR`: remuestrea la trayectoria a **32 puntos** equidistantes, normaliza y transmite `DATA:...` (recolección/entrenamiento) o ejecuta la inferencia embebida y emite `PREDICCION: X`, regresando a `ESPERA`.
* **Interfaz y Dataset (Python / PyQt5):** Visualiza la trayectoria en tiempo real en el lienzo de 100 × 100 mm, registra el dataset etiquetado en CSV y entrena el MLP (`65 -> 32 -> 16 -> 4`, considerando 32 coordenadas X, 32 coordenadas Y y duración normalizada).
* **Despliegue Embebido:** Los pesos entrenados se exportan para su ejecución en C++ dentro del ESP32-S3.

---

## Hardware Principal

* **Microcontrolador:** ESP32-S3 (montado en protoboard con USB-C orientado hacia arriba en el montaje físico).
* **Sensores de distancia:** 2 × **VL53L0X** (uno en pared izquierda para eje $+X$ y otro en pared frontal orientado hacia el usuario para eje $-Y$, con control `XSHUT` y buses $\text{I}^2\text{C}$ separados).
* **Interfaz física:** 2 botones pulsadores (`INICIAR` y `FINALIZAR`), 2 LEDs de estado (verde y rojo) con sus respectivas resistencias limitadoras en serie, y protoboard.

---

## Estado Actual del Repositorio

* **Informe 1:** Documento técnico inicial del proyecto (*ToFWrite AI*).
* **Esquemático electrónico:** Diseño de conexiones de alimentación, buses $\text{I}^2\text{C}$, control `XSHUT`, botones y LEDs.
* **PCB:** Diseño de placa en desarrollo / futura fabricación.
* **Firmware y software:** Código en Arduino IDE para lectura dual VL53L0X / FSM e interfaz en Python (`PyQt5`) para captura, dataset y entrenamiento del MLP.

---

## Flujo de Trabajo

1. **Verificación de sensores y FSM:** Validar lecturas simultáneas de los VL53L0X y transiciones de estado en Arduino IDE (`115200` baudios).
2. **Captura y construcción del dataset:** Conectar el ESP32-S3 a la interfaz `PyQt5` para graficar en vivo y guardar muestras de las cuatro clases («C», círculo, «1» y «2»).
3. **Entrenamiento del MLP en Python:** Entrenar y evaluar el modelo con el dataset recolectado y exportar sus parámetros.
4. **Inferencia embebida:** Integrar el modelo en el ESP32-S3 y validar la clasificación en tiempo real desde `PyQt5`.
