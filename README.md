# Smart Plant Growth Monitor (ELN 233)

An embedded data-acquisition and real-time monitoring system that bridges analog environmental sensors with the digital logic of a **Raspberry Pi 5** using an **MCP3008 10-bit Analog-to-Digital Converter (ADC)** over the **SPI protocol**. 

This project implements an automated, simultaneous alert system designed to eliminate informational gaps in domestic and precision agriculture by constantly tracking plant micro-environments.

---

## 🚀 System Architecture & Hardware Components
Because the Raspberry Pi 5 lacks native analog input pins, this architecture uses the **MCP3008 ADC** as a hardware translator. The components are securely integrated into a custom, dual-chamber enclosure designed in **SolidWorks**.

* **Controller:** Raspberry Pi 5
* **ADC Bridge:** MCP3008 (Interfaced via native SPI Bus 0)
* **Sensors & Circuit Elements:**
  * **Soil Moisture Sensor:** Connected to **CH0** (Blue Wire) for volumetric water tracking
  * **Photoresistor (LDR):** Connected to **CH1** (Gray Wire) utilizing a **10kΩ Voltage Divider** circuit for luminance tracking
  * **TMP36 Temperature Sensor:** Connected to **CH2** (Brown Wire) for ambient thermal monitoring
* **Power Supply:** Official 5V/5A USB-C supply

---

## 📊 Core Features & Engineering Logic

### 1. SPI Protocol Integration
Data transfer is handled via the `spidev` library using a synchronous clock speed of **1MHz** to protect signal integrity. Raw 10-bit digital data values (ranging from `0` to `1023`) are pulled sequentially from the ADC channels.

### 2. Mathematical Scaling & DMM Calibration
Raw data is translated into standard engineering units ($V_{out}$, $\%$, and $°C$) using the following baseline resolution formula:

$$V_{out} = \left(\frac{\text{ADC Reading}}{1023}\right) \times 3.3V$$

* **Thermal Equation:** $Temp (°C) = (V_{out} - 0.5) \times 100$
* **Hardware Offset:** To counteract internal impedance shifts between the sensor and the ADC, a hardware-verified software calibration offset of **$-42.7°C$** is implemented based on external Digital Multimeter (DMM) reference testing.

### 3. Simultaneous Multi-Alert Logic
Unlike basic sequential check loops, the script utilizes a decoupled, independent `if` block structure. This allows the system to process data thresholds natively and print **multiple alert states simultaneously** if environmental boundaries are crossed.

| Metric | Critical Threshold | Console Alert Output |
| :--- | :--- | :--- |
| **Soil Moisture** | $< 30\%$ | `LOW WATER` |
| **Light Intensity** | $< 20\%$ | `LOW LIGHT` |
| **Temperature** | $> 35°C$ | `OVERHEATING` |
| **Nominal State** | All within bounds | `[STATUS: HEALTHY]` |

---

## 🛠️ Repository File Structure
* `/plant_monitor.py` — Fully commented main execution Python script containing SPI bus mapping, mathematical conversions, and threshold logic.
* `/System Schematic.png` — High-resolution hardware circuit drawing including complete wiring layout, pin assignments, and color codes.
* `/CAD/` — Folder containing the parametric 3D design components for the dual-chamber potting and electronics housing.

---

## 💻 How To Run
1. Ensure the SPI interface is enabled on your Raspberry Pi 5 via `sudo raspi-config`.
2. Install the required SPI dependencies:
   ```bash
   pip install spidev
