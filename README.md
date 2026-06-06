#  🛣️ AI-BASED-ROAD-SURFACE-DETECTION-SYSTEM

> An AI-powered embedded system for automated road crack and pothole detection using ESP32-CAM, Edge Impulse ML, and GSM SIM800L — with real-time image processing and instant authority alerts.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [ML Model Pipeline](#ml-model-pipeline)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Road surfaces across highways and urban networks deteriorate due to heavy traffic loads, environmental factors, and deferred maintenance — leading to potholes and cracks that cause accidents and vehicle damage. Traditional inspection relies on manual surveys that are slow, costly, and impractical at scale.

This project solves that with an **automated, real-time road defect detection system** built on embedded edge AI. The ESP32-CAM continuously captures road surface images and runs an on-device machine learning model trained via Edge Impulse to classify defects. Upon detecting a pothole or crack with sufficient confidence, the system dispatches an SMS alert with fault type, confidence score, and an image link to authorities via GSM SIM800L — **all without cloud dependency or human intervention**.

---

## Features

- ✅ Fully automated road defect detection — no human intervention required
- ✅ On-device Edge AI inference via Edge Impulse (no cloud needed)
- ✅ Real-time image capture using OV2640 camera (320×240 QVGA)
- ✅ ML model achieves **~95.7% F1 Score** on crack and pothole detection
- ✅ SMS alerts via GSM SIM800L with fault type, confidence, and image URL
- ✅ Detected images stored locally in SPIFFS flash memory
- ✅ Data logged to Google Sheets + images hosted on IMGBB for remote review
- ✅ Works without internet connectivity (GSM-based alerting)
- ✅ Compact, low-cost, and scalable for large highway networks

---

## System Architecture

The system is divided into three functional layers:

```
┌─────────────────────────────────────────────────────────────┐
│  SENSING LAYER       OV2640 Camera (onboard ESP32-CAM)      │
│                      Real-time QVGA image capture           │
├─────────────────────────────────────────────────────────────┤
│  PROCESSING LAYER    ESP32 (Dual-Core Xtensa LX6 @ 240 MHz) │
│                      Edge Impulse ML Model (TFLite int8)    │
│                      Image preprocessing (DSP block)        │
├─────────────────────────────────────────────────────────────┤
│  COMMUNICATION LAYER GSM SIM800L (AT Commands via UART)     │
│                      SMS alert → Authorities                │
│                      IMGBB image upload + Google Sheets log │
└─────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
OV2640 Camera (onboard)
        │
        ▼
ESP32-CAM Module ──► Edge Impulse ML Model (on-device inference)
        │                      │
        │              Defect Detected?
        │                      │
        ├──► SPIFFS ◄──────────┘  (save image locally)
        │
        └──► GSM SIM800L ──────────────────────► SMS Alert to:
                                                  1. Municipal Authorities
                                                  2. Highway Maintenance Teams
                                                  3. Road Safety Officers

             ┌──────────────┐
             │ IMGBB Website│  (image hosting)
             └──────────────┘
             ┌──────────────┐
             │ Google Sheets│  (fault log: timestamp, type, accuracy, image URL)
             └──────────────┘
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Microcontroller | ESP32-CAM (AI Thinker) | — | Central processing + image capture |
| Camera Sensor | OV2640 (onboard) | Parallel + I2C | Real-time road image acquisition |
| GSM Module | SIM800L | UART (GPIO-1,3) | SMS alert to authorities |
| Power Supply | 5V regulated | — | System power |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **Arduino IDE** | Firmware development, compilation, and flashing |
| **Edge Impulse** | Dataset collection, ML model training, and export |
| **SPIFFS** | Local flash storage for detected images |
| **IMGBB API** | Cloud image hosting for remote viewing |
| **Google Sheets** | Fault log (timestamp, type, accuracy, image URL) |

### Arduino Libraries Used

| Library | Header File | Purpose |
|---|---|---|
| Edge Impulse Model | `road_damage_detection_inferencing.h` | On-device AI inference |
| Edge Impulse DSP | `image.hpp` | Image resize, crop, format conversion |
| ESP32 Camera | `esp_camera.h` | Camera init and frame capture |
| Hardware Serial | `HardwareSerial.h` | UART communication with GSM |
| SPIFFS | `SPIFFS.h` | Save detected images to flash |

---

## Circuit & Pin Connections

### OV2640 Camera → ESP32 (onboard — pre-wired)

| OV2640 Signal | ESP32 GPIO | Description |
|---|---|---|
| D0–D7 | GPIO 5, 18, 19, 21, 36, 39 | Parallel image data lines |
| XCLK | GPIO0 | External clock to camera |
| PCLK | GPIO22 | Pixel clock |
| VSYNC | GPIO25 | Frame sync signal |
| HREF | GPIO23 | Line sync signal |
| SDA (SIOD) | GPIO26 | I2C data for camera config |
| SCL (SIOC) | GPIO27 | I2C clock for camera config |
| PWDN | GPIO32 | Power-down control |

### GSM SIM800L → ESP32-CAM (UART)

| SIM800L Pin | ESP32 Pin | Description |
|---|---|---|
| VCC | 3.7–4.2V (Li-ion) | Power |
| GND | GND | Ground |
| TXD | GPIO3 (RX) | GSM data to ESP32 |
| RXD | GPIO1 (TX) | Commands from ESP32 |
| RST | Optional GPIO | Module reset |

> ⚠️ **Note:** On ESP32-CAM, GPIO0 must be pulled LOW for programming mode. Many GPIO pins are internally used by the camera — GPIO1 and GPIO3 are the only available UART pins for GSM.

---

## How It Works

1. **Boot** — ESP32-CAM initializes SPIFFS, camera (QVGA 320×240), and GSM module.
2. **Continuous Capture** — OV2640 captures road surface images in real time.
3. **Preprocessing** — Images are converted from JPEG → RGB888, resized, cropped, and normalized to match the model's input format.
4. **Inference** — The embedded Edge Impulse model runs `run_classifier()` on-device to detect cracks, potholes, or background (normal road).
5. **Confidence Check** — A confidence threshold filters out false positives before triggering an alert.
6. **Alert Dispatch** — On valid detection, the ESP32 sends AT commands to SIM800L to dispatch an SMS containing:
   - Fault type (crack / pothole)
   - Confidence score
   - IMGBB-hosted image URL
7. **Data Logging** — Detection records (timestamp, fault type, accuracy, image URL) are logged to Google Sheets.
8. **Image Storage** — Detected frame saved locally as `/detected.jpg` in SPIFFS.

### Detection Logic (simplified)

```c
// Run Edge Impulse classifier on captured frame
ei_impulse_result_t result;
run_classifier(&signal, &result, false);

// Check bounding box detections
for (size_t i = 0; i < result.bounding_boxes_count; i++) {
    ei_impulse_result_bounding_box_t bb = result.bounding_boxes[i];
    if (bb.value >= CONFIDENCE_THRESHOLD) {
        save_detected_image();   // store to SPIFFS
        gsmsend();               // trigger SMS alert via SIM800L
    }
}
```

### GSM Alert via AT Commands

```
AT                               // Check communication
AT+CMGF=1                        // Set SMS text mode
AT+CMGS="+91XXXXXXXXXX"          // Set recipient number
> Road Damage! Type: crack       // Message body
  Conf: 73.8%
  Img: https://i.ibb.co/...
[CTRL+Z]                         // Send
```

### Performance Metrics

| Parameter | Value |
|---|---|
| ML Model F1 Score | ~95.7% |
| Training Dataset Size | 75 labeled images |
| Training Cycles | 60 epochs |
| Learning Rate | 0.01 |
| Model Format (deployed) | TensorFlow Lite int8 |
| Camera Resolution | QVGA (320 × 240) |
| Detection Classes | Crack, Pothole, Background |

---

## ML Model Pipeline

The model was built using the **Edge Impulse** platform end-to-end:

1. **Data Collection** — ~75 road surface images labeled as `crack`, `pothole`, or `background` (77% train / 23% test split).
2. **DSP Block** — Images processed through the Image DSP block: resizing, normalization, feature vector generation.
3. **Model Architecture** — Object Detection Neural Network (FOMO-based) trained to output bounding boxes + confidence scores.
4. **Training** — 60 cycles, learning rate 0.01, data augmentation enabled.
5. **Evaluation** — Validated against test set; confusion matrix confirmed minimal misclassification especially for potholes (F1 = 1.00 for pothole class).
6. **Deployment** — Exported as Arduino-compatible C++ library and compiled directly into ESP32-CAM firmware.

---

## Results

The system was successfully tested on hardware. Key outcomes:

- The Edge Impulse model achieved an **F1 Score of ~95.7%**, with perfect pothole detection (F1 = 1.00) and strong crack detection (F1 = 0.91).
- The ESP32-CAM performed real-time on-device inference without any cloud dependency, reducing latency significantly.
- The GSM SIM800L successfully dispatched SMS alerts containing fault type, confidence percentage, and an IMGBB-hosted image link.
- Detected images were correctly saved to SPIFFS flash memory.
- Google Sheets logging captured timestamped records of all detections for remote review.
- The multi-step confidence threshold approach reduced false positive alerts.

---

## Future Scope

- **Larger Dataset** — Expand training data across varied lighting, weather, and road surface types for better generalization
- **Advanced Models** — Explore YOLOv8-nano or MobileNet-based architectures with model quantization and pruning for higher accuracy on-device
- **GPS Integration** — Tag exact defect coordinates for GIS-based road condition mapping
- **Cloud Dashboard** — Centralized monitoring portal for municipal authorities across multiple locations
- **Autonomous Vehicle Integration** — Feed detection data to ADAS or self-driving systems for proactive route adjustment
- **Solar Power** — Enable continuous outdoor deployment with solar-powered operation
- **Mobile App** — Real-time defect notifications for road users and maintenance crews
- **Extended Classes** — Detect road markings, speed breakers, waterlogging, and surface wear

---

> *Built to automate road infrastructure monitoring using edge AI — making India's highways safer, one frame at a time.*
