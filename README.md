# Self-Balancing Robot

A two-wheeled self-balancing robot using MPU6050 gyroscope/accelerometer and PID control algorithm. The robot maintains vertical balance similar to a Segway or hoverboard.

## Overview

This project uses the MPU6050's Digital Motion Processor (DMP) to calculate orientation and a PID controller to maintain balance by adjusting motor speeds in real-time.

## Features

- **Self-balancing capability** using PID control
- **MPU6050 DMP integration** for accurate tilt detection
- **Real-time motor control** with configurable speed factors
- **Tunable PID parameters** (Kp, Ki, Kd)
- **Interrupt-driven sensor reading** for fast response

## Hardware Requirements

### Components
- Arduino Uno/Nano/Mega
- MPU6050 6-axis gyroscope/accelerometer module (GY-521)
- L298N motor driver module (or similar dual H-bridge)
- 2x DC gear motors with encoders (recommended)
- 2x Wheels (60-100mm diameter recommended)
- Robot chassis or custom frame
- Battery pack (7.4V-12V, 2000mAh+ recommended)
- Power switch
- Jumper wires

### Pin Configuration

```
MPU6050 Connections:
VCC  → 5V
GND  → GND
SDA  → A4 (Arduino Uno/Nano) or Pin 20 (Mega)
SCL  → A5 (Arduino Uno/Nano) or Pin 21 (Mega)
INT  → D2 (Interrupt 0)

L298N Motor Driver:
ENA  → Pin 5 (PWM)
IN1  → Pin 6
IN2  → Pin 7
IN3  → Pin 8
IN4  → Pin 9
ENB  → Pin 10 (PWM)

Motor A (Left)  → OUT1, OUT2
Motor B (Right) → OUT3, OUT4

Power:
Battery (+) → L298N +12V
Battery (-) → L298N GND
L298N 5V    → Arduino 5V (if using voltage regulator on L298N)
L298N GND   → Arduino GND (required for common ground)
```

## Required Libraries

This project requires several libraries to be installed in your Arduino IDE.

### Library List

1. **I2Cdev** - I2C communication library
2. **MPU6050** - MPU6050 sensor library with DMP support
3. **PID_v1** - PID controller library by Brett Beauregard
4. **LMotorController** - Custom motor controller library for this project

### Installation Methods

#### Method 1: From This Repository (Recommended)

This repository includes all custom libraries in the `/libraries` folder!

**Step-by-step instructions:**

1. **Clone or download this repository:**
   ```bash
   git clone https://github.com/[your-username]/self-balancing-robot.git
   ```
   Or download as ZIP and extract

2. **Locate your Arduino libraries folder:**
   ```
   Windows: C:\Users\[YourUsername]\Documents\Arduino\libraries\
   Mac: ~/Documents/Arduino/libraries/
   Linux: ~/Arduino/libraries/
   ```

3. **Copy libraries from this repo:**
   - Go to the `/libraries` folder in this repository
   - Copy the **LMotorController** folder
   - Paste it into: `Documents\Arduino\libraries\`

4. **Install external libraries:**
   
   **I2Cdev & MPU6050:**
   - Download from: https://github.com/jrowberg/i2cdevlib
   - Extract the ZIP file
   - Copy `Arduino/I2Cdev/` folder → `Documents\Arduino\libraries\`
   - Copy `Arduino/MPU6050/` folder → `Documents\Arduino\libraries\`

   **PID_v1:**
   - Download from: https://github.com/br3ttb/Arduino-PID-Library
   - Extract and rename folder to **PID_v1**
   - Copy to: `Documents\Arduino\libraries\`

5. **Restart Arduino IDE** completely to load the new libraries

**Your Arduino libraries folder should look like this:**
```
Documents\Arduino\libraries\
├── I2Cdev\
│   ├── I2Cdev.h
│   ├── I2Cdev.cpp
│   └── ...
├── MPU6050\
│   ├── MPU6050.h
│   ├── MPU6050.cpp
│   ├── MPU6050_6Axis_MotionApps20.h
│   └── ...
├── PID_v1\
│   ├── PID_v1.h
│   ├── PID_v1.cpp
│   └── ...
└── LMotorController\          ← From this repository!
    ├── LMotorController.h
    └── LMotorController.cpp
```

#### Method 2: Using Arduino Library Manager (Partial)

You can install PID_v1 via Library Manager, but others must be manual:

1. Open Arduino IDE
2. Go to **Sketch → Include Library → Manage Libraries...**
3. Search for **"PID"**
4. Find **"PID by Brett Beauregard"**
5. Click **Install**

**Note:** I2Cdev, MPU6050, and LMotorController must still be installed manually.

### Verifying Library Installation

After installation, verify libraries are loaded:

1. Open Arduino IDE
2. Go to **Sketch → Include Library**
3. You should see in the list:
   - I2Cdev
   - MPU6050
   - PID_v1
   - LMotorController

If any are missing, restart Arduino IDE and check folder names.

## Setup and Calibration

### 1. MPU6050 Calibration

The gyroscope offsets in the code need to be calibrated for your specific MPU6050:

```cpp
mpu.setXGyroOffset(220);
mpu.setYGyroOffset(76);
mpu.setZGyroOffset(-85);
mpu.setZAccelOffset(1788);
```

**To find your offsets:**
1. Use the MPU6050 calibration sketch from the library examples
2. Run: `File → Examples → MPU6050 → IMU_Zero`
3. Follow the instructions to get your offset values
4. Replace the values in the main code

### 2. Motor Speed Factor Calibration

If your robot drifts to one side, adjust these values:

```cpp
double motorSpeedFactorLeft = 0.58;
double motorSpeedFactorRight = 0.62;
```

- If drifts **left**: Increase `motorSpeedFactorLeft` or decrease `motorSpeedFactorRight`
- If drifts **right**: Decrease `motorSpeedFactorLeft` or increase `motorSpeedFactorRight`
- Values typically range from 0.5 to 1.0

### 3. PID Tuning

The PID values control how aggressively the robot balances:

```cpp
double Kp = 24;   // Proportional - First to adjust
double Ki = 120;  // Integral - Third to adjust
double Kd = 1.6;  // Derivative - Second to adjust
```

**Tuning Guide:**

1. **Start with Kp:**
   - Set Ki=0, Kd=0
   - Increase Kp until robot oscillates
   - Reduce Kp by 20-30%

2. **Add Kd:**
   - Increase Kd to dampen oscillations
   - Too much Kd = sluggish response
   - Too little Kd = continues oscillating

3. **Finally Ki:**
   - Add small Ki to eliminate steady-state error
   - Too much Ki = instability
   - Start with very small values (0.1-1.0)

**Tuning Tips:**
- Make small changes (10-20% at a time)
- Test after each change
- Keep the robot secured during initial tests
- Document values that work

### 4. Balance Point Adjustment

The setpoint determines the target angle:

```cpp
double originalSetpoint = 180;
```

- **180°** = vertical position
- If robot leans forward at rest: increase setpoint (181-185)
- If robot leans backward at rest: decrease setpoint (175-179)

## Assembly Instructions

### 1. Physical Construction

**Robot Design Considerations:**
- **Wheel diameter**: 60-100mm recommended
- **Wheelbase**: 10-15cm apart
- **Center of mass**: Should be slightly above wheel axle
- **Battery placement**: Low and centered
- **Motor placement**: Direct drive to wheels (no gear reduction if possible)

**Build Steps:**
1. Mount motors parallel to each other
2. Attach wheels directly to motor shafts
3. Mount Arduino and motor driver above wheels
4. Place MPU6050 at the center, as close to wheel axle as possible
5. Mount battery low and centered
6. Ensure all wiring is secure and won't interfere with movement

### 2. Wiring

Follow the pin configuration above. Key points:
- **Common ground** is essential - connect all grounds together
- Keep MPU6050 wires short to reduce noise
- Use twisted pair for motor wires to reduce EMI
- Add capacitors (0.1µF) across motor terminals to reduce electrical noise

### 3. Power System

**Recommended Setup:**
```
Battery (7.4V-12V)
    ↓
L298N Motor Driver
    ├→ Motors (high current)
    └→ 5V regulator → Arduino (logic)
```

## Usage

### First Time Setup

1. **Upload the code:**
   - Connect Arduino via USB
   - Disconnect MPU6050 INT pin temporarily
   - Upload the sketch
   - Reconnect INT pin

2. **Serial Monitor Test:**
   - Open Serial Monitor (115200 baud)
   - Should see initialization messages
   - Check for "DMP ready!" message
   - Tilt the MPU6050 and verify yaw/pitch/roll values change

3. **Motor Test:**
   - With robot secured or wheels off ground
   - Power on the robot
   - Tilt forward/backward
   - Motors should respond to balance

4. **Balance Test:**
   - Hold robot vertically
   - Power on while holding
   - Wait for 2-3 seconds for initialization
   - Gently release
   - Robot should attempt to balance

### Operating Tips

- **Always power on while holding the robot upright**
- Give 2-3 seconds for sensor initialization
- Start with gentle releases
- Be ready to catch the robot
- Adjust PID values if too aggressive or too weak
- Test on smooth, flat surfaces first

## Troubleshooting

### Robot Won't Balance

**Possible causes:**
1. **Wrong MPU6050 orientation**
   - Solution: MPU should be horizontal, parallel to ground
   - Chip should face up

2. **Incorrect wiring**
   - Solution: Double-check all connections
   - Verify INT pin connected to D2

3. **PID values too low**
   - Solution: Increase Kp gradually

4. **Center of mass too high**
   - Solution: Lower battery placement

### Robot Oscillates Violently

- **Kp too high**: Reduce proportional gain
- **Kd too low**: Increase derivative gain
- **Motor response too fast**: Add small delay or reduce MIN_ABS_SPEED

### Robot Falls Immediately

- **Kp too low**: Increase proportional gain
- **Motors not responding**: Check motor driver connections and power
- **Wrong setpoint**: Adjust originalSetpoint value
- **Sensor not working**: Check Serial Monitor for DMP initialization

### DMP Initialization Failed

**Error codes:**
- **Code 1**: Memory load failed
  - Solution: Check I2C connections (SDA/SCL)
  - Try different MPU6050 module
  
- **Code 2**: DMP configuration failed
  - Solution: Update MPU6050 library
  - Check power supply stability

### Motors Run at Full Speed

- **MIN_ABS_SPEED too low**: Increase from 20 to 30-40
- **PID output not limiting**: Check SetOutputLimits(-255, 255)
- **Motor driver issue**: Test motors separately

### Serial Monitor Shows "FIFO Overflow"

- **Loop taking too long**: Optimize code
- **I2C speed too slow**: Check TWBR setting (should be 24 for 400kHz)
- **Not critical**: Usually can be ignored if rare

## Advanced Features

### Adding Remote Control

You can add Bluetooth control by:
1. Adding HC-05 module
2. Modifying setpoint based on received commands:
```cpp
if (command == 'F') setpoint = originalSetpoint - 5;  // Lean forward
if (command == 'B') setpoint = originalSetpoint + 5;  // Lean back
```

### Speed Control

Modify the MIN_ABS_SPEED dynamically:
```cpp
MIN_ABS_SPEED = map(desiredSpeed, 0, 100, 20, 100);
```

### Turning Control

Add turning by creating speed difference between motors:
```cpp
motorController.moveWithTurn(output, turnAmount, MIN_ABS_SPEED);
```

## Safety Warnings

⚠️ **Important Safety Information:**

1. **Always test with robot secured initially**
2. **Keep fingers away from wheels during operation**
3. **Use appropriate battery with proper voltage**
4. **Don't leave battery charging unattended**
5. **Ensure emergency shutoff is accessible**
6. **Test in open area away from obstacles**
7. **Be prepared to catch the robot**

## Performance Tips

1. **Use quality motors** with good torque
2. **Keep robot weight low** (under 1kg recommended)
3. **Use fresh batteries** - low voltage affects performance
4. **Ensure tight mechanical connections** - no wobble
5. **Balance weight distribution** - centered and low
6. **Use appropriate wheel size** - larger = easier to balance

## Specifications

| Parameter | Value |
|-----------|-------|
| Operating Voltage | 7.4V - 12V |
| Arduino Board | Uno/Nano/Mega |
| Sensor Update Rate | ~100Hz |
| PID Sample Time | 10ms |
| Motor PWM Range | 0-255 |
| Minimum Speed Threshold | 20 (adjustable) |
| I2C Clock Speed | 400kHz |

## Code Structure

```
setup()
├── Initialize I2C
├── Initialize MPU6050
├── Load DMP firmware
├── Set gyro offsets
├── Enable DMP
├── Setup interrupt
└── Configure PID

loop()
├── Wait for MPU interrupt
├── Read FIFO buffer
├── Calculate yaw/pitch/roll
├── Update PID input
├── Compute PID output
└── Control motors
```

## Resources

- **I2Cdev Library**: https://github.com/jrowberg/i2cdevlib
- **PID Library**: https://github.com/br3ttb/Arduino-PID-Library
- **MPU6050 Datasheet**: Search "MPU6050 datasheet PDF"
- **PID Tuning Guide**: https://www.crossco.com/blog/basics-tuning-pid-loops/

## Contributing

Contributions are welcome! Please:
1. Test thoroughly before submitting
2. Document any hardware changes
3. Include photos/videos of modifications
4. Update this README with new features

## License

MIT License - See LICENSE file

## Credits

- I2Cdev and MPU6050 libraries by Jeff Rowberg
- PID library by Brett Beauregard
- Project by [Moaz Elkholy]

---

**Need help?** Open an issue on GitHub with:
- Clear description of the problem
- Photos of your wiring
- Serial Monitor output.
- PID values you're using..
  
