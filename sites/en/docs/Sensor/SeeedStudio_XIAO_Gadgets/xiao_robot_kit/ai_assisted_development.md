---
description: Type a sentence and let a coding agent edit, flash and test the firmware. Two tasks, easiest first — the Wi-Fi password, then a page button that prints hello world.
title: AI-Assisted Development
keywords:
  - xiao robot kit
  - coding agent
  - ai agent
  - vibe coding
  - claude code
  - codex
  - cursor
  - beginner
  - esp32s3
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_ai_assisted_development
sku: 100058302
sidebar_position: 4
last_update:
  date: 11/17/2026
  author: Atul
createdAt: '2026-11-13'
updatedAt: '2026-11-17'
url: https://wiki.seeedstudio.com/xiao_robot_kit_ai_assisted_development/
---

# AI-Assisted Development

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
    <td><div class="get_one_now_container" style={{textAlign:'center'}}><a class="get_one_now_item" href="https://wiki.seeedstudio.com/xiao_robot_kit_ai_assisted_development/" target="_blank"><strong><span><font color={'FFFFFF'} size={'4'}> Get One Now 🖱️</font></span></strong></a></div></td>
    <td><div class="get_one_now_container" style={{textAlign:'center'}}><a class="get_one_now_item" href="https://wiki.seeedstudio.com/xiao_robot_kit_ai_assisted_development/" target="_blank"><strong><span><font color={'FFFFFF'} size={'4'}> Get One Now 🖱️</font></span></strong></a></div></td>
  </tr>
</table>
</div>

:::info How this page works

**You type a sentence. A coding agent edits the firmware, flashes the board and reads the serial log back to you.**

:::

An *agent* here is a tool that acts on its own: Claude Code, Codex, Cursor or any other AI coding agent. It reads the project folder, runs the compiler and the uploader, and keeps the serial port open. You decide what should change; it types it, flashes it and tells you what came back.

Two tasks below, easiest first: change the Wi-Fi password, then add a button that prints hello world.

## What You Need

- A **XIAO Robot Kit** with a charged battery on the **battery connector** — the motors pull more than a USB port can give
- The **esp32 core 3.x** in the Arduino IDE — [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/) walks through it. If it is missing, just say so: the agent sets it up for you, and asks before installing anything
- The example code: [XIAO-Robot-Kit](https://github.com/Seeed-Projects/XIAO-Robot-Kit) — you do not have to clone it yourself; both prompts below ask the agent to check for the repository and clone it when it is missing
- One AI coding tool, installed and signed in with your own account — **Claude Code** or **Codex**, terminal agents, or **Cursor**, an editor with a chat panel next to the files

Install whichever one you pick from the vendor's own page — everything below starts once it is running. Start it in any folder you can write to, not necessarily the repository. Say `test` first: an answer means the tool is running.

:::tip
Every task below lets the agent flash the board and read its serial port. To upload by hand instead, follow the Arduino IDE steps in [Getting Started](https://wiki.seeedstudio.com/xiao_robot_kit_getting_started/).
:::

## 1. Change the Wi-Fi Password {#first-task}

### The Prompt

Plug the battery in, connect the USB cable, and paste this:

```text
First check where you are: run git remote -v and look for github.com/Seeed-Projects/XIAO-Robot-Kit.
If that remote is not there, clone https://github.com/Seeed-Projects/XIAO-Robot-Kit.git and do
everything below inside it.

The repository holds two example projects: examples/xiao_robot_kit_web_remote (Arduino,
a Wi-Fi remote control) and examples/xiao_robot_kit_ble_control (ESP-IDF, BLE commands).
Read the Wi-Fi remote one and put my home network into it: name MyHome-2.4G, password
12345678.
Tell me which file and which lines you changed.
Then flash the board, keep watching its serial port until it joins the network, and tell me
the address it printed — both the IP and the name it answers to.
```

Swap the two values for your own network before you paste. The board joins only when the name and the password both match your router.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/AI_Assisted_Development/Image/ai_agent_wifi_change.jpg" style={{width:700, height:'auto'}}/></div>

### What Comes Back

Three things, in the order you asked for them: the file and the lines it touched, a sentence on what changed, and the address the board printed once it joined.

The two values sit at the top of `examples/xiao_robot_kit_web_remote/src/app/car_config.h`, in `WIFI_SSID` and `WIFI_PASS` — check that those are the lines it edited and nothing else.

Whatever address comes back belongs to one board on one network. Your router hands out its own, so yours will be different, and it changes again when the car joins a different network.

The log also carries a name, `xiaorobot.local`. That is mDNS: it survives a new IP and is shorter to type, so try it first, and fall back to the IP when your network does not pass the name through.

### Open the Page

Open the address the agent reported and hold a direction key to drive.

:::tip
The board joins **2.4 GHz** networks only. If the name and the password are right and it still will not connect, your router is probably publishing one name for both bands — give the 2.4 GHz band its own SSID.
:::

## 2. Add a Button {#extend}

### The Second Prompt

```text
If you are not inside the XIAO Robot Kit repository yet — github.com/Seeed-Projects/XIAO-Robot-Kit
— clone it first and work inside it.

In examples/xiao_robot_kit_web_remote, add a button to the web page. Pressing it prints
hello world in the serial log. Leave the motors and the driving page alone.
Then flash it, open the page, press the new button, and keep reading the serial port until
hello world appears. Tell me which files you touched and whether the test passed.
```

### What the Agent Tests

It flashes, then knocks on the new control the way a browser would: it opens the page and sends the button press as an HTTP request. The serial port stays open throughout. The report you get is short — files touched, request sent, log line seen — and it arrives only after all three line up.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/AI_Assisted_Development/Image/ai_agent_hello_button_report.jpg" style={{width:700, height:'auto'}}/></div>

### What You Should See

Press the new button yourself and `hello world` shows up in the serial monitor. If it compiles and nothing appears, paste the serial output back into the chat.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/AI_Assisted_Development/Image/ai_agent_hello_button.jpg" style={{width:320, height:'auto'}}/></div>

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
