---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [6]
first_seen: 2026-03-17
prerequisites: [ROS（机器人操作系统）]
related: [URDF（统一机器人描述格式）, Launch 文件, TF（坐标变换）]
contrasts: [RViz, turtlesim]
---

# Gazebo 仿真环境

## 定义

Gazebo 是一个基于 ODE 物理引擎的 **3D 机器人仿真环境**，与 ROS 深度集成，支持物理属性模拟（质量、惯性、碰撞）、传感器仿真（激光雷达、摄像头）和 ROS 控制器插件。

## 直觉理解

好比**机器人版的"模拟城市"**——在真实机器人上场之前，先在三维虚拟世界里摆好模型、调好参数、跑通算法，避免直接在真机上撞坏东西。有重力、有摩擦、有碰撞检测，跟现实物理世界行为一致。

## 前置概念

- ROS（机器人操作系统）：Gazebo 是 ROS 体系中的仿真工具
- URDF（统一机器人描述格式）：Gazebo 中的机器人模型通常由 URDF/XACRO 定义，并需额外添加 `<gazebo>` 标签
- 物理引擎：模拟重力、摩擦、碰撞等物理交互

## 推导到 / 关联到

- URDF（统一机器人描述格式）：URDF 在 Gazebo 中需补充碰撞（`<collision>`）、惯性（`<inertial>`）和控制器插件
- Launch 文件：常通过 Launch 文件启动 Gazebo 并加载机器人模型到仿真世界
- SLAM 仿真：在 Gazebo 中搭建地图 → 运行 GMapping 算法 → move_base 自主导航
- ros_control：五层架构，Gazebo 通过插件配置将 URDF 关节与实际控制器（如 PID 位置控制）关联

## 易混概念

- RViz：RViz 是**数据可视化工具**（显示传感器数据、TF 坐标系、机器人状态），不包含物理引擎，不模拟运动；Gazebo 是**物理仿真环境**，模拟真实的物理运动和传感器数据输出
- turtlesim：turtlesim 是 ROS 的**二维教学仿真器**，界面简单（平面乌龟图标），不包含物理引擎和 3D 渲染，仅用于 ROS 通信机制教学；Gazebo 是**三维物理仿真**，可模拟复杂场景和真实物理交互

## 典型例子

- TurtleBot3 burger 仿真实操：在 Gazebo 中启动 TurtleBot3（迷宫环境/3D 家居环境），键盘 WASD 控制机器人在三维场景中运动
- Building Editor：Gazebo 内置工具，通过二维绘制墙/门窗自动生成三维场景，用于 SLAM 导航地图构建
- 在线模型库：Insert 面板从在线模型库下载救护车（~100 MB）、公寓楼（~700 MB）等装配复杂场景
