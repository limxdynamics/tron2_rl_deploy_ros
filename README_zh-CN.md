<!--
  SPDX-FileCopyrightText: 2024-2026 LimX Dynamics Technology Co., Ltd.
  SPDX-License-Identifier: Apache-2.0
-->

<p align="center">
  <img src="doc/banner.png" alt="tron2_rl_deploy_ros：本仓库可部署 RL 策略的足底与轮足 TRON2A" width="100%">
</p>

<p align="center">
  <a href="README.md">English</a> | <b>中文</b>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-Apache--2.0-2f6feb" alt="License: Apache-2.0"></a>
  <img src="https://img.shields.io/badge/ROS-Noetic-2f6feb" alt="ROS Noetic">
  <img src="https://img.shields.io/badge/ONNX%20Runtime-1.10-2f6feb" alt="ONNX Runtime 1.10">
  <img src="https://img.shields.io/badge/Ubuntu-20.04-2f6feb" alt="Ubuntu 20.04">
</p>

# tron2_rl_deploy_ros

> **发布渠道**。本仓库的主发布地址为
> [`github.com/limxdynamics/tron2_rl_deploy_ros`](https://github.com/limxdynamics/tron2_rl_deploy_ros)。
> LimX 内部 GitLab 上的仓库仅为镜像，Issue 与 Pull Request
> 请统一提交至 GitHub。

在 ROS Noetic 下把强化学习运动策略部署到 TRON2A 双足机器人上，支持足底（`SF_TRON2A`）和轮足（`WF_TRON2A`）。`tron2_hw` 把 LimX SDK 封装成 `ros_control` 硬件接口，`tron2_controller` 用 ONNX Runtime 运行 ONNX 策略和历史编码器。同一套 launch 文件既能驱动真机，也能配合 [tron2_gazebo_ros](https://github.com/limxdynamics/tron2_gazebo_ros) 驱动 Gazebo Classic 11。

## 亮点

- **仿真和真机同一条控制链路。** `tron2_hw_node` -> `RobotHWLoop` -> `controller_manager` -> `tron2_controller`，只换 launch 文件。
- **自带策略。** 两种机型的策略和编码器 ONNX 文件放在 `tron2_controllers/config/<robot_type>/policy/`，`onnxruntime_sdk` 内置 ONNX Runtime 1.10。
- **标准 ROS 接口。** 用 `/cmd_vel` 和 `/tron2_controller/set_mode` 控制，或通过 SDK 用机器人遥控器控制。
- **安全接口。** `tron2_hw` 提供启动、停止、急停服务，`tron2_hw/config/tron2_hw.yaml` 中有逐关节的位置步长、速度和力矩限制。

## 支持的机型

| | 机型 | 说明 | 关节数 / 策略动作数 |
|:-:|---|---|:-:|
| <img src="doc/model_sf.png" width="300"> | `SF_TRON2A` | 足底双足 | 10 / 10 |
| <img src="doc/model_wf.png" width="300"> | `WF_TRON2A` | 轮足双足 | 10 / 10 |

## 工作原理

<p align="center">
  <img src="doc/sim2real_zh.png" alt="同一套控制器：经 Tron2HWSim 插件驱动 Gazebo 中的 TRON2，经 LimX SDK 驱动真机" width="100%">
</p>

## 快速开始

Ubuntu 20.04，已安装 ROS Noetic 和[第 2 节](#2-环境要求)中的依赖：

```bash
mkdir -p ~/limx_ws/src && cd ~/limx_ws/src
git clone https://github.com/limxdynamics/tron2_rl_deploy_ros.git
git clone https://github.com/limxdynamics/tron2_gazebo_ros.git
git clone https://github.com/limxdynamics/tron2-robot-description.git
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
git -C limxsdk-lowlevel checkout 64e3b99   # 4.0.0

cd ~/limx_ws
source /opt/ros/noetic/setup.bash
catkin_make
source devel/setup.bash
roslaunch tron2_hw tron2_hw_sim.launch robot_type:=SF_TRON2A use_joystick:=false
```

然后在另一个终端切换到行走（见 [4.4 节](#44-手柄与话题)）。真机上把最后一行换成 `roslaunch tron2_hw tron2_hw.launch robot_type:=SF_TRON2A robot_ip:=<robot-ip>`，并先阅读 4.4 节和第 5 节。

---

## 1. 目录结构

当前建议结构如下：

```text
~/limx_ws/src
├── CMakeLists.txt                 # catkin 顶层入口（通常为软链接）
├── tron2-robot-description        # 机器人模型描述包 `robot_description`（独立放在 src 一级）
├── limxsdk-lowlevel               # 低层 SDK 包（独立放在 src 一级）
├── tron2_gazebo_ros               # 仿真相关包集合
│   ├── limxsdk-sim
│   └── tron2_gazebo
└── tron2_rl_deploy_ros            # 控制/部署相关包集合（本仓库）
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

先把四个仓库克隆到工作空间：

```bash
mkdir -p ~/limx_ws/src && cd ~/limx_ws/src
git clone https://github.com/limxdynamics/tron2_rl_deploy_ros.git
git clone https://github.com/limxdynamics/tron2_gazebo_ros.git
git clone https://github.com/limxdynamics/tron2-robot-description.git
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
git -C limxsdk-lowlevel checkout 64e3b99   # 4.0.0
```

目前请使用 limxsdk-lowlevel 4.0.0（`64e3b99`）：从 v4.1.1 起它的 `tron2.h` 引用了未随 SDK 发布的 `mros/hand_msgs/*.h`，`tron2_hw` 无法编译。

然后在工作区根目录执行：

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
roslaunch tron2_hw tron2_hw.launch robot_type:=SF_TRON2A robot_ip:=<robot-ip>
```

**关于 `<robot-ip>` 的说明**。文档命令示例中的 `<robot-ip>`
是占位符——运行前请替换为你自己机器人的实际 IP。源码侧的
默认值出现在 `tron2_hw/src/Tron2HW.cpp`、
`tron2_hw/src/tron2_hw_node.cpp`，以及 launch 参数默认值
`tron2_hw/launch/tron2_hw.launch` 中，保留字面量
`10.192.1.2` 作为描述典型实机使用场景的文档示例。这仅是
文档取值，不对应 LimX 任何生产网络中的地址。私网 IP 的
处理策略详见 [`SECURITY.md`](SECURITY.md) 与
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) §7。

### 4.4 手柄与话题

真机上 `tron2_hw` 通过 SDK（`SensorJoy`）读取遥控器；仿真中同样的键位来自 `tron2_gazebo` 的手柄节点（USB 手柄），也可以自己发布话题。

| 输入 | 作用 |
|---|---|
| L1 + Y | 启动 `tron2_controller`；已在运行时切换到 `WALK` |
| L1 + X | 切换到 `IDLE` |
| 左摇杆上 / 下 | `/cmd_vel` linear x |
| 左摇杆左 / 右 | `/cmd_vel` linear y |
| 右摇杆左 / 右 | `/cmd_vel` angular z |

摇杆数值按 `tron2_controllers/config/<robot_type>/params.yaml` 中的 `commands/max` 缩放（`SF_TRON2A` 为 1.0 m/s、1.0 m/s、1.0 rad/s）。没有手柄时：

```bash
rostopic pub -1 /tron2_controller/set_mode std_msgs/String "data: 'WALK'"
rostopic pub -r 10 /cmd_vel geometry_msgs/Twist "{linear: {x: 0.3}}"
```

`tron2_hw` 还提供服务 `/tron2_hw/start_controller`、`/tron2_hw/stop_controller`、`/tron2_hw/emergency_stop` 和 `/tron2_hw/reset_emergency_stop`（见 `tron2_hw/docs/bringup_mvp.md`）。

**启动行为。** 按当前代码，`tron2_hw.launch` 启动后无需按键就会启动 `tron2_controller`：`controller_spawner` 节点会启动它，`tron2_hw_node` 也会在控制循环就绪 1.2 秒后请求启动。控制器从 `STAND` 开始，把所有关节移到默认姿态；机器人上报标定状态 `0` 后指令即下发到关节。launch 之前请先把机器人悬挂好。

## 5. 仿真与实机逻辑说明（重要）

结论：**核心控制链路逻辑一致**，都是：

- `tron2_hw_node`（`Tron2HW`） -> `RobotHWLoop` -> `controller_manager` -> `tron2_controller`

但两者有输入侧差异：

- 仿真（`tron2_hw_sim.launch`）默认还会发 `/cmd_vel` 和 `/tron2_controller/set_mode`。
- 实机（`tron2_hw.launch`）主要走 SDK 订阅的通道。

实机 bring-up 必须始终是操作员显式且受监督的动作。启动控制器时请保持机器人悬挂，并在启用 `controller_manager` 前确认急停位置。详见 `tron2_hw/docs/bringup_mvp.md` 与 `SECURITY.md`。

## 6. 控制参数位置

主要控制参数位于：

- `tron2_rl_deploy_ros/tron2_controllers/config/SF_TRON2A/params.yaml`
- `tron2_rl_deploy_ros/tron2_controllers/config/WF_TRON2A/params.yaml`

## 7. 效果展示

### 7.1 仿真部署

| SF_TRON2A | WF_TRON2A |
| :---: | :---: |
| <img src="doc/sfgazebo-ezgif.com-video-to-gif-converter.gif" alt="SF Gazebo" width="400"> | <img src="doc/wfgazebo.gif" alt="WF Gazebo" width="400"> |

### 7.2 实机部署

实机部署时请悬挂启动控制器

<img src="doc/deploy.jpg" alt="Deploy" width="360">

| SF_TRON2A | WF_TRON2A |
| :---: | :---: |
| <img src="doc/wf.GIF" alt="SF" width="300"> | <img src="doc/sf.GIF" alt="WF" width="300"> |

## 8. 常见问题

- 启动时报找不到包：
  - 确认已执行 `source /opt/ros/noetic/setup.bash`
  - 确认已执行 `source ~/limx_ws/devel/setup.bash`
- 修改目录后编译异常：
  - 在 `~/limx_ws` 下重新执行 `catkin_make`
- Gazebo 插件/控制器加载失败：
  - 先确认 `tron2_gazebo`、`tron2_hw`、`tron2_controllers`均已成功编译
- 编译 `tron2.h` 报 `mros/hand_msgs/HandCmd.h: No such file or directory`：
  - 把 limxsdk-lowlevel 切到 4.0.0（`64e3b99`），见第 3 节
- 编译 `tron2_hw` 报 `GripperCmd does not name a type`：
  - 旧版 SDK 头文件盖住了 limxsdk-lowlevel 的头文件。更新 `tron2_gazebo_ros`（其 `limxsdk-sim` 已不再把头文件拷进 `devel/include`），然后 `rm -rf build devel` 重新编译


## 验证

下列命令与 CI 工作流
[`.github/workflows/ci.yml`](.github/workflows/ci.yml)一致。
在提交 PR 前本地执行可减少 review 往返。

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

# 5. Private-IP scan (only the documented example 10.192.1.2 is allowed;
#    Markdown/YAML examples should use the <robot-ip> substitution token).
grep -RIn --exclude-dir=.git -E \
  '\b(10\.[0-9]+\.[0-9]+\.[0-9]+|192\.168\.[0-9]+\.[0-9]+)\b' .
```


本次整改的 CI（在 [tron2_gazebo_ros](https://github.com/limxdynamics/tron2_gazebo_ros/actions) 中）在 `ros:noetic` 容器里编译了完整工作空间，包括 `tron2_hw`、`tron2_controllers`、`robot_common` 和 `onnxruntime_sdk`。其中无界面 Gazebo 冒烟测试在实时因子约 0.4 的条件下，两台机器人都在 2 秒悬空结束后倒地；尚未在 Ubuntu 20.04 桌面环境复核。真机路径未运行。

---

## 引用与支持

如果你在学术工作或公开材料中使用本部署栈，请引用本仓库：

```
@misc{limx_tron2_rl_deploy_ros_2026,
  title  = {TRON2 RL deployment for ROS},
  author = {LimX Dynamics},
  year   = {2026},
  howpublished = {\url{https://github.com/limxdynamics/tron2_rl_deploy_ros}}
}
```

- **缺陷报告 / 功能请求**：[GitHub Issues](https://github.com/limxdynamics/tron2_rl_deploy_ros/issues)。
- **提问 / 集成协助**：先查看[常见问题](#8-常见问题)，再提交 [GitHub Issue](https://github.com/limxdynamics/tron2_rl_deploy_ros/issues)。
- **安全或机器人安全问题上报**：邮件发送至 `contact@limxdynamics.com`，
  主题前缀请使用 `[tron2-rl-deploy-ros]`；详见 [`SECURITY.md`](SECURITY.md)。
- **公司 / 商务联系**：<https://www.limxdynamics.com>。

---

## 许可与归属

本项目在仓库顶层采用 **Apache License, Version 2.0**
（2004 年 1 月）授权。完整条款请参见 [`LICENSE`](LICENSE) 文件。
SPDX 标识符：`Apache-2.0`。

- [`NOTICE`](NOTICE) — 必需的归属声明，同时列出仓库内尚未
  被顶层 Apache-2.0 授权覆盖的工件。
- [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) — 各工件的
  来源信息与全部 `⚠ TO CONFIRM` 条目。
- [`SECURITY.md`](SECURITY.md) — 漏洞上报流程及实机安全须知。
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — ROS C++ 工作流、模型/
  权重来源检查清单、二进制工件策略、DCO 签署说明。
- [`CHANGELOG.md`](CHANGELOG.md) — 发行说明，包含阻塞首个
  公开 tag 的 “Pending owner sign-off” 列表。

## 适用范围与除外

**本仓库包含**：

- 四个 ROS 包：`tron2_hw`、`tron2_controllers`、`robot_common`、
  `onnxruntime_sdk`。
- Gazebo 仿真与实机 bring-up 的 launch 文件。
- `SF_TRON2A` 与 `WF_TRON2A` 两种配置。

**已包含但仍在等待权利人确认**（详见 `THIRD_PARTY_NOTICES.md`）：

- `onnxruntime_sdk/lib/libonnxruntime.so` 与 `libonnxruntime.so.1.10.0`
  （随附的 ONNX Runtime 二进制，约 14.3 MB — 供应商授权与
  SHA-256 登记待办）。
- `tron2_controllers/config/SF_TRON2A/policy/{policy,encoder}.onnx`
  与 `tron2_controllers/config/WF_TRON2A/policy/{policy,encoder}.onnx`
  （RL 策略 / 编码器权重 — 模型权利人与机器人安全负责人的
  确认待办）。
- 四个 `package.xml`（`tron2_hw`、`tron2_controllers`、`robot_common`、
  `onnxruntime_sdk`）均已声明 `<license>Apache-2.0</license>`。

**按设计排除的内容**：

- 不包含固件或 bootloader 工件。
- 不包含出厂标定值（单机序列号或全局偏移）。
- 不包含运动数据 / bag 数据（`*.bag`、`*.mcap`、HDF5 轨迹记录）。
- 不包含客户或站点定制的配置。
- 实机上没有自动启动行为 — bring-up 始终需要操作员的
  显式动作，详见 `SECURITY.md`。

实机 bring-up 细节（含启动 / 停止 / 急停服务名）记录在
`tron2_hw/docs/bringup_mvp.md`，需机器人安全负责人签署确认后
方可公开发布。
