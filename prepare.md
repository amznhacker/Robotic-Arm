---
layout: default
title: Prepare before powering on
nav_order: 4
---


# Before Powering On

## Zero Position

> ⚠️ **Warning:** Make sure the `z1_ctrl` program is turned off before powering on the robotic arm, otherwise there may be danger.

Position the robotic arm in the zero position before powering on (see figure below). In zero position, the lines on both sides of the joint gaps of J1 and J6 correspond exactly, and the remaining joints can be placed in order.
![Zero Position](Assets/zeroposition.jpg)

Once powered on successfully:
- **Green light** — steady on
- **Blue light** — flashes once after self-check passes

> 📝 **Note:** Each joint must be returned to zero position before every use, so that the theoretical zero position in the control algorithm coincides with the actual mechanical zero position.

> ⚡ **Caution:** The black and white power cord of the motor at the end of the arm carries **DC 24V**. If unused, wrap it with insulating tape to prevent short circuit or other hazards.

---

> 📝 **Note:** If the terminal prints `Arm Version 3-8` after running `z1_Ctrl -v`, the robot may be powered on in any position. In this case, the first action after power on is to press the **`~`** key or execute `backtostart`.

## Network configuration
The default IP address of the robotics arm is 192.168.123.110. The users need to change the IP of the PC before control the robot. 

## Change network segment
With the example of changing to 192.168.123.162, run ifConfig in the terminal and you will find your port name. For example, enpxxx.
```
sudo ifconfig enpxxx down                                   # enpxxx is your PC port 
sudo ifconfig enpxxx 192.168.123.162/24 
sudo ifconfig enpxxx up 
```
The above method is to change the IP temporarily, you can also change the IP of the PC permanently, the operation please see below:

```
sudo vim /etc/network/interfaces
```
Add the following text to the interfaces file opened above.

```
auto enpxxx
iface enpxxx inet static
address 192.168.123.162
netmask 255.255.255.0
```
In this case, you can run the ping 192.168.123.110 command to check whether the connection with the robotic arm is normal.

## Change the default IP address
Insert the telecommunication cable in the auxiliary network port of the robot arm,
then execute z1_controller/unitreeArmTools.py in a terminal and enter command as prompted.

After that, power-on again.

## Singular Position
When the robotic arm is in a singular position, the degrees of freedom will degenerate and it will make the angular velocity of some joints to be infinite, and the robotic will be at the risk of losing control.

It is recommended to run the robotic arm to the forward point after power-on to avoid this area.

![SIngular Position](Assets/singularposition.jpg)
