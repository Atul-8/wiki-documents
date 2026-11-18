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
sidebar_position: 1
type: gettingstarted
last_update:
  date: 11/17/2026
  author: Atul
createdAt: '2026-11-11'
updatedAt: '2026-11-17'
url: https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/
---

# Getting Started with XIAO Robot Kit

<div class="table-center">
<table align="center">
  <tr>
    <th>XIAO Robot Kit</th>
    <th>XIAO Robot Board</th>
  </tr>
  <tr>
    <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:320, height:'auto'}}/></div></td>
    <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:320, height:'auto'}}/></div></td>
  </tr>
  <tr>
    <td><div class="get_one_now_container" style={{textAlign:'center'}}><a class="get_one_now_item" href="https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/" target="_blank"><strong><span><font color={'FFFFFF'} size={'4'}> Get One Now 🖱️</font></span></strong></a></div></td>
    <td><div class="get_one_now_container" style={{textAlign:'center'}}><a class="get_one_now_item" href="https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/" target="_blank"><strong><span><font color={'FFFFFF'} size={'4'}> Get One Now 🖱️</font></span></strong></a></div></td>
  </tr>
</table>
</div>

## Introduction

The XIAO Robot Kit is a modular robotics development kit built around the **Seeed Studio XIAO ESP32-S3 Plus**. The carrier board already holds the electronics of a small mobile robot: an ESP32-S3 controller with Wi-Fi and Bluetooth LE, two N20 gear motors with quadrature encoders, four servo channels, a 6-axis IMU and battery management. The kit adds two Feetech FT1190M metal-gear servos and Grove connectors for whatever you attach later. The body is yours to design and print — the motion hardware is already on the board.

### Specification

<table align="center" style={{display:'table', width:'100%'}}>
  <thead>
    <tr><th>Component</th><th>Specification</th><th>Details</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="12">XIAO Robot Board</td><td>Main Controller</td><td>Seeed Studio XIAO ESP32-S3 Plus</td></tr>
    <tr><td>Processor</td><td>Dual-Core ESP32-S3, up to 240 MHz</td></tr>
    <tr><td>Memory</td><td>8 MB PSRAM + 16 MB Flash</td></tr>
    <tr><td>Wireless</td><td>2.4 GHz Wi-Fi + Bluetooth LE 5.0</td></tr>
    <tr><td>Motor Driver</td><td>2 × GC9112 H-Bridge Drivers</td></tr>
    <tr><td>Motor Interfaces</td><td>2 × DC Motor Outputs<br />2 × Encoder Interfaces</td></tr>
    <tr><td>Grove Servo Interfaces</td><td>4 Channels</td></tr>
    <tr><td>Onboard Sensor</td><td>LSM6DS3TR-C 6-Axis IMU</td></tr>
    <tr><td>Grove Expansion</td><td>2 × I²C<br />1 × UART<br />1 × RGB/Digital</td></tr>
    <tr><td>Onboard Buttons</td><td>1 × User button<br />1 × Reset button</td></tr>
    <tr><td>Power Input</td><td>5V USB-C / 7.4V Lithium Battery System</td></tr>
    <tr><td>Development</td><td>Arduino / ESP-IDF</td></tr>
    <tr><td rowspan="10">N20 Encoder Motors</td><td>Motor Type</td><td>Brushed DC Geared Motor with Quadrature Encoder</td></tr>
    <tr><td>Operating Voltage</td><td>DC 2.5–5.5V</td></tr>
    <tr><td>Gear Ratio</td><td>1:101</td></tr>
    <tr><td>No-Load Speed</td><td>170 RPM ±10%</td></tr>
    <tr><td>No-Load Current</td><td>≤90 mA</td></tr>
    <tr><td>Stall Torque</td><td>0.4 kg·cm</td></tr>
    <tr><td>Stall Current</td><td>≤1.3 A</td></tr>
    <tr><td>Starting Voltage</td><td>≤2.0V</td></tr>
    <tr><td>Output Shaft</td><td>3 mm D-Shaft, 8 mm Length</td></tr>
    <tr><td>Connection</td><td>6-Pin Motor &amp; Encoder Harness</td></tr>
    <tr><td rowspan="8">Feetech FT1190M Servos</td><td>Model</td><td>Customized FT1190M (FT-1190-C016)</td></tr>
    <tr><td>Motor &amp; Gears</td><td>Coreless Motor, Metal Gears</td></tr>
    <tr><td>Input Voltage</td><td>3.3–7.4V</td></tr>
    <tr><td>Stall Torque</td><td>3.5 kg·cm @ 6V</td></tr>
    <tr><td>Rated Torque</td><td>1.15 kg·cm @ 6V</td></tr>
    <tr><td>No-Load Speed</td><td>0.10 sec/60° @ 6V</td></tr>
    <tr><td>Dimensions</td><td>23.2 × 12.1 × 25.25 mm</td></tr>
    <tr><td>Weight</td><td>13 ± 2 g</td></tr>
    <tr><td rowspan="2">Wheels</td><td>Drive Wheels</td><td>Ø34 × 6.5 mm, 3 mm D-Shaft Mount</td></tr>
    <tr><td>Ball Caster Wheels</td><td>25 × 25 × 13 mm (L × W × H), with Steel Ball</td></tr>
  </tbody>
</table>

### Features

- **Build your own robot**: the electronics are done, so the structure is the part you get to invent. Print a wheeled chassis, or a character with moving arms.
- **More ways to move, sense and interact**: two independently driven wheels, four servo channels and a 6-axis IMU, plus the two FT1190M metal-gear servos that come in the kit.
- **Encoder feedback**: each N20 motor reports how far its wheel has actually turned, so a sketch can measure motion instead of assuming it from a delay.
- **Wireless**: Wi-Fi and Bluetooth LE on the ESP32-S3 Plus, enough for a phone-controlled robot or a link to an external AI application.
- **AI-assisted coding**: describe the behavior you want and let a coding agent such as Codex or Claude Code write the Arduino sketch with you.

## Hardware Overview

### XIAO Robot Board

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/XIAO_Robot_Board_Overview_all_annotated.jpg" style={{width:700, height:'auto'}}/></div>

Motor and servo connectors, the IMU, the two USB-C ports, the power switch and the battery connector all sit on this board. Later sections use the same names.

:::tip
Motors and servos draw a large current. A computer's USB port alone can be too weak, so power the kit from the **battery connector** for Examples 3 and 4.
:::

### On-board Peripheral Pin Mapping

The **XIAO ESP32-S3 Plus** module forms the core of the kit. The table below maps each on-board hardware function to its dedicated pin.

:::caution `Dxx` is a silkscreen marking, not a GPIO number
A `Dxx` marking is **not** a GPIO number. For example, `D11` maps to `GPIO38`.
:::

<table align="center" style={{display:'table', width:'100%'}}>
  <thead>
    <tr><th>Function</th><th>Pin</th><th>GPIO</th><th>Note</th></tr>
  </thead>
  <tbody>
    <tr><td>SERVO3</td><td>D0</td><td>GPIO1</td><td>Servo 3</td></tr>
    <tr><td>M1 PWM1</td><td>D1</td><td>GPIO2</td><td>Motor 1 drive</td></tr>
    <tr><td>M1 PWM2</td><td>D2</td><td>GPIO3</td><td>Motor 1 drive</td></tr>
    <tr><td>M1 Encoder A</td><td>D3</td><td>GPIO4</td><td>Motor 1 encoder A</td></tr>
    <tr><td>I²C SDA</td><td>D4</td><td>GPIO5</td><td>on-board IMU</td></tr>
    <tr><td>I²C SCL</td><td>D5</td><td>GPIO6</td><td>on-board IMU</td></tr>
    <tr><td>UART TX</td><td>D6</td><td>GPIO43</td><td>Grove UART TX</td></tr>
    <tr><td>UART RX</td><td>D7</td><td>GPIO44</td><td>Grove UART RX</td></tr>
    <tr><td>SERVO4</td><td>D8</td><td>GPIO7</td><td>Servo 4</td></tr>
    <tr><td>SERVO2</td><td>D9</td><td>GPIO8</td><td>Servo 2</td></tr>
    <tr><td>M2 Encoder B</td><td>D10</td><td>GPIO9</td><td>Motor 2 encoder B</td></tr>
    <tr><td>KEY</td><td>D11</td><td>GPIO38</td><td>User button</td></tr>
    <tr><td>IMU Heater PWM</td><td>D12</td><td>GPIO39</td><td>Drives the heater resistor next to the IMU</td></tr>
    <tr><td>RGB</td><td>D13</td><td>GPIO40</td><td>Grove RGB / Digital</td></tr>
    <tr><td>M1 Encoder B</td><td>D14</td><td>GPIO41</td><td>Motor 1 encoder B</td></tr>
    <tr><td>SERVO1</td><td>D15</td><td>GPIO42</td><td>Servo 1</td></tr>
    <tr><td>ADC</td><td>D16</td><td>GPIO10</td><td>On-board ADC</td></tr>
    <tr><td>M2 Encoder A</td><td>D17</td><td>GPIO13</td><td>Motor 2 encoder A</td></tr>
    <tr><td>M2 PWM2</td><td>D18</td><td>GPIO12</td><td>Motor 2 drive</td></tr>
    <tr><td>M2 PWM1</td><td>D19</td><td>GPIO11</td><td>Motor 2 drive</td></tr>
  </tbody>
</table>

## Getting Started

### Hardware Preparation

**Step 1. Connect to the computer**

##### Method 1

Use a **USB-C data cable** to connect the USB-C port on the **XIAO ESP32-S3 Plus** to your computer, then feed the kit's bottom USB-C / battery port from a separate power source such as a power bank or a battery.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/demo_hardware_connection1.jpg" style={{width:700, height:'auto'}}/></div>

##### Method 2

Use a **USB-C data cable** to connect the kit to your computer through the bottom USB-C port, leaving the USB-C port on the **XIAO ESP32-S3 Plus** unplugged. Slide the power switch to **ON**. Note: this method requires the computer's USB-C port to be backed by a self-powered dock or hub, so that the supply current is sufficient.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/demo_hardware_connection2.jpg" style={{width:700, height:'auto'}}/></div>

:::caution Use a data cable
A charge-only cable shows up as no COM port, or the device does not respond at all — swap in a data cable first.
:::

### Software Preparation

Never used a **XIAO ESP32-S3 Plus**, or the Arduino IDE, before? Stop here and read the board's [Getting Started guide](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/) first. It takes you through installing the Arduino IDE, adding the ESP32 board package, and selecting **XIAO_ESP32S3_PLUS** with the right port.

Come back when **Tools > Port** lists your board. Every example on this page uploads the same way.

## Example 1: Button + Hello World Serial Print

The "hello world" of the kit, and the quickest proof that the toolchain works: it prints `hello world N` once a second and counts presses of the on-board **KEY** button. There is nothing to wire.

`setup()` opens the serial port and puts the KEY pin in `INPUT_PULLUP`, so the pin reads `HIGH` at rest and `LOW` only while the button is held. `loop()` never calls `delay(1000)`: it compares `millis()` against the next deadline, so the printing and the button check keep running side by side. A new level counts only after it has held for 10 ms, which turns one physical press into one line of output.

**Step 1. Create the sketch**

In the Arduino IDE, open **File > New Sketch**, delete the template code, and paste the sketch below:

<details>

<summary> Full code </summary>

```c
/* Example 1: prints "hello world N" once per second,
 * and counts presses of the on-board KEY button.
 *
 * Nothing to wire: the KEY button and the USB serial port are on the board. */
#include <Arduino.h>

#define PIN_BUTTON  38          /* silkscreen KEY = GPIO38 */

void setup()
{
    Serial.begin(115200);       /* must match the Serial Monitor baud rate */
    delay(200);                 /* give the port a moment to come up */

    /* KEY shorts to ground when pressed, so the internal pull-up holds the
     * pin HIGH at rest and lets it drop to LOW while the button is held. */
    pinMode(PIN_BUTTON, INPUT_PULLUP);

    Serial.println("01 button + hello world");
    Serial.println("press the on-board key to count");
}

void loop()
{
    uint32_t now = millis();

    /* ---- print one line every 1000 ms, without blocking the loop ---- */
    static uint32_t nextHello = 0;      /* when the next line is due */
    static uint32_t n = 0;              /* how many lines printed so far */
    if ((int32_t)(now - nextHello) >= 0) {
        nextHello = now + 1000;
        Serial.print("hello world ");
        Serial.println(++n);
    }

    /* ---- watch the button ---- */
    static bool     lastRaw    = HIGH;  /* last level seen on the pin */
    static bool     lastStable = HIGH;  /* last level we accepted */
    static uint32_t lastChange = 0;     /* when that level last moved */
    static uint32_t presses    = 0;

    bool cur = (digitalRead(PIN_BUTTON) == LOW);    /* true = pressed */
    if (cur != lastRaw) { lastRaw = cur; lastChange = now; }

    /* A new level counts only after it has held for 10 ms, so one press
     * becomes one line instead of a burst of them. */
    if (now - lastChange >= 10 && cur != lastStable) {
        lastStable = cur;
        if (lastStable) {               /* count on release, not on press */
            presses++;
            Serial.print("button pressed = ");
            Serial.println(presses);
        }
    }

    delay(10);                  /* keeps the loop light for everything else */
}
```

</details>

**Step 2. Upload the sketch**

Confirm **Tools > Board** is `XIAO_ESP32S3_PLUS` and **Tools > Port** shows your board, then click **Upload** (→).

**Step 3. Observe the result**

Open the **Serial Monitor** (magnifier icon, top right) and set the baud rate to **115200**. You should see one `hello world N` line per second. Now press the on-board **KEY** button a few times — each press appends a `button pressed = N` line:

```
01 button + hello world
press the on-board key to count
hello world 1
hello world 2
button pressed = 1
```

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example1_button_demo.gif" style={{width:700, height:'auto'}}/></div>

Now look at the **Serial Monitor** — the same run looks like this:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example1_button_log.gif" style={{width:700, height:'auto'}}/></div>

:::tip
If the Serial Monitor stays empty or prints garbled text, check the selected port and the 115200 baud setting — see [Troubleshooting](#troubleshooting).
:::

## Example 2: IMU 6-Axis Raw Data + Temperature

Reads the on-board **LSM6DS3TR-C** 6-axis IMU over I2C and prints all seven channels at 20 Hz: acceleration in `g`, angular rate in `dps` and the die temperature in `°C`.

The chip sits on the two-wire bus on `D4`/`D5`, so the sketch needs only three small helpers: `wr()` writes one register, `rd()` bursts bytes out of it, and `i16()` joins two little-endian bytes into a signed 16-bit sample. `setup()` reads `WHO_AM_I` first — a wrong answer means a bus problem, and the sketch stops with `IMU init failed` instead of streaming zeros forever — then sets the output rate and the full-scale range. Each loop reads temperature, gyro and accelerometer in one 14-byte burst and scales the counts with the sensitivity those settings imply.

**Step 1. Create the sketch**

Create a new sketch (**File > New Sketch**) and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 2: streams 6-axis acceleration (g), angular rate (dps)
 * and temperature (C) from the LSM6DS3TR-C IMU over I2C.
 *
 * The IMU is on the board: SDA = D4, SCL = D5, address 0x6A. */
#include <Arduino.h>
#include <Wire.h>
#include <stdlib.h>

#define ADDR     0x6A      /* I2C address of the on-board IMU */
#define PIN_SDA  5         /* silkscreen D4 */
#define PIN_SCL  6         /* silkscreen D5 */

/* Registers used below, straight out of the LSM6DS3TR-C datasheet. */
#define WHO_AM_I 0x0F      /* identity register, always reads 0x6A */
#define CTRL1_XL 0x10      /* accelerometer: output rate + full scale */
#define CTRL2_G  0x11      /* gyroscope: output rate + full scale */
#define CTRL3_C  0x12      /* block data update + register auto-increment */
#define OUT_TEMP 0x20      /* first of the 14 output bytes read here */

/* Write one 8-bit register. */
static void wr(uint8_t r, uint8_t v) {
    Wire.beginTransmission(ADDR); Wire.write(r); Wire.write(v); Wire.endTransmission(true);
}

/* Read n bytes starting at register r; false = the bus did not answer. */
static bool rd(uint8_t r, uint8_t *b, uint8_t n) {
    Wire.beginTransmission(ADDR); Wire.write(r);
    if (Wire.endTransmission(false)) return false;
    if (Wire.requestFrom((int)ADDR, (int)n) != n) return false;
    for (uint8_t i = 0; i < n; i++) b[i] = Wire.read();
    return true;
}

/* Join two little-endian bytes into one signed 16-bit sample. */
static int16_t i16(const uint8_t *p) { return (int16_t)(p[0] | p[1] << 8); }

/* Print one "tag=value" field: value right-aligned in w characters,
 * d digits after the decimal point. */
static void p(const char *tag, float v, int w, int d) {
    char s[10];
    Serial.print(tag);
    Serial.print(dtostrf(v, w, d, s));
    Serial.print(' ');
}

void setup() {
    Serial.begin(115200);
    delay(200);

    Wire.begin(PIN_SDA, PIN_SCL);   /* start the I2C bus on D4 / D5 */
    Wire.setClock(400000);          /* 400 kHz fast mode, plenty for this */

    /* Read the identity register first: a wrong answer means the bus or the
     * address is wrong, so stop with a clear message instead of streaming
     * meaningless zeros forever. */
    uint8_t id = 0;
    if (!rd(WHO_AM_I, &id, 1) || id != 0x6A) {
        Serial.println("IMU init failed");
        while (1) delay(1000);
    }

    wr(CTRL1_XL, 0x40);     /* accel 104 Hz, +/-2 g     -> 0.061 mg/LSB */
    wr(CTRL2_G,  0x4C);     /* gyro  104 Hz, +/-2000 dps -> 0.070 dps/LSB */
    wr(CTRL3_C,  0x44);     /* block data update + register auto-increment */
    Serial.println("IMU ready  (ax/ay/az in g, gx/gy/gz in dps, t in C)");
}

void loop() {
    /* One burst covers temperature, gyro and accelerometer: 14 bytes. */
    uint8_t b[14];
    if (!rd(OUT_TEMP, b, 14)) { delay(50); return; }

    /* Convert the raw counts with the sensitivity set in setup(). */
    float t  = 25.0f + i16(&b[0])  / 256.0f;
    float gx = i16(&b[2])  * 0.070f;
    float gy = i16(&b[4])  * 0.070f;
    float gz = i16(&b[6])  * 0.070f;
    float ax = i16(&b[8])  * 0.000061f;
    float ay = i16(&b[10]) * 0.000061f;
    float az = i16(&b[12]) * 0.000061f;

    /* All seven channels on one line, so a plotter sees one sample per row. */
    p("ax=", ax, 6, 3);
    p("ay=", ay, 6, 3);
    p("az=", az, 6, 3);
    p("gx=", gx, 8, 2);
    p("gy=", gy, 8, 2);
    p("gz=", gz, 8, 2);
    p("t=",  t,  6, 2);
    Serial.println("C");

    delay(50);      /* 50 ms + the read time is about 20 Hz */
}
```

</details>

**Step 2. Upload the sketch**

Upload as in Example 1. On boot the sketch checks the IMU's `WHO_AM_I` register and prints `IMU init failed` if the chip does not answer — otherwise you get `IMU ready`.

**Step 3. Observe the result**

With the board lying flat on the desk, the Serial Monitor shows a steady stream at 20 Hz. Try these checks:

- **Flat and still**: `az ≈ 1.0` (gravity points down), `ax ≈ ay ≈ 0`, all three gyro axes near `0`.
- **Tilt the board**: `ax`/`ay` change as you rotate it — they measure gravity's direction.
- **Shake the board**: the gyro readings spike with the motion.

```
IMU ready  (ax/ay/az in g, gx/gy/gz in dps, t in C)
ax= 0.012 ay= 0.008 az= 0.995 gx=   0.12 gy=  -0.03 gz=   0.05 t=25.31 C
```

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example2_imu_demo.gif" style={{width:700, height:'auto'}}/></div>

Now look at the **Serial Monitor** — the same run looks like this:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example2_imu_data.gif" style={{width:700, height:'auto'}}/></div>

## Example 3: Servo Minimal Example

Drives a position servo on **SERVO1** with a 50 Hz PWM signal whose pulse width sweeps between 500 µs and 2500 µs — one full cycle every 4 s.

A hobby servo does not take an angle, it takes a pulse width. `servoPulseUs()` turns a position between -1.0 and +1.0 into 500–2500 µs and clamps anything outside that range, and `servoWrite()` converts the pulse width into the duty cycle the LEDC peripheral needs at 14-bit resolution over a 20 ms period. `loop()` feeds it `sinf()` of the elapsed time, so the horn glides instead of jumping.

### Hardware Setup

Just connect the included servo and its cable to the **SERVO1** connector on the board.

**Step 1. Create the sketch**

Create a new sketch and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 3: sweeps a position servo smoothly across its full range
 * (one sine cycle every 4 s) on servo port SERVO1.
 *
 * Plug the servo cable into SERVO1; nothing else to wire. */
#include <Arduino.h>
#include <math.h>

#define PIN_SERVO      42          /* silkscreen SERVO1 = GPIO42 */

#define PWM_FREQ       50          /* hobby servos expect 50 Hz, i.e. 20 ms */
#define PWM_RES_BITS   14          /* 16384 steps across one period */
#define PWM_PERIOD_US  20000       /* 1 / 50 Hz, in microseconds */

#define SERVO_PULSE_CENTER   1500  /* center position */
#define SERVO_PULSE_MIN      500   /* full travel one way */
#define SERVO_PULSE_MAX      2500  /* full travel the other way */

#define SWEEP_PERIOD         4000  /* one complete sweep, milliseconds */
#define TWO_PI               6.2831853f

/* Map a position between -1.0 and +1.0 to a pulse width in microseconds.
 * Anything outside that range is clamped, so a bad number cannot drive the
 * servo past its mechanical stop. */
static float servoPulseUs(float normPos) {
    if (normPos < -1.0f) normPos = -1.0f;
    if (normPos >  1.0f) normPos =  1.0f;
    return (normPos < 0.0f)
        ? (float)SERVO_PULSE_CENTER + normPos * (float)(SERVO_PULSE_CENTER - SERVO_PULSE_MIN)
        : (float)SERVO_PULSE_CENTER + normPos * (float)(SERVO_PULSE_MAX - SERVO_PULSE_CENTER);
}

/* Move the servo: -1.0 = one end, 0.0 = center, +1.0 = the other end.
 * The pulse width becomes a duty cycle out of 2^14 steps. */
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

    /* Park the pin low first, then hand it to the LEDC peripheral at 50 Hz
     * with 14-bit resolution. */
    pinMode(PIN_SERVO, OUTPUT); digitalWrite(PIN_SERVO, LOW);
    ledcAttach(PIN_SERVO, PWM_FREQ, PWM_RES_BITS);
    servoWrite(0.0f);       /* start from the center position */
}

void loop() {
    static unsigned long tPrint = 0;
    uint32_t now = millis();

    /* sinf() of the elapsed time gives a smooth -1 .. +1 over SWEEP_PERIOD;
     * the modulo keeps the argument small so precision never degrades. */
    float normPos = sinf(TWO_PI * (float)(now % SWEEP_PERIOD) / (float)SWEEP_PERIOD);
    servoWrite(normPos);

    /* Print 10 times a second: enough to follow, not enough to flood. */
    if (now - tPrint >= 100) {
        tPrint = now;
        Serial.print("normPos=");   Serial.print(normPos, 2);
        Serial.print("  us=");      Serial.println(servoPulseUs(normPos), 0);
    }

    delay(20);
}
```

</details>

**Step 2. Upload the sketch**

Upload as before. On boot the servo is commanded to center (`normPos = 0`, 1500 µs) before the sweep starts.

**Step 3. Observe the result**

After power-up the horn moves to center first, then starts sweeping. The Serial Monitor prints a line every 100 ms: `normPos=` is the commanded position between -1.00 and 1.00, `us=` the matching pulse width in microseconds. Watch the horn and the numbers together — they rise and fall in step, as in the two figures below.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example3_servo_demo.gif" style={{width:700, height:'auto'}}/></div>

Now look at the **Serial Monitor** — the same run looks like this:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example3_servo_log.gif" style={{width:700, height:'auto'}}/></div>

:::tip Calibrating the pulse widths
The three macros `SERVO_PULSE_MIN / CENTER / MAX` map a normalized position to a pulse width. If your servo buzzes at the ends of travel, narrow `MIN`/`MAX` — the mechanical limits differ between servo models and depend on the linkage.
:::

## Example 4: Encoder Motor Minimal Example

Spins the **M1** gear motor with a sine-shaped duty cycle and reads its quadrature encoder, so the Serial Monitor can print the commanded duty, the encoder count and the measured speed.

`motorSet()` takes -100 to +100 and puts the magnitude on one of the two direction pins. The encoder is decoded in an interrupt: `onEncoderChange()` reads both channels and adds -1, 0 or +1 from a 16-entry quadrature table, while `readEnc()` copies the total with interrupts off so a 32-bit read cannot be torn. Speed comes from the counts gained between two prints, divided by the 7200 counts one wheel revolution produces.

### Hardware Setup

Just connect the included motor and its cable to the **encoder motor connector**.

**Step 1. Create the sketch**

Create a new sketch and paste the full code below:

<details>

<summary> Full code </summary>

```c
/* Example 4: drives the M1 gear motor with a sine-shaped duty cycle
 * (one cycle every 4 s) and prints encoder count and speed (rpm).
 *
 * Plug the motor cable into the M1 connector; nothing else to wire. */
#include <Arduino.h>
#include <math.h>

#define PIN_M1_IN1  2       /* silkscreen M1 PWM1, D1 */
#define PIN_M1_IN2  3       /* silkscreen M1 PWM2, D2 */
#define PIN_ENC_A   4       /* silkscreen M1 Encoder A, D3 */
#define PIN_ENC_B   41      /* silkscreen M1 Encoder B, D14 */

#define DRIVE_PERIOD 4000   /* one drive cycle, milliseconds */
#define TWO_PI       6.2831853f
#define DEAD_ZONE    15     /* duty below this cannot move the gearbox */

/* 12 pulses per motor revolution x 4 (quadrature x4) x 150 (gear ratio). */
const uint32_t kCountsPerRev = 7200;

/* Quadrature table: index = (previous state << 2) | new state, where a state
 * is (A << 1) | B. The value is the direction of that one step: -1, 0 or +1. */
static const int8_t kQuad[16] = {
     0,  1, -1,  0,
    -1,  0,  0,  1,
     1,  0,  0, -1,
     0, -1,  1,  0
};
static volatile int32_t encCount = 0;   /* written by the ISR, read by loop */
static volatile uint8_t  encState = 0;  /* previous A/B level */

/* Runs on every edge of A or B and adds -1, 0 or +1 to the count. */
void IRAM_ATTR onEncoderChange() {
    uint8_t a    = (uint8_t)digitalRead(PIN_ENC_A);
    uint8_t b    = (uint8_t)digitalRead(PIN_ENC_B);
    uint8_t next = (uint8_t)((a << 1) | b);
    encCount += kQuad[(encState << 2) | next];
    encState  = next;
}

/* Copy the count with interrupts off, so a 32-bit read cannot be torn. */
static int32_t readEnc() {
    noInterrupts();
    int32_t c = encCount;
    interrupts();
    return c;
}

/* Apply a duty cycle from -100 (reverse) to +100 (forward); 0 = coast.
 * The magnitude goes to one direction pin, the other one stays at zero. */
static void motorSet(int duty) {
    if (duty >  100) duty =  100;
    if (duty < -100) duty = -100;
    if (duty > -DEAD_ZONE && duty < DEAD_ZONE) duty = 0;

    uint32_t d = (uint32_t)((duty < 0 ? -duty : duty) * 1023 / 100);   /* 10-bit */
    if (duty > 0)      { ledcWrite(PIN_M1_IN1, d); ledcWrite(PIN_M1_IN2, 0); }
    else if (duty < 0) { ledcWrite(PIN_M1_IN1, 0); ledcWrite(PIN_M1_IN2, d); }
    else               { ledcWrite(PIN_M1_IN1, 0); ledcWrite(PIN_M1_IN2, 0); }
}

void setup() {
    Serial.begin(115200);
    delay(200);

    Serial.println("encoder motor - auto duty");

    /* Both direction pins low = motor stopped. */
    pinMode(PIN_M1_IN1, OUTPUT); digitalWrite(PIN_M1_IN1, LOW);
    pinMode(PIN_M1_IN2, OUTPUT); digitalWrite(PIN_M1_IN2, LOW);

    /* Encoder channels use the internal pull-ups and interrupt on any edge. */
    pinMode(PIN_ENC_A, INPUT_PULLUP);
    pinMode(PIN_ENC_B, INPUT_PULLUP);
    encState = (uint8_t)((digitalRead(PIN_ENC_A) << 1) | digitalRead(PIN_ENC_B));
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_A), onEncoderChange, CHANGE);
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_B), onEncoderChange, CHANGE);

    /* 20 kHz PWM sits above the audible range, so the motor does not whine. */
    ledcAttach(PIN_M1_IN1, 20000, 10);
    ledcAttach(PIN_M1_IN2, 20000, 10);
    motorSet(0);
}

void loop() {
    static unsigned long tPrint   = 0;
    static int32_t      lastCount = 0;
    uint32_t now = millis();

    /* The sine takes the duty from 0 up to 100, back through 0 down to -100:
     * the wheel speeds up, stops and reverses with no abrupt step. */
    int duty = (int)(100.0f * sinf(TWO_PI * (float)(now % DRIVE_PERIOD) / (float)DRIVE_PERIOD));
    motorSet(duty);

    /* Every 200 ms: turn the counts gained since the last print into rpm. */
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

**Step 2. Upload the sketch**

Upload as before. The motor stays still until the first rising quarter of the sine wave ramps the duty cycle up.

**Step 3. Observe the result**

The wheel speeds up, stops, then reverses, in time with the sine. The Serial Monitor prints a line every 200 ms: `duty=` is the commanded value, `count=` the encoder total, `rpm=` the speed worked out from it. `count` climbs and falls as the wheel turns both ways; `rpm` changes sign together with `duty`. Compare with the two figures below.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example4_motor_demo.gif" style={{width:700, height:'auto'}}/></div>

Now look at the **Serial Monitor** — the same run looks like this:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example4_motor_log.gif" style={{width:700, height:'auto'}}/></div>

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

### Q4: The servo or motor twitches and the board resets

A: The Motor_5V rail is starved when running on USB power alone. Use a lithium battery on the battery connector or an external 5 V supply — see the power note in Hardware Overview.

### Q5: Example 4 prints `rpm=0.0` even though the wheel spins

A: Make sure the cable is pushed fully into its socket and locked. If `rpm` still stays at 0, swap in a motor that has never been dropped or knocked — a damaged encoder inside the motor stops reporting.

## Resources

**Hardware Design**
- **📄[Schematic]** [XIAO Robot Board V1.0 Schematic](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Board_V1_0_SCH_261111.pdf)
- **🗃️[PCB Design Files]**
  - [XIAO Robot Board V1.0 KiCad Project](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Board_V1_0_KICAD_261111.zip)
- **📄[Datasheet]** [N20 Encoder Motor with Cable Harness Spec (5 V / 170 RPM)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/HR-N20-1218-5V-170-101_Encoder_Motor_with_Cable_Spec_261111.pdf)
- **📄[Datasheet]** [FT-1190-C016 Digital Servo Spec (6 V / 1.8 kg·cm)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/FT-1190-C016_Servo_6V_1.8kgcm_Spec_261111.pdf)
<!-- - **[PDF]** [HD-N20 Encoder Motor Datasheet (6 V / 270 RPM, short shaft)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/HD_N20_Encoder_Motor_1218_ShortShaft_6V_270RPM_Datasheet_261111.pdf) -->

<!-- Mechanical Design hidden until the 3D model files are cleared for release -->
<!-- **Mechanical Design** -->
<!-- - **📄[3D Model]** [XIAO Robot Kit 3D Print Files (0924 optimized version)](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Resources/XIAO_Robot_Kit_3D_Print_Files_0924_Optimized.stp) -->
