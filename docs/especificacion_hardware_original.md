# 📋 Especificación Técnica Original del Hardware (Versión Previa)

Este documento recopila la especificación técnica de los circuitos, componentes e interfaces de comunicación desarrollados en la etapa previa del diseño acelerográfico SMD. Sirve como referencia directa para la captura y estandarización en KiCad 10.

---

## 1. Arquitectura y Protocolos de Comunicación

El hardware integra el microcontrolador **ESP32** con sensores y periféricos mediante tres buses principales:

| Periférico | Protocolo | Líneas / Señales |
|---|---|---|
| **Acelerómetro ADXL355Z** | SPI | `SCLK`, `MOSI`, `MISO`, `CS` |
| **Tarjeta MicroSD** | SPI (vía buffer 74LVC125A) | `SCLK`, `MOSI`, `MISO`, `CS` |
| **Módulo GPS (FPGMMOPA6H)** | UART | `TX_GPS`, `RX_GPS` |
| **RTC DS3231** | I2C + Interrupción | `SDA`, `SCL`, `SQW` (1 Hz) |
| **Conversor USB-C / Serial** | UART (CH340G) | `TXD0`, `RXD0`, `DTR`, `RTS` |

> 🔗 **Repositorio de Firmware de Referencia:** [RSA Sensor (GitHub)](https://github.com/JorgeZh-hub/RSA_sensor)  
> *Nota Crítica:* Todos los pines del ESP32 asignados en KiCad deben coincidir con la configuración de GPIOs de este firmware.

---

## 2. Sistema de Alimentación

El circuito opera a dos niveles principales de tensión:
1. **Entrada Principal (+12V DC):** Conector XH de 2 pines con protección por diodos.
2. **Etapa Buck (+12V a +5V):** Regulador conmutado basado en el circuito integrado **MP2307** (eficiencia ~85%).
3. **Etapa Lineal (+5V a +3.3V):** Regulador lineal LDO **LD33V** (encapsulado SOT-223) para alimentar el ESP32, sensores y periféricos digitales.
4. **Alimentación USB-C:** Entrada alternativa de +5V por USB con diodos de protección para permitir programación y depuración sin conectar los +12V.

---

## 3. Circuitos Integrados (ICs) Principales

* **ESP32-WROOM-32E:** Microcontrolador SMD con conectividad Wi-Fi y antena externa.
* **ADXL355Z:** Acelerómetro triaxial de bajo ruido y alta resolución (SPI).
* **DS3231:** Reloj en tiempo real de alta precisión con oscilador TCXO integrado (SOIC-16) y batería de respaldo.
* **MP2307:** Regulador Buck Step-Down síncrono (SOIC-8 Exposed Pad).
* **LD33V:** Regulador de voltaje lineal de +3.3V (SOT-223).
* **CH340G:** Puente USB a UART para programación/depuración (SOP-16).
* **74LVC125A:** Buffer cuádruple de nivel lógico con salidas tri-state (TSSOP-14) para aislar las líneas SPI de la tarjeta MicroSD.

---

## 4. Lista de Componentes Pasivos y Semiconductores

### Resistencias
* `R1`, `R3`: 8.2 kΩ (SMD 2512)
* `R2`: 100 kΩ (SMD 2512)
* `R4`: 220 kΩ (Potenciómetro / Trimmer de ajuste)
* `R5`, `R6`, `R7`, `R8`: 3.3 kΩ (SMD 1206) - Pull-ups / Limitadoras
* `R9`, `R10`, `R11`, `R12`, `R16`: 4.7 kΩ (SMD 1206) - Pull-ups I2C y líneas de control
* `R13`: 200 Ω (SMD 1206) - Resistencia limitadora LED
* `R14`, `R15`: 5.11 kΩ (SMD 1206) - Pull-downs de configuración USB-C CC1/CC2
* `R17`: 4.7 kΩ (SMD 2512)
* `R18`, `R19`: 10 kΩ (SMD 1206)

### Capacitores
* `C1`, `C7`, `C9`, `C13`, `C15`, `C16`, `C18`, `C19`: 100 nF (0.1 µF) - Desacoplo de alta frecuencia
* `C2`, `C3`, `C17`: 10 µF - Filtrado y estabilización
* `C4`: 200 nF (0.2 µF)
* `C5`, `C6`: 100 µF - Filtrado de entrada/salida de potencia
* `C8`, `C12`: 12 µF
* `C10`: 4.7 nF
* `C11`: 10 nF
* `C14`: 1 µF

### Inductor y Diodos
* `L1`: 2 µH (SMD 0805) - Inductor de filtro/conmutación
* `D1`, `D3`, `D4`: Diodos rápidos 1N4148 (SMD SOD-123 / SMA)
* `D5`, `D6`: Diodos rectificadores 1N4001 / S1M (SMA/DO-214AC)
* `D2`, `D7`: LEDs indicadores de estado y encendido (SMD 1206)

---

## 5. Puntos de Prueba (Test Points) Implementados

Para la depuración física con osciloscopio o multímetro se requieren puntos de prueba en:
1. **Líneas SPI:** `SCK`, `MOSI`, `MISO`, `CS_ADXL355`, `CS_SD`.
2. **Líneas UART / GPS:** `TX_GPS`, `RX_GPS`, `TXD0`, `RXD0`.
3. **Líneas I2C / Reloj:** `SDA`, `SCL`, señal `SQW` (1 Hz).
4. **Alimentación y Tierra:** `GND`, `+3.3V`, `+5V`, `+12V`.
