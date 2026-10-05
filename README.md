# HaloTrace

Secure telemetry for sensor networks using hardware entropy and information-theoretic encryption.

HaloTrace is an end-to-end IoT security demo that reads sensor data from an ESP32, generates true random pad material from hardware entropy, and protects telemetry with a One-Time Pad (OTP). The system includes a real-time dashboard to visualize raw values, encrypted payloads, entropy quality, and integrity verification in real time.

> [!IMPORTANT]
> This project is designed as an educational and demonstrative security system. The pad is intentionally transmitted alongside ciphertext for clarity and learning purposes. In a production deployment, pad material must be securely distributed or pre-shared using a trusted key-exchange process.

## Why HaloTrace?

Modern encryption systems are largely based on computational hardness assumptions. While practical today, those assumptions are vulnerable to future advances in cryptanalysis, including quantum computing.

HaloTrace explores a different idea: use physical randomness instead of mathematical complexity to secure measurement data.

- ESP32 generates entropy from hardware noise characteristics
- Fresh pad material is created per frame
- Sensor readings are quantized and masked with the pad
- HMAC-SHA256 verifies integrity and tamper detection
- A real-time dashboard exposes the full pipeline visually

---

## System Overview

```text
ESP32 Sensor Node        Backend Service           Live Dashboard
──────────────────     ───────────────────      ───────────────
     │                           │                          │
     │ Read sensor values         │                          │
     ├──────────────────────────▶│                          │
     │                           │ Quantize readings        │
     │                           ├────────────────────────▶│
     │                           │ Compute HMAC             │
     │                           │ Generate OTP pad         │
     │                           │ Encrypt payload          │
     │                           ├────────────────────────▶│
     │                           │ Decrypt + verify         │
     │                           ├────────────────────────▶│
     │                           │ Display live telemetry   │
     │                           │                          │
```

The underlying flow is:

```text
Sensor → Quantize → HMAC → OTP Mask → Transmit → Unmask → Verify → Visualize
```

### Why mod 257?

The implementation uses a prime modulus of 257 to keep arithmetic uniform and consistent for the OTP-style masking process. This keeps the encryption operation inside a bounded integer range and preserves the theoretical behavior needed for a clean demonstration of information-theoretic masking.

---

## Key Features

- Hardware entropy collection from ESP32 true random generation
- One-Time Pad style encryption for telemetry payloads
- Sensor quantization to discrete value bins
- HMAC-SHA256 integrity verification
- WebSocket streaming for live telemetry updates
- Real-time dashboard with charts, metrics, and tamper detection
- Graceful fallback to simulated sensor data when hardware is unavailable
- Entropy analysis for pad quality and uniformity assessment

---

## Project Structure

```text
├── firmware/
│   └── esp32_otp_sensor.ino      # ESP32 firmware for reading sensors and generating entropy
├── backend/
│   ├── main.py                   # FastAPI service with REST + WebSocket endpoints
│   ├── crypto_core.py            # Quantization, OTP logic, HMAC, entropy analysis
│   ├── serial_bridge.py          # Serial bridge for ESP32 communication
│   ├── simulator.py              # Simulation fallback when hardware is absent
│   ├── config.py                 # Tunable system parameters and channel definitions
│   ├── requirements.txt          # Python package dependencies
│   └── ...
├── dashboard/
│   ├── index.html                # Single-page telemetry interface
│   ├── style.css                 # Dashboard styling and HUD visuals
│   ├── app.js                    # Real-time WebSocket client and rendering logic
│   └── ...
├── README.md
└── LICENSE
```

---

## Quick Start

### Prerequisites

- Python 3.10+
- pip
- Optional: ESP32 board connected over USB

### 1) Install backend dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 2) Verify the crypto pipeline

```bash
cd backend
python crypto_core.py
```

Expected result:

```text
ALL SELF-TESTS PASSED ✓
```

### 3) Start the backend

```bash
cd backend
python main.py
```

The app starts a FastAPI service and automatically attempts to detect an ESP32 or fall back to a software simulation.

### 4) Open the dashboard

Visit:

```text
http://localhost:8000
```

---

## Dashboard Features

The dashboard presents the entire secure telemetry pipeline live:

- Sensor value grid and raw readings
- Quantized state values
- Pad bytes and ciphertext
- Decrypted output and validation results
- HMAC comparison and tamper detection
- Entropy histogram and quality metrics
- Packet log with recent telemetry frames
- Real-time charts for signal trends

---

## API Endpoints

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/api/status` | Current system health and runtime metadata |
| GET | `/api/config` | Public configuration and channel metadata |
| GET | `/api/latest` | Most recent telemetry snapshot |
| GET | `/api/entropy` | Entropy statistics and pad distribution |
| GET | `/api/log` | Recent telemetry log buffer |
| POST | `/api/tamper` | Trigger a deliberate corruption for testing |
| WS | `/ws/telemetry` | Real-time stream of encrypted telemetry frames |

---

## Sensor Channels

The project supports multiple telemetry channels, including environmental and electrical readings.

| Channel | Sensor | Unit |
| --- | --- | --- |
| 0 | Temperature | °C |
| 1 | Humidity | %RH |
| 2 | Pressure | hPa |
| 3 | Light | lux |
| 4 | CO₂ | ppm |
| 5 | Vibration | g |
| 6 | Voltage | V |
| 7 | Current | A |

---

## ESP32 Firmware

To run with actual hardware entropy:

1. Open `firmware/esp32_otp_sensor.ino` in Arduino IDE or PlatformIO
2. Select the ESP32 board profile
3. Set serial monitor baud rate to `115200`
4. Upload the firmware and watch the serial output

If no physical sensors are attached, the firmware can still emit synthetic readings while preserving the demonstration of the secure telemetry pipeline.

---

## Security Note

This system is intentionally educational and is not a production-ready replacement for mature cryptographic protocols such as AES-GCM or ChaCha20-Poly1305.

The project demonstrates a concept: secure telemetry can be built from physical randomness and integrity checks. The same idea should be implemented with proper key management, secure distribution, and hardened operational controls in real-world deployments.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

---

## Contributing

Pull requests and improvements are welcome. If you are extending the system, consider adding:

- better hardware entropy validation
- support for more sensors
- stronger security hardening for deployment scenarios
- additional dashboard analytics
- automated tests for crypto and transport flows
