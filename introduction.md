---
layout: default
title: SDK Introduction
nav_order: 3
---

# Z1 Robot SDK Introduction

Three packages are provided for working with the Z1 robot:

| Package | Purpose |
|---------|---------|
| `z1_controller` | Directly controls the Z1 robot |
| `z1_sdk` | Interfaces for robot control |
| `unitree_ros` | Simulation files for Unitree products (Go1, A1, Aliengo, Laikago, Z1) |

---

## Table of Contents

- [1. z1\_controller](#1-z1_controller)
  - [1.1 z1\_ctrl executable](#11-z1_ctrl-executable)
  - [1.2 sim\_ctrl executable](#12-sim_ctrl-executable)
  - [1.3 unitreeArmTools.py](#13-unitreearmtoolspy)
  - [1.4 config/config.xml](#14-configconfigxml)
  - [1.5 config/saveArmStates.csv](#15-configsavearmstatescsv)
- [2. z1\_sdk](#2-z1_sdk)
  - [2.1 include/](#21-include)
  - [2.2 examples/](#22-examples)
  - [2.3 examples\_py/](#23-examples_py)

---

## 1. z1_controller

> **Note:** You only need to interact with the files described below. Other files in this folder can be ignored.

### 1.1 z1_ctrl executable

On first use, create a `build/` directory and compile the program. The resulting executable is `z1_ctrl`.

```bash
# Check version
./z1_ctrl -v

# Control robot via keyboard
./z1_ctrl k

# Control robot via SDK
./z1_ctrl
```

### 1.2 sim_ctrl executable

If Gazebo is installed, a `sim_ctrl` executable is also generated. It communicates with `unitree_ros` for simulation. Functionally identical to `z1_ctrl` in all other respects.

### 1.3 unitreeArmTools.py

Used to change the robot's lower machine IP address (default: `192.168.123.110`).

**Steps:**
1. Connect the robot to your PC via cable (use the backup network port)
2. Run the tool: `python3 unitreeArmTools.py`
3. Follow the on-screen prompts

### 1.4 config/config.xml

This file is read once when `z1_ctrl` starts. Key settings:

#### IP & Port

| Parameter | Description |
|-----------|-------------|
| `IP` | The robot's lower machine IP. If changed via `unitreeArmTools.py`, update this value to match so `z1_ctrl` can communicate with the robot. |
| `Port` | The robot listens on port `8880`. The PC-side port bound by `z1_ctrl` defaults to `8881`. Change this if you need to control multiple robots from the same PC. |

#### Collision

| Parameter | Description |
|-----------|-------------|
| `open` | Enable or disable collision checking — `Y` or `N` |
| `limitT` | Torque difference threshold used for collision detection |
| `load` | End-effector load on the final joint; affects feedforward torque calculation. Always active. |

### 1.5 config/saveArmStates.csv

Stores named joint angle positions used by `labelRun()` and `labelSave()`.

---

## 2. z1_sdk

### 2.1 include/

Header files for `unitree_arm_sdk`. Inline comments explain each function.

#### unitree_arm_sdk/control/

| File | Description |
|------|-------------|
| `ctrlComponents.h` | Collects all control parameters into one class for easy access |
| `unitreeArm.h` | Encapsulates all robot control interfaces |

#### unitree_arm_sdk/model/

Contains the `armModel` class with:
- Forward & inverse kinematics
- Inverse dynamics
- Spatial Jacobian calculations

Access these via `_ctrlComp->armModel` from within the `unitreeArm` class.

---

### 2.2 examples/

#### 2.2.1 highcmd_basic

A straightforward starting point for robot control. Demonstrates three control methods:

---

**`armCtrlByFSM()`**

Calls `unitreeArm` methods directly — `MoveJ()`, `MoveL()`, `MoveC()`, `backToStart()`. Equivalent to keyboard control (`./z1_ctrl k`).

---

**`armCtrlInJointCtrl()`**

Controls joint rotation direction instead of issuing raw joint commands (`q`, `q̇`). Internally computes:

```
q̇ = directions × ω
q_k = q_(k-1) + q̇ × δt
```

Equivalent to pressing `2` in keyboard mode.

---

**`armCtrlInCartesian()`**

Controls the end-effector direction in Cartesian space, abstracting over raw spatial velocity (`unitreeArm.twist`). Internally computes:

```
posture_k = posture_(k-1) + posture_Δ
T_k = T_Δ + T_(k-1)
[ω] = log(R_(k-1)ᵀ · R_k)
v = p_Δ
```

Where `T` is a homogeneous transformation matrix composed of rotation `R` and position `p`, and `[ω]` is the antisymmetric (skew-symmetric) matrix of `ω`.

Equivalent to pressing `3` in keyboard mode.

---

#### 2.2.2 highcmd_development

For trajectory-based control.

Once `sendRecvThread->start()` runs, `arm.sendRecv()` executes at **500 Hz**.

| Control space | Parameters sent to `z1_controller` |
|---------------|-------------------------------------|
| Joint space | `q`, `qd`, `gripperQ`, `gripperW` |
| Cartesian space | `twist` |

Keep updating these parameters continuously to drive the arm. You can also write a custom thread to call `unitreeArm.sendRecv()` directly.

---

#### 2.2.3 lowcmd_development

For direct motor-level control via PD parameters.

The final output torque for each motor is:

```
τ = kp × 25.6 × (qd − q) + kd × 0.0128 × (q̇d − q̇) + τf
```

> `25.6` and `0.0128` are protocol scaling factors in the motor communication layer.

**Recommended approach:** Define your own thread, call the `run()` function, which calculates motor commands and dispatches them via `sendRecv()` as UDP packets.

> The `sendRecvThread` in `CtrlComponents` is designed for high-level instruction calls (e.g., move forward). For `lowcmd`, use your own thread instead.

##### lowcmd via unitree_sdk2

You can also control motors directly via `unitree_sdk2`:

```bash
cd z1_controller/deploy/${arch}/bin

# Start the UDP service
./z1_udp_service --ns z1 --ip 192.168.123.110 --localport 8881
# Run with -h for full options

# Verify topics
cyclonedds ps | grep z1
cyclonedds subscribe rt/z1/lowstate
```

| Direction | Topic | Message type |
|-----------|-------|--------------|
| Send commands | `rt/z1/lowcmd` | `unitree_go::msg::dds::MotorCmds_` |
| Read state | `rt/z1/lowstate` | `unitree_go::msg::dds::MotorStates_` |

> **Note:** Motor gains do **not** need to be scaled when using `unitree_sdk2`. The torque equation becomes:
>
> ```
> τ = kp × (qd − q) + kd × (q̇d − q̇) + τf
> ```

---

#### 2.2.4 lowcmd_multirobots

Example for controlling multiple robotic arms simultaneously. See the **SDK Run** documentation for details.

---

### 2.3 examples_py/

Python interface for the Z1 SDK, defined in `arm_python_interface.cpp` using [pybind11](https://pybind11.readthedocs.io/en/stable/).

After compilation, a `unitree_arm_interface` library appears in `z1_sdk/lib/`. A pre-compiled `.so` for Python 3.8 on x86_64 is included and ready to use.

> A `unitree_arm_interface.pyi` stub file is included for IDE autocompletion. It has no effect on compilation or runtime — it only provides type hints since `.so` libraries don't expose them natively.

**To run an example:**

```bash
# Terminal 1 — start the controller
./z1_ctrl

# Terminal 2 — run the Python example
cd z1_sdk/examples_py
python3 example_highcmd.py
```

**HTTP service option:** The SDK can also be wrapped as an HTTP API using Python's `fastapi` component. See `example_http_service.py` for details.
