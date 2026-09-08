# xArm ROS 2 Humble learning workspace

This is a throwaway, simulation-first workspace for an expected UFactory xArm-6.
It does not connect to a robot. Lite6 and `realmove` launch files are intentionally
not used.

## Environment verified

- Ubuntu 22.04.5 LTS
- ROS 2 Humble (`/opt/ros/humble`)
- `rosdep` 0.26.0; `python3-colcon-common-extensions` 0.3.0
- UFactory `xarm_ros2`, branch `humble`, commit `62936f7`
- Recursive submodule `xarm_sdk/cxx`, tag `v1.18.1`
- MoveIt 2.5.9; ros2_control/controller-manager 2.54.0; xacro 2.1.1

## Recreate

```bash
mkdir -p ~/xarm_ws/src
cd ~/xarm_ws/src
git clone --branch humble --recurse-submodules \
  https://github.com/xarm-Developer/xarm_ros2.git xarm_ros2
cd ..
source /opt/ros/humble/setup.bash
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro humble -r -y
colcon build --symlink-install --parallel-workers 2
source install/setup.bash
```

If APT says ROS packages are unavailable, the official ROS 2 APT source must be
configured (this container already had the `ros2-apt-source` package):

```bash
sudo apt update
sudo apt install ros2-apt-source
sudo apt update
```

Do not force-remove APT locks; wait for the active transaction to finish. A
container may also need `sudo rosdep fix-permissions` before a non-root
`rosdep update`.

## Verified fake xArm-6 launch

TigerVNC was running on display `:1`. In the VNC terminal:

```bash
export DISPLAY=:1
export XAUTHORITY=/root/.Xauthority   # use your own Xauthority path as needed
source /opt/ros/humble/setup.bash
source /workspace/xarm_ws/install/setup.bash
ros2 launch xarm_moveit_config xarm6_moveit_fake.launch.py
```

Verified: RViz initialized on TigerVNC, `UFRobotFakeSystemHardware` activated,
`joint_state_broadcaster` and `xarm6_traj_controller` configured, and MoveIt
reported “You can start planning now!”. The realtime FIFO warning is expected in
a container. The “no 3D sensor plugin” warning is expected for this sensor-free
fake launch.

## Graph exercise (while launch is running)

In a second sourced terminal, run:

```bash
ros2 node list
ros2 topic list | grep -E 'joint|tf|planning|trajectory|controller'
ros2 topic echo /joint_states --once
ros2 topic hz /joint_states
ros2 topic echo /tf_static --once
ros2 param list /controller_manager
ros2 param get /controller_manager update_rate
```

Expected observations include `/move_group`, `/rviz2`, `/controller_manager`,
`/robot_state_publisher`, `/joint_states` containing six xArm joints, and an
update rate of `150`. Stop `topic hz` with Ctrl-C. This exercise is intentionally
the stopping point before Gazebo.

## Next step (do not run automatically)

After the fake-RViz exercise, the xArm-6 Gazebo launch to investigate is:

```bash
ros2 launch xarm_moveit_config xarm6_moveit_gazebo.launch.py
```

That is a separate simulation milestone; it still does not use physical robot
hardware or an IP address.
