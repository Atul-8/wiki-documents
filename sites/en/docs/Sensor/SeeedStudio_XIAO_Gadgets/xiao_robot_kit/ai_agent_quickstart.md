---
description: Type a sentence, let an AI edit the firmware, press upload and the car does something new. Four tasks, easiest first — Wi-Fi password, top speed, and a button that prints hello world.
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

You type a sentence. An AI edits the firmware. You press upload and the car does something new.

An AI can change values, find constants and explain code. It cannot wire anything, and it cannot feel the car — "too fast" is your call, not its. Three tasks below, easiest first.

## 1. Install One Tool {#setup}

Get the code first: [XIAO_Robot_Kit_Vibe](https://github.com/Atul-8/XIAO_Robot_Kit_Vibe).

```bash
git clone https://github.com/Atul-8/XIAO_Robot_Kit_Vibe.git
```

No Git? Use **Code → Download ZIP** on that page and unzip it.

| Your habit | Use it | Start |
| --- | --- | --- |
| terminal is fine | **Claude Code** | Node.js 18+, then `npm install -g @anthropic-ai/claude-code`, then `claude` inside the folder |
| prefer clicking | **Cursor** | Install, *Open Folder* on the repository, open the chat panel |

Sign in once with the tool's own account, then open the **repository folder** — not your home directory. Test it with one word:

```text
test
```

Any answer back means it is running.

## 2. Change the Wi-Fi Password {#first-task}

Plug in the battery, connect the USB cable, and paste this:

```text
This folder holds two example projects: examples/xiao_robot_kit_web_remote (Arduino,
a Wi-Fi remote control) and examples/xiao_robot_kit_ble_control (ESP-IDF, BLE commands).
Read the Wi-Fi remote one and change its Wi-Fi to my home network:
SSID MyHome-2.4G, password 12345678.
Tell me which file and which lines you changed.
```

It reads the project, edits the values and reports the lines. Then flash it — either way works:

- **You flash it.** Arduino IDE, board **XIAO ESP32S3 Plus**, open the `.ino`, click upload. Or ask it for the two `arduino-cli` commands and run them yourself.
- **Let it flash for you.** It runs commands on your computer, so it asks permission each time — read the port name before you approve. `arduino-cli board list` prints the port; a guessed port fails silently instead of choosing the wrong board.

:::caution
Two things the AI cannot check for you: the battery has to be on the **battery connector**, because the motors draw more than a USB port supplies, and the board it flashes has to be the one you are looking at. Approve a shell command only when the port in it matches.
:::

Open the serial monitor at 115200:

```text
[wifi] connected, IP 192.168.1.42
open http://xiaorobot.local or http://192.168.1.42
```

Then open that address in a browser and hold the forward key.

:::tip
The board joins **2.4 GHz** networks only. Password correct but still connecting usually means your router uses one name for both bands — give 2.4 GHz its own SSID.
:::

**[TBD-46]** Screenshot: the two changed lines side by side.

Sentence, diff, upload. That loop is the whole idea.

## 3. Make It Faster {#tune}

```text
DRIVE_MMPS in examples/xiao_robot_kit_web_remote/src/net/car_web.cpp is 150.
Make it 250, touch nothing else, and tell me the line number.
```

## 4. Add a Button {#extend}

```text
In examples/xiao_robot_kit_web_remote, add a button to the web page. Pressing it prints
hello world in the serial log. Leave the motors and the driving page alone, and tell me
which files you touched.
```

Expect a few rounds — the page, the request handler and the log line all have to line up. Answer its questions in one line each, and paste back the serial output when it compiles but prints nothing.

## When It Goes Wrong

- **"Changed" and nothing changed.** Arduino caches builds. Ask for `compile --clean`, then upload again.
- **It edited five files.** Say *revert everything except `car_config.h`*, then restate the scope.
- **It says "should work".** Ask for the command and the output it actually produced.
- **It guesses about hardware.** Power, pin assignment and flash layout are facts. Stop it, ask what evidence it has, decide yourself.

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