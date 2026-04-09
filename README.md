# Panoramic Vision Helmet

> A real-time embedded system that gives racing drivers panoramic rear vision and a heads-up display — directly integrated into the helmet.

**University of Balamand — Computer Engineering Senior Design Project, Dec 2025**  
*Charbel Mouawad · Karim EL Badaoui EL Najjar · Shaya Melhem*

---

## The Problem

Racing helmets and vehicle bodywork create significant blind spots. During overtaking and high-speed racing, drivers have no reliable way to assess what is behind or beside them without compromising focus on the road ahead.

---

## Solution

A wearable helmet system with two HD cameras, real-time panoramic image stitching, ultrasonic proximity sensing, and an AR heads-up display — all processed on-board with no external compute or cloud dependency.

The driver sees a stitched rear panorama and a color-coded proximity alert overlay directly in their field of view, alongside racing-context flags (clear track, hazard, overlap, danger, finish).

---

## System Architecture

Three concurrent software threads run on the Jetson Orin Nano:

```
┌─────────────────────────────────────────────────────┐
│                  Jetson Orin Nano                   │
│                                                     │
│  Thread 1            Thread 2          Thread 3     │
│  Panoramic           Sensor Data       HUD          │
│  Stitching     ←──   Acquisition  ──→  Rendering    │
│  (OpenCV/VPI)        (Arduino/UART)    (OpenCV)     │
└──────┬───────────────────────┬─────────────┬────────┘
       │                       │             │
  2x HD Webcams         2x HC-SR04    Flexible Display
                         Ultrasonic
                         (via Arduino Uno)
```

---

## Threads

### Thread 1 — Panoramic Image Stitching

Takes live feeds from two side-mounted cameras and produces a single stitched panoramic frame in real time.

**Key functions:** `rot_y()` (camera orientation modeling), `warpPerspective()`, `perspectiveTransform()`, `build_alpha()` (seam smoothing), alpha-blended stitching  
**Libraries:** OpenCV (`cv2`), NVIDIA VPI, NumPy

### Thread 2 — Sensor Data Acquisition (Arduino)

Two HC-SR04 ultrasonic sensors measure left and right distances at 10 Hz. The Arduino converts raw pulse timings to centimeters and streams values to the Jetson over UART.

**Key functions:** `pulseIn()`, `digitalWrite()`, `Serial.print()`  
**Libraries:** Arduino Core, Serial Communication

### Thread 3 — HUD Rendering & Awareness Logic

Composites the panoramic feed with a real-time overlay showing:

**Proximity zones (left & right):**

| Color | Zone | Distance |
|---|---|---|
| 🟢 Green | Safe | > 3 m |
| 🟡 Yellow | Caution | 2 – 3 m |
| 🟠 Orange | Unsafe | 1 – 2 m |
| 🔴 Red | Immediate Danger | < 1 m |

**Racing flag bar:** track clear · hazard ahead · car overlapping · serious danger · race finished · warning

Also displays speed, RPM, lap count, and position readouts.

**Key functions:** `cv2.rectangle()`, `cv2.putText()`, `cv2.addWeighted()`  
**Library:** OpenCV (`cv2`)

---

## Hardware

| Component | Purpose |
|---|---|
| NVIDIA Jetson Orin Nano | Main compute — stitching, HUD, awareness logic |
| 2× HD Webcams | Left and right rear camera feeds |
| Arduino Uno | Ultrasonic sensor interface |
| 2× HC-SR04 Ultrasonic Sensors | Left and right proximity measurement |
| Flexible Display | AR HUD output inside helmet |
| USB Hub | Camera + Arduino connectivity |
| Battery Pack + 5V Regulator | Portable on-helmet power |

**Communication protocols:** UART (Arduino → Jetson), USB (cameras), Mini-HDMI (display), I²C, SPI

---

## Performance

| Metric | Value |
|---|---|
| Frame Rate | ~19.5 FPS |
| End-to-End Latency | ~100 ms |
| Output Resolution | 1280 × 720 px |
| Power Consumption | 23 – 28 W |
| System Weight | ~2.5 kg |

---

## Repository Structure

```
panoramic-vision-helmet/
├── src/
│   ├── stitching/          # Thread 1 — panoramic image stitching
│   ├── hud/                # Thread 3 — HUD rendering and awareness logic
│   └── arduino/            # Thread 2 — ultrasonic sensor firmware (.ino)
├── cad/                    # Helmet mount and enclosure files
└── README.md
```

---

## Setup

### Requirements

- NVIDIA Jetson Orin Nano, JetPack 5.x
- Python 3.8+ with OpenCV, NumPy, NVIDIA VPI
- Arduino IDE (for firmware upload)

### Run on Jetson

```bash
git clone https://github.com/yourusername/panoramic-vision-helmet.git
cd panoramic-vision-helmet
pip install -r requirements.txt
python src/main.py
```

### Arduino Firmware

1. Open `src/arduino/distance_sensor.ino` in Arduino IDE
2. Select board (Uno) and port
3. Upload — the sketch begins streaming distance data over UART automatically

---

## Results

- Functional prototype demonstrated at University of Balamand, December 2025
- Real-time panoramic video with integrated HUD achieved on embedded hardware
- System feasibility validated for driver assistance applications

---

## Future Work

- Mobile app for real-time telemetry data acquisition
- Improved frame rate and reduced latency
- Better enclosure ergonomics and weight distribution
- Wireless communication integration

---

## Standards & Compliance

IEEE 1149.1 · ISO 9001 · IEEE 488 · IEEE P3123 · UN Sustainable Development Goals (SDG 3, SDG 9)

---

## License

© 2025 Charbel Mouawad, Karim EL Badaoui EL Najjar, Shaya Melhem — All Rights Reserved  
University of Balamand, Department of Computer Engineering
