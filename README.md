# 🚁 PID Drone Navigation
 
**Interactive simulation and visualization of PID-based drone control, autonomous path tracking, guidance, and disturbance handling** — built with Python, NumPy, Matplotlib, and Jupyter/Google Colab widgets.
 
This project progressively explores PID controller tuning, feedback-based path tracking, environmental disturbances, and autonomous drone navigation through interactive simulations and animations.
 
---
 
## 📑 Table of Contents
 
- [Overview](#overview)
- [Key Features](#key-features)
- [Simulation Modules](#simulation-modules)
- [Drone Control Architecture](#drone-control-architecture)
- [Takeoff Phase](#takeoff-phase)
- [Wind Disturbance](#wind-disturbance)
- [Performance Metrics](#performance-metrics)
- [Interactive Controls (Task 3 — Figure-8 Navigation)](#interactive-controls-task-3--figure-8-navigation)
- [Visualization](#visualization)
- [Tuning Guide](#tuning-guide)
- [Technologies Used](#technologies-used)
- [Running the Project](#running-the-project)
- [Repository Structure](#repository-structure)
- [Learning Objectives](#learning-objectives)
- [Important Note](#important-note)
- [Project Focus](#project-focus)
- [Author](#author)
- [Contact](#contact)
- [License](#license)
---
 
## Overview
 
This repository contains a collection of **independent** control-system simulations built to understand how **PID controllers and feedback-based guidance** apply to autonomous vehicles.
 
The notebook includes:
 
1. **Task 1 — Drone altitude control** using PID
2. **Task 2 — Guidance and path tracking** using an autonomous boat
3. **Task 3 — PID-based autonomous drone navigation** along a vertical figure-8 trajectory
Each task has its own independent controller, sliders, and default gain values — they do not share state. Controller parameters and environmental disturbances can be adjusted interactively, so their effect on stability, tracking performance, and trajectory shape can be observed in real time.
 
---
 
## Key Features
 
- PID controller implementation for drone altitude control
- Interactive tuning of **Kp, Ki, and Kd**
- Simulation of wind disturbances
- Drone flight animation
- Feedback-based autonomous path tracking
- Guidance and damping control
- Autonomous navigation along a vertical figure-8 trajectory
- Separate X/Y axis PID control
- Takeoff and trajectory-tracking phases
- Wind disturbance during trajectory tracking
- RMSE and maximum tracking-error calculation
- Interactive Jupyter widgets
- Real-time trajectory visualization
- Animated drone flight with trajectory trail
- Configurable path speed and disturbance intensity
---
 
## Simulation Modules
 
### 1. Drone Altitude PID Control (Task 1)
 
Demonstrates PID-based altitude control. The drone starts from ground level and attempts to reach a **10 m altitude setpoint**, with the controller output computed as:
 
```text
u(t) = Kp·e(t) + Ki·∫e(t)dt + Kd·de(t)/dt
```
 
A simple point-mass model uses PID output, gravity, and wind disturbance to determine acceleration.
 
**Parameters**
 
| Parameter    | Description                                   |
| ------------ | ---------------------------------------------- |
| `Kp`         | Proportional gain / controller response        |
| `Ki`         | Integral gain / steady-state error correction   |
| `Kd`         | Derivative gain / damping                       |
| `Wind Noise` | Magnitude of wind disturbance                   |
 
**Default Slider Values (Task 1)**
 
| Control    |  Range | Default |
| ---------- | -----: | ------: |
| Kp         |   0–10 |     1.5 |
| Ki         |    0–2 |     0.1 |
| Kd         |    0–5 |     0.5 |
| Wind Noise |   0–10 |     2.0 |
 
The recommended tuning sequence is **P → D → I**. The first **6 seconds** are disturbance-free, after which wind can be introduced. An animated visualization shows the drone's response to tuning and gusts.
 
### 2. Autonomous Guidance & Path Tracking (Task 2)
 
Demonstrates feedback-based guidance using an autonomous boat as a simplified vehicle model, following a sinusoidal reference path:
 
```text
x(t) = t
y(t) = 8 sin(0.2t)
```
 
The guidance controller computes acceleration commands from position error and velocity damping:
 
```text
ax = Kp·dx - Kd·vx + current_x
ay = Kp·dy - Kd·vy + current_y
```
 
Adjustable parameters:
- Guidance gain (`Kp`, range 0.1–2.0, default 1.0)
- Damping (`Kd`, range 0–3.0, default 0.0)
- X-direction current (range −1.0–1.0, default 0.3)
- Y-direction current (range −1.0–1.0, default 0.3)
The boat's trajectory, desired path, heading, and environmental current are all visualized — an intuitive lead-in to the drone navigation module below.
 
### 3. Autonomous Drone Figure-8 Navigation (Task 3)
 
The main simulator. The drone follows a **vertical standing figure-8 trajectory**, with X and Y position independently controlled via PID:
 
```text
x(t) = B · sin(ωt) · cos(ωt)
y(t) = CENTER + A · sin(ωt)
```
 
**Default geometry**
 
| Parameter                 |  Value |
| -------------------------- | -----: |
| Figure-8 center altitude   |   14 m |
| Half-height                |    5 m |
| Half-width                 |  2.5 m |
| Entry altitude             |    9 m |
| Time step                  | 0.05 s |
 
---
 
## Drone Control Architecture
 
Independent PID controllers drive the X and Y axes:
 
```text
u = Kp·e + Ki·∫e dt + Kd·de/dt
```
 
Control commands are treated as acceleration in a simplified point-mass model, integrated using Euler integration:
 
```text
v(t+dt) = v(t) + a(t)·dt
p(t+dt) = p(t) + v(t+dt)·dt
```
 
The controller continuously computes position error relative to the figure-8 reference trajectory.
 
---
 
## Takeoff Phase
 
Before entering the figure-8, the drone performs a vertical climb from `0 m` to the `9 m` entry altitude using a dedicated PD controller:
 
```text
Kp = 4.0
Kd = 3.0
```
 
This produces a smooth climb before the main tracking controller takes over.
 
---
 
## Wind Disturbance
 
Wind activates after **40%** of the trajectory (when a non-zero wind value is selected), applied independently to X/Y acceleration:
 
```text
ax += random(-wind_max, wind_max)
ay += random(-wind_max, wind_max)
```
 
The affected trajectory section is visually distinguished so the controller's disturbance response is easy to observe.
 
---
 
## Performance Metrics
 
**RMSE**
 
```text
RMSE = sqrt(mean(error²))
error = sqrt((x_ref - x_actual)² + (y_ref - y_actual)²)
```
 
**Maximum Tracking Error**
 
```text
Max Error = max(error)
```
 
**Tracking quality classification**
 
|        RMSE | Classification    |
| -----------: | ------------------ |
|    `< 0.5 m` | Excellent          |
|  `0.5–1.5 m` | Good                |
|    `> 1.5 m` | Poor — tune gains   |
 
---
 
## Interactive Controls (Task 3 — Figure-8 Navigation)
 
| Control     |   Range | Default |
| ----------- | ------: | ------: |
| Kp          |    0–10 |     3.0 |
| Ki          |     0–2 |    0.05 |
| Kd          |     0–8 |     2.0 |
| Wind Noise  |     0–5 |     0.5 |
| Path Speed  | 0.2–2.0 |     0.8 |
 
Sliders automatically rerun the simulation and update the trajectory plot. An **Animate Drone** button visualizes the full flight.
 
> **Note:** These controls are independent of the Task 1 altitude sliders — each task maintains its own gains, ranges, and defaults.
 
---
 
## Visualization
 
The simulator displays:
 
- Desired figure-8 trajectory
- Actual PID-controlled trajectory
- Takeoff path
- Wind-disturbed trajectory section
- Current drone position, ground reference, entry altitude
- RMSE and maximum tracking error
- Live altitude information
- Animated flight trajectory
**Flight phases:**
 
```text
Takeoff → Entry at 9 m → Vertical Figure-8 Tracking → Wind Disturbance → Trajectory Correction
```
 
---
 
## Tuning Guide
 
**Kp — Proportional Gain**
Controls how strongly the drone responds to current position error.
- Higher `Kp` → stronger path chasing
- Excessively high `Kp` → potential oscillation
**Ki — Integral Gain**
Accumulates positional error over time.
- Corrects persistent tracking drift
- Keep relatively small (recommended range 0.01–0.1)
**Kd — Derivative Gain**
Provides damping based on the rate of error change.
- Reduces overshoot and oscillation
- Higher values can improve stability during aggressive tracking
**Wind**
Introduces random external acceleration during the latter portion of the trajectory.
 
**Suggested starting configuration (Task 3 — Figure-8):**
 
```text
Kp = 3.0
Ki = 0.05
Kd = 2.0
Wind = 0
```
 
---
 
## Technologies Used
 
- Python
- NumPy
- Matplotlib
- Matplotlib Animation
- IPyWidgets
- Jupyter Notebook
- Google Colab
- Python Random Module
---
 
## Running the Project
 
### Option 1 — Google Colab
 
1. Open the `.ipynb` notebook in Google Colab.
2. Run the cells sequentially.
3. Adjust the PID parameters using the sliders.
4. Observe the trajectory plots.
5. Run the animation to visualize the drone's flight.
### Option 2 — Jupyter Notebook
 
```bash
pip install numpy matplotlib ipywidgets
jupyter notebook
```
 
Run the cells sequentially to access the interactive simulations.
 
---
 
## Repository Structure
 
```text
PID-Drone-Navigation/
│
├── PID DRONE Navigation.ipynb
├── README.md
└── LICENSE
```
 
---
 
## Learning Objectives
 
This project was built to explore practical concepts in:
 
- PID control
- Controller gain tuning
- Feedback control
- Drone altitude control
- Autonomous navigation
- Path tracking
- Guidance systems
- Trajectory generation
- Environmental disturbance modeling
- Numerical simulation
- Control-system visualization
- Performance evaluation
---
 
## Important Note
 
This repository implements **simplified simulation models** for learning and visualization. The drone is represented as a point-mass system, and PID output is modeled as an acceleration command. This does **not** represent a complete real-world quadrotor flight-control stack, aerodynamic model, sensor-fusion system, or hardware flight controller.
 
---
 
## Project Focus
 
The primary focus of this repository is understanding how **PID control and feedback-based trajectory tracking** can drive autonomous drone navigation — including controller tuning and response to external disturbances.
 
---
 
## Author
 
Know more about the author [Ayush Sharma](https://in.linkedin.com/in/ayush-sharma-student).  
 
---
 
## Contact
 
For questions, suggestions, or issues, feel free to [open an issue](https://github.com/Ayusharma24/PID-Drone-Navigation/issues) on this repository or reach out via Email - alpha992k80@gmail.com.
 
---
 
## License
 
This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.
 
