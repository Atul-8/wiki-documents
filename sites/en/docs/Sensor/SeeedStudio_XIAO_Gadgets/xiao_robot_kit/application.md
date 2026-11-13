---
description: Application examples for the XIAO Robot Kit — BLE control of pre-written motion commands, and a Wi-Fi remote-controlled car with battery monitoring.
title: Application
keywords:
  - xiao robot kit
  - esp32s3
  - wi-fi remote control
  - dc motor car
  - battery monitoring
  - esp-idf
  - ble gatt
  - robot application
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_application
sku: 100058302
sidebar_position: 3
last_update:
  date: 11/13/2026
  author: 划水佬
createdAt: '2026-11-13'
updatedAt: '2026-11-13'
url: https://wiki.seeedstudio.com/xiao_robot_kit_application/
---

# Application

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:800, height:'auto'}}/></div>

<div class="get_one_now_container" style={{textAlign: 'center'}}>
    <a class="get_one_now_item" href="https://www.seeedstudio.com/XIAO-Robot-Kit-p-100058302.html?utm_source=wiki" target="_blank">
            <strong><span><font color={'FFFFFF'} size={"4"}> Get One Now 🖱️</font></span></strong>
    </a>
</div><br />

## Overview

This page introduces two practical examples. The first turns the kit into a BLE peripheral that runs pre-written motion commands written as plain text; the second drives the car live from a browser over Wi-Fi. Both projects live in the repository linked under **[TBD-35]**, one folder each.

These pages cover the basic use only. To build something of your own, the faster route is to open the repository and keep vibe coding there — the chassis, arm and gesture code is all reusable.

Pins, wiring and power are covered in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/) and [Hardware Usage](https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/).

## Application 1: BLE Control of Pre-written Motion Commands

The firmware exposes a single GATT characteristic. Any BLE client — a phone app, a laptop or a test tool — writes a command there as UTF-8 text and reads the result back from the same characteristic.

The text is exactly the command the USB serial shell accepts, so a command can be debugged over the cable before a BLE client is involved.

The project lives in `examples/xiao_robot_kit_ble_control` in **[TBD-35]**.

### Build and Flash

It is an ESP-IDF project, not an Arduino sketch. Use ESP-IDF **v6.0.2** — the only version the repository has verified on hardware.

```bash
cd examples/xiao_robot_kit_ble_control
idf.py set-target esp32s3
idf.py -p <serial-port> flash monitor
```

The monitor lands on a `robot>` prompt, and `help` lists the whole command set.

### Connect over BLE

A new board advertises under the default name `XIAO Robot Kit`, which keeps tutorial screenshots and scanner lists consistent. Rename it once and the name sticks:

```text
btname set XIAO-Robot-Kit-01
```

A name is 1 to 26 characters from `A-Z`, `a-z`, `0-9`, `.`, `_` and `-`; the 26-character cap keeps it inside a single advertising packet. The name is stored in NVS, so it survives a power cycle and a reflash of the application — only `btname reset` or `idf.py erase-flash` restores the default. Renaming restarts advertising immediately, and since BLE shares the command table, a connected client can rename the board too.

| Item | Value |
| --- | --- |
| Service UUID | `a2f39c10-7e4b-4d21-8a61-0f3bc9d15001` |
| Command and status characteristic | `a2f39c11-7e4b-4d21-8a61-0f3bc9d15001` |
| Properties | Read, Write, Write Without Response, Notify |

### Command API

Write UTF-8 text to that characteristic, 63 bytes at most — a full-length command needs the client to negotiate an ATT MTU of 66. Each write is answered on the same characteristic:

```text
OK|<command>|$     the command ran
ERR|UNKNOWN|$      no such command
ERR|EXEC|$         the command failed while running
ERR|LENGTH|$       frame longer than 63 bytes
```

#### Drive

- `drive <left> <right>` — open-loop wheel speed, `-100` to `100`.
- `speed <left> <right>` — same range, closed-loop PID. Floor friction makes anything at or below `25` useless: `30` maps to 1730 CPS, `60` to 3460, `100` to 5766.
- `forward [speed]`, `backward [speed]`, `left [speed]`, `right [speed]` — shorthand, default speed `40`.
- `stop`, `motor <1-2> <-100..100>` and `motorinv <1-2> <0|1>` — both wheels at once, one wheel, and its polarity.

#### Preset chassis actions

- `move_forward`, `move_backward` — 1 s at 55 with IMU heading hold; both refuse to start without a still calibration.
- `turn_left`, `turn_right` — 1.5 s, one wheel driven at 70.
- `reverse_park` — a 90° fan reverse, left wheel slower than the right, until the IMU reaches 90°.
- `spin_clockwise`, `spin_counterclockwise` — one full turn each.
- `greeting` — waves the right arm while driving forward for 1.5 s.
- `emotion_happy`, `emotion_scared`, `emotion_angry` — arms and wheels together; `emotion_happy` spins, so it needs the calibration as well.

#### Arms

- `wave_hand`, `handshake`, `open_arms`, `raise_both_arms`, `close_arms` — one preset at a time.
- `servo <1-2> <0..180>` — 1 is the right arm on `SERVO1`, 2 the left on `SERVO2`.
- `servooff <1-2>` — stop the pulses, so the arm no longer holds its pose.
- `armstop` — cancel the running preset and keep the current pose.

The end effectors were installed around the 1500 µs pulse, so `90` stays the mechanical middle: `0..90` maps to 620–1500 µs and `90..180` to 1500–2500 µs. At power-on both servos go straight to the closed pose — right 0°, left 180° — and the wheels stay still.

#### Light and dance

- `rgb <red> <green> <blue>`, `rainbow`, `rgboff`
- `dance_take_me_hand` — 128 BPM, about 60.7 s.
- `dance_believer` — 129.2 BPM, about 31.4 s plus an IMU finale.
- `dance_new_believer` — 128.7 BPM, about 35.9 s.

Each dance ends parked, with the right arm back at 0° and the left at 180°.

#### Everything else

- `status` — firmware version and the state of every peripheral.
- `battery` — the pack voltage, read on D16 from the battery connector through a 100 kΩ / 30 kΩ divider.
- `enc`, `encreset`, `cps` — encoder counts, and a measured count per second.
- `cpslog <start|stop|list|summary|dump ID|delete ID|clear|status>` — 1200 samples at 40 Hz (30 s) held in RAM, then saved to a 1 MiB SPIFFS partition. `dump` streams CSV over USB only, since a characteristic cannot carry it.
- `imu <status|sample|reg|reinit|calibrate>`, `imulog`, `encdbg`
- `tune <speed|save|arm|show|reset>` — `tune save reverse_park` keeps a reverse speed in NVS.
- `btname <set|show|reset>`

#### Example: wave the hand

```text
wave_hand        →  OK|wave_hand|$
```

The right arm swings between 145° and 125° four times, then returns to 0°.

**[TBD-38]**

### Safety

- Lift the wheels off the desk for the first test, and keep the power switch within reach.
- There is no idle timeout: `drive`, `speed`, `motor` and the presets keep running until `stop` or another drive command arrives.
- A BLE disconnect or a NimBLE reset stops every motor and cancels any running preset.

## Application 2: Wi-Fi Remote-Controlled Car with Battery Monitoring

The car takes commands from a phone browser over Wi-Fi and shows the battery voltage on the same page, so you can watch the pack sag while the motor is under load.

### How it works

**[TBD-37]**

### Battery monitoring

The kit measures its own pack: the battery connector feeds D16 through a 100 kΩ / 30 kΩ divider, so `battery` reports the voltage the firmware actually sees, in volts and millivolts. Read it while the motors are driving and the sag under load shows up directly.

### Required parts

- A **XIAO Robot Kit**
- A charged lithium battery on the **battery connector**
- A phone or laptop with a browser, on the same Wi-Fi network

### Sketch

**[TBD-35]**

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