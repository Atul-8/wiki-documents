---
description: Application examples for the XIAO Robot Kit — flash the web remote sketch and drive the car from a browser page, or flash the BLE firmware and drive it from any BLE tool.
title: Application
keywords:
  - xiao robot kit
  - esp32s3
  - wi-fi remote control
  - web remote
  - battery monitoring
  - ble gatt
  - esp-idf
  - arduino sketch
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

Two examples, easiest first — drive the car from a browser page, then from a BLE tool. Both are the same three moves: flash it, connect to it, drive it.

- **[Application 1: Drive from a Browser](#app-1-browser)** — the board serves its own page over Wi-Fi. Nothing to install.
- **[Application 2: Send BLE Commands](#app-2-ble)** — the board becomes a BLE peripheral, and any BLE client can write commands to it.

Both projects live in the [example repository](https://github.com/Atul-8/XIAO_Robot_Kit_Vibe), one folder each. Open it when you want to change how the car moves. Pins, wiring and power are covered in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/) and [Hardware Usage](https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/).

## Application 1: Drive from a Browser over Wi-Fi {#app-1-browser}

Flash one sketch and the kit becomes a web server: the page it serves is the entire remote control, so there is no app to install and no library to add.

**[TBD-39]** Photo: phone held in portrait with the page open, the car in the same frame.

### Required Parts

- A **XIAO Robot Kit** with a charged battery on the **battery connector** — the motors pull more than a USB port can give
- A phone or laptop on the same Wi-Fi network as the board
- Arduino IDE or `arduino-cli` with the **esp32 core 3.x**

### Flash the Sketch

1. Open `src/app/car_config.h` and fill in `WIFI_SSID` and `WIFI_PASS`. That file is the only place the credentials live.
2. Build and upload:

```bash
arduino-cli compile --fqbn esp32:esp32:XIAO_ESP32S3_Plus examples/xiao_robot_kit_web_remote
arduino-cli upload  --fqbn esp32:esp32:XIAO_ESP32S3_Plus -p <serial-port> examples/xiao_robot_kit_web_remote
```

In the Arduino IDE, select the **XIAO ESP32S3 Plus** board, open the `.ino` and click upload.

:::tip
The board joins **2.4 GHz** networks only. If your router publishes one name for both bands, give the 2.4 GHz band its own SSID and use that one.
:::

### Open the Page

Open the serial monitor at 115200 baud. Two lines tell you where the page is:

```text
[wifi] connected, IP 192.168.1.42
open http://xiaorobot.local or http://192.168.1.42
```

**[TBD-40]** Screenshot: those two serial lines right after a successful boot.

Type `http://xiaorobot.local` into the browser, or the IP address if your network does not resolve `.local` names.

### Drive It

1. Leave the car still for about a second after power-on. The gyro is taking its zero-rate calibration, and the page keeps saying so until it finishes. Press **CALIBRATE** whenever the car has been moved.
2. Hold a direction key to drive, release it to coast to a stop.
3. Hold a second key at the same time to drive forward while turning.
4. Press the center key to stop right away — the space bar and Esc do the same.
5. **SET FORWARD** makes the current heading the new forward; **HOLD** repeats the last direction while no key is pressed.

**[TBD-41]** GIF: hold forward, the car drives, release, the car coasts to a stop.

### What You Should See

- Telemetry arrives five times a second, and a log window mirrors everything the firmware prints, so the car can be watched with no USB cable attached.
- The battery row shows the raw ADC count next to the volts, for example `ADC 2312 · 8.47 V`.
- Kill the Wi-Fi link and the car stops within 1.5 s on its own.

**[TBD-42]** Close-up: the battery row on the page, with the battery cable in frame.

The battery connector feeds D16 through a 100 kΩ / 30 kΩ divider. With USB-C power and no battery fitted, that divider sits at 8.3–8.4 V: the 5 V from the connector passes through the charger and the boost converter first. A reading near 8.5 V is therefore correct, not a calibration error. The sketch treats 8.40 V as full and 6.00 V as empty.

### If the Page Does Not Open

Three checks, in order:

1. **The serial log has to show an IP.** Without one the sketch never joined the network — check the SSID, the password and the 2.4 GHz band.
2. **Press `i` in the monitor.** It prints the Wi-Fi state, the request counters and a one-line verdict.
3. **Still unreachable?** A request that never reaches the board usually means a wrong IP, a phone on a different network, or a router with AP isolation enabled.

### No Browser? Use the Serial Keys

| Key | Action |
| --- | --- |
| `c` | IMU zero-rate calibration, the same routine as CALIBRATE |
| `t` | Bench self test: both wheels forward, backward and a pivot, with pass or fail |
| `l`, `r`, `s` | Pivot left, pivot right, stop |
| `h` | Toggle IMU heading hold |
| `y` | Print yaw, calibration state and standing drift |
| `i` | Print Wi-Fi state, request counters and the network conclusion |

## Application 2: Send BLE Commands from a Tool {#app-2-ble}

Flash one ESP-IDF project and the kit advertises as a BLE peripheral with a single writable characteristic. Whatever your client writes is treated as a command, and it is the same command the USB serial shell accepts.

**[TBD-43]** Screenshot: a BLE tool writing `drive 20 20` and the `OK` reply coming back.

### Required Parts

- A **XIAO Robot Kit** and a USB-C cable
- A BLE client: a phone app such as nRF Connect or LightBlue, or any Web Bluetooth page on a laptop

### Flash the Firmware

It is an ESP-IDF project, not an Arduino sketch, and ESP-IDF **v6.1** builds it. `sdkconfig.defaults` already pins the 16 MB flash, the 8 MB PSRAM and the USB Serial/JTAG console.

```bash
cd examples/xiao_robot_kit_ble_control
idf.py set-target esp32s3
idf.py -p <serial-port> flash monitor
```

The monitor drops you into a `robot>` prompt. Try `help`, then `drive 20 20` and `stop` there first: a command that works over the cable separates a firmware problem from a radio problem.

### Connect from Your Tool

1. Scan for `XIAO Robot Kit`. That is the default name, so every kit shows up under the same name until it is renamed.
2. Connect, then open the service below.
3. Subscribe to notifications on its characteristic, so the replies have somewhere to arrive.

**[TBD-44]** Screenshot: the scanner list with the default name, and the characteristic expanded.

| Item | Value |
| --- | --- |
| Service | `a2f39c10-7e4b-4d21-8a61-0f3bc9d15001` |
| Characteristic | `a2f39c11-7e4b-4d21-8a61-0f3bc9d15001` |
| Properties | Read, Write, Write Without Response, Notify |

### Write Your First Command

Write `drive 20 20` as UTF-8 text. Both wheels turn, and the characteristic notifies `OK|drive 20 20|$`. Write `stop` to end the run.

A write carries 63 bytes at most, so a full-length command needs the client to negotiate an ATT MTU of 66. The other replies are:

```text
OK|<command>|$     the command ran
ERR|UNKNOWN|$      no such command
ERR|EXEC|$         the command failed while running
ERR|LENGTH|$       frame longer than 63 bytes
```

Rename the board by writing `btname set XIAO-Robot-Kit-01` in the same characteristic. A name is 1 to 26 characters from `A-Z`, `a-z`, `0-9`, `.`, `_` and `-`, and it is stored in NVS, so a power cycle or a reflash of the application keeps it.

### Commands Worth Trying

| Command | What it does |
| --- | --- |
| `drive <left> <right>` | Open-loop wheel speed, `-100` to `100` |
| `speed <left> <right>` | The same range under PID; `25` and below cannot beat floor friction |
| `forward`, `backward`, `left`, `right` | Shorthand, default speed `40` |
| `stop` | Stop both wheels now |
| `move_forward`, `reverse_park` | IMU-guided: one second straight, or a 90° fan reverse |
| `spin_clockwise`, `spin_counterclockwise` | One full turn, measured on the IMU |
| `wave_hand`, `close_arms`, `servo <1-2> <0..180>` | Arm presets and a direct angle |
| `rgb <r> <g> <b>`, `rainbow` | Light effects |
| `battery`, `status` | Pack voltage and peripheral state |

The firmware accepts 45 commands in total, and `docs/BLE_COMMANDS.md` in the repository documents every one of them. Each also works in the `robot>` shell, which is the fastest place to debug.

### Safety

- Lift the wheels off the desk for the first test, and keep the power switch within reach.
- There is no idle timeout: `drive`, `speed` and the presets keep running until `stop` or another drive command arrives.
- A BLE disconnect or a NimBLE reset stops every motor and cancels any running preset.

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