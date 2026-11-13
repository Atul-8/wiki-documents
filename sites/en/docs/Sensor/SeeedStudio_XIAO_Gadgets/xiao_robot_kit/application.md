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

This page introduces two practical examples. The first drives the chassis by writing pre-written motion commands over BLE; the second drives the car live from a browser over Wi-Fi. Both projects live in the repository linked under **[TBD-35]**, one folder each.

These pages cover the basic use only. To build something of your own, the faster route is to open the repository and keep vibe coding there — the chassis, arm and gesture code is all reusable.

Pins, wiring and power are covered in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/) and [Hardware Usage](https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/).

## Application 1: BLE Control of Pre-written Motion Commands

The firmware turns the kit into a BLE peripheral. Any BLE client — a phone app, a laptop or a test tool — writes a text command to one GATT characteristic and the chassis moves.

The project lives in `examples/xiao_robot_kit_with_honor_robot_phone` in **[TBD-35]**.

### Build and Flash

It is an ESP-IDF project, not an Arduino sketch. Use ESP-IDF **v6.0.2** — the only version the repo has verified on hardware.

```bash
cd examples/xiao_robot_kit_with_honor_robot_phone
idf.py set-target esp32s3
idf.py -p <serial-port> flash monitor
```

The monitor lands on a `robot>` prompt, so every command below can be tried over USB first.

### Connect over BLE

The board advertises as `XIAO-Robot-Kit-` followed by the last two bytes of its BLE address, for example `XIAO-Robot-Kit-3C7A`. Give a board a permanent name with `btname set XIAO-Robot-Kit-01`; the name lives in NVS and survives reflashing the application.

| Item | Value |
| --- | --- |
| Service UUID | `a2f39c10-7e4b-4d21-8a61-0f3bc9d15001` |
| Command and status characteristic | `a2f39c11-7e4b-4d21-8a61-0f3bc9d15001` |
| Properties | Read, Write, Write Without Response, Notify |

### Command API

Write UTF-8 text to that characteristic, 63 bytes at most — a full-length command needs the phone to negotiate an ATT MTU of 66. Each write is answered on the same characteristic:

```text
OK|<command>|$     the command ran
ERR|UNKNOWN|$      no such command
ERR|EXEC|$         the command failed while running
ERR|LENGTH|$       frame longer than 63 bytes
```

#### Drive

- `drive <left> <right>` — open-loop wheel speed, `-100` to `100`.
- `speed <left> <right>` — same range, but closed-loop PID. A setting at or below `25` cannot overcome floor friction.
- `forward [speed]`, `backward [speed]`, `left [speed]`, `right [speed]` — shorthand, default speed `40`.
- `stop` — stop both wheels immediately.
- `motor <1-2> <-100..100>` and `motorinv <1-2> <0|1>` — one wheel at a time, and its polarity.

The motion frame the phone app already sends still works: `M|20|20|$` is translated to `drive 20 20`.

#### Preset chassis actions

- `move_forward`, `move_backward` — 1 s with IMU heading hold; both need a valid still calibration.
- `turn_left`, `turn_right`
- `greeting` — `wave_hand` and 1.5 s of forward travel at the same time.
- `reverse_park` — IMU-guided 90° fan reverse into a parking spot.
- `spin_clockwise`, `spin_counterclockwise` — one full turn, measured on the IMU.
- `emotion_happy`, `emotion_scared`, `emotion_angry` — arms and wheels together.

#### Arms

- `wave_hand`, `handshake`, `open_arms`, `raise_both_arms`, `close_arms`, `armstop`
- `servo <1-2> <0..180>` and `servooff <1-2>` — a direct angle for calibration, and releasing the hold.

#### Light and dance

- `rgb <red> <green> <blue>`, `rainbow`, `rgboff`
- `dance_take_me_hand`, `dance_believer`, `dance_new_believer`

#### Everything else

- `status`, `battery`, `enc`, `encreset`, `cps`
- `imu calibrate`, `tune speed`, `tune save`, `tune show`, `tune reset`
- `cpslog start`, `cpslog stop`, `cpslog list`, `cpslog summary`, `cpslog dump`
- `btname set`, `btname show`, `btname reset`

#### Example: wave the hand

```text
wave_hand        →  OK|wave_hand|$
```

The right arm waves four times, then returns to 0°.

**[TBD-38]**

### Safety

- Lift the wheels off the desk for the first test, and keep the power switch within reach.
- There is no idle timeout: `drive`, `speed` and `motor` keep running until `stop` or another drive command.
- A BLE disconnect or a NimBLE reset stops every motor.

## Application 2: Wi-Fi Remote-Controlled Car with Battery Monitoring

The car takes commands from a phone browser over Wi-Fi and shows the battery voltage on the same page, so you can watch the pack sag while the motor is under load.

### How it works

**[TBD-37]**

### Battery monitoring

**[TBD-36]**

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
