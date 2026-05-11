---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [5]
first_seen: 2026-04-02
prerequisites: [ROS Topic（话题通信）, TF（坐标变换）]
related: [Gazebo 仿真环境, rqt 可视化工具]
contrasts: [Gazebo 仿真环境]
---

# RViz 可视化工具

## 定义

RViz（ROS Visualization）是 ROS 的 3D 可视化工具，用于实时显示机器人模型、传感器数据（激光扫描、点云、图像）、坐标变换（TF）、路径规划结果等，帮助开发者调试和可视化机器人系统运行状态。

## 直觉理解

好比机器人的"监控大屏"——机器人通过 ROS 话题发布的各种数据（位置、传感器读数、规划路径），都能在 RViz 中以 3D 形式实时呈现出来，让你一眼看清机器人"看到"了什么、"想"做什么。

## 前置概念

- ROS Topic（话题通信）：RViz 通过订阅指定话题获取可视化数据
- TF（坐标变换）：RViz 依赖 TF 确定各坐标系（激光雷达、基座、地图等）之间的相对位置关系
- URDF：RViz 可以加载 URDF 模型文件显示机器人 3D 外观

## 推导到 / 关联到

- rqt 可视化工具：RViz 用于 3D 可视化，rqt 用于 2D GUI 面板组合
- Gazebo 仿真环境：Gazebo 负责物理仿真，RViz 负责结果可视化，两者常配合使用
- rosbag：可录制数据后用 RViz 离线回放

## 易混概念

- RViz vs Gazebo：RViz 是可视化工具（只显示不仿真）；Gazebo 是物理仿真环境（包含动力学引擎）
- RViz vs rqt：RViz 专注 3D 空间可视化；rqt 提供 2D GUI 面板（话题监控、参数配置等）

## 典型例子

SLAM 仿真时同时运行三个窗口：
1. Gazebo：仿真机器人移动和激光雷达
2. RViz：显示激光点云（绿色）、占用网格地图（浅灰/深色）、机器人位姿
3. 键盘控制窗口：W/A/S/D/X 控制机器人移动
