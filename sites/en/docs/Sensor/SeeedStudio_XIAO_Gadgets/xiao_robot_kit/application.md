---
description: Application examples for the XIAO Robot Kit — flash the BLE command firmware and drive it from a BLE tool, or flash the web remote sketch and drive the car from a browser page.
title: Application
keywords:
  - xiao robot kit
  - esp32s3
  - wi-fi remote control
  - web remote
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

This page runs two complete examples. The first drives the chassis from a BLE tool by writing text commands; the second serves its own web page, so a browser is the remote control.

Both follow the same three steps: flash the firmware, open the tool or the page, then send commands. The projects live in the repository linked under **[TBD-35]**, one folder each.

To change how the car behaves, open that repository and keep vibe coding there — the chassis, arm and gesture code is all reusable. Pins, wiring and power are covered in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/) and [Hardware Usage](https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/).

## Application 1: BLE Control of Pre-written Motion Commands

The firmware turns the kit into a BLE peripheral with a single writable characteristic. A scanner app, a Web Bluetooth page or your own client writes a command there as UTF-8 text, and the result comes back on the same characteristic.

The text is the command the USB serial shell already accepts, so a command can be proved over the cable before any BLE client is involved.

### What You Need

- A **XIAO Robot Kit** and a USB-C cable
- A BLE client: a phone app such as nRF Connect or LightBlue, or any Web Bluetooth page on a laptop

### Flash the Firmware

It is an ESP-IDF project, not an Arduino sketch. ESP-IDF **v6.1** builds it, and `sdkconfig.defaults` already pins the 16 MB flash, the 8 MB PSRAM and the USB Serial/JTAG console, so a fresh clone needs no configuration.

```bash
cd examples/xiao_robot_kit_ble_control
idf.py set-target esp32s3
idf.py -p <serial-port> flash monitor
```

The monitor opens a `robot>` prompt where `help` lists everything the firmware accepts.

### Run It from a BLE Tool

**Scan for `XIAO Robot Kit` and connect.** That is the default name, which keeps tutorial screenshots and scanner lists identical across kits.

**Subscribe to notifications on the command characteristic.** Writes go in, results come back, all on the same handle.

**Write `drive 20 20`.** Both wheels turn, and the characteristic notifies `OK|drive 20 20|$`. Write `stop` to end the run.

| Item | Value |
| --- | --- |
| Service UUID | `a2f39c10-7e4b-4d21-8a61-0f3bc9d15001` |
| Command and status characteristic | `a2f39c11-7e4b-4d21-8a61-0f3bc9d15001` |
| Properties | Read, Write, Write Without Response, Notify |

A write carries 63 bytes at most, so a full-length command needs the client to negotiate an ATT MTU of 66. Anything else comes back on the same characteristic:

```text
OK|<command>|$     the command ran
ERR|UNKNOWN|$      no such command
ERR|EXEC|$         the command failed while running
ERR|LENGTH|$       frame longer than 63 bytes
```

To rename a board, write `btname set XIAO-Robot-Kit-01` in the same characteristic. A name is 1 to 26 characters from `A-Z`, `a-z`, `0-9`, `.`, `_` and `-`, and it is stored in NVS, so it survives a power cycle and a reflash of the application.

### Useful Commands

| Command | What it does |
| --- | --- |
| `drive <left> <right>` | Open-loop wheel speed, `-100` to `100` |
| `speed <left> <right>` | Same range under PID; `25` and below cannot beat floor friction |
| `forward`, `backward`, `left`, `right` | Shorthand, default speed `40` |
| `stop` | Stop both wheels now |
| `move_forward`, `move_backward` | One second of PID travel with IMU heading hold |
| `reverse_park` | 90° fan reverse, guided by the IMU |
| `spin_clockwise`, `spin_counterclockwise` | One full turn, measured on the IMU |
| `greeting` | Wave the right arm while driving forward for 1.5 s |
| `wave_hand`, `close_arms`, `servo <1-2> <0..180>` | Arm presets and a direct angle |
| `rgb <r> <g> <b>`, `rainbow`, `dance_take_me_hand` | Light effects and a 60 s choreography |
| `battery`, `status`, `enc` | Pack voltage, peripheral state, encoder counts |

The firmware takes 45 commands in all, and `docs/BLE_COMMANDS.md` in the repository documents every one of them. Each command works in the `robot>` shell too, which makes the cable the fastest place to debug.

### Safety

- Lift the wheels off the desk for the first test, and keep the power switch within reach.
- There is no idle timeout: `drive`, `speed` and the presets keep running until `stop` or another drive command arrives.
- A BLE disconnect or a NimBLE reset stops every motor and cancels any running preset.

## Application 2: Wi-Fi Remote-Controlled Car with Battery Monitoring

The sketch hosts its own HTTP and WebSocket server, so the page it serves is the entire remote control: press the keys, watch the battery, read the firmware log. No app and no third-party library are involved.

### Required Parts

- A **XIAO Robot Kit** and a charged battery on the **battery connector** — the motors draw more than a USB port can supply
- A phone or laptop on the same Wi-Fi network as the board
- Arduino IDE or `arduino-cli` with the **esp32 core 3.x**

### Set the Wi-Fi Credentials

Wi-Fi lives in one place, `WIFI_SSID` and `WIFI_PASS` in `src/app/car_config.h`. Editing the file is the simplest route; to keep credentials out of the source tree, override them on the build line with `-DWIFI_SSID=...` and `-DWIFI_PASS=...` instead.

### Flash the Sketch

```bash
arduino-cli compile --fqbn esp32:esp32:XIAO_ESP32S3_Plus examples/xiao_robot_kit_web_remote
arduino-cli upload  --fqbn esp32:esp32:XIAO_ESP32S3_Plus -p <serial-port> examples/xiao_robot_kit_web_remote
```

`sketch.yaml` in that folder pins the same FQBN, so `arduino-cli` also works from inside it. In the Arduino IDE, select the **XIAO ESP32S3 Plus** board and open the `.ino`; it picks up `src/` by itself.

### Open the Page

Open the serial monitor at 115200 baud and wait for the IP address. The page answers on `http://xiaorobot.local` or on that IP, whichever the network resolves.

Keep the car still for about a second after power-on: the gyro takes its zero-rate calibration in that window, and the page reports an uncalibrated car until it finishes. **CALIBRATE** repeats the same routine whenever the car has been sitting somewhere new.

### Driving from the Page

The page is in English and needs no setup beyond the URL.

- Hold a direction key to drive and release it to coast to a stop; a second key can be held at the same time to drive forward while turning.
- The center key is the emergency stop, and the space bar and Esc do the same thing.
- **SET FORWARD** takes the current heading as the new forward, and **HOLD** keeps repeating the last direction without a finger on the screen.
- Telemetry arrives five times a second, and a log window mirrors everything the firmware prints, so the car can be watched with no USB cable attached.
- If the link dies, the car stops within 1.5 s on its own.

### The Battery Reading

The battery connector feeds D16 through a 100 kΩ / 30 kΩ divider. On USB-C the pack sits at 8.3–8.4 V rather than 5 V, because the Type-C input goes through the charger and boost first — a reading near 8.5 V is correct, not a calibration error.

The page shows the raw ADC count next to the converted volts, and the sketch treats 8.40 V as full and 6.00 V as empty.

### If the Page Does Not Open

Three checks, in this order. The serial log has to print the IP address first. Pressing `i` in the monitor then prints the Wi-Fi state, the request counters and a one-line conclusion. And a request that never reaches the board points at a wrong IP, a phone on another network, or a router with AP isolation switched on.

### Serial Keys Without a Browser

| Key | Action |
| --- | --- |
| `c` | IMU zero-rate calibration, the same routine as CALIBRATE |
| `t` | Bench self test: both wheels forward, backward and a pivot, with pass or fail |
| `l`, `r`, `s` | Pivot left, pivot right, stop |
| `h` | Toggle IMU heading hold |
| `y` | Print yaw, calibration state and standing drift |
| `i` | Print Wi-Fi state, request counters and the network conclusion |

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