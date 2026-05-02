# Robotics and Intelligent Systems Integration

Autonomous navigation project for the PuzzleBot mobile robot, implementing the ROS 2 Navigation2 (Nav2) stack and SLAM Toolbox within a Gazebo simulation environment. The system supports two operation modes: autonomous navigation using a pre-built map, and online mapping via SLAM.

---

## Team Members

| Name                       | ID         |
| -------------------------- | ---------- |
| Laura Helena Molina Jiménez | A01706282  |
| Zoé Rebolledo Argueta      | A01709875  |
| Ulises Carrizalez Lerín    | A01027715  |
| Tomás Pérez Vera           | A01028008  |

---

## Prerequisites

### System Requirements

| Tool | Recommended Version |
|------|-------------------|
| Operating System | Ubuntu 22.04 LTS |
| ROS 2 | Humble Hawksbill |
| Gazebo | Fortress (Ignition) / Classic |
| Nav2 | ROS 2 Humble compatible release |
| SLAM Toolbox | ROS 2 Humble compatible release |
| Python | 3.10 or higher |
| CMake | 3.8 or higher |

### ROS 2 Package Dependencies

Install the required ROS 2 packages before building the workspace:

```bash
sudo apt install ros-humble-navigation2 \
                 ros-humble-nav2-bringup \
                 ros-humble-slam-toolbox \
                 ros-humble-ros-gz-bridge \
                 ros-humble-xacro \
                 ros-humble-robot-state-publisher \
                 ros-humble-joint-state-publisher
```

---

## Package Structure

```
puzzlebot_ws/
└── src/
    └── puzzlebot_ros2/
        ├── puzzlebot_description/      # Robot URDF/Xacro model
        ├── puzzlebot_gazebo/           # Gazebo simulation environment
        └── puzzlebot_navigation2/      # Nav2 and SLAM configuration
```

### `puzzlebot_description`

Contains the full robot description in URDF/Xacro format, including chassis geometry, wheel joints, sensor definitions, and differential drive control configuration.

```
puzzlebot_description/
├── CMakeLists.txt
├── package.xml
├── launch/
│   └── puzzlebot_description.launch.xml   # Launches robot_state_publisher
├── meshes/
│   ├── base/       # Chassis STL mesh (Jetson Lidar Edition)
│   ├── sensors/    # Sensor STL meshes
│   ├── wheels/     # Drive wheel and caster wheel meshes
│   └── misc/
├── rviz/
│   └── puzzlebot_description.rviz         # Base RViz visualization config
└── urdf/
    ├── puzzlebot.urdf.xacro               # Root robot description file
    ├── base.xacro                         # Chassis and main body
    ├── wheels.xacro                       # Drive and caster wheels
    ├── joints.xacro                       # Joint definitions
    ├── sensors.xacro                      # LiDAR and sensor plugins
    ├── sensors_properties.xacro           # LíDAR macros
    ├── macros.xacro                       # Puzzlebot macros
    └── control.xacro                      # Differential drive controller
```

### `puzzlebot_gazebo`

Provides the simulation environment configuration, including the maze world definition and the ROS–Gazebo topic bridge required for sensor and actuator communication.

```
puzzlebot_gazebo/
├── CMakeLists.txt
├── package.xml
├── config/
│   └── gazebo_bridge.yaml             # ROS–Gazebo topic bridge configuration
├── launch/
│   └── puzzlebot_gazebo.launch.xml    # Launches Gazebo, spawns robot, starts bridge
└── worlds/
    └── maze_world.world               # Maze environment for navigation testing
```

### `puzzlebot_navigation2`

Core navigation package. Contains the main launch files for both operation modes, Nav2 and SLAM Toolbox parameter files, pre-built maps, and RViz configuration profiles.

```
puzzlebot_navigation2/
├── CMakeLists.txt
├── package.xml
├── config/
│   ├── nav2_params.yaml               # Nav2 stack parameters
│   └── slam_toolbox.yaml              # SLAM Toolbox parameters
├── launch/
│   ├── nav2.launch.xml                # Primary launch file — Navigation mode
│   ├── nav2_core.launch.xml           # Internal Nav2 node launcher
│   ├── slam.launch.xml                # Primary launch file — SLAM mode
│   └── slam_core.launch.xml           # Internal SLAM Toolbox node launcher
├── maps/
│   ├── map_maze.pgm                   # Occupancy grid image of the maze
│   └── map_maze.yaml                  # Map metadata file
└── rviz/
    ├── nav2.rviz                      # RViz configuration for navigation mode
    └── slam.rviz                      # RViz configuration for SLAM mode
```

---

## Installation and Build

### 1. Clone the Repository
It is understood that the user already counts with a worksapce, this repo should be clone as shown below:

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

> **Note:** It is recommended to add `source ~/puzzlebot_ws/install/setup.bash` to your `~/.bashrc` file to avoid sourcing the workspace manually on every new terminal session.

---

## Launching the Project

### Mode 1 — Autonomous Navigation with Nav2

This mode launches the Gazebo simulation with the robot in the maze environment and starts the complete Nav2 navigation stack using the pre-built map (`map_maze.yaml`). Navigation goals can be sent directly from RViz.

```bash
ros2 launch puzzlebot_navigation2 nav2.launch.xml
```

**Available arguments:**

| Argument | Default Value | Description |
|----------|--------------|-------------|
| `use_sim_time` | `true` | Use simulation time from Gazebo |
| `headless` | `false` | Run Gazebo without graphical interface |
| `map_path` | `maps/map_maze.yaml` | Path to the map file |
| `nav2_params_file` | `config/nav2_params.yaml` | Path to the Nav2 parameters file |

**Example with custom arguments:**

```bash
ros2 launch puzzlebot_navigation2 nav2.launch.xml headless:=true
```

Once the system is running, use the **"2D Goal Pose"** tool in RViz to send navigation goals to the robot.

---

### Mode 2 — Online Async Mapping with SLAM Toolbox

This mode launches the Gazebo simulation alongside SLAM Toolbox in online mapping mode. The robot can be teleoperated to explore the environment and build a map in real time.

```bash
ros2 launch puzzlebot_navigation2 slam.launch.xml
```

**Available arguments:**

| Argument | Default Value | Description |
|----------|--------------|-------------|
| `use_sim_time` | `true` | Use simulation time from Gazebo |
| `headless` | `false` | Run Gazebo without graphical interface |

**Example with custom arguments:**

```bash
ros2 launch puzzlebot_navigation2 slam.launch.xml headless:=true
```

To save the generated map, execute the following command in a separate terminal:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/my_map
```

This will produce a `my_map.pgm` and `my_map.yaml` pair that can later be used in navigation mode.

---

## Robot Description

The **PuzzleBot Jetson Lidar Edition** is a differential-drive mobile robot with the following characteristics:

- **Drive system:** Two-wheeled differential drive with a passive caster wheel
- **Primary sensor:** 2D LiDAR for obstacle detection, navigation, and SLAM
- **Onboard computer:** NVIDIA Jetson (physical platform)
- **Simulation:** Full Gazebo model with ROS 2 topic bridge for sensor and actuator data

---

## Additional Notes

- The `gazebo_bridge.yaml` file defines the topic mappings between Gazebo and ROS 2. If new sensors are added to the URDF, this file must be updated accordingly.
- The Nav2 parameters in `nav2_params.yaml` have been tuned for the included maze environment. Adjustments may be required for different maps or robot configurations.
- Navigation mode and SLAM mode are mutually exclusive. Both launch files must not be executed simultaneously.