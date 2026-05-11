---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [6]
first_seen: 2026-04-09
prerequisites: [URDF（统一机器人描述格式）, Gazebo 仿真环境]
related: [Gazebo 仿真环境, URDF（统一机器人描述格式）, Launch 文件]
contrasts: [Gazebo 仿真环境, ROS 节点]
---

# ros_control 控制框架

## 定义

ros_control 是 ROS 中用于**机器人硬件控制的通用框架**，为开发者提供了一套标准化的控制器接口（Controller Interface）、硬件资源接口（Hardware Resource Interface）和传输层抽象（Transmission Layer），使上层控制器代码与底层硬件解耦。

## 直觉理解

好比遥控车的**遥控器与底盘之间的标准化接口**——不管遥控车是四驱越野还是两驱跑车（不同的硬件），只要它们用同样的遥控器协议（标准化接口），同一套前后左右的控制指令就能驱动不同类型的车。ros_control 就是 ROS 世界的"标准化遥控协议"。

## 前置概念

- URDF：ros_control 读取 URDF 中的传动（Transmission）标签来配置关节与电机的关系
- Gazebo 仿真环境：ros_control 是连接 Gazebo 仿真与 ROS 控制指令的桥梁
- PID 控制器：ros_control 内置 PID 控制器用于位置/速度/力矩控制

## 推导到 / 关联到

- 控制器（Controller）：ros_control 支持多种控制器类型（Joint Position、Joint Velocity、Joint Trajectory 等）
- 硬件接口（Hardware Interface）：定义了下位机（电机驱动/单片机）与 ROS 之间的数据交换协议
- 传动（Transmission）：URDF 中 `<transmission>` 标签指定电机轴与关节之间的机械传动关系

## 易混概念

- ros_control vs ROS 节点：ros_control 是一个框架/基础设施；普通 ROS 节点通过 Topic 直接发布指令
- ros_control vs Gazebo 插件：两者都在 Gazebo 中配合使用——ros_control 提供控制器接口标准，Gazebo 插件实现物理仿真层面的驱动
- 硬件接口 vs 控制器接口：硬件接口 = 与底层硬件（电机编码器）的数据交换；控制器接口 = 与上层算法（路径规划）的数据交换

## 典型例子

ros_control 的五层架构（从硬件到 ROS）：
1. 硬件（电机/编码器）
2. 硬件接口（Hardware Interface）
3. 传输层（Transmission）
4. 控制器（Controller）
5. ROS API（Topic/Service）

在 Gazebo 仿真中使用：URDF 中配置 `<transmission>` 标签 → Gazebo 加载 ros_control 插件 → PID 控制器接收 `/cmd_vel` 话题驱动仿真机器人运动。
