# Virtual-Temperature-Monitoring-PCB-Design

A computer-based design of a temperature-monitoring PCB using an **STM32G431CBUx** microcontroller.

The project covers the design process from schematic capture and analog signal conditioning through SPICE simulation, PCB layout, firmware development, and manufacturing-file generation.

## Overview

The system models a temperature sensor with a linear output of:

**V<sub>TEMP</sub> = 0.02 × T**

This gives a sensor output of **0 V at 0 °C** and **2.0 V at 100 °C**.

The sensor signal passes through a passive RC low-pass filter before being measured by the STM32 ADC. The calculated temperature is then reported through UART.

### Signal Path

```text
Temperature
     ↓
Virtual Sensor
     ↓
RC Low-Pass Filter
     ↓
STM32 ADC
     ↓
Temperature Calculation
     ↓
UART
     ↓
Host PC
```

### Power Path

```text
5 V Input → AMS1117-3.3 → 3.3 V Rail → STM32 + Supporting Circuitry
```

## PCB Preview

<img width="404" height="245" alt="VirtualTemperatureMonitoringPCB_3D" src="https://github.com/user-attachments/assets/a3ecc7b7-3c3c-4e6b-ad42-c9296d7fb148" />


*3D representation of the virtual PCB design.*

## Main Specifications

| Parameter                  | Value         |
| -------------------------- | ------------- |
| MCU                        | STM32G431CBUx |
| Intended temperature range | 0–100 °C      |
| Sensor model               | 0.02 V/°C     |
| Sensor output              | 0–2.0 V       |
| ADC resolution             | 12-bit        |
| ADC reference              | 3.3 V         |
| RC filter                  | 1 kΩ / 100 nF |
| Filter cutoff frequency    | 1.59 kHz      |
| Supply input               | 5 V           |
| Regulated supply           | 3.3 V         |
| UART baud rate             | 115200 bit/s  |

## Hardware Design

The PCB was designed in **KiCad 10** and includes the STM32G431CBUx, analog input filtering, 3.3 V regulation, decoupling, power input, and UART interface.

The RC filter uses:

* **R1 = 1 kΩ**
* **C1 = 100 nF**
* **Cutoff frequency ≈ 1.59 kHz**

The 3.3 V supply is generated from the nominal 5 V input using an **AMS1117-3.3** linear regulator.

### Schematic

<img width="404" height="245" alt="VirtualTemperatureMonitoringPCB_Schematic" src="https://github.com/user-attachments/assets/3ad24ec4-859f-4658-8d0d-4161521fbfd5" />

*KiCad schematic showing the temperature-sensing, filtering, power, MCU, and UART sections.*

### PCB Layout

<img width="404" height="245" alt="VirtualTemperatureMonitoringPCB_pcb" src="https://github.com/user-attachments/assets/cf4023e5-31c0-4a86-9a18-1a740bd3b6a0" />

*PCB layout showing component placement, routing, and ground copper.*

## Firmware

The STM32 firmware was developed using **STM32CubeIDE** and the **STM32 Hardware Abstraction Layer (HAL)**.

The firmware configures:

* ADC1 on **PA0 (ADC1_IN1)**
* 12-bit, single-ended ADC conversion
* USART1 TX on **PA9**
* USART1 RX on **PA10**
* UART at **115200 bit/s**

The ADC reading is converted to voltage using the 3.3 V reference and then converted to temperature using the sensor model.

Example UART output:

```text
ADC: 1241, Voltage: 1.000 V, Temperature: 50.00 C
```

The theoretical ADC range extends to approximately 3.3 V. With the sensor model of 0.02 V/°C, this corresponds to a theoretical maximum of **165 °C**, although the intended operating range is **0–100 °C**.

### Firmware Flow

<img width="404" height="245" alt="mermaid-flowchart-2026-08-23T03-05-18" src="https://github.com/user-attachments/assets/d367f673-8a98-4420-86b0-341db08137c2" />

```text

Simulation and Analysis

SPICE transient simulation was used to examine the behaviour of the RC filter.

The simulated filtered signal showed the expected low-pass behaviour, smoothly approaching the applied sensor voltage following a step change.

The ideal 12-bit ADC resolution is:

```text
3.3 V / 4096 ≈ 0.806 mV/count
```

Some expected values are:

| Temperature | Sensor Voltage | Approx. ADC Code |
| ----------: | -------------: | ---------------: |
|        0 °C |          0.0 V |                0 |
|       25 °C |          0.5 V |              620 |
|       50 °C |          1.0 V |             1241 |
|       75 °C |          1.5 V |             1861 |
|      100 °C |          2.0 V |             2482 |

### SPICE Simulation

<img width="404" height="245" alt="PCB_TempFiltered vs TempSensor AT 2V" src="https://github.com/user-attachments/assets/ab388c55-5e97-44fd-8b53-3d76acc024fd" />

*Simulated transient response of the RC low-pass filter at 2.0V.*

## Verification

The project was verified using computer-based design and simulation tools.

Completed checks include:

* KiCad schematic and ERC verification
* SPICE transient simulation
* ADC resolution calculations
* PCB layout and 3D inspection
* Manufacturing-file generation
* STM32 firmware compilation

The final STM32 Debug build completed with:

```text
0 errors
0 warnings
```

The build also generated the firmware output files required for future programming of the microcontroller.

## Lessons Learned

This project provided hands-on experience with:

* Full PCB design workflow (schematic → layout → simulation → manufacturing outputs)
* RC filter analysis (time constant, cutoff frequency, transient response)
* ADC resolution and quantization analysis
* Power regulation and MCU decoupling strategies
* STM32 firmware development (ADC configuration, UART communication)
* Professional documentation and version control practices

## Future Work

The next stage of the project would be to:

1. Fabricate the PCB.
2. Assemble the components.
3. Program the STM32 with the completed firmware.
4. Test the ADC using known input voltages.
5. Verify temperature conversion and UART output.
6. Compare measured results with the theoretical and simulated results.

## Repository Structure

```text
.
├── Firmware/
│   └── STM32CubeIDE project
│
├── KiCad/
│   ├── Schematic
│   ├── PCB layout
│   └── Project files
│
├── Simulation/
│   └── SPICE files
│
├── Manufacturing/
│   ├── Gerbers
│   ├── Drill files
│   └── BOM
│
├── Project report/
│   
│
├── Images/
│   ├── pcb-3d.png
│   ├── schematic.png
│   ├── pcb-layout.png
│   └── spice-response.png
│
└── README.md
```

## Tools

* **KiCad 10** — schematic capture and PCB design
* **STM32CubeIDE** — firmware development
* **STM32 HAL** — MCU peripheral configuration
* **SPICE** — analog simulation
* **Git / GitHub** — project version control

