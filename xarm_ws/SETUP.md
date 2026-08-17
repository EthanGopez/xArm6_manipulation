# xArm ROS 2 Humble learning workspace

Simulation-only setup for ECLAIR HAND of God. This workspace targets an
**xArm-6**. Do not supply a robot IP address or use a `realmove` launch file.

For the complete reproducible remote-desktop and fake-planner handoff, see
[`REMOTE_RVIZ_HANDOFF.md`](REMOTE_RVIZ_HANDOFF.md).

## Environment recorded

- Ubuntu 22.04.5 LTS
- ROS 2 Humble (`ros-humble-desktop` 0.10.0)
- Upstream: `https://github.com/xarm-Developer/xarm_ros2`
- Branch/revision: `humble` / `62936f7`
- Submodule: `xarm_sdk/cxx` at `v1.18.1`
- Useful resolved packages: MoveIt 2.5.9, MoveIt Servo 2.5.9, xacro 2.1.1

## Reproduce

```bash
source /opt/ros/humble/setup.bash
mkdir -p ~/xarm_ws/src
git clone --branch humble --recurse-submodules \
  https://github.com/xarm-Developer/xarm_ros2.git ~/xarm_ws/src/xarm_ros2
cd ~/xarm_ws
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro humble -r -y
colcon build --symlink-install
```

If the base image lacks the ROS APT source, install the official
`ros2-apt-source` configuration package first, refresh APT, and then run the
`rosdep` command. `Unable to locate package ros-humble-*` means that ROS APT
source is absent or its package index has not been refreshed; it is not an
xArm build error.

## xArm-6 fake MoveIt/RViz verification

Open a terminal in this directory and run:

```bash
cd /workspace/xarm_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 launch xarm_moveit_config xarm6_moveit_fake.launch.py
```

This is the verified installed launch-file path. It uses fake hardware: no
robot IP or physical arm is involved. In RViz, set the fixed frame to `world`
if needed, confirm the displayed model is an xArm-6, and use the Motion
Planning panel to plan a small pose change. Planning/execution should update
the fake joint-state model only.

## Graph inspection exercise

Leave the fake launch running in terminal A. In terminal B, source the same
two setup files, then run:

```bash
ros2 node list
ros2 topic list -t
ros2 topic echo --once /joint_states
ros2 param get /move_group robot_description_semantic
ros2 run tf2_ros tf2_echo world link6
```

Identify the publisher of `/joint_states` with:

```bash
ros2 topic info /joint_states -v
```

Then move a joint target in RViz and repeat `ros2 topic echo --once
/joint_states`. Explain which joint values changed, and compare `world` to
`link6` with the TF command. This connects the MoveIt interface, joint-state
stream, and kinematic transform tree without touching hardware.

## Troubleshooting

- `Package 'xarm_moveit_config' not found`: source Humble, then this
  workspace's `install/setup.bash`, in that order.
- RViz cannot display in OrbStack: keep using the graphical environment that
  already lets `rqt` run; the ROS launch itself is otherwise headless-safe.
- `rosdep` warns about root: use a normal user and initialize/update rosdep
  under that user when possible; the warning does not indicate an xArm error.
- APT lock errors: another package transaction is running. Wait for it; do
  not delete lock files.

### TigerVNC desktop for RViz

The devcontainer starts an XFCE desktop on display `:1` after each container
start. Both services listen only on the container loopback interface, so use
VS Code's **Ports** panel rather than exposing them directly on the network.

- Preferred: forward port `5901`, then connect a native TigerVNC Viewer to
  `localhost:5901`. This avoids browser rendering overhead.
- Fallback: open the forwarded port `6080` and visit
  `/vnc.html?autoconnect=true&resize=remote`.

The initial configuration intentionally uses no VNC password because neither
port is reachable outside the container without VS Code port forwarding. Do
not publish either port on a shared network without changing this to VNC
password authentication.

Inside the desktop, open a terminal and run the fake MoveIt command above.
For an RViz-only test, set `DISPLAY=:1` in a normal devcontainer terminal
before launching RViz. The current container forces software OpenGL, so VNC
can reduce transport latency but cannot make RViz rendering GPU-accelerated.

## Next command (not run in this setup)

After completing the fake-RViz exercise, the expected Gazebo direction is:

```bash
ros2 launch xarm_gazebo xarm6_gazebo.launch.py
```

Inspect `ros2 launch xarm_gazebo xarm6_gazebo.launch.py --show-args` first
and keep it simulation-only.
