# omni_robot_bridge

Unified, robot-side ROS 2 bridge for omni quadrupeds. The repository name
describes the product boundary; the ROS package and executable names remain
`rosdeck_robot_bridge` for compatibility with deployed units.

This repository was history-preservingly extracted from `rosdeck/robot`. It
does not contain the Rosdeck App or WebSocket gateway. Its responsibilities
are deliberately limited to:

- exactly one owner of the selected vendor SDK;
- VBot and ZsiBot robot-model adapters;
- navigation, docking and teleoperation velocity arbitration;
- velocity limits, command watchdogs and fail-closed E-stop supervision;
- posture, locomotion and compatibility mapping commands;
- battery, adapter diagnostics and aggregate `RobotState` publication;
- signed/offline A/B deployment tooling for the robot runtime.

See [the complete ROS interface contract](docs/INTERFACES.md),
[the repository relationship map](docs/REPOSITORY_ARCHITECTURE.md) and
[the engineering backlog](docs/TODO.md).

## Runtime processes

| Process | Current ROS name | Purpose |
|---|---|---|
| Bridge | `/rosdeck_robot_bridge` | Adapter, lease, command arbiter and state aggregation |
| Safety supervisor | `/rosdeck_safety_supervisor` | Independent latched E-stop heartbeat and arm/latch services |

The `rosdeck_*` node names are a compatibility debt, not a design
recommendation. Migration to `omni_robot_bridge` and
`omni_safety_supervisor` needs aliases, launch tests and a major-version
deprecation window.

## Build the bridge core

Place this repository, `omni_robot_interfaces` and `omni_tf_manager` in one colcon
workspace, then build without a vendor SDK:

```bash
source /opt/ros/humble/setup.bash
colcon build \
  --packages-select omni_robot_interfaces omni_tf_manager rosdeck_robot_bridge \
  --cmake-args \
    -DROSDECK_BUILD_VBOT_ADAPTER=OFF \
    -DROSDECK_BUILD_ZSIBOT_ADAPTER=OFF
colcon test --packages-select rosdeck_robot_bridge
colcon test-result --verbose
```

VBot builds additionally require `function_msgs` and `software_msgs`. ZsiBot
builds require the matching vendor SDK root and model:

```bash
colcon build --packages-select rosdeck_robot_bridge --cmake-args \
  -DROSDECK_BUILD_VBOT_ADAPTER=OFF \
  -DROSDECK_BUILD_ZSIBOT_ADAPTER=ON \
  -DROSDECK_ZSIBOT_SDK_ROOT=/path/to/zsibot_sdk-main \
  -DROSDECK_ZSIBOT_MODEL=zsl-1
```

`scripts/build.sh`, `deploy.sh` and the A/B bundle scripts were migrated with
the deployed product and still contain optional Rosdeck gateway/mission
composition logic. Core CI uses colcon directly; decoupling those product
composition scripts is tracked as P0.

## Launch

```bash
source install/setup.bash
ros2 launch rosdeck_robot_bridge bridge.launch.py \
  config:=$(ros2 pkg prefix --share rosdeck_robot_bridge)/config/zsibot.yaml
```

Only one bridge/vendor adapter may run. Before a ZsiBot launch, use the
installed `assert-no-legacy-zsibot-owner.sh` check and stop any legacy SDK
bridge.

## Compatibility and release policy

- Custom product contracts are imported from `omni_robot_interfaces` and the
  manager-owned SLAM status from `omni_tf_manager`; this repository owns
  neither IDL.
- `/rosdeck/*` topics and package/executable names remain operational until a
  tested `/omni/*` replacement has shipped through a deprecation window.
- The extracted package metadata says `Apache-2.0`, while the former monorepo
  root carried `GPL-3.0`. The owner must resolve that provenance and add an
  authoritative root `LICENSE` before any external binary/source release.
