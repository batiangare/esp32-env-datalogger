# 🌡️ IoT Smart Environmental Monitor & Data Logger

An advanced, standalone IoT monitoring system built on the ESP32 microcontroller. This project is designed to precisely track environmental conditions (temperature and humidity) while ensuring high system reliability through self-monitoring features like power tracking and hardware watchdogs.

## ✨ Key Features
* **Real-time IoT Dashboard:** Live remote monitoring via the Blynk platform.
* **Automated Data Logging:** Environmental data is saved locally to a MicroSD card (`serverdatalog.csv`) every 1 minute.
* **Accurate Timestamping:** Integrated with an NTP server (`pool.ntp.org`) to ensure all logged data has accurate real-world timestamps.
* **Power Monitoring:** Utilizes an INA219 sensor to track bus voltage. Features an automated low-power alert (triggers an LED if voltage drops below 6.5V).
* **High Reliability (Watchdog Timer):** Implements a Hardware Watchdog Timer (WDT) and auto-reconnect protocols to automatically reboot the ESP32 in case of system freezes or WiFi dropouts, ensuring 24/7 continuous operation.
* **Local Display:** Real-time data visualization on an I2C 16x2 LCD.

## 🛠️ Hardware Components
* Microcontroller: ESP32
* Environmental Sensor: DHT22 (Pin 4)
* Power/Voltage Sensor: Adafruit INA219 (I2C)
* Storage: MicroSD Card Adapter (SPI, CS Pin 5)
* Display: 16x2 LCD with I2C Backpack
* Indicator: 5mm LED (Pin 12) for low-voltage alert

## 📌 Pin Configuration Summary
| Component | ESP32 Pin |
| :--- | :--- |
| DHT22 Data | GPIO 4 |
| SD Card CS | GPIO 5 |
| I2C SDA (LCD & INA219) | GPIO 21 |
| I2C SCL (LCD & INA219) | GPIO 22 |
| Alert LED | GPIO 12 |

## 🚀 Setup & Installation
1. Install the required libraries in the Arduino IDE (`BlynkSimpleEsp32`, `DHT sensor library`, `LiquidCrystal_I2C`, `Adafruit_INA219`).
2. Open the source code and update the **WiFi credentials** (`ssid` and `password`).
3. Update the **Blynk Auth Token** with the token generated from your Blynk Web Dashboard.
4. Ensure a MicroSD card (formatted to FAT32) is inserted into the module.
5. Compile and upload the code to the ESP32.

## 📂 Repository Structure
* `maincode.txt` : Main source code for the ESP32 containing all sensor integrations, logging, and IoT logic.
* `Circuit_Schematic.fzz` : Fritzing wiring schematic.
* `Gerber_Files_JLCPCB.zip` : Ready-to-print PCB production files.
* `3D_Enclosure_Design.zip` : 3D models for the device casing.
