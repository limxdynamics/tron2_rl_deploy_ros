# TRON2 实机 MVP 联调记录模板

## 0. 环境准备

- `source ~/limx_ws/devel/setup.bash`
- 确认 `policy.onnx` 和 `encoder.onnx` 放在：
  - `tron2_controllers/config/SF_TRON2A/policy/`
  - `tron2_controllers/config/WF_TRON2A/policy/`
- 确认 `robot_ip` 与实机一致：本文档命令示例中的 `<robot-ip>`
  是占位符，运行前请替换为你自己机器人的实际 IP；
  `tron2_hw/launch/tron2_hw.launch` 的 `robot_ip` 参数没有默认值，
  必须显式传入（详见 `SECURITY.md`）。
- 确认遥控器按键映射与 `tron2_hw/config/joystick.yaml` 一致

## 1. 启动命令

### SF_TRON2A

```bash
roslaunch tron2_hw tron2_hw.launch robot_type:=SF_TRON2A robot_ip:=<robot-ip>
```

### WF_TRON2A

```bash
roslaunch tron2_hw tron2_hw.launch robot_type:=WF_TRON2A robot_ip:=<robot-ip>
```

## 2. 控制器与安全接口

- 启动行为：`tron2_hw.launch` 启动后会自动启动 `tron2_controller`
  （`controller_spawner` 以及 `tron2_hw_node` 在 1.2 秒后的启动请求），
  无需按键。控制器从 `STAND` 开始把关节移到默认姿态；关节指令从第一个
  控制周期起下发，只有机器人上报非 0 的 `calibration` 诊断时才暂停。
  因此 launch 之前必须先把机器人悬挂好，`roslaunch` 本身就是上电动作。

- 启动控制器（服务）：

```bash
rosservice call /tron2_hw/start_controller
```

- 停止控制器（服务）：

```bash
rosservice call /tron2_hw/stop_controller
```

- 急停（服务）：

```bash
rosservice call /tron2_hw/emergency_stop
```

- 清除急停（标定状态满足时）：

```bash
rosservice call /tron2_hw/reset_emergency_stop
```

- 遥控快捷键：
  - `L1 + Y`：控制器未运行时启动控制器；已运行时切换到 `WALK`
  - `L1 + X`：切换到 `IDLE`（不是急停；急停请调用 `/tron2_hw/emergency_stop`）

## 3. MVP 分阶段验收

### 阶段 A：单关节读写回环（离地/工装）

- [ ] `joint_states` 连续发布且关节名称顺序符合 `Tron2Cfg.init_state.joint_names`
- [ ] IMU 句柄可用（`limx_imu`），姿态/角速度/加速度数值正常
- [ ] 设置低增益后，单关节指令方向与幅值正确

### 阶段 B：全关节空载低增益

- [ ] 所有关节无异常振荡/打颤
- [ ] 停止控制器后命令能回落到阻尼状态
- [ ] 急停服务与遥控急停立即生效

### 阶段 C：站立与低速运动

- [ ] 30 秒内稳定进入控制状态
- [ ] `/cmd_vel` 指令可驱动机器人响应
- [ ] 低速直行、转向可控，无持续异常抖振

### 阶段 D：SF/WF 双机型复核

- [ ] SF_TRON2A 完成 A/B/C
- [ ] WF_TRON2A 完成 A/B/C
- [ ] 两机型均可重复启动与停控

## 4. 问题记录

| 日期 | 机型 | 现象 | 复现步骤 | 处理结果 |
|---|---|---|---|---|
| YYYY-MM-DD | SF/WF | - | - | - |
