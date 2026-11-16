---
description: Application examples for the XIAO Robot Kit — flash the web remote project and drive the car from a browser page, or flash the BLE firmware and drive it from any BLE tool.
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

Upload one project and the kit becomes a web server: the page it serves is the entire remote control, so there is no app to install and no library to add.

### Required Parts

- A **XIAO Robot Kit** with a charged battery on the **battery connector** — the motors pull more than a USB port can give
- A phone or laptop on the same Wi-Fi network as the board
- Arduino IDE with the **esp32 core 3.x** installed, as described in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/)

### Set Your Wi-Fi

Wi-Fi lives in one place: `WIFI_SSID` and `WIFI_PASS`, the two lines at the top of `examples/xiao_robot_kit_web_remote/src/app/car_config.h`, both set to `HelloWorld`. Replace them with your own network name and password — the board joins only after both match your router, and a reflash onto a different network also changes the address the board prints at boot.

:::tip
The board joins **2.4 GHz** networks only. If your router publishes one name for both bands, give the 2.4 GHz band its own SSID and use that one.
:::

### Upload the Project

- Open `examples/xiao_robot_kit_web_remote/xiao_robot_kit_web_remote.ino` in the Arduino IDE.
- Set **Tools > Board** to `XIAO_ESP32S3_PLUS`.
- Set **Tools > Port** to the port that appeared when you plugged the board in.
- Click **Upload** (→).

### Open the Page

Open the serial monitor at 115200 baud. Once the board has joined your network it prints two lines: the IP it was given, and the address of the page. Your router hands out its own, so the address in the screenshot below will not be the one you get.

Copy the address from your own log and paste it into the browser. That one always works.

Try `http://xiaorobot.local` first: it is shorter to type, and it survives a new IP because the firmware advertises the name itself over mDNS. Some networks block mDNS and the name will not resolve there — the IP from your log is then the way in.

![The two lines the board prints after the upload](https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app1_serial_log.jpg)

### Drive It

1. Switch the car on and leave it still for 3–5 s: the gyro takes an uninterrupted zero-rate reading, and a disturbed attempt is retried once.
2. Hold a direction key to drive, and release it to stop.
3. Press **CALIBRATE** on the page whenever the car has been picked up or bumped, and keep it still until the *Calibrating* note clears — that is what re-zeros the gyro.
4. Press the center key to stop right away — the space bar and Esc do the same.
5. **SET FORWARD** makes the current heading the new forward; **HOLD** repeats the last direction while no key is pressed.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app1_phone_driving.gif" style={{width:320, height:'auto'}}/></div>

### What You Should See

- Telemetry arrives five times a second, and a log window mirrors everything the firmware prints, so the car can be watched with no USB cable attached.
- The battery row shows the raw ADC count next to the volts, for example `ADC 2312 · 8.47 V`.
- Kill the Wi-Fi link and the car stops within 1.5 s on its own.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app1_page_on_phone.jpg" style={{width:320, height:'auto'}}/></div>

:::tip
The battery sense sits on the **battery connector**, not on the 5 V input. The BQ25886 PMIC charges and boosts the pack, so on USB-C alone that connector already reads 8.3–8.4 V — 8.5 V is normal, not a calibration error. The firmware calls 8.40 V full and 6.00 V empty.
:::

## Application 2: Send BLE Commands from a Tool {#app-2-ble}

This one is an ESP-IDF project rather than an Arduino sketch. It makes the kit a BLE peripheral with a single writable characteristic, and whatever you write there runs as the same command the `robot>` shell accepts — the phone is a remote control for the firmware you already have.

### Required Parts

- A **XIAO Robot Kit** and a USB-C cable
- **VS Code** with the **ESP-IDF extension** and ESP-IDF **v6.1**
- A BLE scanner app on your phone, such as nRF Connect

### Flash the Firmware

- Right-click the folder `examples/xiao_robot_kit_ble_control` and choose **Open with Code** — the extension takes over the project from there.
- If it asks for a target, choose `esp32s3`; if it asks for a port, take the one that appeared when you plugged the board in.
- Press **Build** in the ESP-IDF status bar, then **Flash**.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app2_idf_build_bar.jpg" style={{width:600, height:'auto'}}/></div>

Nothing else needs configuring: the 16 MB flash, the 8 MB PSRAM and the USB Serial/JTAG console are already pinned in `sdkconfig.defaults`.

Open the monitor when the flash finishes — it drops into a `robot>` prompt. Type `drive 20 20`, then `stop`. If the car drives over the cable, the firmware is fine and whatever is left to fix is in the radio.

### Connect and Write

1. Scan for **XIAO Robot Kit** and connect — the screenshots below use **nRF Connect**, and [Experiment 2 in Hardware Usage](https://wiki.seeedstudio.com/xiao_robot_kit_hardware_usage/#experiment-2-ble-control-of-the-encoder-motor) covers that app in more detail.
2. Open the service `a2f39c10-7e4b-4d21-8a61-0f3bc9d15001`, pick the single characteristic `a2f39c11-7e4b-4d21-8a61-0f3bc9d15001` under it, and subscribe to notifications.
3. Keep the write format at **UTF-8** and write `drive 20 20`.

<table align="center">
   <tr>
       <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app2_ble_scan.jpg" style={{width:220, height:'auto'}}/></div></td>
       <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app2_ble_write_format.jpg" style={{width:200, height:'auto'}}/></div></td>
  </tr>
</table>

Both wheels turn and the characteristic notifies back `OK|drive 20 20|$`. Write `stop` to end the run.

<table align="center">
   <tr>
       <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app2_ble_write.jpg" style={{width:260, height:'auto'}}/></div></td>
       <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Application/Image/app2_ble_reply.jpg" style={{width:260, height:'auto'}}/></div></td>
  </tr>
</table>

A command the shell does not know answers `ERR|UNKNOWN|$`, one that failed while running answers `ERR|EXEC|$`, and a write over 63 bytes answers `ERR|LENGTH|$` — 63 bytes is all a single write carries, so a full-length command needs an ATT MTU of 66.

The board leaves the factory called **XIAO Robot Kit**. Rename it by writing `btname set XIAO-Robot-Kit-01`: 1 to 26 characters from `A-Z`, `a-z`, `0-9`, `.`, `_` and `-`, kept in NVS, so it survives power cycles and reflashes.

### Commands Worth Trying

| Command | What it does |
| --- | --- |
| `drive <left> <right>` | Open-loop PWM, `-100` to `100`, running until `stop` |
| `speed <left> <right>` | The same range on the encoder PID: `100` asks for 5766 counts/s |
| `stop` | Cancels whatever is running and switches both wheels off |
| `move_forward`, `spin_clockwise` | IMU-measured: one second straight, or a full 360° turn |
| `rgb <r> <g> <b>` | The 10-pixel light strip |

The firmware accepts 45 commands in total, and `docs/BLE_COMMANDS.md` in the repository documents every one of them. Each also works in the `robot>` shell, which is the fastest place to debug.

### Safety

- Lift the wheels off the desk for the first test, and keep the power switch within reach.
- There is no idle timeout: `drive`, `speed` and the presets keep running until `stop` or another drive command arrives.
- A BLE disconnect or a NimBLE reset stops every motor and cancels any running preset.

## FAQs

### Q1: The page does not open in the browser

A: Read the serial log first. Without an IP line the board never joined the network — check the SSID, the password and the 2.4 GHz band. If the log does show an IP and the page still will not load, the request is not reaching the board: a wrong address, a phone on a different network, or a router with AP isolation turned on.

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