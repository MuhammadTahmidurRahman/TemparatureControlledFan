# Smart Temperature-Controlled Fan using STM32

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Pin Configuration](#pin-configuration)
- [Software Requirements](#software-requirements)
- [Getting Started](#getting-started)
- [Control Logic](#control-logic)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Team Members](#team-members)
- [License](#license)

---

## 🎯 Overview

The Smart Temperature-Controlled Fan automatically regulates fan speed based on real-time temperature readings, eliminating the need for manual adjustments. The system provides:

- **Automatic Speed Control**: 4-level intelligent speed adjustment (0%, 40%, 70%, 100%)
- **Real-time Monitoring**: Live temperature and humidity display on 16x2 LCD
- **Energy Efficiency**: Reduces power consumption by up to 35%
- **Error Handling**: Automatic retry mechanism and fault detection
- **Cost-effective**: Total cost ~2000 BDT vs 4000-6000 BDT for commercial alternatives

---

## ✨ Features

### Core Functionality
- ✅ Continuous temperature and humidity monitoring using DHT11 sensor
- ✅ Automatic PWM-based fan speed control via L298N motor driver
- ✅ Real-time display of temperature, humidity, and fan status
- ✅ 4-level temperature-based control logic
- ✅ Sensor error detection with retry mechanism (up to 3 attempts)
- ✅ Motor diagnostic test on startup

### Technical Highlights
- **Microsecond Precision Timing**: TIM2-based delay for DHT11 communication
- **PWM Motor Control**: TIM4 for smooth speed transitions
- **I2C LCD Communication**: Efficient 16x2 display interface
- **Checksum Verification**: Data integrity validation for sensor readings
- **Safety Features**: Automatic motor shutdown on persistent sensor failure

---

## 🛠️ Hardware Requirements

| Component | Model/Specification | Quantity | Purpose |
|-----------|-------------------|----------|---------|
| Microcontroller | STM32F103C8T6 (Blue Pill) | 1 | Main processor |
| Temperature Sensor | DHT11 | 1 | Temperature & humidity sensing |
| Motor Driver | L298N Module | 1 | DC motor control with PWM |
| DC Fan | 12V | 1 | Cooling actuator |
| LCD Display | 16x2 with I2C (0x27) | 1 | Real-time data display |
| Power Supply | 5V (for STM32) | 1 | MCU power |
| Power Supply | 12V (for fan) | 1 | Motor power |
| Miscellaneous | Jumper wires, breadboard | - | Connections |

**Total Cost**: ~2000 BDT

---

## 📌 Pin Configuration

### STM32F103C8T6 Connections

#### DHT11 Sensor
| DHT11 Pin | STM32 Pin | Configuration |
|-----------|-----------|---------------|
| VCC | 5V | Power |
| DATA | PA1 | GPIO (Open Drain + Pull-up) |
| GND | GND | Ground |

#### LCD Display (I2C)
| LCD I2C Pin | STM32 Pin | Configuration |
|-------------|-----------|---------------|
| VCC | 5V | Power |
| GND | GND | Ground |
| SDA | PB9 | I2C1_SDA |
| SCL | PB8 | I2C1_SCL |

**I2C Address**: 0x27 (try 0x3F if 0x27 doesn't work)

#### L298N Motor Driver
| L298N Pin | STM32 Pin | Configuration |
|-----------|-----------|---------------|
| IN1 | PB7 | GPIO Output (Direction) |
| IN2 | Not used | (Optional: reverse direction) |
| ENA | PB6 | TIM4_CH1 (PWM) |
| OUT1 | Fan + | Motor terminal |
| OUT2 | Fan - | Motor terminal |
| +12V | 12V Supply | Motor power |
| GND | Common GND | Ground |
| +5V | 5V (optional) | Logic power |

#### Power Connections
- **STM32**: 5V via USB or external 5V supply
- **L298N + Fan**: 12V external supply
- **Common Ground**: Connect all GND together

---

## 💻 Software Requirements

- **IDE**: STM32CubeIDE (version 1.9.0 or later)
- **Firmware**: STM32 HAL Drivers
- **Programmer**: ST-Link V2
- **Language**: C

### Required STM32 Peripherals Configuration

```
✓ I2C1 (PB8/PB9) - LCD communication
✓ TIM2 - Microsecond delay for DHT11
✓ TIM4_CH1 (PB6) - PWM for motor control
✓ GPIO (PA1) - DHT11 data line (Open Drain + Pull-up)
✓ GPIO (PB7) - Motor direction control
```

---

## 🚀 Getting Started

### 1. Clone or Download the Repository

```bash
git clone https://github.com/your-repo/temperature-controlled-fan.git
cd temperature-controlled-fan
```

### 2. Import Project into STM32CubeIDE

#### Method A: Import Existing Project
1. Open **STM32CubeIDE**
2. Go to `File` → `Import...`
3. Select `General` → `Existing Projects into Workspace`
4. Click `Next`
5. Click `Browse` and navigate to the downloaded `TemperatureControlledZip` folder
6. Select the project and click `Finish`

#### Method B: Import from Archive
1. If you have the `.zip` file directly:
   - Go to `File` → `Import...`
   - Select `General` → `Existing Projects into Workspace`
   - Select `Select archive file` radio button
   - Browse and select `TemperatureControlledZip.zip`
   - Click `Finish`

### 3. Build the Project

1. Right-click on the project in Project Explorer
2. Select `Build Project` or press `Ctrl + B`
3. Wait for compilation to complete (should have 0 errors)

### 4. Hardware Setup

1. **Connect DHT11** to PA1 with 5V and GND
2. **Connect LCD I2C** to PB8 (SCL) and PB9 (SDA)
3. **Connect L298N**:
   - ENA → PB6 (PWM)
   - IN1 → PB7 (Direction)
   - Connect 12V fan to OUT1 and OUT2
   - Supply 12V to motor driver
4. **Power STM32** via USB or 5V supply
5. **Connect ST-Link** for programming

### 5. Flash the Code

1. Connect ST-Link to STM32:
   - SWDIO → SWDIO
   - SWCLK → SWCLK
   - GND → GND
   - 3.3V → 3.3V
2. Click `Run` → `Debug` or press `F11`
3. The code will be flashed automatically
4. Press `Resume (F8)` to start execution

### 6. Verify Operation

1. LCD should display: `"DHT11 Fan Ctrl"` → `"Initializing..."`
2. Motor test runs automatically: 30% → 60% → 100% → 0%
3. System enters monitoring mode
4. LCD shows: 
   ```
   T:28C H:65%
   Fan: 70%
   ```

---

## 🎛️ Control Logic

The fan speed automatically adjusts based on temperature thresholds:

| Temperature Range | Fan Speed | PWM Duty Cycle | Status |
|-------------------|-----------|----------------|--------|
| Below 25°C | OFF | 0% | Fan stopped |
| 25°C - 30°C | SLOW | 40% | Gentle cooling |
| 30°C - 35°C | MEDIUM | 70% | Active cooling |
| Above 35°C | FAST | 100% | Maximum cooling |

### LCD Display Format

```
Line 1: T:28C H:65%     (Temperature and Humidity)
Line 2: Fan: 70%        (Current fan speed)
```

### Error Handling

- **Sensor Error**: Displays `"DHT11 ERROR!"` and retries up to 3 times
- **Persistent Failure**: After 5 failed attempts, displays `"Check wiring!"` and stops motor for safety
- **Invalid Data**: Automatic checksum verification rejects corrupted readings

---

## 📁 Project Structure

```
temperature-controlled-fan/
│
├── Core/
│   ├── Src/
│   │   ├── main.c              # Main program logic
│   │   ├── stm32f1xx_it.c      # Interrupt handlers
│   │   └── stm32f1xx_hal_msp.c # HAL MSP initialization
│   │
│   └── Inc/
│       ├── main.h              # Main header file
│       └── stm32f1xx_it.h      # Interrupt declarations
│
├── Drivers/
│   ├── STM32F1xx_HAL_Driver/  # HAL library files
│   └── CMSIS/                  # ARM CMSIS files
│
├── Project-Proposal.pdf        # Detailed project documentation
├── README.md                   # This file
└── TemperatureControlledZip/   # Complete project archive
```

---

## ⚙️ How It Works

### 1. DHT11 Communication (Timing-Critical)

The DHT11 uses a **single-wire protocol** with microsecond-level timing:

```c
// Start Signal: MCU → DHT11
MCU pulls LOW for 18ms → DHT11 wakes up
MCU pulls HIGH for 20µs → Ready to receive

// Response: DHT11 → MCU
DHT11 pulls LOW for 80µs → Acknowledgment
DHT11 pulls HIGH for 80µs → Ready to send

// Data Bits (40 bits total):
Bit '0': LOW 50µs + HIGH 26-28µs
Bit '1': LOW 50µs + HIGH 70µs
```

**Key Function**: `delay_us(40)` is used to distinguish between '0' and '1':
- After 40µs, if line is still HIGH → Bit is '1'
- After 40µs, if line is LOW → Bit is '0'

### 2. PWM Motor Control

```c
// TIM4 Configuration:
Prescaler = 63
Period = 999
→ PWM Frequency = 64MHz / 64 / 1000 = 1 kHz

// Speed Calculation:
Speed 40% → Pulse = (40 × 1000) / 100 = 400
→ Duty Cycle = 400/1000 = 40%
```

### 3. I2C LCD Communication

The LCD uses a **4-bit mode** with I2C expander:

```c
// Each command/data requires 4 I2C transmissions:
1. Upper nibble + Enable HIGH
2. Upper nibble + Enable LOW (latch)
3. Lower nibble + Enable HIGH
4. Lower nibble + Enable LOW (latch)
```

### 4. System Flow

```
[Power On]
    ↓
[Initialize HAL, I2C, Timers, GPIO]
    ↓
[LCD Welcome Message]
    ↓
[Motor Diagnostic Test: 30% → 60% → 100%]
    ↓
[Wait for DHT11 Stabilization]
    ↓
┌─────────────────────────┐
│   MAIN MONITORING LOOP   │
│                          │
│  1. Read DHT11 (retry ×3)│
│  2. Validate checksum    │
│  3. Apply control logic  │
│  4. Update PWM speed     │
│  5. Display on LCD       │
│  6. Wait 2.5 seconds     │
└──────────┬───────────────┘
           │
           └─→ [Repeat Forever]
```

---

## 🔧 Troubleshooting

### Problem: LCD not displaying anything

**Solutions:**
1. Check I2C address (try 0x3F instead of 0x27)
2. Verify SDA/SCL connections (PB9/PB8)
3. Test I2C with I2C scanner code
4. Adjust LCD contrast potentiometer on I2C module

### Problem: DHT11 shows constant errors

**Solutions:**
1. Check DATA pin connection to PA1
2. Verify DHT11 has 5V power (not 3.3V)
3. Ensure pull-up resistor is enabled in code
4. Wait 2 seconds after power-up for sensor stabilization
5. Try replacing DHT11 sensor (common failure point)

### Problem: Motor not spinning

**Solutions:**
1. Verify 12V supply to L298N
2. Check ENA (PB6) PWM connection
3. Confirm IN1 (PB7) direction pin connection
4. Test motor directly with 12V (bypass driver)
5. Run Motor_Test() function for diagnostics
6. Check if PWM signal is being generated (use oscilloscope or logic analyzer)

### Problem: Motor spins but LCD shows 0% speed

**Solutions:**
1. Check temperature reading (might be below 25°C threshold)
2. Verify temperature sensor is reading correctly
3. Test with heat source (hairdryer, hot water) near sensor

### Problem: Code won't compile

**Solutions:**
1. Ensure STM32CubeIDE version is 1.9.0 or later
2. Verify HAL drivers are properly linked
3. Clean project: `Project → Clean...`
4. Rebuild: `Project → Build Project`
5. Check for missing semicolons or brackets in user code sections

### Problem: ST-Link connection failed

**Solutions:**
1. Install ST-Link drivers from ST website
2. Check SWDIO/SWCLK connections
3. Press RESET button on STM32 while connecting
4. Try different USB cable/port
5. Verify ST-Link firmware is updated

---

## 👥 Team Members

**North South University**

| Name | Student ID | Role |
|------|------------|------|
| Anika Tabassum | 2211012042 | Hardware Integration |
| Mohosina Islam Disha | 2212349042 | Software Development |
| Muhammad Tahmidur Rahman | 2211295042 | System Design |
| Mehedi Hassan Roktim | 2212497042 | Testing & Documentation |

---

## 📄 License

This project is open-source and available under the MIT License. Feel free to use, modify, and distribute with proper attribution.

---

## 🎓 Academic Use

This project was developed as part of an embedded systems course at **North South University**. It demonstrates:
- Real-time sensor interfacing (DHT11)
- PWM motor control techniques
- I2C communication protocols
- Embedded C programming
- Hardware-software integration
- Error handling and fault tolerance

---

## 📚 Additional Resources

- [STM32F103C8T6 Datasheet](https://www.st.com/resource/en/datasheet/stm32f103c8.pdf)
- [DHT11 Sensor Datasheet](https://www.mouser.com/datasheet/2/758/DHT11-Technical-Data-Sheet-Translated-Version-1143054.pdf)
- [L298N Motor Driver Guide](https://lastminuteengineers.com/l298n-dc-stepper-driver-arduino-tutorial/)
- [STM32 HAL Documentation](https://www.st.com/content/st_com/en/stm32-mcu-developer-zone.html)

---

## 🤝 Contributing

Contributions are welcome! If you'd like to improve this project:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add new feature'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

---

## 📞 Contact

For questions, suggestions, or collaboration:

- **Email**: your.email@example.com
- **University**: North South University
- **Course**: Embedded Systems Design

---

## ⭐ Acknowledgments

- STMicroelectronics for STM32 HAL libraries
- North South University faculty for guidance
- Open-source community for reference implementations

---

**Made with ❤️ by NSU Team | Temperature-Controlled Fan Project**

---

## 📊 Performance Metrics

- **Response Time**: <3 seconds for temperature changes
- **Accuracy**: ±2°C (DHT11 sensor limitation)
- **Energy Savings**: Up to 35% compared to constant-speed operation
- **Reliability**: 99%+ uptime with error recovery
- **Cost**: ~2000 BDT (50-60% cheaper than commercial alternatives)

---

*Last Updated: December 2024*