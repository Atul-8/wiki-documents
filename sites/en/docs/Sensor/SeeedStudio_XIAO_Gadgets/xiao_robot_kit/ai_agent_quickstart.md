---
description: No coding needed — install an AI tool in three minutes, paste one sentence, and watch it change your car's Wi-Fi password and speed. A step-by-step first taste of vibe coding on the XIAO Robot Kit.
title: AI Agent Quickstart
keywords:
  - xiao robot kit
  - ai agent
  - vibe coding
  - claude code
  - cursor
  - beginner
  - esp32s3
  - wifi password
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

## What You Will Do on This Page

You do not need to write code. You type one sentence in Chinese or English, an AI changes the firmware for you, you press upload, and the car does something new.

Three steps, easiest first:

- **[Install an AI tool](#setup)** — three minutes, one download
- **[Change the Wi-Fi password](#first-task)** — one sentence, the fastest thing you can actually see working
- **[Make the car faster, then add a new move](#tune)** — the same trick, on a real feature

## What an AI Can Do Here — and What It Cannot

Three things it is genuinely good at, all of them on this kit:

| You say | It does |
| --- | --- |
| "Change the Wi-Fi password to my home network" | Edits the two lines in `car_config.h` |
| "The car is too slow, make the top speed 250 mm/s" | Finds the speed constant and changes it |
| "Why does the car freeze when I open the serial monitor?" | Reads the code and explains it — the fix is one line |

Two things it cannot do, and no prompt will change that:

- **It cannot wire anything.** It does not know where your battery is plugged in, and if you ask it to guess, it will guess wrong.
- **It cannot feel the car.** "Too twitchy", "too slow" — that judgement is yours. You turn it into a number, it makes the change.

:::note
This repository was written with an AI in the loop, so it already carries the rules: `AGENTS.md` at the root and a working manual under `.agents/skills/`. Your AI reads them on its own and inherits the lessons — for example that a `Serial` print can freeze the whole car when no monitor is attached. You do not have to explain any of that.
:::

## 1. Install an AI Tool {#setup}

Pick one. Both are ordinary desktop tools; neither needs an account on this kit.

| If you... | Use | How to start |
| --- | --- | --- |
| are fine with a terminal | **Claude Code** | Install Node.js 18+, run `npm install -g @anthropic-ai/claude-code`, then type `claude` inside the repository folder |
| prefer clicking | **Cursor** (or the AI panel in VS Code) | Install it, choose *Open Folder* on the repository, open the chat panel |

Either way you will be asked to sign in or paste an API key once — that is the tool's own account, not something from Seeed.

**First, get the code.** Everything lives in one repository — [XIAO_Robot_Kit_Vibe](https://github.com/Atul-8/XIAO_Robot_Kit_Vibe):

```bash
git clone https://github.com/Atul-8/XIAO_Robot_Kit_Vibe.git
```

No Git installed? Open the repository page, choose **Code → Download ZIP**, and unzip it somewhere you can find again.

Two things matter more than which tool you pick:

1. **Open the repository folder itself**, not your home directory. The AI looks for its instructions in the folder you opened.
2. **Start with a question, not a change.** Send this first:

```text
Read AGENTS.md and the skill under .agents/skills/ before doing anything.
Then tell me, in plain language, what examples/xiao_robot_kit_web_remote does
and which file holds the Wi-Fi name and password. Change nothing yet.
```

If the answer names `src/app/car_config.h`, you are set up correctly. If it starts inventing, close the session and open the repository folder properly.

**[TBD-45]** Screenshot: the AI's first answer, naming `car_config.h`.

## 2. First Real Change: the Wi-Fi Password {#first-task}

This is the best first task on the whole kit: two strings change, and you find out within a minute whether it worked.

**Step 1.** Set up the car as described in [Application 1](https://wiki.seeedstudio.com/xiao_robot_kit_application/#app-1-browser) — battery plugged in, USB cable to your computer, Arduino IDE or `arduino-cli` ready.

**Step 2.** Paste this, with your own network name and password:

```text
把 examples/xiao_robot_kit_web_remote/src/app/car_config.h 里的 Wi-Fi 改成我家的：
WIFI_SSID 改成 MyHome-2.4G
WIFI_PASS 改成 12345678
只改这两个值，别的文件不要动。改完告诉我改了哪两行，以及怎么烧录。
```

**Step 3.** It answers with the two lines it wants to change. Say yes. It edits the file.

**Step 4.** Upload. In the Arduino IDE: board **XIAO ESP32S3 Plus**, open the `.ino`, click upload. If you would rather not click, ask the AI for the `arduino-cli` command and it will give you the exact two lines.

**Step 5.** Open the serial monitor at 115200 and watch for this:

```text
[wifi] connected, IP 192.168.1.42
open http://xiaorobot.local or http://192.168.1.42
```

Then type `http://xiaorobot.local` in a browser on the same network. Hold the forward key and the car drives.

:::tip
The board joins **2.4 GHz** networks only. If the password is right and it still says connecting, your router is almost certainly offering the 5 GHz band under the same name — give 2.4 GHz its own SSID and use that one.
:::

**[TBD-46]** Screenshot: the AI showing the two changed lines side by side.

That is the whole loop: one sentence → a diff → an upload → the car behaves differently. Everything after this is the same loop with a bigger sentence.

## 3. Now Make It Yours {#tune}

**Make it faster.** Half a minute, and you will feel the difference:

```text
The browser remote tops out at 150 mm/s. Change it to 250 mm/s.
Only touch DRIVE_MMPS in examples/xiao_robot_kit_web_remote/src/net/car_web.cpp.
Tell me the line number, then I will upload and check.
```

The page looks identical. The car does not. That feeling — *I said a number, it found the line* — is the whole point of vibe coding.

**Add a move that does not exist yet.** This one takes a few rounds, which is the honest experience:

```text
In examples/xiao_robot_kit_ble_control, add a command called hello.
It should turn 90 degrees left, 90 degrees right, then drive forward 1 second.
Update docs/BLE_COMMANDS.md too, and tell me how to test it from a BLE tool.
```

If it asks three questions back, answer them in one line each. If it produces something that compiles but does not work, send it the serial log — that is all the feedback it needs.

## When It Goes Wrong

Four things happen often enough to recognise:

- **It says "changed" and nothing changed.** Arduino caches builds. Ask it for `compile --clean` and upload again.
- **It edited five files instead of one.** Say *revert everything except `car_config.h`* and it will. Then re-send the task with the allowed file written out.
- **It claims it works.** "Should work" is not a result. Ask for the exact command and its output — compiling, uploading, and the line the board printed.
- **It guesses about hardware.** Power, pin assignment and flash layout are facts, not preferences. If it proposes moving a motor pin, stop it, ask what evidence it has, and decide yourself.

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