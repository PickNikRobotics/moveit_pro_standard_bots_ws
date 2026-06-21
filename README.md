# MoveIt Pro — Standard Bots Workspace

[MoveIt Pro](https://picknik.ai/moveit-pro/) configuration packages for [Standard Bots](https://standardbots.com/) robot arms.

Forked from [`PickNikRobotics/moveit_pro_empty_ws`](https://github.com/PickNikRobotics/moveit_pro_empty_ws). Customer engagements should fork *this* workspace into a `<customer>_ws` repo under `PickNikRoboticsServices` rather than starting from scratch.

## Config Packages

| Package | Robot | Hardware | Status |
|---|---|---|---|
| `sbot_ro1_mock` | Standard Bots RO1 (6-DOF, 18 kg payload, 1.3 m reach) | `mock_components/GenericSystem` | Builds, launches, motion verified |
| `sbot_ro1_sim` | Standard Bots RO1 — physics simulation (inherits from `sbot_ro1_mock`) | `picknik_mujoco_ros/MujocoSystem` | Builds, launches, motion + Reset Simulation verified |

## Robot Description

The arm description (URDF, xacro macros, meshes) lives under `src/external_dependencies/standard_bots_description/` and is provided by Standard Bots directly. It exposes:

- `urdf/sbot/sbot_macro.xacro` — link/joint macro (`<xacro:sbot prefix="" parent="...">`)
- `urdf/sbot/sbot_macro.ros2_control.xacro` — ros2_control macro that switches between `mock_components/GenericSystem` (when `use_mock_hardware:=true`), Gazebo plugins, or `standard_bots_hardware_interface/SbotHW` (real hardware) based on its parameters.
- `meshes/sbot/ro1/{visual,collision}/*.STL` — per-robot mesh layout. Tracked via [Git LFS](https://git-lfs.github.com).

`sbot_ro1_mock/description/sbot_ro1.urdf.xacro` is a thin wrapper that includes the macro, attaches it to a `world` frame, adds a `grasp_link` for IK targeting, and instantiates the ros2_control macro with `use_mock_hardware:=true`.

## Setup

Git LFS is required to fetch the mesh files:

```bash
brew install git-lfs   # macOS; on Linux: sudo apt-get install git-lfs
git lfs install
git clone git@github.com:PickNikRobotics/moveit_pro_standard_bots_ws.git
cd moveit_pro_standard_bots_ws
git lfs pull
```

Then point MoveIt Pro at the workspace and a config package:

```bash
moveit_pro configure -w "$PWD" -c sbot_ro1_mock     # or sbot_ro1_sim for physics
moveit_pro build
moveit_pro run
```

Web UI: <http://localhost> (or the VM's IP if running in a Parallels VM).

## Test motions

Two motion objectives are wired up (and inherited by `sbot_ro1_sim`):

- **Move to Home** — non-singular elbow-up tucked pose `[0.0, -0.6, 1.2, -0.6, 0.0, 0.0]`
- **Move to Ready** — visibly distinct test pose `[0.7, -1.2, 1.6, -0.4, 0.5, 0.0]`

`sbot_ro1_sim` also adds **Reset Simulation** (Simulation category) which deactivates trajectory controllers and snaps the MuJoCo state back to the `default` keyframe.

Click either in the web UI's Motion category, or send via ROS action from inside the agent_bridge or drivers container:

```bash
ros2 action send_goal /do_objective \
  moveit_studio_sdk_msgs/action/DoObjectiveSequence \
  '{objective_name: "Move to Ready"}'
```

## Stop

```bash
moveit_pro down
```

## Mac + Parallels host workflow notes

If you're driving `moveit_pro` over SSH from a Mac into a Parallels Linux VM, the wrapper requires `DISPLAY` and `XAUTHORITY` in the SSH session — even for mock configs that do no rendering. See [`docs_nonpublic/Developer-Mac-Guide.md`](https://github.com/PickNikRobotics/moveit_pro/blob/main/docs_nonpublic/Developer-Mac-Guide.md) in the main moveit_pro repo for the full workaround.
