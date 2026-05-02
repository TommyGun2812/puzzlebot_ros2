# PuzzleBot ROS 2 — Autonomous Navigation & SLAM

> Robotics and Intelligent Systems Integration · Tecnológico de Monterrey

Autonomous navigation project for the **PuzzleBot Jetson Lidar Edition** mobile robot, implementing the ROS 2 Navigation2 (Nav2) stack and SLAM Toolbox within a Gazebo Fortress simulation environment. The system supports two mutually exclusive operation modes: autonomous navigation using a pre-built occupancy grid map, and online asynchronous mapping via SLAM Toolbox.

---

## Team Members

| Name | Student ID |
|---|---|
| Laura Helena Molina Jiménez | A01706282 |
| Zoé Rebolledo Argueta | A01709875 |
| Ulises Carrizalez Lerín | A01027715 |
| Tomás Pérez Vera | A01028008 |

---

## Prerequisites

### System Requirements

| Component | Tested Version |
|---|---|
| Operating System | Ubuntu 22.04 LTS |
| ROS 2 | Humble Hawksbill |
| Gazebo | Fortress (Ignition) |
| Nav2 | Humble-compatible release |
| SLAM Toolbox | Humble-compatible release |
| Python | 3.10+ |
| CMake | 3.8+ |

### ROS 2 Package Dependencies

Install all required packages before building the workspace:

```bash
sudo apt install ros-humble-navigation2 \
                 ros-humble-nav2-bringup \
                 ros-humble-slam-toolbox \
                 ros-humble-ros-gz-bridge \
                 ros-humble-ros-gz-sim \
                 ros-humble-xacro \
                 ros-humble-robot-state-publisher \
                 ros-humble-joint-state-publisher
```

---

## Repository Structure

```
puzzlebot_ws/
└── src/
    └── puzzlebot_ros2/
        ├── puzzlebot_description/      # Robot URDF/Xacro model and meshes
        ├── puzzlebot_gazebo/           # Gazebo simulation environment and bridge
        └── puzzlebot_navigation2/      # Nav2 stack, SLAM config, maps, and RViz profiles
```

### `puzzlebot_description`

Contains the complete robot description in modular URDF/Xacro format. Defines chassis geometry, drive and caster wheel joints, the RPLidar 2D sensor, and the Gazebo differential drive controller plugin.

```
puzzlebot_description/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── puzzlebot_description.launch.xml   # Publishes robot_state_publisher
├── meshes/
│   ├── base/       # Chassis STL (Jetson Lidar Edition)
│   ├── sensors/    # RPLidar STL
│   └── wheels/     # Drive wheel and caster wheel STLs
├── rviz/
│   └── puzzlebot_description.rviz
└── urdf/
    ├── puzzlebot_base.urdf.xacro          # Root description file
    ├── base.xacro                         # Chassis and footprint
    ├── wheels.xacro                       # Drive and caster wheels
    ├── joints.xacro                       # Joint definitions
    ├── sensors.xacro                      # LiDAR link and Gazebo plugin
    ├── sensor_properties.xacro            # LiDAR macro parameters
    ├── macros.xacro                       # Shared geometry macros
    └── puzzlebot_control.xacro            # Differential drive controller
```

### `puzzlebot_gazebo`

Provides the Gazebo Fortress simulation environment. Includes the maze world SDF, the `ros_gz_bridge` configuration for bidirectional topic bridging between ROS 2 and Gazebo, and the top-level launch file that spawns the robot at position `(x=0.0, y=1.55)` with heading `yaw=-1.57 rad`.

```
puzzlebot_gazebo/
├── CMakeLists.txt
├── package.xml
├── config/
│   └── gazebo_bridge.yaml        # Topic bridge: /clock, /cmd_vel, /odom, /tf, /scan, /joint_states
├── launch/
│   └── puzzlebot_gazebo.launch.xml
└── worlds/
    └── maze_world.world
```

**Bridged topics:**

| ROS 2 Topic | Direction | Message Type |
|---|---|---|
| `/clock` | GZ → ROS | `rosgraph_msgs/msg/Clock` |
| `/cmd_vel` | ROS → GZ | `geometry_msgs/msg/Twist` |
| `/odom` | GZ → ROS | `nav_msgs/msg/Odometry` |
| `/tf` | GZ → ROS | `tf2_msgs/msg/TFMessage` |
| `/joint_states` | GZ → ROS | `sensor_msgs/msg/JointState` |
| `/scan` | GZ → ROS | `sensor_msgs/msg/LaserScan` |

### `puzzlebot_navigation2`

Core navigation package. Contains the primary launch files for both operation modes, the tuned Nav2 and SLAM Toolbox parameter files, a pre-built maze occupancy grid, and RViz configuration profiles for each mode.

```
puzzlebot_navigation2/
├── CMakeLists.txt
├── package.xml
├── config/
│   ├── nav2_params.yaml          # Full Nav2 stack parameters (AMCL, planner, controller, costmaps)
│   └── slam_toolbox.yaml         # SLAM Toolbox online async mapping parameters
├── launch/
│   ├── nav2.launch.xml           # Entry point — Navigation mode
│   ├── nav2_core.launch.xml      # Internal Nav2 node bringup
│   ├── slam.launch.xml           # Entry point — SLAM mode
│   └── slam_core.launch.xml      # Internal SLAM Toolbox bringup
├── maps/
│   ├── map_maze.pgm              # Occupancy grid (0.05 m/px resolution)
│   └── map_maze.yaml             # Map metadata (origin, thresholds, resolution)
└── rviz/
    ├── nav2.rviz
    └── slam.rviz
```

---

## Installation and Build

> **Prerequisite:** A ROS 2 Humble workspace at `~/puzzlebot_ws` is assumed to already exist.

### 1. Clone the Repository

```bash
cd ~/puzzlebot_ws/src
git clone https://github.com/TommyGun2812/puzzlebot_ros2.git
cd ~/puzzlebot_ws
```

### 2. Install Dependencies

```bash
rosdep install --from-paths src --ignore-src -r -y
```

### 3. Build the Workspace

```bash
colcon build
```

### 4. Source the Workspace

```bash
source install/setup.bash
```

> **Tip:** Add `source ~/puzzlebot_ws/install/setup.bash` to `~/.bashrc` to avoid re-sourcing on every new terminal session.

---

## Launching the Project

> **Important:** Navigation mode and SLAM mode are **mutually exclusive**. Do not run both launch files simultaneously.

---

### Mode 1 — Autonomous Navigation (Nav2)

Launches the Gazebo simulation and the complete Nav2 stack using the pre-built maze map (`map_maze.yaml`). Localization is performed by AMCL with a tuned differential motion model. Path planning uses `NavfnPlanner` (Dijkstra/A\*) and trajectory tracking uses `RegulatedPurePursuitController`.

The robot spawns at `(x=0.0, y=1.55, yaw=-1.57 rad)`, which matches the initial pose configured in `nav2_params.yaml`.

```bash
ros2 launch puzzlebot_navigation2 nav2.launch.xml
```

**Launch arguments:**

| Argument | Default | Description |
|---|---|---|
| `use_sim_time` | `true` | Sync to Gazebo simulation clock |
| `headless` | `false` | Suppress Gazebo GUI (server-only mode) |
| `map_path` | `maps/map_maze.yaml` | Path to the occupancy grid map |
| `nav2_params_file` | `config/nav2_params.yaml` | Path to Nav2 parameter file |

**Example — headless run:**

```bash
ros2 launch puzzlebot_navigation2 nav2.launch.xml headless:=true
```

Once all nodes are active, use the **2D Goal Pose** tool in RViz to publish a `geometry_msgs/PoseStamped` navigation goal.

---

### Mode 2 — Online Async Mapping (SLAM Toolbox)

Launches the Gazebo simulation alongside SLAM Toolbox in `mapping` mode using the Ceres solver (`SPARSE_NORMAL_CHOLESKY` + `LEVENBERG_MARQUARDT`). The robot can be teleoperated to explore the environment and build a map in real time. Loop closure is enabled with a search distance of 3.0 m.

```bash
ros2 launch puzzlebot_navigation2 slam.launch.xml
```

**Launch arguments:**

| Argument | Default | Description |
|---|---|---|
| `use_sim_time` | `true` | Sync to Gazebo simulation clock |
| `headless` | `false` | Suppress Gazebo GUI (server-only mode) |

**Example — headless run:**

```bash
ros2 launch puzzlebot_navigation2 slam.launch.xml headless:=true
```

**Saving the generated map:**

Once the environment has been fully explored, run the following in a separate terminal to serialize the occupancy grid:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/my_map
```

This produces `my_map.pgm` and `my_map.yaml`, which can be passed to `nav2.launch.xml` via the `map_path` argument.

---

## Robot Description

The **PuzzleBot Jetson Lidar Edition** is a compact differential-drive mobile robot designed for indoor autonomous navigation research.

| Property | Value |
|---|---|
| Drive system | Two-wheeled differential drive + passive caster |
| Primary sensor | 2D RPLidar (scan topic: `/scan`) |
| Controller | Gazebo differential drive plugin |
| Onboard computer | NVIDIA Jetson (physical platform) |
| Simulation | Gazebo Fortress with `ros_gz_bridge` |
| Robot radius | 0.15 m (costmap footprint) |
| Max velocity | 0.10 m/s linear · 0.8 rad/s angular |

---

## Configuration Reference

### Nav2 Key Parameters (`nav2_params.yaml`)

| Component | Plugin / Setting |
|---|---|
| Localization | `nav2_amcl::DifferentialMotionModel` · 200–800 particles |
| Planner | `nav2_navfn_planner/NavfnPlanner` |
| Controller | `RegulatedPurePursuitController` |
| Local costmap inflation | radius 0.10 m · scaling 3.5 |
| Global costmap inflation | radius 0.12 m · scaling 3.0 |
| LiDAR range | 0.15 m – 3.5 m |

### SLAM Toolbox Key Parameters (`slam_toolbox.yaml`)

| Setting | Value |
|---|---|
| Solver | `CeresSolver` — `SPARSE_NORMAL_CHOLESKY` |
| Trust strategy | `LEVENBERG_MARQUARDT` |
| Map resolution | 0.05 m/px |
| Map update interval | 5.0 s |
| Loop closure search distance | 3.0 m |
| Min travel before new scan | 0.5 m / 0.5 rad |

### Pre-built Map (`map_maze.yaml`)

| Property | Value |
|---|---|
| Resolution | 0.05 m/px |
| Origin | `[-2.0, -1.82, 0.0]` |
| Occupied threshold | 0.65 |
| Free threshold | 0.25 |

---

## Notes

- If new sensors are added to the URDF, the corresponding bridge entries must be added to `gazebo_bridge.yaml`.
- The Nav2 parameters in `nav2_params.yaml` have been tuned specifically for `maze_world.world`. Different maps or environments may require adjusting costmap inflation radii, planner tolerances, and AMCL particle counts.
- The `headless:=true` argument passes the `-s` flag to `gz_sim`, launching only the Gazebo server without a GUI — useful for CI or resource-constrained machines.