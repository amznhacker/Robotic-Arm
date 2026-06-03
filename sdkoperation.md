---
layout: default
title: SDK Operation
nav_order: 8
---

# Unitree Z1 Setup & Operation Guide

## 0. System Prerequisites

**Supported ROS Versions**
- ROS Noetic (Ubuntu 20.04)
- ROS Melodic (Ubuntu 18.04)

**Install ros_control stack (Noetic)**
```bash
sudo apt install \
  ros-noetic-controller-manager \
  ros-noetic-gazebo-ros-control \
  ros-noetic-joint-state-controller \
  ros-noetic-effort-controllers \
  ros-noetic-joint-trajectory-controller
```

---

## 1. Create and Build the ROS Workspace

**Required workspace layout:**
```
~/unitree_ws/
├── src/
│   ├── unitree_ros/
│   └── unitree_legged_msgs/
```

```bash
source /opt/ros/noetic/setup.bash
mkdir -p ~/unitree_ws/src
cd ~/unitree_ws/src
```

Clone the repositories:
```bash
git clone https://github.com/unitreerobotics/unitree_ros.git
git clone https://github.com/unitreerobotics/unitree_ros_to_real.git
```

> `unitree_legged_msgs` is located inside `unitree_ros_to_real`. Move or symlink it into `~/unitree_ws/src/`.

Build:
```bash
cd ~/unitree_ws
catkin_make
```

**Activate the workspace:**
```bash
source ~/unitree_ws/devel/setup.bash
```

Verify:
```bash
rospack find unitree_gazebo
```

Launch simulation:
```bash
roslaunch unitree_gazebo z1.launch
```

✔ If Gazebo opens and the Z1 arm appears, the ROS path is correct.

---

## 2. Configure Communication Mode

Edit:
```
~/unitree_ws/src/unitree_ros/z1_controller/CMakeLists.txt
```

Only **one** mode must be active at a time:

**For ROS simulation:**
```cmake
set(COMMUNICATION ROS)
# set(COMMUNICATION UDP)
```

**For hardware / SDK testing:**
```cmake
set(COMMUNICATION UDP)
# set(COMMUNICATION ROS)
```

Rebuild after any change:
```bash
cd ~/unitree_ws
catkin_make
```

---

## 3. Build z1_controller (SDK / Hardware Mode)

This is standalone SDK testing — not ROS.

```bash
cd ~/unitree_ws/src/unitree_ros/z1_controller
mkdir -p build && cd build
cmake ..
make
```

Run the controller:
```bash
./z1_ctrl
```

Keyboard control mode:
```bash
./z1_ctrl k
```

**Expected output when no arm is connected:**
```
[WARNING] UDPPort::recv, unblock version, wait time out
```
This is normal. The terminal will continuously print this until an SDK client connects.

---

## 4. Build and Test the Z1 SDK

```bash
cd ~/unitree_ws/src/unitree_ros/z1_sdk
mkdir -p build && cd build
cmake ..
make
```

Run the demo:
```bash
./highcmd_basic
```

**Keyboard control sequence (SDK):**
1. Press `2` → enter labeled state
2. Press `0` → confirm
3. Type `forward` → execute motion
4. Press `~` → return to home position

Joint control mode activates automatically after these steps.

---

## 5. Real Machine Control

① Power on the robotic arm, then verify network connectivity:
```bash
ping 192.168.123.110
```

② Build `z1_controller` with `COMMUNICATION UDP` set (see Section 2), then:
```bash
cd z1_controller/build
./z1_ctrl
```

③ Open `z1_sdk/build` and run the example:
```bash
./highcmd_basic
```

Monitor the `z1_ctrl` terminal — it prints arm state and warnings continuously.

---

## 6. ROS Simulation Workflow

Full sequence: **Run ROS → Run sim_ctrl → Run SDK example**

① Launch simulation:
```bash
roslaunch unitree_gazebo z1.launch
```

② Open `z1_controller/build` and run:
```bash
./sim_ctrl
```

③ Open `z1_sdk/build` and run:
```bash
./highcmd_basic
```

---

## 7. Multiple Robot Control

To control two arms simultaneously (e.g., `192.168.123.110` and `192.168.123.111`):

**① Copy z1_controller:**
```bash
cp -r z1_controller z1_controller_111
```

**② Modify `z1_controller_111` for the second arm:**

`main.cpp` — update the UDP port (line ~51):
```cpp
ctrlComp->cmdPanel = new ARMSDK(events, emptyAction, "127.0.0.1", 8074, 8073, 0.002);
```

`config.xml` — set IP and port:
```xml
<IP>192.168.123.111</IP>
<Port>8882</Port>
```

`unitreeArmTools.py` — set the second arm's IP to `192.168.123.111`.

**③ Launch in three terminals:**

| Terminal | Command | Purpose |
|----------|---------|---------|
| 1 | `./z1_ctrl` in `z1_controller/build` | Controls arm at `.110` |
| 2 | `./z1_ctrl` in `z1_controller_111/build` | Controls arm at `.111` |
| 3 | `./lowcmd_multirobots` in `z1_sdk/build` | Sends commands to both |

Both arms' Joint1 will rotate simultaneously.

---

## 8. Environment Rules

`.bashrc` should contain **only**:
```bash
source /opt/ros/noetic/setup.bash
```

> ❗ Do **not** permanently source the Unitree workspace in `.bashrc`. The online guide's instruction to do so (`echo "source ~/unitree_ros/devel/setup.bash" >> ~/.bashrc`) is incorrect — it also references the wrong path.

Activate the Unitree workspace manually when needed:
```bash
source ~/unitree_ws/devel/setup.bash
```

Optional alias for convenience:
```bash
alias unitree='source ~/unitree_ws/devel/setup.bash'
```

**Validation checklist (run after activation):**
```bash
echo $ROS_DISTRO           # should print: noetic
which roscore              # should print: /opt/ros/noetic/bin/roscore
rospack find unitree_gazebo
```

---

> **Key corrections from the online guide:**
> - The `.bashrc` path was wrong (`~/unitree_ros/` instead of `~/unitree_ws/`) and permanently sourcing the workspace is discouraged
> - `sim_ctrl` is the correct binary for ROS simulation mode — it's a separate step between launching Gazebo and running the SDK
> - `mkdir build & cd build` in the online guide is a shell bug (`&` runs in background); the correct form is `mkdir -p build && cd build`
