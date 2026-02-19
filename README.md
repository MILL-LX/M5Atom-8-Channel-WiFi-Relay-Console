# M5Atom 8-Channel WiFi Relay Console

This project consists of a wireless control console capable of managing 8 relays remotely. The system uses a Master-Slave architecture with low-latency communication (ESP-NOW) and features a hybrid interface: physical buttons for local control and a responsive WebApp for control via smartphone/PC.

## 📋 Project Description

The system comprises **one Master** (The Console) and **two Slaves** (The Actuators).
The Master reads 8 physical buttons and hosts a WebApp. Based on the inputs, it sends wireless commands to the Slaves, which trigger the relay modules via I2C.

### Key Features
*   **Hybrid Communication:** Uses **ESP-NOW** for instant device-to-device communication and **WiFi AP** to serve the WebApp simultaneously.
*   **Split Architecture:**
    *   **Buttons 1-4:** Control Slave 1.
    *   **Buttons 5-8:** Control Slave 2.
*   **Control Logic:**
    *   **Physical:** *Toggle* operation (press to turn ON, press to turn OFF).
    *   **Web:** Responsive interface with instant feedback.
*   **Safety (Watchdog):** Automatic timer that turns off any active relay after **30 seconds** of inactivity.
*   **Visual Feedback:** The M5Atom's RGB LED indicates communication status (Green = Success/Connected, Red = Error/Disconnected).
*   **Multicore Processing:** The Master utilizes both ESP32 cores (Core 0 for WiFi/Web, Core 1 for Buttons/Logic) to ensure zero latency.

## 🛠 Hardware Used

### Console (Master)
*   1x **M5Stack ATOM Lite** (ESP32-PICO).
*   8x **Pushbuttons** (SPST).
*   8x Resistors (for External Pull-Up configuration).
*   Connecting wires.

### Actuators (Slaves)
*   2x **M5Stack ATOM Lite**.
*   2x **M5Stack Unit 4Relay**.
*   Grove Cables (I2C Connection).

## 🔌 Pinout & Wiring

### Master (Buttons)
Buttons must be wired with an **External Pull-Up** configuration (Resistor connected to 3.3V, Button closes to GND).

| Button ID | M5Atom GPIO | Target |
| :--- | :--- | :--- |
| **1** | GPIO 21 | Slave 1 - Relay 1 |
| **2** | GPIO 25 | Slave 1 - Relay 2 |
| **3** | GPIO 23 | Slave 1 - Relay 3 |
| **4** | GPIO 33 | Slave 1 - Relay 4 |
| **5** | GPIO 22 | Slave 2 - Relay 1 |
| **6** | GPIO 19 | Slave 2 - Relay 2 |
| **7** | GPIO 26 | Slave 2 - Relay 3 |
| **8** | GPIO 32 | Slave 2 - Relay 4 |

### Slaves (Unit 4Relay)
Connection via the standard Grove port on the M5Atom Lite (I2C):
*   **SCL:** GPIO 32
*   **SDA:** GPIO 26
*   **Internal LED:** GPIO 27 (Controlled via FastLED).

## 📚 Libraries & Dependencies

The project was developed using the **Arduino IDE** (Board: M5Stack-ATOM). It is recommended to use the mentioned versions or higher:

| Library | Recommended Version | Function |
| :--- | :--- | :--- |
| **FastLED** | `3.6.0` | Control of the internal RGB LED (Pin 27) to avoid white flash issues. |
| **M5Unit-RELAY** | `0.0.2` | Control of the Unit 4Relay module. |
| **WiFi** | (Built-in ESP32) | Access Point and Station management. |
| **esp_now** | (Built-in ESP32) | Master-Slave communication protocol. |
| **WebServer** | (Built-in ESP32) | HTTP Server for the WebApp. |

> **Note:** The `M5Unified` or `M5Atom` libraries are **not** used to avoid conflicts with the LED startup state and GPIO management.

## 💻 Code Description

### Master (`Master.ino`)
*   **WiFi Setup:** Starts in `AP_STA` mode (Access Point + Station). Creates the "M5-Console" network on Channel 1.
*   **Tasks:**
    *   `loop()` (Core 1): Reads the 8 physical buttons with *debounce*, manages the 30s timeout for every channel, and sends commands via ESP-NOW.
    *   `TaskWeb` (Core 0): Handles asynchronous HTTP requests from the WebApp.
*   **WebApp:** HTML/JS interface stored in flash memory (`PROGMEM`). Features a **2-column layout** (Left: Slave 1, Right: Slave 2) with **rectangular buttons** (2:1 aspect ratio) optimized for mobile touch events.
*   **Feedback:** The RGB LED is managed via `FastLED` on Pin 27.

### Slaves (`Slave.ino`)
*   **Initialization:** Sets up the Unit 4Relay and the RGB LED.
*   **Communication:** Starts in Station mode on a fixed Channel (1) and registers a *callback* function for ESP-NOW.
*   **Action:** Upon receiving a packet, it verifies the Relay ID, triggers the corresponding hardware, and provides visual feedback (Green LED).

## 🚀 How to Use

1.  **Get MAC Addresses:**
    *   Before uploading the main code, open and upload the provided utility file: **`Read_MAC_ADD_M5Stack_ATOM_Lite.ino`**.
    *   Open the Serial Monitor (115200 baud) and write down the MAC Address displayed for each Slave unit.
2.  **Configuration:**
    *   Open the **Master** code.
    *   Update the variables `uint8_t slave1Address[]` and `uint8_t slave2Address[]` with the MAC addresses you obtained in step 1.
3.  **Flash:**
    *   Upload the **Master** code to the Console unit.
    *   Upload the **Slave** codes to the respective Actuator units.
4.  **Operation:**
    *   **Physical:** Press the buttons on the console to Toggle the relays.
    *   **Web:** Connect to the WiFi network `M5-Console` (Password: `12345678`), and access `192.168.4.1` in your browser.

## 📄 License

This project is open-source. Feel free to modify and adapt it to your needs.

---
*Author: Generated by AI Assistant guided by Mauricio Martins.*
