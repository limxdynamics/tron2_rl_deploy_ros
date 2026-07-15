<!--
  SPDX-FileCopyrightText: 2024-2026 LimX Dynamics Technology Co., Ltd.
  SPDX-License-Identifier: Apache-2.0
-->

# TRON2 ROS 工作区说明（`~/limx_ws/src`）

本目录是 TRON2 在 ROS Noetic 下的源码空间（catkin `src`），包含仿真侧与控制侧两类包。

## 1. 目录结构

当前建议结构如下：

```text
~/limx_ws/src
├── CMakeLists.txt                 # catkin 顶层入口（通常为软链接）
├── robot-description              # 机器人模型描述包（独立放在 src 一级）
├── limxsdk-lowlevel               # 低层 SDK 包（独立放在 src 一级）
├── tron2-gazebo-ros               # 仿真相关包集合
│   ├── limxsdk-sim
│   └── tron2_gazebo
└── tron2-rl-deploy-ros            # 控制/部署相关包集合
    ├── onnxruntime_sdk
    ├── robot_common
    ├── tron2_controllers
    └── tron2_hw
```

> 注意：catkin 支持递归发现包，因此子目录分组不会影响编译，只要每个 ROS 包内有合法 `package.xml` 与 `CMakeLists.txt`。

## 2. 环境要求

- Ubuntu 20.04
- ROS Noetic（建议 `ros-noetic-desktop-full`）
- Gazebo 11（ROS Noetic 默认）
- 常见依赖（按需补齐）：

```bash
sudo apt-get update
sudo apt-get install -y \
  ros-noetic-gazebo-ros-pkgs \
  ros-noetic-gazebo-ros-control \
  ros-noetic-ros-control \
  ros-noetic-ros-controllers \
  ros-noetic-controller-manager \
  ros-noetic-joint-state-controller \
  ros-noetic-rqt-controller-manager \
  ros-noetic-robot-state-publisher \
  libeigen3-dev
```

## 3. 编译

在工作区根目录执行：

```bash
cd ~/limx_ws
source /opt/ros/noetic/setup.bash
catkin_make
```

编译成功后建议加载环境：

```bash
source ~/limx_ws/devel/setup.bash
```

## 4. 运行示例

### 4.1 启动仿真（完整部署）

```bash
cd ~/limx_ws
source /opt/ros/noetic/setup.bash
source devel/setup.bash
# 启动包含 Gazebo 场景、硬件节点和控制器的完整仿真
roslaunch tron2_hw tron2_hw_sim.launch robot_type:=SF_TRON2A
```

`robot_type` 可按你的配置切换（例如 `SF_TRON2A` / `WF_TRON2A`）。

### 4.2 仅启动控制器（仿真模式）

如果你已经手动启动了 Gazebo 场景，可以只启动硬件节点和控制器：

```bash
roslaunch tron2_hw tron2_controller_sim.launch robot_type:=SF_TRON2A
```

### 4.3 实物部署

```bash
roslaunch tron2_hw tron2_hw.launch robot_type:=SF_TRON2A robot_ip:=10.192.1.2
```

> Note on `10.192.1.2`: this is an internal-lab example address and
> appears in the same form in `tron2_hw/launch/tron2_hw.launch` and
> `tron2_hw/docs/bringup_mvp.md`. Its value is intentionally left
> unchanged in this scaffolding pass — legal / SRE sign-off is
> pending on whether to replace it with a documented placeholder
> (e.g. `<ROBOT_IP>`). See `THIRD_PARTY_NOTICES.md` §7.

## 5. 仿真与实机逻辑说明（重要）

结论：**核心控制链路逻辑一致**，都是：

- `tron2_hw_node`（`Tron2HW`） -> `RobotHWLoop` -> `controller_manager` -> `tron2_controller`

但两者有输入侧差异：

- 仿真（`tron2_hw_sim.launch`）默认还会发 `/cmd_vel` 和 `/tron2_controller/set_mode`。
- 实机（`tron2_hw.launch`）主要走 SDK 订阅的通道。

## 6. 控制参数位置

主要控制参数位于：

- `tron2-rl-deploy-ros/tron2_controllers/config/SF_TRON2A/params.yaml`
- `tron2-rl-deploy-ros/tron2_controllers/config/WF_TRON2A/params.yaml`

## 7. 效果展示

### 7.1 仿真部署

![SF Gazebo](doc/sfgazebo-ezgif.com-video-to-gif-converter.gif)
![WF Gazebo](doc/wfgazebo.gif)

### 7.2 实机部署

实机部署时请悬挂启动控制器

![Deploy](doc/deploy.jpg)

![SF](doc/sf.GIF)
![WF](doc/wf.GIF)

## 8. 常见问题

- 启动时报找不到包：
  - 确认已执行 `source /opt/ros/noetic/setup.bash`
  - 确认已执行 `source ~/limx_ws/devel/setup.bash`
- 修改目录后编译异常：
  - 在 `~/limx_ws` 下重新执行 `catkin_make`
- Gazebo 插件/控制器加载失败：
  - 先确认 `tron2_gazebo`、`tron2_hw`、`tron2_controllers`均已成功编译

## 6. License

[Apache 2.0](LICENSE)

---

## License & attribution

This project is licensed under the **Apache License, Version 2.0**
(January 2004) at the repository top level. See the [`LICENSE`](LICENSE)
file for the full text. SPDX identifier: `Apache-2.0`.

- [`NOTICE`](NOTICE) — required attribution notice, and the list of
  in-tree artifacts whose license status is **not** yet covered by
  the top-level Apache-2.0 grant.
- [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) — per-artifact
  provenance and every `⚠ TO CONFIRM` item.
- [`SECURITY.md`](SECURITY.md) — how to report a vulnerability, plus
  real-hardware safety notes.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — ROS C++ workflow, model /
  weight provenance checklist, binary artifact policy, DCO sign-off.
- [`CHANGELOG.md`](CHANGELOG.md) — release notes, including the
  "Pending owner sign-off" list that blocks the first public tag.

## Scope / not included

**Included** in this repository:

- Four ROS packages: `tron2_hw`, `tron2_controllers`, `robot_common`,
  `onnxruntime_sdk`.
- Launch files for Gazebo simulation and real-hardware bring-up.
- Configuration for the `SF_TRON2A` and `WF_TRON2A` variants.

**Included but pending owner clearance** (see `THIRD_PARTY_NOTICES.md`):

- `onnxruntime_sdk/lib/libonnxruntime.so` and `libonnxruntime.so.1.10.0`
  (bundled ONNX Runtime binary, ~14.3 MB — vendor clearance and
  SHA-256 recording pending).
- `tron2_controllers/config/SF_TRON2A/policy/{policy,encoder}.onnx`
  and `tron2_controllers/config/WF_TRON2A/policy/{policy,encoder}.onnx`
  (RL policy / encoder weights — model owner and robot-safety owner
  clearance pending).
- `tron2_hw/package.xml` and `tron2_controllers/package.xml` currently
  declare `<license>Proprietary</license>`; `robot_common/package.xml`
  and `onnxruntime_sdk/package.xml` currently declare
  `<license>TODO</license>` — legal / OSPO review pending.

**Excluded — by design:**

- No firmware or bootloader artifacts.
- No factory-calibration values (per-serial or global offsets).
- No motion / bag data (`*.bag`, `*.mcap`, HDF5 trajectory captures).
- No customer-specific or site-specific configuration.
- No auto-start behaviour on real hardware — bring-up always
  requires an explicit operator action; see `SECURITY.md`.

Real-hardware bring-up details, including start / stop / emergency-stop
service names, are documented in `tron2_hw/docs/bringup_mvp.md` and
are pending robot-safety owner sign-off before public release.

## Verification

The commands below mirror the CI workflow at
[`.github/workflows/ci.yml`](.github/workflows/ci.yml). Running them
locally before opening a PR saves review round-trips.

```bash
# 1. Build succeeds against the sibling workspace
cd ~/limx_ws
source /opt/ros/noetic/setup.bash
catkin_make

# 2. Every package.xml is well-formed
xmllint --noout $(find . -name 'package.xml')

# 3. Every unresolved <license> line still carries the ⚠ TO CONFIRM
#    annotation (CI fails if any silently loses the annotation).
find . -name 'package.xml' -exec grep -Hn '<license>' {} \;

# 4. No uncontrolled binaries / weights / bags. The output must exactly
#    match the pre-existing controlled set in THIRD_PARTY_NOTICES.md.
git ls-files | grep -iE '\.(onnx|pt|pth|ckpt|so(\.[0-9.]+)?|dll|dylib|lib|whl|bag|mcap)$'

# 5. Private-IP scan (only the documented example 10.192.1.2 is allowed).
grep -RIn --exclude-dir=.git -E \
  '\b(10\.[0-9]+\.[0-9]+\.[0-9]+|192\.168\.[0-9]+\.[0-9]+)\b' .
```

## Cite & support

If you use this deployment stack in academic or public work, please
cite the repository:

```
@misc{limx_tron2_rl_deploy_ros_2026,
  title  = {TRON2 RL deployment for ROS},
  author = {LimX Dynamics},
  year   = {2026},
  howpublished = {\url{https://github.com/limx-tron2/tron2-rl-deploy-ros}}
}
```

- **Bug reports / feature requests:** [GitHub Issues](https://github.com/limx-tron2/tron2-rl-deploy-ros/issues).
- **Questions / integration help:** [GitHub Discussions](https://github.com/limx-tron2/tron2-rl-deploy-ros/discussions).
- **Security or robot-safety reports:** email `contact@limxdynamics.com`
  with subject prefix `[tron2-rl-deploy-ros]`; see [`SECURITY.md`](SECURITY.md).
- **Company / commercial contact:** <https://www.limxdynamics.com>.
