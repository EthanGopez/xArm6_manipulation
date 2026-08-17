# Remote RViz / MoveIt handoff guide

This document describes the supported way to run the xArm-6 fake planner with
RViz from this devcontainer. It is simulation-only: do not add a robot IP or
use a `realmove` launch file.

## What is configured in the repository

The configuration lives in the workspace, so a recreated container does not
depend on manually installed GUI packages. Commit the files listed under
**Relevant files** with the project before handing it to another developer:

- `.devcontainer/devcontainer.json` installs `xfce4`, `xfce4-goodies`,
  `tigervnc-standalone-server`, and `novnc` in `postCreateCommand`.
- The same file starts `.devcontainer/start-vnc.sh` in `postStartCommand` and
  forwards ports `5901` and `6080`.
- `.devcontainer/start-vnc.sh` starts an XFCE desktop on X display `:1` and
  TigerVNC on `127.0.0.1:5901`. It also serves noVNC on
  `127.0.0.1:6080` and recovers safely from stale VNC lock files after an
  interrupted container shutdown.
- `8765` remains reserved for the Foxglove Bridge. Foxglove is not required
  for the RViz/MoveIt workflow and no ROS launch files were changed for it.

RViz currently uses software OpenGL (`LIBGL_ALWAYS_SOFTWARE=1`). TigerVNC can
improve remote-desktop transport versus host X11 forwarding, but it does not
turn RViz rendering into GPU acceleration.

## First use or after pulling these changes

1. Open the repository in VS Code and choose **Dev Containers: Rebuild and
   Reopen in Container**. This is required once after a change to
   `devcontainer.json`; it installs the GUI packages in the new container.
2. Wait for the container lifecycle commands to finish. On later normal
   reopen/start operations, the VNC desktop starts automatically.
3. Open VS Code's **Ports** panel. Ports `5901` and `6080` should appear. If
   not, use **Forward a Port** and enter the port number.

The older commented host-X11 instructions in `devcontainer.json` are not part
of this workflow and are unnecessary when using TigerVNC.

## Connect to the desktop

Use a native VNC client for the best latency:

1. Forward port `5901` in VS Code.
2. On the host, connect TigerVNC Viewer (or another compatible VNC client) to
   `localhost:5901`.

Browser fallback:

1. Forward/open port `6080` in VS Code.
2. Visit `http://localhost:6080/vnc.html?autoconnect=true&resize=remote`.

Both services bind to container-localhost. The VNC server intentionally uses
no password because access is expected to be through VS Code's local port
forwarding. Do not publish either port directly to a shared LAN or internet
endpoint without changing the VNC server to use password authentication.

## Run the verified xArm-6 fake planner

From a normal devcontainer terminal:

```bash
cd /workspace/xarm_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export DISPLAY=:1
ros2 launch xarm_planner xarm6_planner_fake.launch.py
```

If the command is run from a terminal inside the VNC XFCE desktop,
`DISPLAY=:1` is normally already set. This launch file comes from the
`xarm_planner` package and includes the fake six-DOF xArm MoveIt setup; no
physical robot is contacted.

For the base MoveIt configuration instead of the planner launch, use:

```bash
ros2 launch xarm_moveit_config xarm6_moveit_fake.launch.py
```

## Quick verification

In RViz:

- Set the fixed frame to `world` if it is not already set.
- Confirm that the displayed model is an xArm-6.
- Use the Motion Planning panel to plan a small pose change. In this fake
  setup, execution updates only the simulated joint state.

In a second sourced terminal, these commands confirm the ROS graph:

```bash
ros2 node list
ros2 topic echo --once /joint_states
ros2 run tf2_ros tf2_echo world link6
```

## Troubleshooting

| Symptom | Action |
| --- | --- |
| `Package ... not found` | Source `/opt/ros/humble/setup.bash`, then `install/setup.bash`, in that order. |
| RViz opens on the wrong display or fails to open from a normal terminal | Run `export DISPLAY=:1`, then launch again. |
| VNC desktop is absent after the container starts | Run `bash /workspace/.devcontainer/start-vnc.sh`; then check the Ports panel. |
| Browser desktop is slow | Prefer the native client on port `5901`; noVNC is a convenience fallback. |
| An arm link looks malformed/misaligned | Set RViz fixed frame to `world`; enable the TF display and check for errors. A screenshot plus the affected link name is useful for diagnosing a mesh versus transform issue. |
| RViz is slow in either client | This is likely software OpenGL rendering, not a VNC configuration failure. |

## Relevant files

- Devcontainer lifecycle and ports: `.devcontainer/devcontainer.json`
- Desktop launcher: `.devcontainer/start-vnc.sh`
- ROS workspace overview: `SETUP.md`
- xArm fake planner launch: `src/xarm_ros2/xarm_planner/launch/xarm6_planner_fake.launch.py`
