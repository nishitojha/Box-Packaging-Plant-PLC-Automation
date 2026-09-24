# Box Packaging Plant

A PLC automation project for a box packaging line, programmed in **Ladder Diagram (LD)** using **OpenPLC Editor** (IEC 61131-3).

The program handles machine start/stop with an emergency stop, conveyor motor control, proximity-triggered filling, camera-based packaging verification, and a second conveyor stage.

## Tools

- OpenPLC Editor (Ladder Diagram)
- IEC 61131-3 standard timers: `TON` (on-delay) and `TP` (pulse)

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

## Program Description

| Rung | Name | Logic |
|------|------|-------|
| 1 | Start_Stop_System | `/Emergency_Stop` AND (`Start_Button` OR `LED`) AND `/Stop_Button` starts a 3 s TON timer; its output drives `LED`. The `LED` contact in parallel with Start acts as a seal-in (latch). |
| 2 | Motor1_Signal | `LED` AND `/Filling_Process` AND `/Linear_actuator` energizes `Motor_1`. Motor 1 runs while the machine is on and stops during filling or actuator movement. |
| 3 | Proximity_Sensor_Triggering_for_Filling_Process | `P_Sensor` AND `/Stop_Button` triggers a 10 s pulse (TP0) on `Filling_Process`. |
| 4 | Camera_Sensor_To_Verify_Packaging | `C_Sensor` AND `Semi_Filled` AND `/Stop_Button` triggers a 5 s pulse (TP1) on `Linear_actuator`. |
| 5 | Motor2_Signal | `Linear_actuator` triggers a pulse (TP2) on `Motor_2`. |

## Process Flow

1. Press **Start** - after the 3 s delay the machine turns on (LED lit) and Motor 1 runs.
2. A box reaches the filling station - **P_Sensor** triggers filling for 10 s and Motor 1 pauses.
3. The camera checks the box - if the conditions in rung 4 are met, the **linear actuator** fires for 5 s.
4. The actuator signal starts **Motor 2** for its timer duration.
5. **Stop** or **Emergency Stop** halts the system (see notes below).

## How to Open the Project

1. Install [OpenPLC Editor](https://autonomylogic.com/docs/installing-openplc-editor/).
2. Clone this repository:
   ```bash
   git clone https://github.com/<your-username>/Box-Packaging-Plant.git
   ```
3. In OpenPLC Editor, choose **Open Project** and select the cloned project folder.
4. Compile, then upload to an OpenPLC Runtime target or run it in the simulator.

## Notes / Known Points

- `Stop_Button` and `Emergency_Stop_Button` use normally-closed contacts in the logic.
- The seal-in on `LED` is driven by the timer output, so `Start_Button` must be held for the full 3 s before the machine latches on.
- `TP2` is set to `T#10m` (10 **minutes**). If 10 seconds was intended, change it to `T#10s`.
- Rungs 3-5 are not gated by `LED` or `Emergency_Stop_Button`, so those outputs can still trigger when the machine is off.

## Screenshots

Add screenshots of the ladder rungs and variable table to a `docs/` folder and link them here:

```markdown
![Ladder logic](docs/ladder_1.png)
```

## Author

Your Name - [GitHub profile](https://github.com/<your-username>)

## License

Add a license (e.g., MIT) if you want others to reuse this project.
