---
description: Hand the XIAO Robot Kit repository to a coding agent — what the repo already tells the agent, the first prompt to send, three tasks to copy, and how to check its work.
title: AI Agent Quickstart
keywords:
  - xiao robot kit
  - ai agent
  - vibe coding
  - claude code
  - coding agent
  - esp32s3
  - embedded workflow
  - agents.md
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /xiao_robot_kit_ai_agent_quickstart
sku: 100058302
sidebar_position: 4
last_update:
  date: 11/13/2026
  author: 划水佬
createdAt: '2026-11-13'
updatedAt: '2026-11-13'
url: https://wiki.seeedstudio.com/xiao_robot_kit_ai_agent_quickstart/
---

# AI Agent Quickstart

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/xiao_gadgets_xiao_robot_kit/Getting_Started/Image/bazaar_xiao_robot_board1.jpg" style={{width:800, height:'auto'}}/></div>

<div class="get_one_now_container" style={{textAlign: 'center'}}>
    <a class="get_one_now_item" href="https://www.seeedstudio.com/XIAO-Robot-Kit-p-100058302.html?utm_source=wiki" target="_blank">
            <strong><span><font color={'FFFFFF'} size={"4"}> Get One Now 🖱️</font></span></strong>
    </a>
</div><br />

## Overview

This page hands the kit to a coding agent: you describe the change, the agent edits the firmware, flashes it, and shows you the car doing it.

The repository is already written for that. It carries its own rules and a debugging playbook, so the agent starts with the same conventions the code was built with — no briefing needed.

- **[What the repository gives an agent](#repo)** — the files it reads on its own
- **[First prompt](#first-prompt)** — copy it, send it, no edits yet
- **[Three tasks you can copy](#tasks)** — each with a way to prove it worked
- **[How to check its work](#verify)** — what "done" has to look like

## What the Repository Gives an Agent {#repo}

Three files do the briefing:

| File | What it does |
| --- | --- |
| `AGENTS.md` | Eight standing rules: verify by running, never guess hardware, short ISRs, one-way layering, evidence over claims |
| `.agents/skills/embedded-vibe-assistant/SKILL.md` | The working manual: build-chain workflow, code guardrails, eight field-tested debugging playbooks, a definition of done |
| `.claude/skills/embedded-vibe-assistant/SKILL.md` | The same skill as a short entry point for Claude Code |

Claude Code and pi pick up `AGENTS.md` from the repository root by themselves. Any other agent just needs one sentence: *read `AGENTS.md` and `.agents/skills/embedded-vibe-assistant/SKILL.md` before touching code.*

That manual is not decoration. Its playbooks are the failures that actually cost hours on this board: a `Serial` call stalling the whole loop, a BLE peripheral that stops advertising after one disconnect, an upload that silently runs the previous binary.

## 1. Clone, Then Start the Agent Inside It {#step-1}

```bash
git clone <repository-url> xiao-robot-kit
cd xiao-robot-kit
```

Start the agent in that folder, not in your home directory. `AGENTS.md` and the skills are discovered relative to the working directory, so starting one level up means the rules never load.

**[TBD-45]** Screenshot: cloning the repository and starting the agent inside it.

## 2. First Prompt — Make It Read Before It Writes {#first-prompt}

Send this as it is. It asks three questions and forbids edits:

```text
Read AGENTS.md and .agents/skills/embedded-vibe-assistant/SKILL.md first.
Then read examples/xiao_robot_kit_web_remote/src/app/car_config.h,
src/hw/car_hw.h and src/net/car_web.cpp.

Answer three things, and change nothing:
1. Where is the Wi-Fi SSID set, and which single line do I edit?
2. What exact command compiles and uploads this sketch to the board?
3. How would you prove the change took effect on real hardware?

Quote file paths and line numbers for each answer.
```

If it cannot name the file where `WIFI_SSID` lives, it has not read the code. Start a new session instead of correcting it — an agent that skipped the reading step will guess its way through the rest.

**[TBD-46]** Screenshot: the agent answering the three questions with file paths.

## 3. A Task in Four Parts {#tasks}

A prompt that works has four parts: the goal, how you will check it, the files it may touch, and what it must leave alone. Without those, the agent will helpfully refactor half the project.

```text
Goal: raise the browser page's driving speed from 150 to 250 mm/s.
Allowed: examples/xiao_robot_kit_web_remote/src/net/car_web.cpp
Verify: arduino-cli compile --fqbn esp32:esp32:XIAO_ESP32S3_Plus passes, then
        upload and the car drives visibly faster while telemetry keeps updating.
Do not touch: pin definitions, sdkconfig, or any other example.
Show me the diff first, wait for my OK, then edit.
```

Three tasks worth copying, smallest first:

| Task | What it changes | How you prove it |
| --- | --- | --- |
| Make the car faster | The `DRIVE_MMPS` value in `src/net/car_web.cpp`, currently `150` | Compile, upload, drive it — the page still shows live telemetry |
| Add a BLE command | `register_motor_commands()` in `main/main.c`, plus `docs/BLE_COMMANDS.md` | Write the new command from a BLE tool and get an `OK` reply |
| Slow the wave down | `run_wave_hand()` in `main/arm_action.c` | Send `wave_hand` over USB serial and time the motion |

The last one is the best first exercise: the effect is visible, nothing else depends on it, and reverting is a one-line diff.

## 4. How to Check Its Work {#verify}

An agent finishing a task reports a conclusion. Check three things before you believe it:

1. **The compiler output.** Not "it compiles" — the last lines of a real `arduino-cli compile` or `idf.py build`.
2. **The behaviour on the board.** The line the firmware printed, or the page moving. If it could not flash, the honest answer is "not verified on hardware", and that is still a useful answer.
3. **The diff.** `git diff --stat` should touch only the files you allowed. Anything else gets reverted first, then discussed.

AGENTS.md ends with the same checklist, so you can just ask: *run the definition of done in the skill and show me each item with its evidence.*

## Five Things That Go Wrong {#pitfalls}

- **It guesses the pins.** Tell it to read `src/hw/car_hw.h` for the Arduino sketch, or `main/motor_control.c` for the ESP-IDF firmware, before it reasons about wiring.
- **It compiles and stops there.** "Compiles" is step one of four. Put the upload and the observation into the acceptance line of the prompt.
- **The car freezes with the log open.** `Serial` on the ESP32-S3 blocks the loop when no monitor is attached; the agent knows this as playbook P1 and should route output through the non-blocking path.
- **The upload "succeeds" and nothing changes.** `arduino-cli` reuses a stale build cache. Ask for `compile --clean`.
- **The code and the docs drift apart.** For BLE commands, `docs/BLE_COMMANDS.md` is part of the task, not an afterthought.

## What the Agent Should Not Decide

Power, pin assignment and partition layout are hardware facts, not coding preferences. An agent that proposes moving a motor pin or changing `sdkconfig.defaults` has left its lane — ask it to explain the evidence, then decide yourself.

And it should not edit `AGENTS.md` or the skill. Those are the rules it is being held to.

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