---
layout: default
title: SDK Introduction
nav_order: 3
---

# Introduction to SDK
{: .no_toc }

Three files are provided for working with the Z1 robot: `z1_controller`, `z1_sdk`, and `unitree_ros`.
{: .fs-6 .fw-300 }

---

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

| Package | Purpose |
|---------|---------|
| `z1_controller` | Directly controls the Z1 robot |
| `z1_sdk` | Contains interfaces for robot control |
| `unitree_ros` | Simulation files for Unitree products (Go1, A1, Aliengo, Laikago, Z1) |

---

## 1. z1_controller

### 1.1 build/z1_ctrl

On first use, create a `build` directory and compile the program. The resulting executable is `z1_ctrl`.

```bash
./z1_ctrl -v       # Check version information
./z1_ctrl k        # Control robot by keyboard
./z1_ctrl          # Control robot via SDK
```

### 1.2 build/sim_ctrl

If Gazebo is installed, a `sim_ctrl` executable will be generated for communicating with `unitree_ros`. Functionally identical to `z1_ctrl` otherwise.

### 1.3 unitreeArmTools.py

Used to change the lower machine IP address (default: `192.168.123.110`).

1. Connect the robot to your PC via cable (use the backup network port)
2. Run `python3 unitreeArmTools.py`
3. Follow the prompts

### 1.4 config/config.xml

Read once at `z1_ctrl` startup. Key parameters:

**IP & Port**
- `IP` — The robot's lower machine IP (set via `unitreeArmTools`). Must match for `z1_ctrl` to communicate with the robot.
- `Port` — Robot port is `8880`. The native PC port bound during `z1_ctrl` defaults to `8881`. Change this to control multiple robots from the same PC.

**Collision**
- `open` — Enable or disable collision checking (`Y` / `N`)
- `limitT` — Torque difference threshold used for collision detection
- `load` — End-effector load attached to the final joint; affects feedforward torque calculation

### 1.5 config/saveArmStates.csv

Stores joint angle labels used by `labelRun()` and `labelSave()`.

---

## 2. z1_sdk

### 2.1 include

Contains header files for `unitree_arm_sdk`. Comments within explain each function.

**`unitree_arm_sdk/control`**
- `ctrlComponents.h` — Collects all control parameters into a single class for easy access
- `unitreeArm.h` — Encapsulates all robot control interfaces

**`unitree_arm_sdk/model`**

Contains the `armModel` class with functions for forward & inverse kinematics, inverse dynamics, and spatial Jacobian calculations. Access via `_ctrlComp->armModel` inside the `unitreeArm` class.

### 2.2 examples

#### 2.2.1 highcmd_basic

A simple starting point for robot control. Includes three control methods:

**`armCtrlByFSM()`** — Calls `unitreeArm` methods directly (`MoveJ()`, `MoveL()`, `MoveC()`, `backToStart()`). Equivalent to keyboard control with `./z1_ctrl k`.

**`armCtrlInJointCtrl()`** — Controls joint rotation direction rather than issuing raw joint commands. Equivalent to pressing the `2` key in keyboard mode.

**`armCtrlInCartesian()`** — Controls the end-effector direction in Cartesian space, abstracting over raw spatial velocity (`unitreeArm.twist`). Equivalent to pressing the `3` key in keyboard mode.

#### 2.2.2 highcmd_development

For trajectory-based control. Once `sendRecvThread->start()` runs, `arm.sendRecv()` executes at **500 Hz**.

- **Joint space** — Sends `q`, `qd`, `gripperQ`, `gripperW` to `z1_controller`
- **Cartesian space** — Sends `twist` to `z1_controller`

Update these parameters continuously to drive the arm. You can also write a custom thread to call `unitreeArm.sendRecv()` directly.

#### 2.2.3 lowcmd_development

For direct motor-level control via PD parameters. `25.6` and `0.0128` are protocol scaling factors in the motor communication layer.

It is recommended to define your own thread, call the `run()` function, which calculates the appropriate motor commands and calls `sendRecv()` to dispatch UDP packets.

#### 2.2.4 lowcmd_multirobots

Example for controlling multiple robotic arms simultaneously. See **SDK Run** for details.

### 2.3 examples_py

Python interface for the Z1 SDK, defined in `arm_python_interface.cpp` using [pybind11](https://pybind11.readthedocs.io/). After compilation, a `unitree_arm_interface` library will appear in `z1_sdk/lib`.

**To run:**

1. In terminal 1:
```bash
./z1_ctrl
```

2. In terminal 2:
```bash
cd z1_sdk/examples_py
python3 example_highcmd.py
```
