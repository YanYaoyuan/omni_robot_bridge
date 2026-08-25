# Unified robot bridge engineering TODO

## P0 — repository and interface blockers

- [ ] Implement `/omni/control/authority` (`ControlAuthority`) as the single
  typed façade over the existing lease state machine; retain and test the
  `/rosdeck/control_*` adapter during migration.
- [ ] Split legacy product composition from `scripts/build.sh`, `deploy.sh`
  and A/B packaging so the bridge, mission manager and Rosdeck gateway are
  independently versioned artifacts assembled by a separate deployment
  manifest.
- [ ] Add a machine-readable compatibility matrix pinning bridge,
  `omni_robot_interfaces`, `omni_tf_manager`, mission and docking
  versions/SHAs.
- [ ] Rename default node names to `omni_robot_bridge` and
  `omni_safety_supervisor` with a tested deprecation path; keep the ROS
  package/executable rename for a major release.
- [ ] Gate navigation/docking authority on `/omni/tf_manager/ready` and fresh
  localization state.
- [ ] Resolve the extracted-source license conflict and add the authoritative
  root `LICENSE`.

## P1 — production safety and portability

- [ ] Move the internal `/omni/cmd_vel/final` seam into an in-process adapter
  call or enforce publisher identity with SROS2 and launch audits.
- [ ] Replace mapping/posture/locomotion string topics with typed Omni
  services/actions; publish compatibility aliases during migration.
- [ ] Add robot-model schema validation for all topic/service names, network
  settings, limits, timeouts and adapter capabilities.
- [ ] Add hardware-in-the-loop tests for SDK disconnect, duplicate owner,
  stale command, E-stop, vendor process recovery and shutdown ordering.
- [ ] Define real-time budgets for arbitration-to-SDK latency and watchdog
  jitter; collect field metrics.
- [ ] Add lifecycle/readiness, systemd watchdog and structured diagnostic
  reason codes.

## P2 — release engineering

- [ ] Run x86_64 core CI plus Orin and S100 cross/native builds for every PR.
- [ ] Produce SBOM, provenance, signatures and reproducible artifacts per
  robot model; make SDK blobs explicit non-redistributable inputs.
- [ ] Move product-level OTA composition into a dedicated deployment repo and
  support independent component rollback with compatibility checks.
- [ ] Add SROS2 enclaves so only the arbiter can publish final motion and only
  authorized clients can request control.
