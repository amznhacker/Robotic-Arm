🤖 Z1 + Modern Robotics Ch. 2 — Project Ideas
1. Degrees of Freedom (DOF) Verification — Grübler's Formula
Topic: C-space dimension, Grübler's formula
Manually model the Z1 as a kinematic chain and apply Grübler's formula to verify it has 6 DOF. Then extend it: add a gripper — how does that change the count? What if you mount it on a mobile base?
What you'll do: Draw the joint/link diagram, classify each joint type (all revolute on Z1), apply the formula, verify against the 6-axis spec with joints J1–J6. Unitree Robotics

2. Joint Space Visualization
Topic: Configuration space as an abstract space
Write a Python script that samples random valid joint configurations (θ1...θ6) within the Z1's joint limits and visualizes the reachable envelope in 3D using FK.
Joint limits to use:
J1: ±150°, J2: 0–180°, J3: –165° to 0°, J4: ±80°, J5: ±85°, J6: ±160° Unitree Robotics
What you'll learn: How the C-space (a 6D torus/box) maps to task-space reachability.

3. Singularity Explorer
Topic: Configuration space singularities / workspace boundaries
Find and physically demonstrate singular configurations of the Z1 — poses where the arm loses a DOF (e.g., fully extended, or wrist axes aligned). Log joint torques via the SDK when near singularities.
What you'll do: Use the joint space control and low-level motor control from the SDK to sweep through near-singular poses and observe behavior. Unitree

4. C-Space Obstacle Mapping
Topic: C-space obstacles, free C-space
Place a physical obstacle on a table. For a 2-joint subset of the Z1 (hold J3–J6 fixed), map out which (θ1, θ2) combinations cause collision. Plot the C-space obstacle as a 2D region.
What you'll learn: The direct intuition behind why C-space planning is hard even for simple arms.

5. Task Space vs. Joint Space Control Demo
Topic: Implicit vs. explicit configuration representation
The Z1 supports both joint space control and Cartesian space control. Write a demo that moves the end-effector along a straight line in Cartesian space and logs the resulting joint trajectories — then repeat in pure joint space. Compare path quality and singularity behavior. Unitree

6. Holonomic Constraint Counter
Topic: Constraints reducing DOF, task constraints
Attach the gripper and have the Z1 hold a rigid rod while you add a second constraint (e.g., keep the rod horizontal). Formally count the constraints and resulting DOF using the Ch. 2 framework, then verify with the arm's behavior.

🛠 Tools You'll Need
ToolUsePython + numpyFK, joint sampling, visualizationROS + Z1 SDK (C++/Python)Real hardware controlmatplotlib / open3dC-space and workspace plotsroboticstoolbox-python (Peter Corke)Symbolic modeling, DH parameters

Best starter project: #2 (Joint Space Visualization) — it's immediately rewarding, uses real hardware limits, and builds the intuition that every later chapter depends on. Want code scaffolding for any of these?
