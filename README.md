# RTL-HAOS: RTL-433 to Home Assistant Bridge


This project turns one or more RTL-SDR dongles into a Home Assistant-friendly sensor bridge. It also acts as a **System Monitor**, reporting the host machine's CPU, RAM, Disk, and Temperature stats to Home Assistant.

It:

- Runs `rtl_433` and parses its JSON output
- Normalizes and flattens sensor data
- Optionally averages/buffers readings to reduce noise
- Publishes everything to an MQTT broker using **Home Assistant MQTT Discovery**
- Publishes **system/bridge diagnostics** (CPU, RAM, disk, host model, IP, device list)

The goal is a “drop-in” bridge: plug in RTL-SDR(s), run this script, and watch devices appear in Home Assistant with clean names, units, and icons.

## ✨ Features

* **MQTT Auto-Discovery:** Sensors appear automatically in Home Assistant without manual YAML configuration.
* **Field metadata**: units, device_class, icons, friendly names (for HA) 
* **System monitor**:
  - CPU %, RAM %, disk %, CPU temp, script memory
  - Bridge uptime, OS version, model, IP
  - Count/list of active RF devices
* **Dew point calculation** derived from temperature + humidity
* **Filtering:** Built-in Whitelist and Blacklist support to ignore neighbor's sensors.
* **Multi-Radio Support:** Can manage multiple SDR dongles on different frequencies simultaneously.
* **Data Averaging:** Buffers and averages sensor readings over a set interval (e.g., 30s) to reduce database noise.

---

## 📂 Project Layout

* `rtl_mqtt_bridge.py`: Main entry point. Manages threads for `rtl_433`, buffering, and system stats.
* `config.py`: **User-Editable Settings** (Radios, Filters, MQTT).
* `mqtt_handler.py`: Handles MQTT connection and Home Assistant Auto-Discovery.
* `field_meta.py`: Definitions for icons, units, and device classes.
* `system_monitor.py`: Collects hardware metrics (CPU/RAM/Disk).

---

## 🛠️ Hardware Requirements

* **Host:** Raspberry Pi (3/4/5), Mini PC, or any Linux machine.
* **Radio:** RTL-SDR USB Dongle (RTL-SDR Blog V3, Nooelec, etc.).

---

## 🚀 Installation

### 1. System Dependencies
Install Python and the `rtl_433` binary.

```bash
# Debian / Ubuntu / Raspberry Pi OS
sudo apt update
sudo apt install -y rtl-433 git python3 python3-pip python3-venv libatlas-base-dev
```

> **⚠️ Important:** Ensure your user has permission to access the USB stick!
> If you haven't already, install the rtl-sdr udev rules:
> `sudo apt install rtl-sdr` (or download the rules manually). Unplug and replug your dongle after installing.

### 2. Clone & Setup
```bash
git clone https://github.com/jaronmcd/rtl-haos.git
cd rtl-haos

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python requirements
pip3 install -r requirements.txt
```

### 3. Configuration
Copy the example config and edit it.

```bash
cp config.example.py config.py
nano config.py
```

**Key Configuration Examples:**

```python
# --- MQTT ---
MQTT_SETTINGS = {
    "host": "192.168.1.100",
    "user": "mqtt_user",
    "pass": "password"
}

# --- MULTI-RADIO SETUP ---
# You can define multiple radios here.
RTL_CONFIG = [
    {"name": "Weather Radio", "id": "0", "freq": "433.92M", "rate": "250k"},
    {"name": "Utility Meter", "id": "1", "freq": "915M",    "rate": "250k"},
]

# --- FILTERING ---
# Ignore specific neighbors by ID or Model
DEVICE_BLACKLIST = [
    "SimpliSafe*",
    "EezTire*",
]
```

---

## ▶️ Usage

Run the bridge manually to test connection:

```bash
source venv/bin/activate
python3 rtl_mqtt_bridge.py
```

**Expected Output:**
```text
[STARTUP] Connecting to MQTT Broker at 192.168.1.100...
[MQTT] Connected Successfully.
[RTL] Manager started for Weather Radio. Monitoring...
 -> TX RaspberryPi (b827eb...) [radio_status_0]: Scanning...
 -> TX RaspberryPi (b827eb...) [radio_status_0]: Online
 -> TX Acurite-5n1 (1234) [temperature]: 72.3
```

---

## 🤖 Running as a Service

To keep the bridge running 24/7, use `systemd`.

1.  **Create the service file:**
    ```bash
    sudo nano /etc/systemd/system/rtl-bridge.service
    ```

2.  **Paste the configuration** (Update paths to match your username!):
    ```ini
    [Unit]
    Description=RTL-HAOS MQTT Bridge
    After=network.target

    [Service]
    Type=simple
    User=pi
    # UPDATE THIS PATH
    WorkingDirectory=/home/pi/rtl-haos
    # UPDATE THIS PATH
    ExecStart=/home/pi/rtl-haos/venv/bin/python /home/pi/rtl-haos/rtl_mqtt_bridge.py
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```

3.  **Enable and Start:**
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable rtl-bridge.service
    sudo systemctl start rtl-bridge.service
    ```



