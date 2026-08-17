# 🚀 Rediseño y Estandarización de PCB Acelerográfica en KiCad 10 (PCBA JLCPCB)

Este repositorio contiene el espacio de trabajo, documentación de referencia y material de apoyo para el desarrollo de la pasantía de **Joel Suárez** en la **Red Sísmica del Austro (RSA)**.

El objetivo central es capturar, auditar, optimizar y estandarizar el diseño del **Sistema Acelerográfico con ESP32 (SMD)** en el entorno **KiCad 10**, garantizando que el diseño sea 100% compatible con el firmware de adquisición original y esté preparado para ensamblaje automático (**PCBA**) en **JLCPCB**.

---

## 📂 Organización del Repositorio

Para mantener el flujo de trabajo simple y ordenado, el proyecto está estructurado en tres carpetas:

```text
RSA-Intern-HW-Acelerografo_ESP32_KiCad/
│
├── .gitignore                                 # Configurado para ignorar temporales y respaldos de KiCad
├── README.md                                  # Guía operativa e instrucciones de trabajo
│
├── docs/                                      # Documentación técnica de apoyo y planificación
│   ├── Planificacion_Migracion_EAGLE_KiCad.pdf # Plan de trabajo oficial detallado (96 horas)
│   └── especificacion_hardware_original.md    # Detalle de circuitos integrados, valores pasivos y pines
│
├── eagle/                                     # Material de referencia del diseño previo
│   ├── BOM_Assembly.txt                       # Lista de materiales (BOM) con valores, referencias y encapsulados
│   ├── images_schematic/                      # Capturas de los esquemas por bloques (Páginas 1 a 7 y Full)
│   └── images_pcb_3d_view/                    # Vistas 3D de referencia de la placa previa
│
└── kicad/                                     # 👉 Espacio de trabajo exclusivo para el nuevo proyecto KiCad 10
```

---

## 🧭 Metodología de Trabajo y Fases (96 horas)

Dado que el diseño previo proviene de una versión en Autodesk Fusion Electronics / EAGLE sin archivos de proyecto nativos disponibles, el trabajo se desarrollará directamente en **KiCad 10** en la carpeta [`kicad/`](./kicad) siguiendo estas 6 fases:

### 1. Captura de Esquemático y Auditoría de Pines (Pinout Audit - 16 h)
* **Captura en KiCad 10:** Crear el proyecto en [`kicad/`](./kicad) y dibujar los bloques esquemáticos utilizando como referencia las capturas en [`eagle/images_schematic/`](./eagle/images_schematic), la lista de materiales en [`eagle/BOM_Assembly.txt`](./eagle/BOM_Assembly.txt) y la [`especificacion_hardware_original.md`](./docs/especificacion_hardware_original.md).
* **Auditoría estricta contra Firmware:** Cotejar pin a pin la asignación de GPIOs del ESP32 con el código fuente del firmware de adquisición disponible en [RSA Sensor (GitHub)](https://github.com/JorgeZh-hub/RSA_sensor):
  * **ADXL355Z (SPI):** `MOSI`, `MISO`, `SCK`, `CS`.
  * **Tarjeta MicroSD (SPI + Buffer 74LVC125A):** `MOSI`, `MISO`, `SCK`, `CS`.
  * **GPS (UART):** `TX`, `RX`.
  * **RTC DS3231 (I2C + SQW):** `SDA`, `SCL`, interrupción `SQW` (1 Hz).
  * **Conversor USB/Serial (UART):** `TXD0`, `RXD0`, `DTR`, `RTS` hacia el CH340G.

### 2. Estandarización de Componentes y Códigos LCSC (16 h)
* Asignar a cada símbolo el campo de metadatos `LCSC` con el número de parte (*LCSC Part Number*) del catálogo de JLCPCB.
* Priorizar componentes clasificados como **Basic Parts** de JLCPCB para reducir costos de montaje.
* Seleccionar encapsulados SMD estándar adecuados (0805, 0603, 1206, SOT-23, SOIC).

### 3. Gestión de Librerías Locales Relativas (12 h)
* Utilizar la herramienta `jlc2kicadlib` para descargar componentes específicos que no estén en la librería oficial de KiCad.
* Alojar todos los símbolos (`.kicad_sym`), huellas (`.kicad_mod` en carpeta `.pretty`) y modelos 3D (`.step`) dentro de carpetas locales del proyecto (ej. `kicad/libs/`).
* Configurar las rutas mediante variables relativas del entorno KiCad (ej. `${KIPRJMOD}`) para que el repositorio sea **100% portable y autónomo** al clonarse en cualquier computador.

### 4. Layout Físico, Integridad y Optimización (24 h)
* **Desacoplo:** Ubicar condensadores de desacoplo (0.1 µF y 10 µF) lo más próximos posible a los pines `VCC`/`VDD` del ESP32, ADXL355 y RTC.
* **Plano de Masa:** Implementar planos de tierra robustos de cobre (GND) en ambas capas (Top y Bottom).
* **Señales Críticas:** Rutar con trazas cortas y directas las líneas de datos SPI del acelerómetro y MicroSD, minimizando el número de vías.
* **Test Points:** Incluir puntos de prueba accesibles (pads/headers) para señales críticas (`3.3V`, `5V`, `GND`, `TX/RX`, `SQW 1Hz`, bus SPI).
* **Pick & Place:** Colocar al menos 3 marcas fiduciarias (*Fiducials*) en las esquinas de la PCB.
* **DRC:** Ejecutar el Design Rules Check de KiCad y asegurar 0 errores/advertencias.

### 5. Generación y Validación de Archivos de Fabricación (14 h)
* Instalar y utilizar el plugin **Fabrication Toolkit** en KiCad para generar automáticamente el paquete PCBA:
  * Archivos Gerber y NC Drill.
  * Lista de Materiales (`BOM.csv`).
  * Archivo de Posición y Coordenadas (`CPL.csv`).
* Cargar el paquete en el visor web de JLCPCB para auditar y corregir rotaciones u offsets de los componentes SMD.

### 6. Cierre y Documentación (14 h)
* Documentar las decisiones técnicas y optimizaciones realizadas (planos GND, desacoplo, corrección de pines).
* Generar las instrucciones de actualización y exportación en el README final del proyecto.

---

## 🏆 Criterios de Éxito y Entregables Finales

1. **Proyecto KiCad 10 Funcional y Portable:** Esquemático y PCB completos en `kicad/` con librerías locales relativas.
2. **Paquete de Fabricación JLCPCB Validado:** Gerbers, BOM y CPL listos para producción sin requerir correcciones manuales en el visor de JLCPCB.
3. **Auditoría de Pines Verificada:** Informe que certifique la compatibilidad total del pinout con el firmware de adquisición.
