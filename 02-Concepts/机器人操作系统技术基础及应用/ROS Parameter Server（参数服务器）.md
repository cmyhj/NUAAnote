---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [2]
first_seen: 2026-03-10
prerequisites: [ROS（机器人操作系统）, ROS Topic（话题通信）, ROS Service（服务通信）]
related: [ROS Topic（话题通信）, ROS Service（服务通信）, Launch 文件]
contrasts: [ROS Topic（话题通信）, ROS Service（服务通信）]
---

# ROS Parameter Server（参数服务器）

## 定义

参数服务器（Parameter Server）是 ROS 中一种**全局共享的键-值存储系统**，运行在 ROS Master 中，用于在多个节点之间共享静态或半静态的配置数据（如参数、阈值、文件名等）。节点可通过 `rosparam` 命令行或 API 对参数进行增/删/改/查。

## 直觉理解

好比办公室里的**公共白板**——任何人都能在上面写信息（如"会议室温度设为 25℃"），其他人随时可以查看和使用这些信息，不需要事先约定通信方式。但白板上写的只是固定数据，不能像 Topic 一样传输实时数据流。

## 前置概念

- ROS（机器人操作系统）：参数服务器是 ROS 三大通信机制之一
- ROS Master：参数服务器运行在 Master 进程中
- 键-值存储：通过字符串键名（key）存取任意类型的数据（value）

## 推导到 / 关联到

- Launch 文件：Launch 文件中用 `<param>` 标签向参数服务器写入参数
- 动态参数配置（dynamic_reconfigure）：运行时动态修改参数服务器中的参数
- 命名空间：参数名受 ROS 命名机制影响，可通过命名空间组织参数

## 易混概念

- 参数服务器 vs Topic：Topic 是连续数据流（实时通信）；参数服务器是静态存储（读写即用）
- 参数服务器 vs Service：Service 需要编程定义接口并注册回调；参数服务器的读写是内置 API，无需额外定义
- `<param>` vs `<arg>`：`<param>` 写入参数服务器（全局可见）；`<arg>` 是 Launch 文件局部变量（仅文件内可见）

## 典型例子

- turtlesim 中的背景色：`rosservice call /clear` 配合 `rosparam set /turtlesim/background_r 255` 改变乌龟模拟器背景色
- 机器人参数：`/robot_description` 存储 URDF 模型字符串，由 `robot_state_publisher` 读取并发布 TF
