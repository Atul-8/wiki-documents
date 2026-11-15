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

You type a sentence. An AI edits the firmware. You press upload and the car does something new.

An AI can change a value, add a small feature and explain code it did not write. It cannot connect anything for you, and it cannot feel the car. Three short steps below, easiest first.

## 1. Install One Tool {#setup}

Get the code first: [XIAO_Robot_Kit_Vibe](https://github.com/Atul-8/XIAO_Robot_Kit_Vibe).

```bash
git clone https://github.com/Atul-8/XIAO_Robot_Kit_Vibe.git
```

No Git? Use **Code → Download ZIP** on that page and unzip it.

| Your habit | Tool | Start |
| --- | --- | --- |
| terminal is fine | **Claude Code** | Node.js 18+, then `npm install -g @anthropic-ai/claude-code`, then `claude` inside the folder |
| prefer clicking | **Cursor** | Install it, choose *Open Folder* on the repository, open the chat panel |

Sign in once with your own account for that tool, then open the **repository folder** — not your home directory. Test it with one word:

```text
test
```

If it answers, it is running.

## 2. Change the Wi-Fi Password {#first-task}

Plug in the battery, connect the USB cable, and paste this:

```text
This folder holds two example projects: examples/xiao_robot_kit_web_remote (Arduino,
a Wi-Fi remote control) and examples/xiao_robot_kit_ble_control (ESP-IDF, BLE commands).
Read the Wi-Fi remote one and change its Wi-Fi to my home network:
SSID MyHome-2.4G, password 12345678.
Tell me which file and which lines you changed.
```

It reads the project, edits the values and reports the lines.

**[TBD-46]** Screenshot: the two changed lines side by side.

Then flash it — either way works:

- **You flash it.** Arduino IDE, board **XIAO ESP32S3 Plus**, open the `.ino`, click upload. Or ask it for the two `arduino-cli` commands and run them yourself.
- **Let it flash for you.** It has to run commands on your computer, and it will ask permission for each one — read the port name before you approve. `arduino-cli board list` prints the port that is really connected; a guessed port fails with no output at all.

:::caution
Two things the AI cannot check. The battery has to be on the **battery connector**, because the motors draw more than a USB port supplies. And the board it flashes has to be the one on your desk — approve a shell command only when the port in it matches that board.
:::

Open the serial monitor at 115200:

```text
[wifi] connected, IP 192.168.1.42
open http://xiaorobot.local or http://192.168.1.42
```

Yours will print a different IP, so open the address in your own log — or `xiaorobot.local` — and hold the forward key on the page.

:::tip
The board joins **2.4 GHz** networks only. If the password is right and it still will not connect, your router is probably publishing one name for both bands — give the 2.4 GHz band its own SSID.
:::

One sentence, one diff, one upload. That loop is the whole idea.

## 3. Add a Button {#extend}

```text
In examples/xiao_robot_kit_web_remote, add a button to the web page. Pressing it prints
hello world in the serial log. Leave the motors and the driving page alone, and tell me
which files you touched.
```

Expect a few rounds — the page, the request handler and the log line all have to line up. Answer its questions in one line each, and paste back the serial output when it compiles but prints nothing.

## When It Goes Wrong

- **It reports the edit, but nothing changed.** Arduino caches builds. Ask for `compile --clean`, then upload again.
- **It edited five files.** Say *revert everything except `car_config.h`*, then repeat the task with the scope spelled out.
- **It says "should work".** Ask for the command it ran and the output it produced.
- **It guesses about hardware.** Power, pin assignment and flash layout are facts. Stop it, ask what it based the answer on, and decide yourself.

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