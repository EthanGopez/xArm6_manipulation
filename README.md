# xArm ROS 2 workspace

A ROS 2 Humble development workspace for an xArm-6,
with MoveIt, RViz, Gazebo-related packages, and a reproducible VS Code
devcontainer. The current workflow is simulation-only: it uses fake hardware
and must not be given a real robot IP address.

Serves as a reference for [ECLAIR](https://github.com/ECLAIR-Robotics)'s upcoming HAND project, a fully human-in-the-loop glove interface to control the xArm with both electrode stimulated input and haptic feedback.

## Repository layout

- `.devcontainer/` — development-container and remote RViz desktop setup
- `xarm_ws/src/` — ROS 2 source packages, including the upstream xArm stack
- `xarm_ws/SETUP.md` — full environment, dependency, and verification notes
- `xarm_ws/REMOTE_RVIZ_HANDOFF.md` — VNC/noVNC and RViz handoff instructions

Generated `xarm_ws/build/`, `xarm_ws/install/`, and `xarm_ws/log/` directories
are intentionally ignored by Git; build using colcon to run locally.

## Quick start

Open the repository in VS Code and use **Dev Containers: Rebuild and Reopen in
Container**. Then build and source the workspace:

```bash
source /opt/ros/humble/setup.bash
cd /workspace/xarm_ws
rosdep install --from-paths src --ignore-src --rosdistro humble -r -y
colcon build --symlink-install
source install/setup.bash
```

Launch the verified fake xArm-6 planner:

```bash
# Required when launching from a normal devcontainer terminal.
bash /workspace/.devcontainer/start-vnc.sh
export DISPLAY=:1
ros2 launch xarm_planner xarm6_planner_fake.launch.py
```

Forward port `5901` in VS Code to use a native VNC client, or port `6080` for
the browser-based noVNC fallback. See the handoff guide for details.

For more setup details, see [`SETUP.md`](xarm_ws/SETUP.md) and [`REMOTE_RVIZ_HANDOFF.md`](xarm/wsREMOTE_RVIZ_HANDOFF.md)

## Safety note

Use fake or simulation launch files during this project workflow. Do not use a
`realmove` launch file or configure a physical robot connection without first
reviewing the safety and deployment requirements.
