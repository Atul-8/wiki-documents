---
description: Type a sentence, let an AI edit the firmware, press upload and the car does something new. Two tasks, easiest first — the Wi-Fi password, then a page button that prints hello world.
title: AI Agent Quickstart
keywords:
  - xiao robot kit
  - ai agent
  - vibe coding
  - claude code
  - cursor
  - beginner
  - esp32s3
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_ai_agent_quickstart
sku: 100058302
sidebar_position: 4
last_update:
  date: 11/15/2026
  author: 划水佬
createdAt: '2026-11-13'
updatedAt: '2026-11-15'
url: https://wiki.seeedstudio.com/xiao_robot_kit_ai_agent_quickstart/
---

# AI Agent Quickstart

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:800, height:'auto'}}/></div>

<div class="get_one_now_container" style={{textAlign: 'center'}}>
    <a class="get_one_now_item" href="https://www.seeedstudio.com/XIAO-Robot-Kit-p-100058302.html?utm_source=wiki" target="_blank">
            <strong><span><font color={'FFFFFF'} size={"4"}> Get One Now 🖱️</font></span></strong>
    </a>
</div><br />

You type a sentence. An AI edits the firmware. You upload it and the car does something new.

Two tasks below, easiest first: change the Wi-Fi password, then add a button that prints hello world.

## What You Need

- A **XIAO Robot Kit** with a charged battery on the **battery connector** — the motors pull more than a USB port can give
- Arduino IDE with the **esp32 core 3.x** installed, as described in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/)
- The example code: [XIAO-Robot-Kit](https://github.com/Seeed-Projects/XIAO-Robot-Kit) — `git clone https://github.com/Seeed-Projects/XIAO-Robot-Kit.git`, or **Code → Download ZIP** on that page if you have no Git
- One AI coding tool, installed and signed in with your own account — **Claude Code**, a terminal agent you start inside the project folder, or **Cursor**, an editor with a chat panel next to the files

Install whichever one you pick from the vendor's own page — everything below starts once it is running. Open the **repository folder**, not your home directory, and say `test`: an answer means it is running.

## 1. Change the Wi-Fi Password {#first-task}

### The Prompt

Plug the battery in, connect the USB cable, and paste this:

```text
This folder holds two example projects: examples/xiao_robot_kit_web_remote (Arduino,
a Wi-Fi remote control) and examples/xiao_robot_kit_ble_control (ESP-IDF, BLE commands).
Read the Wi-Fi remote one and change its Wi-Fi to my home network:
SSID MyHome-2.4G, password 12345678.
Tell me which file and which lines you changed.
```

It reads the project, edits the two values and reports which lines changed.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/AI_Agent_Quickstart/Image/ai_agent_wifi_change.jpg" style={{width:700, height:'auto'}}/></div>

### Upload It

1. Open `examples/xiao_robot_kit_web_remote/xiao_robot_kit_web_remote.ino` in the Arduino IDE.
2. Set **Tools > Board** to `XIAO_ESP32S3_PLUS`.
3. Set **Tools > Port** to the port that appeared when you plugged the board in.
4. Click **Upload** (→).

Ask the tool to run the `arduino-cli` compile and upload for you instead, and it will ask permission for each command. Read the port in that command before you approve: `arduino-cli board list` prints the port that is really connected, and a guessed one fails with no output at all.

### Check the Log

Open the serial monitor at 115200 baud. The address below belongs to one computer on one network: your router hands out its own, so yours will be different, and it changes again on a different network.

```text
[wifi] connected, IP 192.168.1.42
open http://xiaorobot.local or http://192.168.1.42
```

Read the address from your own log and open it in the browser, then hold a direction key on the page. `http://xiaorobot.local` is shorter to type and survives a new IP — try it first, and fall back to the IP in your log if the name does not resolve.

:::tip
The board joins **2.4 GHz** networks only. If the password is right and it still will not connect, your router is probably publishing one name for both bands — give the 2.4 GHz band its own SSID.
:::

## 2. Add a Button {#extend}

### The Second Prompt

```text
In examples/xiao_robot_kit_web_remote, add a button to the web page. Pressing it prints
hello world in the serial log. Leave the motors and the driving page alone, and tell me
which files you touched.
```

### What You Should See

Upload again, open the page and press the new button: the serial monitor prints `hello world`. If it compiles and nothing appears, paste the serial output back into the chat.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/AI_Agent_Quickstart/Image/ai_agent_hello_button.jpg" style={{width:320, height:'auto'}}/></div>

:::note
What you get may look different — the button lands wherever your agent decided to put it, so treat this as one possible result. What has to be true is the message: press the button, `hello world` shows up in the serial monitor.
:::

## FAQs

### Q1: It reports the edit, but nothing changed

A: Arduino caches builds. Ask for `compile --clean`, then upload again.

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