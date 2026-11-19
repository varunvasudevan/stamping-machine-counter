# 🏭 Stamping Machine Production Counter

**Real-time stroke counting for stamping machines, with counts sent to a central server and visible to every location on the network.**

Each stamping machine has an IR sensor that detects every stroke. An ESP32 with wired Ethernet reads up to six machines and sends one event per stroke to a server on a Raspberry Pi. The server stores every count in MySQL and pushes live totals to dashboards over WebSockets, so supervisors, the production office and other plant locations can monitor output as it happens. Counts reset automatically at the start of each day's shift and survive reboots and power cuts.

> ℹ️ This is a showcase repository. The source code is private; this page describes the design, features and tech used. Device IDs, MAC/IP addresses, counts and screenshots are sample data, and the screenshots are illustrative dashboard mockups.

---

## 📸 Screenshots

### Live production dashboard
Per-machine stroke totals with today's increments, plus device, database and next-reset status.

![Live production dashboard](dashboard.png)

### Timestamped stroke history
Every stroke is stored as its own event, along with the daily reset marker.

![Stroke history](event-history.png)

### Device and machine status
The counter device, its network addresses and the state of each machine's sensor input.

![Device and machine status](device-status.png)

---

## 🏗️ Architecture

```text
Stamping machines (1..6) with IR sensors
       |
       v
ESP32 + W5500 Ethernet  --- HTTP over LAN --->  Raspberry Pi server (FastAPI)
                                                    |              \
                                                    v               \ WebSocket
                                                  MySQL              v
                                              Dashboards at other locations
                                              (production office, supervisors,
                                               management)
```

The ESP32 sends a **stroke event** for each detection rather than a locally kept total. The server is the single source of truth, so a counter reboot never loses or double-counts production.

## ✨ Features

| Area | What it does |
|---|---|
| **Stroke counting** | Up to six stamping machines per counter, each with its own IR sensor, with edge detection and debouncing |
| **Wired network** | ESP32 + W5500 Ethernet for reliable shop-floor communication; failed sends are queued and retried |
| **Central server** | FastAPI service for stroke events, current totals and date-wise history |
| **Storage** | MySQL with a complete stroke log plus daily totals per machine |
| **Multi-location monitoring** | Live WebSocket dashboards on any system on the network |
| **Daily reset** | Automatic reset at 07:30 (shift start), runs once per day even after restarts, and keeps all history |
| **Recovery** | Restores the day's totals after a restart or power interruption |

## 🧠 Design decisions

- **Events, not totals:** each stroke is one transactional insert plus a counter increment, so history and totals always agree.
- **Reset once per day:** a unique reset marker per machine and date stops a reboot from resetting counts twice.
- **Saved before acknowledging:** a stroke is written to MySQL before the counter gets a success response.
- **Stable device identity:** each counter is registered with one fixed identifier.
- **Shop-floor safety:** 3.3 V logic, with level shifting for 5 V or industrial sensor outputs.

## 🛠️ Tech Stack

![ESP32](https://img.shields.io/badge/ESP32-E7352C?style=flat&logo=espressif&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00878F?style=flat&logo=arduino&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat&logo=raspberrypi&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)
![WebSockets](https://img.shields.io/badge/WebSockets-010101?style=flat&logo=socketdotio&logoColor=white)

- ESP32 firmware (Arduino) with a W5500 SPI Ethernet module
- Raspberry Pi running FastAPI as a system service
- MySQL for the stroke log and daily totals
- WebSockets for live dashboards
- HTML/JavaScript dashboard

## 🗺️ Roadmap

- Shift-wise totals and targets vs actual
- Machine downtime detection from stroke gaps
- Per-device API keys and device registration
- Multi-line, multi-plant dashboards
- CSV and Excel export of daily and shift-wise production

## 👤 My role

Designed the system end to end: sensor wiring and ESP32 firmware, the network event protocol, FastAPI server, MySQL schema, daily reset and recovery logic, and the live monitoring dashboard.

---

📫 Want to know more? See my [portfolio](https://varunvasudevan.github.io).
