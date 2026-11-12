---
description: Getting Started with the XIAO Robot Kit — step-by-step setup, button, IMU, servo and encoder motor examples.
title: Getting Started with XIAO Robot Kit
keywords:
  - xiao robot kit
  - esp32s3
  - arduino
  - servo
  - encoder motor
  - imu
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_getting_started
sku: 100058302
type: gettingstarted
last_update:
  date: 11/11/2026
  author: 划水佬
createdAt: '2026-11-11'
updatedAt: '2026-11-11'
url: https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/
---

# Getting Started with XIAO Robot Kit

## Introduction

The XIAO Robot Kit is a modular robotics development kit built around the **Seeed Studio XIAO ESP32-S3 Plus**. The carrier board already holds the electronics of a small mobile robot: an ESP32-S3 controller with Wi-Fi and Bluetooth LE, two N20 gear motors with quadrature encoders, four servo channels, a 6-axis IMU and battery management. The kit adds two Feetech FT1190M metal-gear servos and Grove connectors for whatever you attach later. The body is yours to design and print — the motion hardware is already on the board.

- **Build your own robot**: the electronics are done, so the structure is the part you get to invent. Print a wheeled chassis, or a character with moving arms.
- **More ways to move, sense and interact**: two independently driven wheels, four servo channels and a 6-axis IMU, plus the two FT1190M metal-gear servos that come in the kit.
- **Encoder feedback**: each N20 motor reports how far its wheel has actually turned, so a sketch can measure motion instead of assuming it from a delay.
- **Wireless**: Wi-Fi and Bluetooth LE on the ESP32-S3 Plus, enough for a phone-controlled robot or a link to an external AI application.
- **AI-assisted coding**: describe the behavior you want and let a coding agent such as Codex or Claude Code write the Arduino sketch with you.

The rest of this page covers the board layout and four minimal examples, one per subsystem.

## Hardware Overview

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/XIAO_Robot_Board_Front_Side.jpg" style={{width:800, height:'auto'}}/></div>

The **XIAO ESP32-S3 Plus** module forms the core of the kit. The table below maps each on-board hardware function to its dedicated pin.

:::caution `Dxx` is a silkscreen marking, not a GPIO number
A `Dxx` marking is **not** a GPIO number. For example, `D11` maps to `GPIO38`.
:::

### On-board Peripheral Pin Mapping

| Function | Pin | GPIO | Note |
| --- | --- | --- | --- |
| SERVO3 | D0 | GPIO1 | Servo 3 |
| M1 PWM1 | D1 | GPIO2 | Motor 1 drive |
| M1 PWM2 | D2 | GPIO3 | Motor 1 drive |
| M1 Encoder A | D3 | GPIO4 | Motor 1 encoder A |
| I²C SDA | D4 | GPIO5 | on-board IMU |
| I²C SCL | D5 | GPIO6 | on-board IMU |
| UART TX | D6 | GPIO43 | Grove UART TX |
| UART RX | D7 | GPIO44 | Grove UART RX |
| SERVO4 | D8 | GPIO7 | Servo 4 |
| SERVO2 | D9 | GPIO8 | Servo 2 |
| M2 Encoder B | D10 | GPIO9 | Motor 2 encoder B |
| KEY | D11 | GPIO38 | User button |
| IMU Heater PWM | D12 | GPIO39 | Drives the heater resistor next to the IMU |
| RGB | D13 | GPIO40 | Grove RGB / Digital |
| M1 Encoder B | D14 | GPIO41 | Motor 1 encoder B |
| SERVO1 | D15 | GPIO42 | Servo 1 |
| ADC | D16 | GPIO10 | On-board ADC |
| M2 Encoder A | D17 | GPIO13 | Motor 2 encoder A |
| M2 PWM2 | D18 | GPIO12 | Motor 2 drive |
| M2 PWM1 | D19 | GPIO11 | Motor 2 drive |

### Power

The kit has two power inputs: the **USB-C port** and the **lithium battery connector**. Motors and servos run from a separate **Motor_5V** rail, not from the 3.3 V logic rail — driving a loaded servo or motor from USB alone can brown out the board, so use a battery for Examples 3 and 4.

## Getting Started

### Hardware Preparation

#### Step 1. Connect to the computer

##### Method 1

Use a **USB-C data cable** to connect the USB-C port on the **XIAO ESP32-S3 Plus** to your computer, then feed the kit's bottom USB-C / battery port from a separate power source such as a power bank or a battery.

![Hardware connection](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/demo_hardware_connection1.jpg)

##### Method 2

Use a **USB-C data cable** to connect the kit to your computer through the bottom USB-C port, leaving the USB-C port on the **XIAO ESP32-S3 Plus** unplugged. Slide the power switch to **ON**. Note: this method requires the computer's USB-C port to be backed by a self-powered dock or hub, so that the supply current is sufficient.

![Hardware connection](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/demo_hardware_connection2.jpg)

:::caution Use a data cable
A charge-only cable shows up as no COM port, or the device does not respond at all — swap in a data cable first.
:::

### Software Preparation

#### Step 1. Install the Arduino IDE

Download the latest **Arduino IDE 2.x** from [arduino.cc/en/software](https://www.arduino.cc/en/software) and install it for your operating system.

#### Step 2. Add the ESP32 board package

Launch Arduino IDE, open **File > Preferences**, and paste the URL below into **"Additional Boards Manager URLs"**:

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Then open **Tools > Board > Boards Manager**, search for `esp32`, and install **esp32 by Espressif Systems**.

#### Step 3. Select the board and port

Connect the kit with the USB-C cable, then in **Tools > Board > esp32** choose **XIAO_ESP32S3_PLUS**. Open **Tools > Port** and pick the new COM port that appeared when you plugged the board in.

:::tip More detailed setup guide
For more detailed **XIAO ESP32-S3 Plus** setup instructions, see the Software Preparation section of the [Seeed Studio XIAO ESP32-S3 Getting Started](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/#software-preparation) guide.
:::

## Example 1: Button + Hello World Serial Print

This is the "hello world" of the kit and the quickest way to confirm that your toolchain works. The sketch does two things: it prints `hello world N` once per second, and it counts every press of the on-board **KEY** button — no wiring needed, because the button is already on the board.

### Step 1. Create the sketch

In the Arduino IDE, open **File > New Sketch**, delete the template code, and paste the sketch below:

<details>

<summary> Full code </summary>

```c
/* Example 1: prints "hello world N" once per second,
 * and counts presses of the on-board KEY button. */
#include <Arduino.h>

#define PIN_BUTTON  38

void setup()
{
    Serial.begin(115200);
    delay(200);
    pinMode(PIN_BUTTON, INPUT_PULLUP);

    Serial.println("01 button + hello world");
    Serial.println("press the on-board key to count");
}

void loop()
{
    uint32_t now = millis();

    static uint32_t nextHello = 0;
    static uint32_t n = 0;
    if ((int32_t)(now - nextHello) >= 0) {
        nextHello = now + 1000;
        Serial.print("hello world ");
        Serial.println(++n);
    }

    static bool     lastRaw    = HIGH;
    static bool     lastStable = HIGH;
    static uint32_t lastChange = 0;
    static uint32_t presses    = 0;

    bool cur = (digitalRead(PIN_BUTTON) == LOW);
    if (cur != lastRaw) { lastRaw = cur; lastChange = now; }
    if (now - lastChange >= 10 && cur != lastStable) {
        lastStable = cur;
        if (lastStable) {
            presses++;
            Serial.print("button pressed = ");
            Serial.println(presses);
        }
    }

    delay(10);
}
```

</details>

### Step 2. Upload the sketch

Confirm **Tools > Board** is `XIAO_ESP32S3_PLUS` and **Tools > Port** shows your board, then click **Upload** (→).

### Step 3. Observe the result

Open the **Serial Monitor** (magnifier icon, top right) and set the baud rate to **115200**. You should see one `hello world N` line per second. Now press the on-board **KEY** button a few times — each press appends a `button pressed = N` line:

```
01 button + hello world
press the on-board key to count
hello world 1
hello world 2
button pressed = 1
```

![Button demo](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example1_button_demo.gif)

![Button serial log](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example1_button_log.gif)

:::tip
If the Serial Monitor stays empty or prints garbled text, check the selected port and the 115200 baud setting — see [Troubleshooting](#troubleshooting).
:::

## Example 2: IMU 6-Axis Raw Data + Temperature

This example reads the on-board **LSM6DS3TR-C** 6-axis IMU (3-axis accelerometer + 3-axis gyroscope) and streams all seven channels at 20 Hz — acceleration in `g`, angular rate in `dps`, and die temperature in `°C` — so you can watch the readings respond as you move the board. Again, no wiring is needed.

### Step 1. Create the sketch

Create a new sketch (**File > New Sketch**) and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 2: streams 6-axis acceleration (g), angular rate (dps)
 * and temperature (C) from the LSM6DS3TR-C IMU over I2C. */
#include <Arduino.h>
#include <Wire.h>
#include <stdlib.h>

#define ADDR     0x6A
#define PIN_SDA  5
#define PIN_SCL  6

#define WHO_AM_I 0x0F
#define CTRL1_XL 0x10
#define CTRL2_G  0x11
#define CTRL3_C  0x12
#define OUT_TEMP 0x20

/* write one register */
static void wr(uint8_t r, uint8_t v) {
    Wire.beginTransmission(ADDR); Wire.write(r); Wire.write(v); Wire.endTransmission(true);
}

/* read n bytes starting at register r */
static bool rd(uint8_t r, uint8_t *b, uint8_t n) {
    Wire.beginTransmission(ADDR); Wire.write(r);
    if (Wire.endTransmission(false)) return false;
    if (Wire.requestFrom((int)ADDR, (int)n) != n) return false;
    for (uint8_t i = 0; i < n; i++) b[i] = Wire.read();
    return true;
}

/* little-endian 16-bit word */
static int16_t i16(const uint8_t *p) { return (int16_t)(p[0] | p[1] << 8); }

/* print one "tag=value" field */
static void p(const char *tag, float v, int w, int d) {
    char s[10];
    Serial.print(tag);
    Serial.print(dtostrf(v, w, d, s));
    Serial.print(' ');
}

void setup() {
    Serial.begin(115200);
    delay(200);
    Wire.begin(PIN_SDA, PIN_SCL);
    Wire.setClock(400000);

    uint8_t id = 0;
    if (!rd(WHO_AM_I, &id, 1) || id != 0x6A) {
        Serial.println("IMU init failed");
        while (1) delay(1000);
    }

    wr(CTRL1_XL, 0x40);
    wr(CTRL2_G,  0x4C);
    wr(CTRL3_C,  0x44);
    Serial.println("IMU ready  (ax/ay/az in g, gx/gy/gz in dps, t in C)");
}

void loop() {
    uint8_t b[14];
    if (!rd(OUT_TEMP, b, 14)) { delay(50); return; }

    float t  = 25.0f + i16(&b[0])  / 256.0f;
    float gx = i16(&b[2])  * 0.070f;
    float gy = i16(&b[4])  * 0.070f;
    float gz = i16(&b[6])  * 0.070f;
    float ax = i16(&b[8])  * 0.000061f;
    float ay = i16(&b[10]) * 0.000061f;
    float az = i16(&b[12]) * 0.000061f;

    p("ax=", ax, 6, 3);
    p("ay=", ay, 6, 3);
    p("az=", az, 6, 3);
    p("gx=", gx, 8, 2);
    p("gy=", gy, 8, 2);
    p("gz=", gz, 8, 2);
    p("t=",  t,  6, 2);
    Serial.println("C");

    delay(50);
}
```

</details>

### Step 2. Upload the sketch

Upload as in Example 1. On boot the sketch checks the IMU's `WHO_AM_I` register and prints `IMU init failed` if the chip does not answer — otherwise you get `IMU ready`.

### Step 3. Observe the result

With the board lying flat on the desk, the Serial Monitor shows a steady stream at 20 Hz. Try these checks:

- **Flat and still**: `az ≈ 1.0` (gravity points down), `ax ≈ ay ≈ 0`, all three gyro axes near `0`.
- **Tilt the board**: `ax`/`ay` change as you rotate it — they measure gravity's direction.
- **Shake the board**: the gyro readings spike with the motion.

```
IMU ready  (ax/ay/az in g, gx/gy/gz in dps, t in C)
ax= 0.012 ay= 0.008 az= 0.995 gx=   0.12 gy=  -0.03 gz=   0.05 t=25.31 C
```

![IMU demo](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example2_imu_demo.gif)

![IMU data](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example2_imu_data.gif)

## Example 3: Servo Minimal Example

This example drives a **position servo** connected to servo port **SERVO1**. It outputs a 50 Hz PWM signal whose pulse width follows a sine wave between 500 µs and 2500 µs, sweeping the servo horn smoothly across its full travel. The Serial Monitor prints the commanded position and pulse width so you can correlate PWM with motion.

### Hardware Setup

Just connect the included servo and its cable to the **SERVO1** connector on the board.

### Step 1. Create the sketch

Create a new sketch and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 3: sweeps a position servo smoothly across its full range
 * (one sine cycle every 4 s) on servo port SERVO1. */
#include <Arduino.h>
#include <math.h>

#define PIN_SERVO      42

#define PWM_FREQ       50
#define PWM_RES_BITS   14
#define PWM_PERIOD_US  20000

#define SERVO_PULSE_CENTER   1500
#define SERVO_PULSE_MIN      500
#define SERVO_PULSE_MAX      2500

#define SWEEP_PERIOD         4000
#define TWO_PI               6.2831853f

/* map position -1.0..+1.0 to pulse width in us */
static float servoPulseUs(float normPos) {
    if (normPos < -1.0f) normPos = -1.0f;
    if (normPos >  1.0f) normPos =  1.0f;
    return (normPos < 0.0f)
        ? (float)SERVO_PULSE_CENTER + normPos * (float)(SERVO_PULSE_CENTER - SERVO_PULSE_MIN)
        : (float)SERVO_PULSE_CENTER + normPos * (float)(SERVO_PULSE_MAX - SERVO_PULSE_CENTER);
}

/* set servo position, -1.0..+1.0 */
static void servoWrite(float normPos) {
    float us = servoPulseUs(normPos);
    uint32_t duty = (uint32_t)(us * (1u << PWM_RES_BITS) / (float)PWM_PERIOD_US + 0.5f);
    ledcWrite(PIN_SERVO, duty);
}

void setup() {
    Serial.begin(115200);
    delay(200);

    Serial.println("servo - full-range sweep (SERVO1)");
    Serial.println("WARN: servos on Motor_5V rail - use battery / external 5V");

    pinMode(PIN_SERVO, OUTPUT); digitalWrite(PIN_SERVO, LOW);
    ledcAttach(PIN_SERVO, PWM_FREQ, PWM_RES_BITS);
    servoWrite(0.0f);
}

void loop() {
    static unsigned long tPrint = 0;
    uint32_t now = millis();

    float normPos = sinf(TWO_PI * (float)(now % SWEEP_PERIOD) / (float)SWEEP_PERIOD);
    servoWrite(normPos);

    if (now - tPrint >= 100) {
        tPrint = now;
        Serial.print("normPos=");   Serial.print(normPos, 2);
        Serial.print("  us=");      Serial.println(servoPulseUs(normPos), 0);
    }

    delay(20);
}
```

</details>

### Step 2. Upload the sketch

Upload as before. On boot the servo is commanded to center (`normPos = 0`, 1500 µs) before the sweep starts.

### Step 3. Observe the result

After power-up, watch the servo horn move and check the Serial Monitor output — it should match the two figures below.

![Servo demo](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example3_servo_demo.gif)

![Servo serial log](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example3_servo_log.gif)

:::tip Calibrating the pulse widths
The three macros `SERVO_PULSE_MIN / CENTER / MAX` map a normalized position to a pulse width. If your servo buzzes at the ends of travel, narrow `MIN`/`MAX` — the mechanical limits differ between servo models and depend on the linkage.
:::

## Example 4: Encoder Motor Minimal Example

This example drives the **M1 DC gear motor** and reads its **quadrature encoder** to measure speed. The Serial Monitor prints the commanded duty cycle, the encoder count and the resulting speed.

### Hardware Setup

Just connect the included motor and its cable to the **encoder motor connector**.

### Step 1. Create the sketch

Create a new sketch and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 4: drives the M1 gear motor with a sine-shaped duty cycle
 * (one cycle every 4 s) and prints encoder count and speed (rpm). */
#include <Arduino.h>
#include <math.h>

#define PIN_M1_IN1  2
#define PIN_M1_IN2  3
#define PIN_ENC_A   4
#define PIN_ENC_B   41

#define DRIVE_PERIOD 4000
#define TWO_PI       6.2831853f
#define DEAD_ZONE    15

const uint32_t kCountsPerRev = 7200;

/* quadrature transition table (x4 decoding) */
static const int8_t kQuad[16] = {
     0,  1, -1,  0,
    -1,  0,  0,  1,
     1,  0,  0, -1,
     0, -1,  1,  0
};
static volatile int32_t encCount = 0;
static volatile uint8_t  encState = 0;

/* encoder ISR: quadrature decode */
void IRAM_ATTR onEncoderChange() {
    uint8_t a    = (uint8_t)digitalRead(PIN_ENC_A);
    uint8_t b    = (uint8_t)digitalRead(PIN_ENC_B);
    uint8_t next = (uint8_t)((a << 1) | b);
    encCount += kQuad[(encState << 2) | next];
    encState  = next;
}

/* atomic snapshot of the encoder count */
static int32_t readEnc() {
    noInterrupts();
    int32_t c = encCount;
    interrupts();
    return c;
}

/* apply motor duty, -100..100 (0 = coast) */
static void motorSet(int duty) {
    if (duty >  100) duty =  100;
    if (duty < -100) duty = -100;
    if (duty > -DEAD_ZONE && duty < DEAD_ZONE) duty = 0;

    uint32_t d = (uint32_t)((duty < 0 ? -duty : duty) * 1023 / 100);
    if (duty > 0)      { ledcWrite(PIN_M1_IN1, d); ledcWrite(PIN_M1_IN2, 0); }
    else if (duty < 0) { ledcWrite(PIN_M1_IN1, 0); ledcWrite(PIN_M1_IN2, d); }
    else               { ledcWrite(PIN_M1_IN1, 0); ledcWrite(PIN_M1_IN2, 0); }
}

void setup() {
    Serial.begin(115200);
    delay(200);

    Serial.println("encoder motor - auto duty");

    pinMode(PIN_M1_IN1, OUTPUT); digitalWrite(PIN_M1_IN1, LOW);
    pinMode(PIN_M1_IN2, OUTPUT); digitalWrite(PIN_M1_IN2, LOW);

    pinMode(PIN_ENC_A, INPUT_PULLUP);
    pinMode(PIN_ENC_B, INPUT_PULLUP);
    encState = (uint8_t)((digitalRead(PIN_ENC_A) << 1) | digitalRead(PIN_ENC_B));
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_A), onEncoderChange, CHANGE);
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_B), onEncoderChange, CHANGE);

    ledcAttach(PIN_M1_IN1, 20000, 10);
    ledcAttach(PIN_M1_IN2, 20000, 10);
    motorSet(0);
}

void loop() {
    static unsigned long tPrint   = 0;
    static int32_t      lastCount = 0;
    uint32_t now = millis();

    int duty = (int)(100.0f * sinf(TWO_PI * (float)(now % DRIVE_PERIOD) / (float)DRIVE_PERIOD));
    motorSet(duty);

    if (now - tPrint >= 200) {
        int32_t c  = readEnc();
        float   dt = (now - tPrint) / 1000.0f;
        int32_t d  = c - lastCount;
        lastCount  = c;
        tPrint     = now;
        float rpm  = dt > 0.0f ? (float)d * 60.0f / ((float)kCountsPerRev * dt) : 0.0f;

        Serial.print("duty="); Serial.print(duty);
        Serial.print("  count="); Serial.print(c);
        Serial.print("  rpm="); Serial.println(rpm, 1);
    }

    delay(5);
}
```

</details>

### Step 2. Upload the sketch

Upload as before. The motor stays still until the first rising quarter of the sine wave ramps the duty cycle up.

### Step 3. Observe the result

Watch the wheel driven by M1 and check the Serial Monitor output — it should match the two figures below.

![Motor demo](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example4_motor_demo.gif)

![Motor serial log](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example4_motor_log.gif)

:::tip Counts per revolution
`kCountsPerRev = 7200` = 12 PPR × 4 (quadrature ×4 decoding) × 150 (gear ratio). If you swap the gear motor, update this constant so the rpm reading stays correct.
:::

## Troubleshooting

### Q1: No COM port appears after I plug in the board

A: You are most likely using a charge-only USB cable — swap in a data cable. If the port still does not appear, try another USB port on the PC.

### Q2: The upload fails and the console keeps repeating `Connecting.....`

A: Hold the **BOOT** button on the XIAO, tap **RST**, release **BOOT** to force the board into bootloader mode, then start the upload again.

### Q3: The Serial Monitor prints garbled characters

A: The examples run at **115200** baud — make sure the baud rate selected in the lower-right corner of the Serial Monitor matches.

### Q4: Example 2 prints `IMU init failed`

A: Power-cycle the kit (unplug and replug the USB cable). If the message persists, reseat the XIAO on the carrier board so the I2C pins (D4/D5) make good contact.

### Q5: The servo or motor twitches and the board resets

A: The Motor_5V rail is starved when running on USB power alone. Use a lithium battery on the battery connector or an external 5 V supply — see the power note in Hardware Overview.

### Q6: Example 4 prints `rpm=0.0` even though the wheel spins

A: The encoder leads are not connected — the motor is probably plugged into the **DC motor connector** instead of the **encoder motor connector**. Move the cable to the encoder motor connector so both the motor and encoder leads reach the board.

## Resources

- **[PDF]** [XIAO Robot Board V1.0 Schematic](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Board_V1_0_SCH_261111.pdf)
- **[ZIP]** [XIAO Robot Board V1.0 KiCad Project](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Board_V1_0_KICAD_261111.zip)
- **[STEP]** [XIAO Robot Kit 3D Print Files (0924 optimized version)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Kit_3D_Print_Files_0924_Optimized.stp)
{/* Commented out for now: HD_N20 datasheet (uploaded, not linked yet) - https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/HD_N20_Encoder_Motor_1218_ShortShaft_6V_270RPM_Datasheet_261111.pdf */}
- **[PDF]** [N20 Encoder Motor + Cable Harness Spec (5 V / 170 RPM)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/HR-N20-1218-5V-170-101_Encoder_Motor_with_Cable_Spec_261111.pdf)
- **[PDF]** [FT-1190-C016 Digital Servo Spec (6 V / 1.8 kg·cm)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/FT-1190-C016_Servo_6V_1.8kgcm_Spec_261111.pdf)
