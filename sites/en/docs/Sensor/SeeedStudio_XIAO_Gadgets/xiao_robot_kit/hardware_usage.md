---
description: Hardware usage guide for the XIAO Robot Kit — IMU zero calibration with Euler-angle fusion, BLE control of the encoder motor, and a button-switched WS2812 light board.
title: Hardware Usage
keywords:
  - xiao robot kit
  - esp32s3
  - imu
  - mahony
  - euler angle
  - roll pitch yaw
  - ble
  - servo
  - encoder motor
  - ws2812
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_hardware_usage
sku: 100058302
sidebar_position: 2
last_update:
  date: 11/11/2026
  author: 划水佬
createdAt: '2026-11-11'
updatedAt: '2026-11-11'
url: https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/
---

# Hardware Usage

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:800, height:'auto'}}/></div>

<div class="get_one_now_container" style={{textAlign: 'center'}}>
    <a class="get_one_now_item" href="https://www.seeedstudio.com/XIAO-Robot-Kit-p-100058302.html?utm_source=wiki" target="_blank">
            <strong><span><font color={'FFFFFF'} size={"4"}> Get One Now 🖱️</font></span></strong>
    </a>
</div><br />

## Getting Started

### Hardware Preparation

To work through all three experiments you need the **[XIAO Robot Kit](https://www.seeedstudio.com/XIAO-Robot-Kit-p-100058302.html?utm_source=wiki)**, a USB-C **data** cable, and a computer. Experiments 1 and 2 add nothing: the IMU sits on the carrier board, and the encoder motor arrives with its cable in the box. Experiment 3 is the only one that asks you to supply a part — a WS2812 light board. The photos show a board from CH Maker Ed, but nothing in the sketch depends on it: any WS2812 board wired to `D13` works, and the single number you have to match is `LED_COUNT`, the number of LEDs it carries.

:::tip
Motors and servos draw a large current. A computer's USB port alone can be too weak, so power the kit from the **battery connector** when you drive them.
:::

### Software Preparation

Arduino IDE with the ESP32 board package installed, and `XIAO_ESP32S3_PLUS` selected under **Tools > Board** — [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/#software-preparation) covers the install if you have not done it yet. Nothing beyond the ESP32 core needs installing: all three sketches build with only what the board package already ships with.

## Experiment 1: IMU Zero Calibration + Euler Angle Fusion

**Step 1. Create the sketch**

In the Arduino IDE, open **File > New Sketch**, delete the template code, and paste the sketch below:

<details>

<summary> Full code </summary>

```c
/* SPDX-License-Identifier: MIT */
/* Experiment 1: LSM6DS3TR-C + Mahony AHRS -> roll/pitch/yaw (deg), zero-lib.
 * Warm-up 12s -> auto-cal 6s (hold still). Still-tracker nulls yaw drift.
 * Press KEY (D11) to re-zero. I2C SDA=D4 SCL=D5, addr 0x6A, Serial 115200. */

#include <Arduino.h>
#include <Wire.h>
#include <math.h>
#include <stdlib.h>

#define ADDR       0x6A
#define WHO_AM_I   0x0F
#define CTRL1_XL   0x10
#define CTRL2_G    0x11
#define CTRL3_C    0x12
#define OUT_TEMP_L 0x20
#define OUTX_L_G   0x22
#define OUTX_L_XL  0x28

#define PIN_SDA    5      // D4
#define PIN_SCL    6      // D5
#define PIN_BUTTON 38     // KEY (D11), active low

#define ACCEL_LSB  0.000061f  // +-2g, g/LSB
#define GYRO_LSB   0.070f     // +-2000dps, dps/LSB

#define ODR_HZ     104
#define PERIOD_US  (1000000u / ODR_HZ)
#define CALIB_MS   6000
#define SETTLE_MS  12000

#define KP         2.0f
#define KI         0.0f
#define STILL_DPS  1.0f    // sum|gyro| under this = still
#define STILL_MS   1500    // still time before auto-null
#define BIAS_ALPHA 0.01f   // bias leak per tick

#define RAD2DEG    57.2958f
#define PI         3.14159265f

struct Quat { float w, x, y, z; };
struct Mahony { Quat q; float kp, ki, dt; float integral[3]; };
struct ImuSample { float gx, gy, gz, ax, ay, az; };
struct ImuBias { float gx, gy, gz, ax, ay, az; };

static Mahony    mahony;
static ImuBias   bias;
static ImuSample cur;
static float     imuTemp;
static unsigned long gLastUs;

static void wr(uint8_t r, uint8_t v) {
    Wire.beginTransmission(ADDR); Wire.write(r); Wire.write(v); Wire.endTransmission(true);
}
static bool rd(uint8_t r, uint8_t *b, uint8_t n) {
    Wire.beginTransmission(ADDR); Wire.write(r);
    if (Wire.endTransmission(false)) return false;
    if (Wire.requestFrom((int)ADDR, (int)n) != n) return false;
    for (uint8_t i = 0; i < n; i++) b[i] = Wire.read();
    return true;
}
static int16_t i16(const uint8_t *p) { return (int16_t)(p[0] | p[1] << 8); }

static bool readImu() {
    uint8_t b[6];
    if (!rd(OUTX_L_G, b, 6)) return false;
    cur.gx = i16(&b[0]) * GYRO_LSB;
    cur.gy = i16(&b[2]) * GYRO_LSB;
    cur.gz = i16(&b[4]) * GYRO_LSB;
    if (!rd(OUTX_L_XL, b, 6)) return false;
    cur.ax = i16(&b[0]) * ACCEL_LSB;
    cur.ay = i16(&b[2]) * ACCEL_LSB;
    cur.az = i16(&b[4]) * ACCEL_LSB;
    uint8_t t[2];
    if (rd(OUT_TEMP_L, t, 2)) imuTemp = 25.0f + (float)i16(t) / 256.0f;
    return true;
}

static void quat_identity() { mahony.q.w = 1.0f; mahony.q.x = mahony.q.y = mahony.q.z = 0.0f; }

static void quat_normalize() {
    float n = sqrtf(mahony.q.w * mahony.q.w + mahony.q.x * mahony.q.x
                  + mahony.q.y * mahony.q.y + mahony.q.z * mahony.q.z);
    if (n < 1e-6f) { quat_identity(); return; }
    mahony.q.w /= n; mahony.q.x /= n; mahony.q.y /= n; mahony.q.z /= n;
}

// quaternion -> ZYX euler (rad)
static void quat_to_euler(float *roll, float *pitch, float *yaw) {
    float w = mahony.q.w, x = mahony.q.x, qy = mahony.q.y, z = mahony.q.z;
    *roll  = atan2f(2.0f * (w * x + qy * z), 1.0f - 2.0f * (x * x + qy * qy));
    float sp = 2.0f * (w * qy - z * x);
    *pitch = (sp >= 1.0f) ? PI * 0.5f : (sp <= -1.0f) ? -PI * 0.5f : asinf(sp);
    *yaw   = atan2f(2.0f * (w * z + x * qy), 1.0f - 2.0f * (qy * qy + z * z));
}

static void mahony_init() {
    quat_identity();
    mahony.kp = KP; mahony.ki = KI; mahony.dt = 1.0f / ODR_HZ;
    mahony.integral[0] = mahony.integral[1] = mahony.integral[2] = 0.0f;
}

static void mahony_update(float ax, float ay, float az, float gx, float gy, float gz) {
    const float dt = mahony.dt;
    const float twoKp = 2.0f * mahony.kp, twoKi = 2.0f * mahony.ki;

    float n = sqrtf(ax * ax + ay * ay + az * az);
    if (n < 1e-6f) return;
    ax /= n; ay /= n; az /= n;

    float q0 = mahony.q.w, q1 = mahony.q.x, q2 = mahony.q.y, q3 = mahony.q.z;

    // estimated gravity dir, error = cross(measured, estimated)
    float vx = 2.0f * (q1 * q3 - q0 * q2);
    float vy = 2.0f * (q0 * q1 + q2 * q3);
    float vz = q0 * q0 - q1 * q1 - q2 * q2 + q3 * q3;
    float ex = ay * vz - az * vy;
    float ey = az * vx - ax * vz;
    float ez = ax * vy - ay * vx;

    mahony.integral[0] += twoKi * ex * dt;
    mahony.integral[1] += twoKi * ey * dt;
    mahony.integral[2] += twoKi * ez * dt;

    const float d2r = PI / 180.0f;   // dps -> rad/s
    gx = gx * d2r + twoKp * ex + mahony.integral[0];
    gy = gy * d2r + twoKp * ey + mahony.integral[1];
    gz = gz * d2r + twoKp * ez + mahony.integral[2];

    // first-order quaternion integration
    float hdt = 0.5f * dt;
    mahony.q.w += hdt * (-q1 * gx - q2 * gy - q3 * gz);
    mahony.q.x += hdt * ( q0 * gx + q2 * gz - q3 * gy);
    mahony.q.y += hdt * ( q0 * gy - q1 * gz + q3 * gx);
    mahony.q.z += hdt * ( q0 * gz + q1 * gy - q2 * gx);

    quat_normalize();
}

static void calibrateImu() {
    float gxs = 0, gys = 0, gzs = 0, axs = 0, ays = 0, azs = 0;
    int n = 0;
    unsigned long t0 = millis(), tick = micros();
    while (millis() - t0 < CALIB_MS) {
        if ((unsigned long)(micros() - tick) >= PERIOD_US) {
            tick = micros();
            if (readImu()) {
                gxs += cur.gx; gys += cur.gy; gzs += cur.gz;
                axs += cur.ax; ays += cur.ay; azs += cur.az;
                n++;
            }
        }
    }
    float inv = 1.0f / n;
    float gxm = gxs * inv, gym = gys * inv, gzm = gzs * inv;
    float axm = axs * inv, aym = ays * inv, azm = azs * inv;

    // remove 1g from the dominant accel axis
    float aa[3] = { fabsf(axm), fabsf(aym), fabsf(azm) };
    int axis = 0;
    if (aa[1] > aa[0]) axis = 1;
    if (aa[2] > aa[axis]) axis = 2;
    float av[3] = { axm, aym, azm };
    float sign = (av[axis] > 0.0f) ? 1.0f : -1.0f;

    bias.gx = gxm; bias.gy = gym; bias.gz = gzm;
    bias.ax = axm; bias.ay = aym; bias.az = azm;
    if (axis == 0)       bias.ax -= sign * 1.0f;
    else if (axis == 1)  bias.ay -= sign * 1.0f;
    else                 bias.az -= sign * 1.0f;
}

static void recalibrate() {
    Serial.println("re-cal (keep still...)");
    calibrateImu();
    mahony_init();
    gLastUs = micros();
}

// right-aligned, fixed-width "+12.3" style
static void fmtAng(char *out, float deg, int width) {
    char m[12];
    dtostrf(deg < 0.0f ? -deg : deg, width - 1, 1, m);
    int i = 0;
    while (i < width - 1 && m[i] == ' ') i++;   // skip dtostrf pad
    int o = 0;
    while (o < i) out[o++] = ' ';
    out[o++] = (deg < 0.0f) ? '-' : '+';
    for (int k = i; k < width - 1; k++) out[o++] = m[k];
    out[o] = '\0';
}

void setup() {
    Serial.begin(115200);
    delay(200);
    Wire.begin(PIN_SDA, PIN_SCL);
    Wire.setClock(400000);
    pinMode(PIN_BUTTON, INPUT_PULLUP);

    uint8_t id = 0;
    if (!rd(WHO_AM_I, &id, 1) || id != 0x6A) {
        Serial.println("IMU init failed");
        while (1) delay(1000);
    }
    wr(CTRL1_XL, 0x40);   // +-2g @104Hz
    wr(CTRL2_G,  0x4C);   // +-2000dps @104Hz
    wr(CTRL3_C,  0x44);   // BDU + IF_INC
    mahony_init();

    Serial.println("warm-up...");
    for (unsigned long s = millis(); millis() - s < SETTLE_MS; ) {
        readImu(); delay(500);
        Serial.print("temp="); Serial.println(imuTemp, 1);
    }
    Serial.println("keep still...");
    calibrateImu();
    gLastUs = micros();
    Serial.println("roll=deg pitch=deg yaw=deg temp=C");
}

void loop() {
    static unsigned long tPrint = 0;
    static uint32_t stillSince = 0;

    // button: debounce + falling edge -> re-zero
    static bool lastRaw = HIGH, lastStable = HIGH;
    static uint32_t lastChange = 0;
    uint32_t bNow = millis();
    bool bCur = (digitalRead(PIN_BUTTON) == LOW);
    if (bCur != lastRaw) { lastRaw = bCur; lastChange = bNow; }
    if ((uint32_t)(bNow - lastChange) >= 10 && bCur != lastStable) {
        lastStable = bCur;
        if (lastStable) recalibrate();
    }

    // non-blocking tick, measured dt
    unsigned long nowUs = micros();
    if (nowUs - gLastUs >= PERIOD_US) {
        mahony.dt = (float)(nowUs - gLastUs) / 1000000.0f;
        gLastUs = nowUs;
        if (readImu()) {
            float gx = cur.gx - bias.gx;
            float gy = cur.gy - bias.gy;
            float gz = cur.gz - bias.gz;

            // still-tracker: null residual zero-rate so yaw holds flat
            if (fabsf(gx) + fabsf(gy) + fabsf(gz) < STILL_DPS) {
                if (stillSince == 0) stillSince = millis();
                if (millis() - stillSince >= STILL_MS) {
                    bias.gx += BIAS_ALPHA * gx;
                    bias.gy += BIAS_ALPHA * gy;
                    bias.gz += BIAS_ALPHA * gz;
                }
            } else {
                stillSince = 0;
            }

            mahony_update(cur.ax - bias.ax, cur.ay - bias.ay, cur.az - bias.az,
                          gx, gy, gz);
        }
    }

    uint32_t now = millis();
    if (now - tPrint >= 20) {
        tPrint = now;
        float roll, pitch, yaw;
        quat_to_euler(&roll, &pitch, &yaw);
        char rb[12], pb[12], yb[12];
        fmtAng(rb, roll  * RAD2DEG, 6);
        fmtAng(pb, pitch * RAD2DEG, 6);
        fmtAng(yb, yaw   * RAD2DEG, 7);
        Serial.print("roll=");   Serial.print(rb);
        Serial.print("  pitch="); Serial.print(pb);
        Serial.print("  yaw=");   Serial.print(yb);
        Serial.print("  temp=");  Serial.print(imuTemp, 1);
        Serial.println("C");
    }
    delay(0);   // yield to RTOS (loop is non-blocking but time-stamped)
}
```

</details>

**Step 2. Upload the sketch**

Confirm **Tools > Board** is `XIAO_ESP32S3_PLUS` and **Tools > Port** shows your board, then click **Upload** (→).

**Step 3. Observe the result**

Open the **Serial Monitor** (the magnifier icon, top right) and set the baud rate to **115200**. For the first 12 seconds the sketch prints the die temperature twice a second; the next 6 seconds measure the gyro offset; after that the angles start scrolling:

```
warm-up...
temp=27.4
keep still...
roll=deg pitch=deg yaw=deg temp=C
roll=  +0.3  pitch=  -1.2  yaw=   +0.0  temp=27.9C
roll=  +0.4  pitch=  -1.1  yaw=   +0.1  temp=27.9C
```

Tilt the board forward and back, then roll it side to side, and watch which angle answers:

![IMU demo](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/example2_imu_demo.gif)

![IMU Euler angle serial log](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment1_imu_euler_log.gif)

Set the board down flat and press the on-board **KEY** button: the sketch prints `re-cal (keep still...)`, measures the offset again, and the angles return to near zero.

:::tip
`yaw` is relative, not absolute. There is no magnetometer on the board, so the sketch only knows how far it has turned since the filter was last reset — it has no idea which way north is. Press **KEY** to re-zero, and expect a slow drift over long sessions.
:::

### Re-zeroing and tuning

A gyro never reads exactly zero, and the leftover offset gets integrated into a `yaw` that slowly drifts. The sketch measures that offset while the board sits still (that is the 18-second warm-up), and **KEY** re-measures it on demand — no reflash needed.

Four `#define`s at the top of the sketch cover the cases where the defaults do not fit. The defaults are what produced the log above.

| Constant | Default | Reach for it when |
| --- | --- | --- |
| `KP` | 2.0 | A tilt takes too long to follow (raise it), or the angles twitch (lower it). |
| `STILL_DPS` | 1.0 | `yaw` still drifts with the board sitting flat (lower it), or a slow deliberate turn is mistaken for standing still (raise it). |
| `BIAS_ALPHA` | 0.01 | Drift comes back quickly (raise it) — but a genuine slow rotation then gets pulled toward zero. |
| `CALIB_MS` | 6000 | Re-zeroing is not repeatable enough (raise it), at the cost of a longer wait on **KEY**. |

## Experiment 2: BLE Control of the Encoder Motor

The hardware side is one cable — no jumpers, no breadboard:

![Experiment 2 hardware connection](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_hardware_connection.jpg)

**Step 1. Create the sketch**

In the Arduino IDE, open **File > New Sketch**, delete the template code, and paste the sketch below:

<details>

<summary> Full code </summary>

```c
/* SPDX-License-Identifier: MIT */
/* Experiment 2: BLE Encoder Motor, BLE | Serial -> M1 encoder motor, exact position move.
 *   set 1  forward 10 rev    set 0  stop (coast)    set 1 20  forward 20 rev
 *   set -1 reverse 10 rev    revs = 1..255, default REVS_PER_CMD
 * target = count_now + dir * revs * 7200 counts, so overshoot never accumulates.
 * The move is measured when the command is executed (BLE writes are queued, so
 * that is at most one loop period after arrival). Encoder: 7200 counts/rev (x4
 * quadrature). Commands are case-insensitive.
 * BLE: "XIAO Robot Kit", service d3f1e2a4-0000-..., char d3f1e2a4-0001-... (WRITE|WRITE_NR). */

#include <Arduino.h>
#include <string.h>
#include <stdio.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEAdvertising.h>

#define PIN_M1_IN1          2      /* D1 */
#define PIN_M1_IN2          3      /* D2 */
#define PIN_ENC_A           4      /* D3 */
#define PIN_ENC_B           41     /* D14 */

#define MOTOR_PWM_FREQ      20000
#define MOTOR_PWM_RES       10
#define DEAD_ZONE           15     /* |duty| below this coasts */
#define RUN_DUTY            70     /* duty % while moving */
/* Drive polarity calibration: +duty MUST increase the encoder count, otherwise the
 * position loop can never converge. Measured on this board: +duty decreased the
 * count, hence MOTOR_INVERT = 1. If `set 1` spins the wrong way physically, swap
 * the two M1 wires AND set this to 0 (both changes together keep the loop valid). */
#define MOTOR_INVERT        1

#define REVS_PER_CMD        10     /* default travel of `set ±1`; `set 1 <n>` overrides */
#define COUNTS_PER_REV      7200   /* 12 PPR x 4 quadrature x 150 gear */
#define GUARD_REVS          2      /* max wrong-way travel before abort (capped by revs) */
#define CMD_MAX_LEN         32     /* longest accepted command line, Serial and BLE */

#define SERVICE_UUID        "d3f1e2a4-0000-4b6e-9c1d-5a7b8c9d0e1f"
#define CHARACTERISTIC_UUID "d3f1e2a4-0001-4b6e-9c1d-5a7b8c9d0e1f"
#define BLE_DEVICE_NAME     "XIAO Robot Kit"

/* command = run state; "stop" is simply the command to go back to idle */
enum : uint8_t { CMD_NONE = 0, CMD_FWD, CMD_REV };
static const char *const kStateName[] = { "IDLE", "FWD", "REV" };

static uint8_t gState  = CMD_NONE;   /* loop only */
static int32_t gTarget = 0;          /* absolute encoder count to reach */
static int32_t gStart  = 0;          /* count when the current move began */
static int32_t gGuard  = 0;          /* wrong-way abort distance, in counts */

/* 4-slot queue of packed commands ((revs<<8)|cmd): BLE task writes, loop reads */
#define CMD_QUEUE_LEN 4
static volatile uint16_t gQueue[CMD_QUEUE_LEN];
static volatile uint8_t gQHead = 0, gQTail = 0;

/* quadrature x4 transition table: (prev<<2)|next -> step */
static const int8_t kQuad[16] = { 0, 1, -1, 0,  -1, 0, 0, 1,  1, 0, 0, -1,  0, -1, 1, 0 };
static volatile int32_t encCount = 0;
static volatile uint8_t encState = 0;

static void IRAM_ATTR onEncoderChange() {
    uint8_t n = (uint8_t)((digitalRead(PIN_ENC_A) << 1) | digitalRead(PIN_ENC_B));
    encCount += kQuad[(encState << 2) | n];
    encState  = n;
}

static int32_t readEnc() { noInterrupts(); int32_t c = encCount; interrupts(); return c; }

static void pushCmd(uint16_t packed) {
    uint8_t h = gQHead;
    if ((uint8_t)(h - gQTail) >= CMD_QUEUE_LEN) return;   /* full: drop */
    gQueue[h % CMD_QUEUE_LEN] = packed;
    gQHead = (uint8_t)(h + 1);
}

/* duty -100..100, 0 = coast, +duty = count increasing */
static void motorSet(int duty) {
    if (MOTOR_INVERT) duty = -duty;
    if (duty >  100) duty =  100;
    if (duty < -100) duty = -100;
    if (duty > -DEAD_ZONE && duty < DEAD_ZONE) duty = 0;
    uint32_t d = (uint32_t)((duty < 0 ? -duty : duty) * ((1u << MOTOR_PWM_RES) - 1) / 100);
    ledcWrite(PIN_M1_IN1, duty > 0 ? d : 0);
    ledcWrite(PIN_M1_IN2, duty < 0 ? d : 0);
}

/* all motor action happens here, in the loop task. packed = (revs<<8)|cmd,
 * with revs already guaranteed to be 1..255 by parseCmd() */
static void execCmd(uint16_t packed) {
    uint8_t cmd  = (uint8_t)(packed & 0xFF);
    int32_t revs = (int32_t)(packed >> 8);
    if (cmd == CMD_NONE) {
        gState = CMD_NONE;
        motorSet(0);
        Serial.println("stop (coast)");
        return;
    }
    gState = cmd;
    int8_t  dir = (cmd == CMD_FWD) ? +1 : -1;
    gStart  = readEnc();
    gTarget = gStart + dir * revs * COUNTS_PER_REV;
    gGuard  = (revs < GUARD_REVS ? revs : GUARD_REVS) * COUNTS_PER_REV;
    motorSet(dir * RUN_DUTY);
    Serial.printf("go %s %ld rev (from %ld target %ld)\n", dir > 0 ? "fwd" : "rev",
                  (long)revs, (long)gStart, (long)gTarget);
}

/* parse "set N [revs]" -> packed (revs<<8)|CMD_*, 0 if invalid (case tolerant) */
static uint16_t parseCmd(const char *s) {
    char buf[CMD_MAX_LEN];
    strncpy(buf, s, sizeof(buf) - 1);
    buf[sizeof(buf) - 1] = '\0';
    for (char *p = buf; *p; p++) if (*p >= 'a' && *p <= 'z') *p -= 32;

    int n = 0, revs = 0;
    /* 3-digit width caps keep a long digit run from overflowing int */
    if (sscanf(buf, "SET %3d %3d", &n, &revs) < 1) {
        Serial.println("bad cmd (set 1 | set 0 | set -1 [revs])");
        return 0;
    }
    uint8_t cmd = (n > 0) ? CMD_FWD : (n < 0) ? CMD_REV : CMD_NONE;   /* 0 = stop */
    if (revs < 1 || revs > 255) revs = REVS_PER_CMD;   /* out of range -> default */
    return (uint16_t)(((uint16_t)revs << 8) | cmd);
}

class CmdCallbacks : public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *p) override {
        String v = p->getValue();
        uint16_t c = v.length() ? parseCmd(v.c_str()) : (uint16_t)0;
        if (c) pushCmd(c);
    }
};

/* BLE stops advertising on connect and never resumes it: re-arm every time */
static volatile bool gConnected = false;   /* written from the BLE task */
static volatile bool gAdvActive = false;

static void startAdvertising() {
    BLEAdvertising *adv = BLEDevice::getAdvertising();
    adv->setScanResponse(true);

    BLEAdvertisementData advData;                       /* primary packet */
    advData.setFlags(0x06);                             /* general discoverable */
    advData.setCompleteServices(BLEUUID(SERVICE_UUID));
    adv->setAdvertisementData(advData);

    BLEAdvertisementData respData;                      /* scan response: full name */
    respData.setName(BLE_DEVICE_NAME);
    adv->setScanResponseData(respData);

    BLEDevice::startAdvertising();
    gAdvActive = true;
}

class ServerCallbacks : public BLEServerCallbacks {
    void onConnect(BLEServer *) override {
        gConnected = true;
        gAdvActive = false;            /* the stack stopped advertising */
        Serial.println("[BLE] connected");
    }
    void onDisconnect(BLEServer *) override {
        gConnected = false;
        gAdvActive = false;            /* loop() re-arms the advertising */
        Serial.println("[BLE] disconnected -> will re-advertise");
    }
};

static bool bleInit() {
    if (!BLEDevice::init(BLE_DEVICE_NAME)) return false;
    BLEServer *srv = BLEDevice::createServer();
    srv->setCallbacks(new ServerCallbacks());
    BLEService *svc = srv->createService(SERVICE_UUID);
    BLECharacteristic *ch = svc->createCharacteristic(
        CHARACTERISTIC_UUID, BLECharacteristic::PROPERTY_WRITE | BLECharacteristic::PROPERTY_WRITE_NR);
    ch->setCallbacks(new CmdCallbacks());
    svc->start();
    startAdvertising();
    return true;
}

void setup() {
    Serial.begin(115200);
    delay(200);
    Serial.println("Experiment 2: BLE Encoder Motor (M1 D1/D2, enc D3/D14)");

    pinMode(PIN_M1_IN1, OUTPUT); digitalWrite(PIN_M1_IN1, LOW);
    pinMode(PIN_M1_IN2, OUTPUT); digitalWrite(PIN_M1_IN2, LOW);
    pinMode(PIN_ENC_A, INPUT_PULLUP);
    pinMode(PIN_ENC_B, INPUT_PULLUP);
    encState = (uint8_t)((digitalRead(PIN_ENC_A) << 1) | digitalRead(PIN_ENC_B));
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_A), onEncoderChange, CHANGE);
    attachInterrupt(digitalPinToInterrupt(PIN_ENC_B), onEncoderChange, CHANGE);

    ledcAttach(PIN_M1_IN1, MOTOR_PWM_FREQ, MOTOR_PWM_RES);
    ledcAttach(PIN_M1_IN2, MOTOR_PWM_FREQ, MOTOR_PWM_RES);
    motorSet(0);

    if (!bleInit()) { Serial.println("BLE init failed"); while (1) delay(1000); }

    Serial.printf("BLE up: name=%s\n  service=%s\n  char(WRITE)=%s\n",
                  BLE_DEVICE_NAME, SERVICE_UUID, CHARACTERISTIC_UUID);
    Serial.println("phone: scan with nRF Connect / LightBlue (not system Bluetooth)");
    Serial.println("cmds: set 1 | set 0 | set -1   (optional revs: set 1 20)");
}

void loop() {
    static String   line;
    static uint32_t tPrint = 0, tBlue = 0;
    static int32_t  lastCount = 0;
    uint32_t now = millis();

    /* local Serial command (same task as the consumer, so run it directly) */
    while (Serial.available()) {
        char c = (char)Serial.read();
        if (c == '\n' || c == '\r') {
            line.trim();
            if (line.length()) {
                uint16_t cmd = parseCmd(line.c_str());
                if (cmd) execCmd(cmd);
            }
            line = "";
        } else if (line.length() < CMD_MAX_LEN - 1) {
            line += c;
        }
    }

    /* drain BLE commands */
    while (gQTail != gQHead) {
        uint16_t cmd = gQueue[gQTail % CMD_QUEUE_LEN];
        gQTail = (uint8_t)(gQTail + 1);
        execCmd(cmd);
    }

    /* reached target? also abort if it runs the wrong way (polarity/encoder fault) */
    if (gState != CMD_NONE) {
        int32_t c   = readEnc();
        int32_t dir = (gState == CMD_FWD) ? +1 : -1;
        if (dir * (gTarget - c) <= 0) {
            Serial.printf("done: %.2f rev, enc=%ld (%.2f rev past target)\n",
                          (double)(dir * (c - gStart)) / COUNTS_PER_REV, (long)c,
                          (double)(dir * (c - gTarget)) / COUNTS_PER_REV);
            execCmd(CMD_NONE);
        } else if (dir * (c - gStart) < -gGuard) {
            Serial.println("ABORT: motor turns the wrong way (check MOTOR_INVERT / wiring)");
            execCmd(CMD_NONE);
        }
    }

    /* keep the device discoverable */
    if (now - tBlue >= 1000 && !gConnected && !gAdvActive) {
        tBlue = now;
        startAdvertising();
        Serial.println("[BLE] advertising restarted");
    }

    /* telemetry */
    if (now - tPrint >= 200) {
        int32_t c  = readEnc();
        float   dt = (now - tPrint) / 1000.0f;
        float   rpm = dt > 0.0f
                    ? (float)(c - lastCount) * 60.0f / ((float)COUNTS_PER_REV * dt) : 0.0f;
        lastCount = c;
        tPrint    = now;
        Serial.printf("state=%s count=%ld rpm=%.1f remain=%ld (%.2f rev) target=%ld\n",
                      kStateName[gState], (long)c, rpm,
                      (long)(gTarget - c),
                      (double)(gTarget - c) / COUNTS_PER_REV, (long)gTarget);
    }

    delay(5);
}
```

</details>

**Step 2. Upload the sketch**

Confirm **Tools > Board** is `XIAO_ESP32S3_PLUS` and **Tools > Port** shows your board, then click **Upload** (→).

**Step 3. Send commands over Bluetooth LE**

The sketch advertises as **XIAO Robot Kit** and waits for a write. Your phone's system Bluetooth settings cannot write to a characteristic, so you need a BLE scanner app — the walkthrough below uses **nRF Connect**:

- Android: [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en)
- iOS: [nRF Connect](https://apps.apple.com/us/app/nrf-connect-for-mobile/id1054362403)

:::note
nRF Connect is a third-party app published by Nordic Semiconductor. It is not part of the kit, Seeed Studio does not maintain it, and the two links above point at the official store listings. *nRF Connect* is a trademark of Nordic Semiconductor.
:::

Then follow the four screenshots: scan for **XIAO Robot Kit** and connect, open **Unknown Service**, and type a command into the write field. Keep the data format at **UTF-8** rather than hex.

<table align="center">
 <tr>
     <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_nrfconnect_scan_and_connect.jpg" style={{width:200, height:'auto'}}/></div></td>
     <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_nrfconnect_open_characteristic.jpg" style={{width:200, height:'auto'}}/></div></td>
     <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_nrfconnect_write_format.jpg" style={{width:200, height:'auto'}}/></div></td>
     <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_nrfconnect_send_command.jpg" style={{width:200, height:'auto'}}/></div></td>
  </tr>
</table>

Write one of these lines — the Serial Monitor accepts the same ones, so you can skip the phone entirely:

| Command | What the motor does |
| --- | --- |
| `set 1` | Forward for 10 revolutions, then stop. |
| `set -1` | Reverse for 10 revolutions, then stop. |
| `set 1 20` | Forward for 20 revolutions. The second number is the revolution count, 1–255. |
| `set 0` | Stop immediately and let the motor coast. |

Commands are case-insensitive, and a command without a count uses the default 10 revolutions.

### Reading the log

Open the **Serial Monitor** at **115200**. The sketch prints its BLE name and UUIDs once at boot, then one line per command plus a telemetry line every 200 ms. Here is a session started from the phone:

```
[BLE] connected
go fwd 10 rev (from 0 target 72000)
state=FWD count=14400 rpm=150.0 remain=57600 (8.00 rev) target=72000
done: 10.00 rev, enc=72000 (0.00 rev past target)
```

![BLE encoder motor serial log](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment2_ble_encoder_motor_log.gif)

## Experiment 3: Button-Switched RGB LED

:::note
The light board used in this experiment is a WS2812 board from **CH Maker Ed**, a third party. It is not part of the kit — you supply your own — and it is not required to be that particular board: the sketch only sends the standard WS2812 bit stream, so any WS2812 board wired to `D13` behaves the same way.
:::

**Step 1. Create the sketch**

In the Arduino IDE, open **File > New Sketch**, delete the template code, and paste the sketch below.

Set `LED_COUNT` to the number of LEDs on your board before uploading; the sketch ships with `8`. Data is generated for that many LEDs only, so a longer strip stays dark past `LED_COUNT`, and on a shorter strip the extra data reaches no LED at all.

<details>

<summary> Full code </summary>

```c
/* SPDX-License-Identifier: MIT */
/* Experiment 3: Button-Switched RGB LED - press KEY and the whole WS2812 strip switches to the next color,
 * but the LEDs turn over one at a time: LED0 first, then one more every
 * SWEEP_MS, so the change crawls along the strip instead of jumping. The palette
 * loops OFF -> RED -> GREEN -> BLUE. The sweep is non-blocking, so a press
 * arriving mid-sweep is simply ignored.
 * Zero library, core RMT only. Button KEY = D11/GPIO38 (active low), strip = D13.
 */

#include <Arduino.h>
#include <esp32-hal-rmt.h>

/* ── pins ── */
#define PIN_WS2812  40          /* D13, strip data in */
#define PIN_BUTTON  38          /* D11 / KEY, low while pressed */

/* ── strip size and WS2812 bit timing ── */
#define LED_COUNT   8           /* raise if your strip has more LEDs */
#define RMT_TICK_HZ 20000000u   /* 20 MHz: one RMT tick is 50 ns */
#define T1H         16          /* 16 * 50 ns = 800 ns high */
#define T1L         9           /*  9 * 50 ns = 450 ns low */
#define T0H         8           /*  8 * 50 ns = 400 ns high */
#define T0L         17          /* 17 * 50 ns = 850 ns low */
#define RESET_US    60          /* low for > 50 us latches the frame */

/* ── behavior ── */
#define SWEEP_MS    60          /* time between two LEDs during a change */
#define DEBOUNCE_MS 10          /* the button must be stable that long */

/* Palette, cycled in order; index 0 is "off". */
static const uint8_t kPalette[][3] = {
    {0, 0, 0}, {255, 0, 0}, {0, 255, 0}, {0, 0, 255},
};
#define N_COLOR ((uint8_t)(sizeof(kPalette) / sizeof(kPalette[0])))

static uint8_t sPix[LED_COUNT][3];              /* r, g, b of every LED */
static rmt_data_t sFrame[LED_COUNT * 24];       /* RMT symbols, 24 bits per LED */

static uint8_t gColor = 0;          /* palette index being painted */
static uint8_t gNext = LED_COUNT;   /* next LED to paint; LED_COUNT = idle */
static unsigned long gStepAt = 0;   /* millis() when the next LED is due */

/* One bit = level0/duration0 then level1/duration1, 1.25 us in total. */
static void putBit(rmt_data_t *d, uint8_t b) {
    d->duration0 = b ? T1H : T0H;
    d->duration1 = b ? T1L : T0L;
    d->level0 = 1;
    d->level1 = 0;
}

/* Encode 8 bits MSB first and return the next free symbol. */
static rmt_data_t *putByte(rmt_data_t *d, uint8_t v) {
    for (int8_t i = 7; i >= 0; i--) putBit(d++, (uint8_t)((v >> i) & 1u));
    return d;
}

/* Push the whole strip as GRB, then hold the line low to latch the frame. */
static void show() {
    rmt_data_t *d = sFrame;
    for (uint8_t n = 0; n < LED_COUNT; n++) {
        d = putByte(d, sPix[n][1]);   /* G */
        d = putByte(d, sPix[n][0]);   /* R */
        d = putByte(d, sPix[n][2]);   /* B */
    }
    rmtWrite(PIN_WS2812, sFrame, (size_t)LED_COUNT * 24, 100);
    delayMicroseconds(RESET_US);
}

/* Switch to the next palette color; ignored while a sweep is still running. */
static void nextColor() {
    if (gNext < LED_COUNT) return;
    gColor = (uint8_t)((gColor + 1) % N_COLOR);
    gNext = 0;
    gStepAt = millis();             /* the next loop paints LED0 */
}

/* Paint the next LED once its slot has come up. */
static void sweepStep() {
    if (gNext >= LED_COUNT) return;
    unsigned long now = millis();
    if ((long)(now - gStepAt) < 0) return;          /* wrap-safe "not yet" */
    for (uint8_t k = 0; k < 3; k++) sPix[gNext][k] = kPalette[gColor][k];
    show();
    gNext++;
    gStepAt = now + SWEEP_MS;
}

/* Falling edge = one trigger (raw/stable start as "released", so a button held
 * down at power-up counts as one press and then waits for a new edge). */
static void pollButton() {
    static bool raw = false, stable = false;
    static unsigned long changedAt = 0;
    bool pressed = (digitalRead(PIN_BUTTON) == LOW);
    if (pressed != raw) { raw = pressed; changedAt = millis(); }
    if (raw == stable || millis() - changedAt < DEBOUNCE_MS) return;
    stable = raw;
    if (stable) nextColor();
}

void setup() {
    pinMode(PIN_BUTTON, INPUT_PULLUP);
    pinMode(PIN_WS2812, OUTPUT);
    digitalWrite(PIN_WS2812, LOW);   /* idle low until the first frame */
    rmtInit(PIN_WS2812, RMT_TX_MODE, RMT_MEM_NUM_BLOCKS_4, RMT_TICK_HZ);
    rmtSetEOT(PIN_WS2812, 0);        /* keep the line low between frames */
    show();                          /* sPix is zero-initialized = dark strip */
}

void loop() {
    pollButton();
    sweepStep();
    delay(1);
}
```

</details>

**Step 2. Upload the sketch**

Confirm **Tools > Board** is `XIAO_ESP32S3_PLUS` and **Tools > Port** shows your board, then click **Upload** (→).

**Step 3. Press KEY**

The board starts dark. Every press of **KEY** advances the palette by one step, and the LEDs take the new color in order rather than all at once:

| Press | Color |
| --- | --- |
| 1 | red |
| 2 | green |
| 3 | blue |
| 4 | off again |

![Button-switched RGB board](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Hardware_Usage/Image/experiment3_button_rgb_sweep.gif)

:::tip
A press that lands while a sweep is still running is ignored — wait until the last LED has changed color before pressing again.
:::

## Troubleshooting

### Q1: Roll and pitch are fine but yaw drifts

A: Three different situations all show up as a drifting yaw, and only the first one is a fault.

- **Zeroed while the board was moving.** The 6-second average stored a wrong offset. Hold the board still and press **KEY** again.
- **Still inside the first 18 seconds.** Nothing is compensated yet, so the raw gyro offsets go straight into the angles. Once the header line `roll=deg pitch=deg yaw=deg temp=C` shows up, this is over.
- **Yaw is relative on this board.** There is no magnetometer, so it only knows how far the board turned since the last reset, and a long session will drift. While the board is still, the still-tracker (`STILL_DPS` / `STILL_MS` / `BIAS_ALPHA`) folds the leftover rate back into the bias and holds yaw flat. Start moving and it can no longer help.

## Tech Support & Product Discussion

Thank you for choosing our products! We are here to provide you with different support to ensure that your experience with our products is as smooth as possible. We offer several communication channels to cater to different preferences and needs.

<div class="button_tech_support_container">
<a href="https://forum.seeedstudio.com/" class="button_forum"></a>
<a href="https://www.seeedstudio.com/contacts" class="button_email"></a>
</div>

<div class="button_tech_support_container">
<a href="https://discord.gg/eWkprNDMU7" class="button_discord"></a>
<a href="https://github.com/Seeed-Studio/wiki-documents/discussions/69" class="button_discussion"></a>
</div>
