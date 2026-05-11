---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [1]
first_seen: 2026-03-03
prerequisites: [Linux 基础]
related: [ROS Topic（话题通信）, ROS Service（服务通信）, catkin 工作空间]
contrasts: [传统操作系统（Windows/Linux）]
---

# ROS（机器人操作系统）

## 定义

ROS（Robot Operating System）是一个适用于机器人的**开源元操作系统（Meta-OS）**，运行于 Linux 之上，提供硬件抽象、设备控制、消息传递和包管理等服务。

## 直觉理解

ROS 不是真正的操作系统（如 Windows/Linux），而是运行在操作系统与应用软件之间的**中间件/软件框架**，负责让机器人各个模块（传感器、执行器、算法）能相互通信、协同工作——好比一个城市的"交通系统"，让不同部门（节点）之间有序地收发信息。

## 前置概念

- Linux 基础：ROS 主要运行于 Ubuntu Linux，需掌握基本命令行操作
- 软件框架/中间件：介于底层 OS 与应用层之间的服务层

## 推导到 / 关联到

- ROS Topic（话题通信）：ROS 中最核心的异步通信机制
- ROS Service（服务通信）：ROS 中的同步请求-应答通信机制
- catkin 工作空间：ROS 包编译与组织的标准目录结构
- Launch 文件：一次性启动多节点的 XML 配置文件
- Gazebo 仿真环境：与 ROS 深度集成的 3D 物理仿真平台

## 易混概念

- 传统操作系统（Windows/Linux）：真正的操作系统，管理硬件资源；ROS 是运行于其上的元操作系统，依赖底层 OS 的进程调度与驱动
- ROS 2：第二代 ROS 架构，2017 年推出，解决了 ROS 1 基于 Master 的单点故障问题，架构全面重构，支持多平台

## 典型例子

- TurtleBot 海龟机器人：低成本 ROS 学习平台（~几千元），以树莓派做主控，运行 Ubuntu + ROS
- 百度 Apollo 无人驾驶计划：基于 ROS 开发的开放自动驾驶平台
- NASA 月球极地探测器：控制系统基于 ROS 构建
