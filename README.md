🤖 Arduino Uno Self-Balancing Robot

A **DIY two-wheel self-balancing robot** built using an **Arduino Uno, MPU6050 IMU, L298N motor driver, and two 6V geared DC motors**.

The MPU6050 detects the robot's tilt angle and movement. The Arduino Uno processes the sensor data and uses a **PID control algorithm** to continuously adjust the motor speed and direction, keeping the robot upright.

This project is designed for **students, electronics enthusiasts, robotics learners, and embedded-systems developers** who want to understand practical applications of **IMU sensors, PID control, motor control, and feedback systems**.

---

## 📸 Project Hardware

The prototype consists of:

* Arduino Uno
* MPU6050
* L298N motor driver
* 2 × 6V geared DC motors
* Two wheels
* Robot chassis
* Battery/power supply

---

# 🔍 Project Overview

The self-balancing robot uses two wheels as its only point of contact with the ground.

When the robot starts tilting forward or backward, the MPU6050 detects the change in orientation. The Arduino calculates the required correction using a PID controller and commands both motors to move in the appropriate direction.

The wheels continuously move underneath the robot's center of mass, allowing it to maintain balance.

### Basic working principle

```text
                 ┌─────────────────┐
                 │    MPU6050      │
                 │  Tilt Sensor    │
                 └────────┬────────┘
                          │
                          │ I2C
                          ▼
                 ┌─────────────────┐
                 │   Arduino UNO   │
                 │ Sensor + PID    │
                 │   Processing    │
                 └────────┬────────┘
                          │
                       PWM + DIR
                          │
                          ▼
                 ┌─────────────────┐
                 │     L298N       │
                 │  Motor Driver   │
                 └───────┬─┬───────┘
                         │ │
                    ┌────┘ └────┐
                    ▼           ▼
               LEFT MOTOR   RIGHT MOTOR
                    │           │
                    ▼           ▼
                  LEFT        RIGHT
                  WHEEL       WHEEL

                         ↑
                         │
                     Robot tilt
                         │
                         └──── MPU6050
```

---

# ✨ Features

* 🤖 Two-wheel self-balancing
* 📐 MPU6050 tilt sensing
* 🧠 PID-based balance control
* ⚙️ Dual DC motor control
* 🔌 L298N motor driver
* 🔄 Real-time feedback control
* 🎯 Adjustable PID parameters
* 🛑 Automatic motor cutoff when the robot falls
* 📊 Serial Monitor debugging
* 🧪 Wokwi simulation
* 💻 Arduino Uno based
* 🔧 Open-source hardware and software

---

# 🧰 Components Required

| Component                        |    Quantity | Purpose                 |
| -------------------------------- | ----------: | ----------------------- |
| Arduino Uno                      |           1 | Main controller         |
| MPU6050                          |           1 | Tilt/orientation sensor |
| L298N                            |           1 | Dual motor driver       |
| 6V geared DC motor, ~200–230 RPM |           2 | Robot drive             |
| Wheel                            |           2 | Robot movement          |
| Chassis                          |           1 | Mechanical structure    |
| Battery                          |           1 | Power supply            |
| Jumper wires                     | As required | Electrical connections  |
| USB cable                        |           1 | Programming             |

---

# 🔌 Circuit Diagram

## MPU6050 → Arduino Uno

```text
        MPU6050
     ┌─────────────┐
     │             │
 VCC │─────────────┼──── Arduino 5V
 GND │─────────────┼──── Arduino GND
 SDA │─────────────┼──── Arduino A4
 SCL │─────────────┼──── Arduino A5
 INT │─────────────┼──── Arduino D2
     │             │
     └─────────────┘
```

### Connection table

| MPU6050 Pin | Arduino Uno | Function      |
| ----------- | ----------- | ------------- |
| VCC         | 5V          | Power         |
| GND         | GND         | Ground        |
| SDA         | A4          | I2C Data      |
| SCL         | A5          | I2C Clock     |
| INT         | D2          | DMP interrupt |

> Note: The exact safe supply voltage depends on the MPU6050 breakout board being used. Check the module's regulator/logic-level design before applying 5V.

---

# ⚙️ L298N Motor Connections

```text
Arduino UNO              L298N
--------------------------------

D11  ─────────────────── ENA
D7   ─────────────────── IN1
D6   ─────────────────── IN2

D5   ─────────────────── IN3
D4   ─────────────────── IN4
D10  ─────────────────── ENB
```

Motor connections:

```text
L298N OUT1 / OUT2  → Left Motor

L298N OUT3 / OUT4  → Right Motor
```

### Motor-control table

| L298N Pin | Arduino Uno | Function        |
| --------- | ----------: | --------------- |
| ENA       |         D11 | Left motor PWM  |
| IN1       |          D7 | Left direction  |
| IN2       |          D6 | Left direction  |
| IN3       |          D5 | Right direction |
| IN4       |          D4 | Right direction |
| ENB       |         D10 | Right motor PWM |

The pin arrangement above matches the motor-controller configuration in your provided balancing code. 

---

# 🔋 Power Connections

The motors should **not** be powered from the Arduino's 5V output.

```text
              Battery
                 │
                 ▼
              L298N
             Motor Supply
              │      │
              ▼      ▼
         Left Motor  Right Motor

Arduino UNO
     │
     └── regulated supply

All GNDs
    │
    ├── Arduino GND
    ├── MPU6050 GND
    └── L298N GND
```

Use a motor supply appropriate for your specific motors and a suitable regulated supply for the Arduino.

---

# 🧠 How It Works

The balancing system can be divided into four stages.

## 1. Tilt Detection

The MPU6050 measures the robot's orientation.

```text
Robot tilts
     ↓
MPU6050
     ↓
Gyroscope + Accelerometer
     ↓
Orientation data
```

---

## 2. DMP Processing

The MPU6050's Digital Motion Processor can provide orientation information through the DMP interface.

```text
MPU6050
   ↓
DMP
   ↓
Quaternion
   ↓
Gravity vector
   ↓
Yaw / Pitch / Roll
```

Your code extracts the pitch value:

```cpp
mpu.dmpGetQuaternion(&q, fifoBuffer);
mpu.dmpGetGravity(&gravity, &q);
mpu.dmpGetYawPitchRoll(ypr, &q, &gravity);

input = ypr[1] * 180 / M_PI + 180;
```

---

# 3. PID Control

The Arduino compares the measured angle with the desired balance angle.

```text
Desired angle
      │
      ▼
    Error
      │
      ▼
     PID
      │
      ▼
 Motor output
```

The PID controller contains three terms:

### Proportional

```text
P = Kp × Error
```

Responds to the current tilt.

### Integral

```text
I = Ki × accumulated Error
```

Helps compensate for persistent offset.

### Derivative

```text
D = Kd × rate of change
```

Helps reduce oscillation and improve stability.

The starting PID parameters in the supplied code are:

```cpp
double Kp = 60;
double Kd = 2.2;
double Ki = 270;
```

These values are intended to be adjusted for the physical robot. 

---

# 4. Motor Control

The PID output controls both motors.

```text
             PID
              │
       ┌──────┴──────┐
       ▼             ▼
 Left motor       Right motor
       │             │
       └──────┬──────┘
              ▼
          Robot moves
```

### Forward tilt

```text
Robot falls forward
        ↓
MPU6050 detects tilt
        ↓
PID correction
        ↓
Motors move forward
        ↓
Robot returns toward upright
```

### Backward tilt

```text
Robot falls backward
        ↓
MPU6050 detects tilt
        ↓
PID correction
        ↓
Motors move backward
        ↓
Robot returns toward upright
```

---

# 📐 Balance Angle

The robot has a specific upright reference angle.

The code starts with:

```cpp
double originalSetpoint = 181.0;
```

and uses:

```cpp
double setpoint = originalSetpoint;
```

The actual balance point normally needs to be tuned for the physical construction.

Even a small difference in:

* battery location
* MPU6050 mounting
* chassis height
* wheel diameter
* motor position

can change the optimum balance angle.

---

# ⚙️ Motor Speed Configuration

The motor controller allows separate scaling for the two motors:

```cpp
double motorSpeedFactorLeft = 0.9;
double motorSpeedFactorRight = 0.9;
```

This can compensate for small differences between the motors.

For example:

```cpp
double motorSpeedFactorLeft = 0.90;
double motorSpeedFactorRight = 0.95;
```

can be used when one motor consistently runs slightly differently from the other.

---

# 🧭 MPU6050 Interrupt

The MPU6050 interrupt signal is connected to Arduino interrupt 0:

```cpp
attachInterrupt(0, dmpDataReady, RISING);
```

The interrupt function sets a flag:

```cpp
volatile bool mpuInterrupt = false;

void dmpDataReady()
{
    mpuInterrupt = true;
}
```

This allows the program to know when new DMP data is available.

---

# 💻 Software & Libraries

## Development Environment

* Arduino IDE 1.8.19
* C/C++
* Arduino Uno

## Required Arduino Libraries

```text
I2Cdev
MPU6050
PID_v1
LMotorController
```

The project uses:

```cpp
#include <PID_v1.h>
#include <LMotorController.h>
#include "I2Cdev.h"
#include "MPU6050_6Axis_MotionApps20.h"
```

---

# 🧪 Wokwi Simulation

A Wokwi simulation is available for the project:

[Wokwi Balancing Bot Simulation](https://wokwi.com/projects/473799284567120897?utm_source=chatgpt.com)

The simulation can be used to experiment with:

* Arduino Uno
* MPU6050
* Motor-control logic
* PID parameters
* Sensor data
* Program behavior

> The real physical robot will behave differently from simulation because actual motors introduce gearbox friction, inertia, voltage drop, mechanical backlash, and other non-ideal effects.

---

# 🧪 Testing Procedure

## Test 1 — MPU6050

Verify that the MPU6050 is detected correctly.

```text
MPU6050
   ↓
I2C communication
   ↓
Arduino Uno
```

---

## Test 2 — DMP

Check that the serial monitor reports successful DMP initialization.

Expected sequence:

```text
MPU6050 initialized
        ↓
DMP initialization
        ↓
DMP ready
```

---

## Test 3 — Sensor Movement

Keep the robot in your hand and slowly tilt it.

Check that:

```text
Forward tilt  → Pitch changes
Backward tilt → Pitch changes
```

The angle must change continuously before motor balancing is attempted.

---

## Test 4 — Motor Direction

Lift the robot so the wheels are free.

Tilt it forward.

```text
Forward tilt
     ↓
Motors should drive forward
```

Tilt it backward.

```text
Backward tilt
     ↓
Motors should drive backward
```

The correction direction is critical.

---

## Test 5 — PID

Start with conservative PID values and gradually tune the controller.

Recommended tuning sequence:

```text
Kp
 ↓
Kd
 ↓
Ki
```

Start with:

```cpp
Ki = 0;
```

and introduce integral gain only after basic balance response is working.

---

# ⚠️ Safety

Do not place the robot on the floor immediately after uploading new PID parameters.

First:

```text
1. Check sensor
2. Check angle
3. Check motor direction
4. Hold robot above the ground
5. Check correction
6. Start with low motor output
7. Test on the floor
```

The robot can accelerate quickly when the control direction is incorrect.

---

# 🛠️ Troubleshooting

## MPU6050 not responding

Check:

```text
VCC → correct supply
GND → GND
SDA → A4
SCL → A5
INT → D2
```

Also check that all ground connections are common.

---

## Pitch does not change

Check:

```text
MPU6050 SDA
MPU6050 SCL
MPU6050 INT
```

and verify that the DMP is actually providing new FIFO packets.

---

## Robot immediately falls

Check motor correction direction first.

If the robot tilts forward:

```text
Motor → forward
```

must be the correction.

If the motors drive backward when the robot falls forward, reverse the motor direction logic.

---

## Robot oscillates

The PID gains may be too aggressive.

Try reducing:

```cpp
Kp
```

or increasing:

```cpp
Kd
```

in small increments.

---

## Robot reacts too slowly

Increase `Kp` gradually.

Avoid immediately increasing all three PID terms.

---

## One wheel is faster

Adjust:

```cpp
motorSpeedFactorLeft
motorSpeedFactorRight
```

Example:

```cpp
double motorSpeedFactorLeft = 0.9;
double motorSpeedFactorRight = 0.95;
```

---

# 🚀 Future Improvements

Possible upgrades:

* 🔋 Rechargeable battery system
* 🖥️ OLED display
* 🎮 Wireless remote control
* 📱 Bluetooth control
* 🎯 Automatic PID tuning
* 🧭 Improved sensor fusion
* ⚙️ Better motor driver
* 🔄 Encoder feedback
* 📊 Real-time PID monitoring
* 🔋 Battery voltage monitoring
* 🚦 Status LED
* 🛞 Higher-torque geared motors
* 🧠 Advanced balance controller
* 📡 Wi-Fi configuration
* 🖨️ Custom 3D-printed chassis

---

# 📚 Learning Outcomes

This project demonstrates practical concepts in:

* Embedded systems
* Arduino programming
* I2C communication
* MPU6050 IMU sensors
* Digital Motion Processor
* Gyroscope and accelerometer data
* PID control
* Closed-loop feedback
* DC motor control
* PWM
* Motor drivers
* Robotics
* Real-time control systems
* Hardware prototyping

---

# 📁 Project Structure

```text
Balancing-bot/
│
├── Self_Balancing_Robot.ino
│
├── README.md
│
├── images/
│   └── Balancing_Bot_Hardware.png
│
└── wokwi/
    ├── sketch.ino
    └── diagram.json
```

---

# 🔗 Wokwi Simulation

```text
https://wokwi.com/projects/473799284567120897
```

---

# 👨‍💻 Author

**solo-prince**

Arduino Uno Self-Balancing Robot project.

---

# 📄 License

This project is intended for **educational and personal development purposes**.

You may modify and improve the project for robotics, embedded-systems learning, and experimental development.
