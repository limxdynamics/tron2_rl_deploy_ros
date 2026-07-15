# Third-Party Notices

`tron2-rl-deploy-ros` (TRON2 RL deployment for ROS) is distributed under
the Apache License 2.0 at the repository top level (see [`LICENSE`](LICENSE)
and [`NOTICE`](NOTICE)).

However, several in-tree artifacts and ROS packages are **not** yet
cleared under that license. This file lists every such artifact, its
provenance status, and the owner action required before a public
release.

> **Status:** items marked `⚠ TO CONFIRM` are pending sign-off from
> the responsible legal / hardware / model / SDK / safety owner. Do
> not cut a public release while any `⚠ TO CONFIRM` entry remains.

---

## 1. ROS packages in this repository — resolved (2026-07-14)

All four ROS packages now declare `<license>Apache-2.0</license>`,
consistent with the top-level [`LICENSE`](LICENSE).

| ROS package | Path | `<license>` | Status |
|-------------|------|-------------|--------|
| `tron2_hw`          | `tron2_hw/package.xml`          | `Apache-2.0` | ✅ resolved |
| `tron2_controllers` | `tron2_controllers/package.xml` | `Apache-2.0` | ✅ resolved |
| `robot_common`      | `robot_common/package.xml`      | `Apache-2.0` | ✅ resolved |
| `onnxruntime_sdk`   | `onnxruntime_sdk/package.xml`   | `Apache-2.0` | ✅ resolved — covers the LimX-authored CMake wrapper; the bundled ONNX Runtime binary in §2 keeps its own `⚠ TO CONFIRM` row for provenance / checksum / redistribution notice. |

The `⚠ TO CONFIRM` inline comments in each `package.xml` have been
removed. The CI guard remains: any future `<license>TODO` or
`<license>Proprietary` value regresses the sign-off and fails CI unless
paired with a fresh `⚠ TO CONFIRM` annotation.

---

## 2. Bundled ONNX Runtime binary

| Path | Size | Purpose |
|------|------|---------|
| `onnxruntime_sdk/lib/libonnxruntime.so`        | ~14.3 MB | Runtime linked by `tron2_controllers` (see `tron2_controllers/CMakeLists.txt`). |
| `onnxruntime_sdk/lib/libonnxruntime.so.1.10.0` | ~14.3 MB | Versioned SONAME target (identical bytes). |
| `onnxruntime_sdk/include/*`                    | headers  | Public C / C++ headers for the same version. |

- **Upstream project:** Microsoft ONNX Runtime — <https://github.com/microsoft/onnxruntime>
- **Upstream license:** MIT (see the ONNX Runtime `LICENSE`).
- **Bundled version:** filename suggests **1.10.0**. ⚠ TO CONFIRM by
  SDK owner: exact upstream tag / commit, build flags (CPU / CUDA /
  TensorRT / …), target platform (`linux/x86_64` vs `linux/aarch64`),
  and SHA-256 of the two `.so` files.
- **Redistribution clearance:** ⚠ TO CONFIRM. MIT permits
  redistribution with attribution, but this repo does **not** yet
  ship the ONNX Runtime `LICENSE` and `ThirdPartyNotices.txt` next to
  the binary as MIT requires. Before release, either:
  - **(A)** vendor the upstream `LICENSE` / `ThirdPartyNotices.txt`
    into `onnxruntime_sdk/`, record the SHA-256, and cite the exact
    upstream tag; **or**
  - **(B)** remove the binary and headers, and require downstream
    users to install `onnxruntime` from upstream releases (documented
    in `README.md`).

Owner action: **SDK lead + legal / OSPO**.

---

## 3. RL policy / encoder ONNX weights

Four ONNX files are checked into the tree:

| Path | Size | Kind |
|------|------|------|
| `tron2_controllers/config/SF_TRON2A/policy/policy.onnx`  | ~791 KB | RL policy weights, SF variant. |
| `tron2_controllers/config/SF_TRON2A/policy/encoder.onnx` | ~589 KB | Observation encoder weights, SF variant. |
| `tron2_controllers/config/WF_TRON2A/policy/policy.onnx`  | ~770 KB | RL policy weights, WF variant. |
| `tron2_controllers/config/WF_TRON2A/policy/encoder.onnx` | ~503 KB | Observation encoder weights, WF variant. |

For **each** of the four files, the model owner must resolve the
following before a public release:

- ⚠ TO CONFIRM — training pipeline, commit hash / experiment ID.
- ⚠ TO CONFIRM — training data provenance and licensing (was any
  third-party motion capture, imitation data, or asset used?).
- ⚠ TO CONFIRM — public-release approval (may LimX distribute these
  weights under Apache-2.0, under a separate model licence, or not
  at all?).
- ⚠ TO CONFIRM — model card: input / output tensor shapes, expected
  observation frame, action bounds, safety envelope, and known
  failure modes.
- ⚠ TO CONFIRM — real-hardware safety review: the same policy is
  loaded on the physical robot by `tron2_hw.launch`; the robot-safety
  owner must sign that the shipped weights are the ones cleared for
  public bring-up.

If any of the four cannot be cleared, remove the file (and add its
path to `.gitignore`) before release. Do **not** silently swap in
different weights — record the substitution here.

Owner action: **model lead + robot-safety lead + legal**.

---

## 4. External repository dependencies (not vendored)

`tron2-rl-deploy-ros` depends on the following LimX repositories at
build time. They are **not** vendored here; the workspace layout in
`README.md` expects sibling checkouts.

| Repository | Role | License status |
|------------|------|----------------|
| `robot-description`  | URDF / xacro / MuJoCo / STL / USD for TRON2A. Referenced via `<depend>robot_description</depend>` in `tron2_controllers/package.xml`. | Apache-2.0 (see the sibling repo's `LICENSE` and `NOTICE`). ⚠ TO CONFIRM per-asset mesh / USD provenance is tracked in that repo's `THIRD_PARTY_NOTICES.md`. |
| `limxsdk-lowlevel`   | Low-level SDK used by `tron2_hw`. Referenced via `<depend>limxsdk_lowlevel</depend>` in `tron2_hw/package.xml`. | ⚠ TO CONFIRM — SDK owner must publish the license before this repo can be built from a public clone. |

---

## 5. ROS runtime dependencies (not redistributed)

The following ROS / third-party packages are declared as `<depend>` in
this repo's `package.xml` files and are **runtime dependencies only** —
they are neither vendored nor shipped here.

| Dependency | Where declared | License | Where obtained |
|------------|----------------|---------|----------------|
| `roscpp`, `std_msgs`, `std_srvs`, `geometry_msgs`, `urdf`, `realtime_tools` | `tron2_hw`, `tron2_controllers`, `robot_common` | BSD-3-Clause | ROS Noetic distribution |
| `controller_manager`, `controller_manager_msgs`, `controller_interface`, `hardware_interface` | `tron2_hw`, `tron2_controllers`, `robot_common` | BSD-3-Clause | `ros-noetic-ros-control` / `ros-noetic-ros-controllers` |
| `eigen`     | `tron2_controllers` (`<build_depend>`) | MPL-2.0 / BSD-3-Clause | `libeigen3-dev` |
| `rclcpp`    | `onnxruntime_sdk` (ROS 2 branch) | Apache-2.0 | ROS 2 distribution |
| ONNX Runtime (linked, C++) | `tron2_controllers/CMakeLists.txt` links `${ONNXRUNTIME_SDK_LIB_FILES}` | MIT | See §2 above |

Downstream users must obtain and license these independently.

---

## 6. Documentation media

| Path | Kind | Provenance | License |
|------|------|------------|---------|
| `doc/deploy.jpg`                                     | Photo of a real-hardware bring-up (robot on a hoist) | ⚠ TO CONFIRM — must not disclose office interior, individuals, or non-public products. | ⚠ TO CONFIRM |
| `doc/sf.GIF`, `doc/wf.GIF`                           | Real-hardware locomotion capture           | ⚠ TO CONFIRM | ⚠ TO CONFIRM |
| `doc/sfgazebo-ezgif.com-video-to-gif-converter.gif`  | Screen-recorded Gazebo simulation          | ⚠ TO CONFIRM | ⚠ TO CONFIRM |
| `doc/wfgazebo.gif`                                   | Screen-recorded Gazebo simulation          | ⚠ TO CONFIRM | ⚠ TO CONFIRM |

Before release, run:

```bash
exiftool doc/* | grep -iE '(gps|serial|make|model|software|author|artist|copyright)'
```

and strip anything that discloses office locations, camera serials, or
individual contributors' names, unless intentionally kept:

```bash
exiftool -all= doc/*
```

Also visually re-review every frame of the GIFs for people, whiteboards,
badges, and screens containing internal URLs / hostnames.

---

## 7. Sensitive on-robot configuration referenced in-tree

Not shipped as separate files, but present in text and therefore
subject to legal / safety review before publication:

- Private-network robot IP `10.192.1.2` in `README.md`,
  `tron2_hw/launch/tron2_hw.launch`, `tron2_hw/docs/bringup_mvp.md`.
  ⚠ TO CONFIRM — is this an internal-only example that should be
  replaced with a documented placeholder (e.g. `<ROBOT_IP>`) before
  release, or does it correspond to a customer-facing default?
- Emergency-stop / start / stop service names and topics documented
  in `tron2_hw/docs/bringup_mvp.md`. ⚠ TO CONFIRM by robot-safety
  owner: publishing these interface names is safe (they are already
  visible over the ROS graph on a bring-up network).

---

## 8. Update procedure

Whenever a package, binary, model, image, or upstream reference is
added or changed:

1. Update the corresponding section of this file.
2. Re-run the EXIF strip (§6) on any changed doc media.
3. If the change touches an `⚠ TO CONFIRM` row, block the merge on
   written sign-off from the responsible owner (see `.github/CODEOWNERS`).
4. Record any SHA-256 change of `libonnxruntime.so*` and re-run the
   SDK owner's clearance.
