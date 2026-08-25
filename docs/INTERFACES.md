# Unified robot bridge interface contract

## Safety ownership

The bridge is the only component allowed to publish the vendor SDK-facing
velocity stream. Planner, docking and App/teleop commands enter separate
topics, then pass through authority selection, freshness checks, E-stop,
limits and the adapter. A second bridge or a direct publisher to the final
topic is a deployment error.

## Canonical command interfaces

| Direction | Default name | Type | Owner/use |
|---|---|---|---|
| In | `/omni/cmd_vel/teleop` | `TwistStamped` | Authorized App/joystick input |
| In | `/scan_planner/cmd_vel` | `Twist` | SCAN-Planner navigation input |
| In | `/omni/cmd_vel/docking` | `Twist` | omni_docking final approach input |
| In | `/omni/safety/estop` | `Bool` | Safety supervisor heartbeat/latch state |
| Out | `/omni/cmd_vel/final` | `Twist` | In-process ZsiBot adapter only |
| Out | `/omni/cmd_vel/arbiter_status` | `String` | Selected source, authority and rejection state |
| Service | `/omni/safety/reset_estop` | `Trigger` | Reset bridge latch after supervisor recovery |
| Service | `/omni/safety/arm_supervisor` | `Trigger` | Arm independent supervisor |
| Service | `/omni/safety/latch_estop` | `Trigger` | Explicitly latch supervisor |
| In | `/omni/safety/estop_request` | `Bool` | External safety request; false never clears latch |
| Out | `/omni/safety/supervisor_status` | `String` | Supervisor state heartbeat |

`/omni/cmd_vel/final` is an internal ROS seam required by the current adapter
implementation. It is not a public integration API and must be isolated by
deployment checks or a future in-process call path.

## State and telemetry

| Direction | Default name | Type | Meaning |
|---|---|---|---|
| Out | `/omni/robot_state` | `RobotState` | Aggregate localization, mission, battery and E-stop snapshot |
| In | `/omni/slam/status` | `omni_tf_manager/SlamStatus` | SLAM mode/readiness/localization state |
| In | `/omni/mission/status` | `MissionStatus` | Active mission state |
| Out | `/battery_state` | `sensor_msgs/BatteryState` | Merged BMS/SDK battery sample |
| Out | `/diagnostics` | `DiagnosticArray` | Adapter and bridge diagnostics |
| Out | `/omni/robot/connection` | `String` | Vendor adapter connectivity |
| Out | `/omni/robot/mode` | `String` | Vendor locomotion/posture mode |
| Out | `/omni/robot/sdk_error` | `String` | Last sanitized SDK error |
| Out | `/omni/robot/adapter_status` | `String` | Monotonic adapter summary |

`RobotState` is the high-level gate consumed by mission and docking. Raw
string telemetry is operational compatibility data and should not become a
new business-logic dependency.

## Authority and legacy robot commands

The implemented control lease uses:

| Direction | Default name | Type |
|---|---|---|
| In | `/rosdeck/control_command` | `String` |
| Out | `/rosdeck/control_status` | `String` |

Payloads are `acquire:<client_id>`, `heartbeat:<client_id>` and
`release:<client_id>`. This protocol currently gates ZsiBot SDK ownership and
is also used by `omni_docking`. The target API is
`/omni/control/authority` (`ControlAuthority`) backed by the same internal
lease state machine. Until that service exists, the mission manager and
bridge do not form a complete control path.

Legacy compatibility commands are configurable `/rosdeck/*` string/bool
topic pairs for mapping, posture and locomotion. They must be replaced by
typed `/omni/*` services/actions before removing the compatibility topics.

## Vendor boundary

| Adapter | External dependency | Current capability |
|---|---|---|
| VBot | `function_msgs`, `software_msgs` services | locomotion, posture, legacy mapping script |
| ZsiBot | vendor HighLevel SDK | exclusive SDK ownership, posture, locomotion, velocity, battery/diagnostics |

Robot-model-specific topic/service names, IP addresses, ports and limits are
configuration. High-level consumers must never branch on vendor type.
