### What Is a Driver?

A **driver** is the lowest-level software component that communicates directly with hardware (or simulated hardware) through its native control interface.

In robotics, a driver:

1. Publishes motor commands in the exact message format expected by the robot  
2. Runs at a fixed control frequency  
3. Owns joint-level authority  
4. Translates desired motion into valid actuator commands  

Everything above the driver (e.g., keyboard control, trajectory planning, inverse kinematics) depends on it.

---

### What We Are Building

We will create a ROS driver node that:

- Publishes `unitree_legged_msgs/MotorCmd`  
- Sends commands to `/z1_gazebo/JointXX_controller/command`  
- Runs continuously at a fixed rate  
- Provides deterministic joint-level control of the Unitree Z1  

This driver will serve as the foundation for all higher-level control.

## Creating the `z1_driver` Package

Before we can move the robot in simulation, we must create a driver.

A driver is the lowest-level software component that sends valid motor commands to the robot at a fixed rate.  
Without a driver, Gazebo will load the robot but nothing will move.

---

### Step 1 — Navigate to the Workspace Source Directory

```bash
cd ~/unitree_ws/src
```

### Step 2 — Create the Driver Package
```
catkin_create_pkg z1_driver roscpp unitree_legged_msgs
```
This command creates 
```
z1_driver/
├── CMakeLists.txt
├── package.xml
└── src/

```
It does not create a running node.
It creates a container where we will define one.

### Step 3 - Create a Source File for the Driver Node
```
mkdir -p z1_driver/src
touch z1_driver/src/z1_driver.cpp

```

ROS does not assume which files are executable.
We must explicitly define the source file that will become our driver node.

```
#z1_driver.cpp
#include <ros/ros.h>
#include <unitree_legged_msgs/MotorCmd.h>

int main(int argc, char** argv){
    ros::init(argc, argv, "z1_driver");
    ros::NodeHandle nh; 

    // Publisher for Joint01 only (start small)
    ros::Publisher joint1_pub = nh.advertise<unitree_legged_msgs::MotorCmd>(
        "/z1_gazebo/Joint01_controller/command", 1); 

    // Fixed-rate control loop 
    ros::Rate rate(500); // 500 Hz is conservative and safe 

    unitree_legged_msgs::MotorCmd cmd;
    cmd.mode = 10; 
    cmd.dq = 0.0; 
    cmd.tau = 0.0; 
    cmd.Kp = 15.0; 
    cmd.Kd = 1.0;

    double t = 0.0; 

    while (ros::ok()){

        // Simple deterministic motion
        cmd.q = 0.3 * std::sin(t); 

        joint1_pub. publish(cmd); 

        t += 0.002; // controls speed of motion
        rate.sleep(); 
    }

}
```

### Step 4 — Register the Executable in CMakeLists.txt
```
nano z1_driver/CMakeLists.txt
```

Add the following to the bottom. 

```
include_directories(${catkin_INCLUDE_DIRS})

add_executable(z1_driver src/z1_driver.cpp)
target_link_libraries(z1_driver ${catkin_LIBRARIES})

```
### Step 5 — Build the Workspace 
```
cd ~/unitree_ws
catkin_make
```

### Step 6 — Source the Workspace
```
source ~/unitree_ws/devel/setup.bash

```
