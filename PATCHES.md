# braid branch — patches over upstream

Fork of [Slamtec/rplidar_sdk](https://github.com/Slamtec/rplidar_sdk).
This `braid` branch is upstream `master` (99478e5, "Bugfix: Get incorrect
sample delay offset in ultradense mode") plus three independent fixes,
one commit each. It is the source of truth for the prebuilt
`ofLibs_rplidar_macos.zip` consumed by braid's `braid-rplidar` addon
(pinned by commit in ofLibs `rplidar/chalet.yaml`).

## The patches (`git log master..braid`)

1. **Dense-capsule sync-bit cross-contamination** —
   `sdk/src/dataunpacker/unpacker/handler_capsules.{cpp,h}`:
   `_onScanNodeDenseCapsuleData` kept `lastNodeSyncBit` as a function-local
   static, shared by every `UnpackerHandler_DenseCapsuleNode` instance; with
   multiple lidars in one process, one device's sync transitions corrupted
   another's node flags. Hoisted to a per-instance `_lastNodeSyncBit` member.
2. **Compiler-predefined platform macros** —
   `sdk/src/hal/{event.h,locker.h,thread.cpp}`, `sdk/src/sdkcommon.h`:
   `_MACOS` → `__APPLE__` and (arch selection) `__GNUC__` → `__linux__`.
   Building no longer requires `-D_MACOS`.
3. **RPM motor-control payload** — `sdk/src/sl_lidar_driver.cpp`:
   the `MotorCtrlSupportRpm` case sent `sl_lidar_payload_motor_pwm_t` with
   `SL_LIDAR_CMD_HQ_MOTOR_SPEED_CTRL`, which expects
   `sl_lidar_payload_hq_spd_ctrl_t`; same first-field layout, wrong
   semantics. Now sends the RPM struct.

Each is a candidate for an upstream pull request on its own branch.

## Refreshing against upstream

```sh
git fetch https://github.com/Slamtec/rplidar_sdk master
git rebase FETCH_HEAD braid        # or cherry-pick the three commits
```

Then update the pinned commit in ofLibs `rplidar/chalet.yaml` and push
ofLibs so CI republishes the archive.

## History

Braid previously vendored this SDK as a patched source tree
(hand-assembled from `rplidar_ros/sdk`, which at the time led
`rplidar_sdk`; the two are identical as of 2026-07). That tree carried
the same fixes plus local debug leftovers, which were deliberately not
brought here. This fork replaced it 2026-07-17.
