# CPE 631 Final Project: Socially Aware Robot Navigation in a Cafe Environment
Final Project for CPE 631 Cooperating Autonomous Mobile Robots

## Overview
This project runs a TurtleBot3 Burger robot in a Gazebo cafe environment using ROS 2 Jazzy and Nav2. The final project compares two navigation setups in the same map: a default Nav2 setup using NavFn and DWB, and a dynamic obstacle avoidance setup using NavFn, MPPI, and more conservative collision monitor settings. A separate Python metrics script is used to record goal performance after a Nav2 goal is sent in RViz.

## Dependencies
This project was made for ROS 2 Jazzy on Ubuntu 24.04. The main required packages are:

```bash
sudo apt install \
  ros-jazzy-ros-gz-sim \
  ros-jazzy-ros-gz-bridge \
  ros-jazzy-ros-gz-image \
  ros-jazzy-nav2-bringup \
  ros-jazzy-slam-toolbox \
  ros-jazzy-turtlebot3-gazebo \
  ros-jazzy-turtlebot3-navigation2 \
  ros-jazzy-rviz2
```

The Python metrics script also uses standard ROS 2 Python packages and message types, including `rclpy`, `action_msgs`, `nav_msgs`, and `sensor_msgs`.

## Main Files Used

The main files used to run the project are:

- `launch/cafe.launch.py`: launches the default Nav2 setup with NavFn and DWB.
- `launch/cafe_dynamic.launch.py`: launches the dynamic setup with NavFn and MPPI.
- `param/nav2_burger.yaml`: Nav2 parameters for the default setup, including the default collision monitor settings.
- `param/nav2_dynamic_conservative.yaml`: Nav2 parameters for the dynamic obstacle avoidance setup, including the more conservative collision monitor settings.
- `maps/cafe.yaml` and `maps/cafe.pgm`: static map used for localization and navigation.
- `worlds/cafe.world`: Gazebo cafe simulation world.
- `models/`: Gazebo models for the TurtleBot3, cafe, tables, and pedestrians.
- `rviz/navigation.rviz`: RViz layout used for sending navigation goals.
- `metrics_scripts/nav2_goal_metrics.py`: script used to collect navigation performance metrics.

## Build and Source the Workspace
Place this package inside the `src` folder of a ROS 2 workspace. Then build and source the workspace:

```bash
cd <your_ros2_ws>
colcon build --packages-select cpe631_ros2
source install/setup.bash
```

Run `source install/setup.bash` again whenever a new terminal is opened.

## RViz Goal Setup Order
After launching either navigation setup, use RViz to set up and send the robot goal in this order:

1. Click **2D Pose Estimate** and set the robot's initial pose on the map.
2. Click **Nav2 Goal** and select the goal location for the robot.

The initial pose should be set before sending a Nav2 goal so AMCL knows where the robot starts on the map.

## Running the Default Nav2 Setup
Use this command to launch the cafe world with the default Nav2 configuration:

```bash
ros2 launch cpe631_ros2 cafe.launch.py navigation:=true map_file:=maps/cafe.yaml enable_peds:=true
```

This setup uses:

- The launch file for the default Nav2 setup located at `launch/cafe.launch.py`.
- Default collision monitor settings from `param/nav2_burger.yaml`.
- The saved cafe map located at `maps/cafe.yaml`. (Note: `maps/cafe.pgm` is the actual map image used by the YAML file)
- The Gazebo cafe simulation world located at `worlds/cafe.world`
- The RViz layout for navigation located at `rviz/navigation.rviz`

This version uses NavFn as the global planner and DWB as the local controller. It also includes collision monitor settings through `param/nav2_burger.yaml`. Pedestrians are enabled in the Gazebo environment.

## Running the Dynamic Navigation Setup
Use this command to launch the cafe world with the dynamic navigation configuration:

```bash
ros2 launch cpe631_ros2 cafe_dynamic.launch.py navigation:=true map_file:=maps/cafe.yaml enable_peds:=true
```

This setup uses:

- The launch file for conservative dynamic obstacle avoidance located at `launch/cafe_dynamic.launch.py`
- A more conservative collision monitor configuration for dynamic obstacle avoidance from `param/nav2_dynamic_conservative.yaml`
- The same saved cafe map located at `maps/cafe.yaml` (Note: `maps/cafe.pgm` is still used)
- The same Gazebo cafe simulation world located at `worlds/cafe.world`
- The same RViz layout for navigation located at `rviz/navigation.rviz`

This version uses NavFn as the global planner, MPPI as the local controller, and more conservative collision monitor settings for dynamic obstacle avoidance around pedestrians.

## Running the Metrics Script
After launching either navigation setup, open another terminal, source the workspace, and run:

```bash
cd <your_ros2_ws>/src/CPE631_Final_Project_Codes_Shaligram
python3 metrics_scripts/nav2_goal_metrics.py --ros-args -p use_sim_time:=true
```

Then go back to RViz, click **2D Pose Estimate** to set the initial pose, and use **Nav2 Goal** to send a goal. The script will print performance metrics in the terminal and save the metrics in a CSV file after the goal finishes.

The metrics script records information such as:

- Goal result status
- Time to complete the goal
- Distance traveled using `/odom`
- Estimated pedestrian encounters using `/scan`

The default output file is:

```text
nav2_goal_metrics.csv
```

## Launch Argument Explanations
The two ROS 2 launch commands use launch arguments after the launch file name. These arguments change how the launch file starts the simulation and Nav2.

### Arguments used in `cafe.launch.py`

```bash
ros2 launch cpe631_ros2 cafe.launch.py navigation:=true map_file:=maps/cafe.yaml enable_peds:=true
```

- `navigation:=true` starts the Nav2 navigation stack instead of only launching the simulation.
- `map_file:=maps/cafe.yaml` tells Nav2 which saved map YAML file to load for localization and planning.
- `enable_peds:=true` spawns the pedestrian models in the cafe environment.

Other supported launch arguments include:

- `mapping:=true` starts SLAM mapping mode instead of navigation mode (no pedestrians).
- `use_sim_time:=true` makes ROS nodes use Gazebo simulation time.
- `model:=burger` selects the TurtleBot3 Burger model.

### Arguments used in `cafe_dynamic.launch.py`

```bash
ros2 launch cpe631_ros2 cafe_dynamic.launch.py navigation:=true map_file:=maps/cafe.yaml enable_peds:=true
```

- `navigation:=true` starts Nav2 navigation.
- `map_file:=maps/cafe.yaml` loads the saved cafe map.
- `enable_peds:=true` enables the pedestrians in the environment.

This launch file also supports extra arguments:

- `use_composition:=False` controls whether Nav2 nodes are launched using composition.
- `nav2_log_level:=info` sets the logging level for Nav2.

The main difference is that `cafe_dynamic.launch.py` loads `param/nav2_dynamic_conservative.yaml`, which uses MPPI and more conservative collision monitor behavior.

## ROS Arguments for the Metrics Script
The metrics script is run with ROS arguments:

```bash
python3 metrics_scripts/nav2_goal_metrics.py --ros-args -p use_sim_time:=true
```

- `--ros-args` tells ROS 2 that the following options are ROS-specific arguments.
- `-p use_sim_time:=true` sets the `use_sim_time` parameter to true, so the script uses Gazebo simulation time instead of real wall-clock time.

The script also has optional ROS parameters that can be changed using the same `-p` format. Examples include:

```bash
python3 metrics_scripts/nav2_goal_metrics.py --ros-args \
  -p use_sim_time:=true \
  -p encounter_range:=1.8 \
  -p front_angle_deg:=160.0 \
  -p min_cluster_points:=2
```

Important optional parameters:

- `odom_topic` sets the odometry topic. Default: `/odom`.
- `scan_topic` sets the lidar topic. Default: `/scan`.
- `status_topic` sets the Nav2 action status topic. Default: `/navigate_to_pose/_action/status`.
- `csv_path` sets the output CSV file path. Default: `nav2_goal_metrics.csv`.
- `encounter_range` sets the scan range used to estimate pedestrian encounters.
- `front_angle_deg` sets the front field of view used for encounter detection.
- `min_cluster_points` sets the minimum number of lidar points needed to count an obstacle cluster.

## Typical Workflow
Use the following workflow for a full test:

1. Launch either the default setup or the dynamic setup.
2. Wait for Gazebo, RViz, and Nav2 to finish loading.
3. In a second terminal, run the metrics script with `use_sim_time:=true`.
4. In RViz, click **2D Pose Estimate** and set the initial robot pose.
5. In RViz, click **Nav2 Goal** and send the robot to a goal location.
6. After the goal succeeds, aborts, or cancels, check the terminal and `nav2_goal_metrics.csv` for the results.
