## **Description :**


PLC Project: Box Packaging Plant

The Box Packaging Plant is a beginner-friendly PLC automation project designed to demonstrate the basic operation of a packaging system using Ladder Logic. The program helps understand the fundamental functionality of Contacts, Coils, Function Blocks, and Rung Layout, along with the concept of current/logic flow through each rung. It provides a simple practical approach to understanding how inputs control outputs and how different control conditions are connected to automate a packaging process.

## I/O List

### Inputs

| Name | Type | Address | Description |
|------|------|---------|-------------|
| Start_Button | BOOL | %IX0.0 | Start push button |
| Stop_Button | BOOL | %IX0.1 | Stop push button (wired normally closed in logic) |
| Emergency_Stop_Button | BOOL | %IX0.2 | Emergency stop (normally closed contact) |
| P_Sensor | BOOL | %IX0.3 | Proximity sensor - detects a box at the filling station |
| C_Sensor | BOOL | %IX0.4 | Camera sensor - verifies packaging |
| Semi_Filled | BOOL | %IX0.5 | Semi-filled box status signal |

### Outputs

| Name | Type | Address | Description |
|------|------|---------|-------------|
| LED | BOOL | %QX0.0 | Machine running indicator (also the system-run latch) |
| Motor_1 | BOOL | %QX0.1 | Conveyor / motor 1 |
| Motor_2 | BOOL | %QX0.2 | Conveyor / motor 2 |
| Linear_actuator | BOOL | %QX0.3 | Linear actuator |
| Filling_Process | BOOL | %QX0.4 | Filling process output |

### Internal Function Blocks

| Name | Type | Preset | Used in |
|------|------|--------|---------|
| Machine_Start_Timer | TON | T#3000ms | Rung 1 - start delay |
| TP0 | TP | T#10s | Rung 3 - filling duration |
| TP1 | TP | T#5s | Rung 4 - actuator pulse |
| TP2 | TP | T#10m | Rung 5 - motor 2 run time |


## Explanation

What this system does

This is a box-filling and inspection line: it starts/stops the machine safely, runs a conveyor, fills boxes when one arrives, checks with a camera whether the fill was successful, and diverts (rejects) boxes that aren't filled correctly using a second conveyor and an actuator.

Rung-by-rung breakdown :

Rung 1 — Start/Stop System -->

The Emergency Stop and Stop Button are wired as normally-closed contacts (shown with the diagonal slash), so the rung is only alive when neither is pressed.
Pressing Start_Button feeds into a 3-second TON (on-delay) timer. Once 3 seconds elapse with Start still held, the timer output energizes LED.
The LED contact wired in parallel with Start_Button is a seal-in (latch): once LED turns on, it keeps feeding the timer's input even after you release Start_Button, so the machine stays "on" continuously.
Releasing Stop_Button or hitting Emergency_Stop breaks the rung instantly and drops LED (and everything downstream that depends on it).

Rung 2 — Motor1_Signal -->

Motor_1 (the main conveyor) runs whenever the machine is on, except while a box is actively being filled or while the actuator is diverting a rejected box.
This pause-during-fill logic makes sense mechanically — you don't want the conveyor moving a box while it's under the filling nozzle, or while the actuator is mid-stroke ejecting a box.

Rung 3 — Proximity Sensor Triggering for Filling Process -->

When a box reaches the P_Sensor (and the system isn't stopped), it fires a 10-second pulse (TP) on Filling_Process.
A TP (pulse timer) ignores further input changes once triggered — it runs its full 10 seconds regardless of what P_Sensor does afterward. This guarantees a fixed, repeatable fill duration.
This is also what pauses Motor_1 via Rung 2 for those 10 seconds.

Rung 4 — Camera Sensor To Verify Packaging -->

After filling, the camera (C_Sensor) checks the box. If Semi_Filled is also true — meaning the box was not filled correctly — this rung fires a 5-second pulse on Linear_actuator.
The actuator only activates on the bad outcome (Semi_Filled true); correctly-filled boxes don't trigger this rung and pass through untouched.

Rung 5 — Motor2 Signal -->

When the actuator fires, it in turn triggers Motor_2 (the reject conveyor) for a pulse duration set to T#10m — 10 minutes.
This runs the reject conveyor to carry the diverted box away.

## How This Demonstrates the Core Concepts :

Contacts  -->

every input in the program is either NO (| |, passes power when the bit is true — Start_Button, P_Sensor, C_Sensor, Semi_Filled) or NC (|/|, passes power when the bit is false — Emergency_Stop_Button, Stop_Button, Filling_Process, Linear_actuator). The choice of NO vs. NC is doing real work here: safety inputs and interlocks are NC so their default (unpowered/inactive) state is "permit," and a fault or an active downstream process is what removes permission — not the other way around.

Coils -->

five coils in the program (LED, Motor_1, Filling_Process, Linear_actuator, Motor_2), and every one of them is also read as a contact somewhere else in the program. That's the whole architecture in one sentence — outputs from one rung become interlocking or triggering inputs for the next.
Function Blocks: one TON (delay before the machine reports "running") and three TPs (fixed-duration pulses for fill, verify/actuate, and discharge). The distinction matters: TON answers "how long until this turns on," TP answers "how long should this stay on once triggered" — and the program uses each for exactly the case it's suited to.

Rung layout / logic flow  -->

each rung reads left-to-right as a power-flow path — series contacts are AND conditions, the parallel LED branch in Rung 1 is the one OR condition in the whole program. Nothing in this project uses branching logic beyond that single seal-in, which is consistent with the "beginner-friendly" framing.

## Process Flow :

Operator holds Start for 3 seconds → machine latches ON (LED lit), Motor_1 starts running.

A box reaches the filling station → P_Sensor triggers a 10-second fill (Filling_Process); Motor_1 pauses during this window.

After filling, the camera inspects the box.

If filled correctly → nothing further happens, Motor_1 resumes, box continues down the main line.

If Semi_Filled → the Linear_actuator fires for 5 seconds, ejecting the box off the main conveyor.

The actuator firing starts Motor_2, running the reject conveyor to carry the ejected box away.

Stop or Emergency Stop at any time drops LED and halts the sequence (subject to the note below).


## How to Open the Project

1. Install [OpenPLC Editor](https://autonomylogic.com/docs/installing-openplc-editor/).
2. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/Box-Packaging-Plant.git
   ```
3. In OpenPLC Editor, choose **Open Project** and select the cloned project folder.
4. Compile, then upload to an OpenPLC Runtime target or run it in the simulator.
