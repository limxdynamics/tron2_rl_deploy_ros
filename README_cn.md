# 中文 | [English](README.md)

# tron2_rl_deploy_ros

LimX TRON2 强化学习部署 — 通过 ROS Noetic 将训练好的 ONNX 策略部署到真机或 Gazebo 仿真。

## 前提条件

- Ubuntu 20.04
- ROS Noetic
- ONNX Runtime 1.10.0

## 编译与运行

```bash
mkdir -p ~/limx_ws/src && cd ~/limx_ws/src
git clone https://github.com/limxdynamics/limxsdk-lowlevel.git
git clone https://github.com/limxdynamics/tron2-robot-description.git
git clone https://github.com/limxdynamics/tron2_rl_deploy_ros.git
cd ~/limx_ws
catkin_make install
source install/setup.bash

# 设置机器人型号
echo 'export ROBOT_TYPE=SF_TRON2A' >> ~/.bashrc && source ~/.bashrc

# 启动
roslaunch robot_hw tron2_hw_sim.launch
```
