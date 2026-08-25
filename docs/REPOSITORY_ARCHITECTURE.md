# Robot-side repository architecture

## Ownership map

```text
Rosdeck App / omni_ws_gateway
          |
          | typed mission/control APIs
          v
omni_mission_manager ---- FollowRoute ----> SCAN-Planner
          |                                    |
          | Dock/GetDockConfig                 | navigation cmd_vel
          v                                    v
     omni_docking ---- docking cmd_vel ---> omni_robot_bridge ---> vendor SDK
          ^                                    ^
          | RobotState/Battery                 | SlamStatus
          +------------------------------------+---- omni_slam
                                                   omni_tf_manager
```

## Repository roles

| Repository | Owns | Must not own |
|---|---|---|
| `omni_mission_manager` | Mission state, routes/checkpoints, persistence, orchestration | SDK, velocity arbitration, SLAM, final docking |
| `omni_docking` | Dock geometry, servo/undock, charge verdict | Global planner, SDK, App transport |
| `omni_robot_bridge` | Vendor adaptation, sole SDK ownership, authority, safety arbitration, aggregate state | Mission policy, route planning, App UI/protocol |
| `omni_robot_interfaces` | Stable cross-repository ROS IDL and constants | Runtime behavior |
| `omni_slam` | Mapping/localization pose and status | Canonical sensor/static TF ownership |
| `omni_tf_manager` | Canonical frame names, static extrinsics, TF readiness | SLAM estimation or mission logic |
| `SCAN-Planner` | Route/goal planning and navigation command generation | SDK transport and authority policy |
| `rosdeck` | App and App-facing authenticated gateway | Robot SDK, mission/docking implementations |

## Startup/readiness order

1. `omni_tf_manager` loads a robot model, validates static extrinsics and
   publishes `/omni/tf_manager/ready`.
2. `omni_slam` starts mapping/localization and publishes
   `/omni/slam/status` plus the dynamic map/odom/base chain.
3. `omni_robot_bridge` starts fail-closed, acquires the sole vendor SDK lock,
   starts safety supervision and publishes `RobotState`.
4. `omni_docking` and `omni_mission_manager` start but reject operations until
   robot/TF/localization readiness gates pass.
5. SCAN-Planner accepts goals only after TF readiness and localization.
6. The App-facing gateway starts last and exposes only typed, authenticated
   commands.

All processes may start under systemd in parallel, but their public readiness
must follow these gates. A process being alive is not equivalent to being
ready for motion.
