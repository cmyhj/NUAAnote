---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [4]
first_seen: 2026-03-31
prerequisites: [ROS（机器人操作系统）, ROS Topic（话题通信）, ROS Service（服务通信）]
related: [catkin 工作空间, TF（坐标变换）]
contrasts: [Shell 脚本]
---

# Launch 文件

## 定义

Launch 文件是 ROS 中 `.launch` 后缀的 **XML 配置文件**，用于一次性启动多个 ROS 节点、自动启动 rosmaster、设置参数服务器参数和命名重映射等。

## 直觉理解

好比**一键启动脚本的升级版**——不用手动开好几个终端分别 `rosrun`，写一个 Launch 文件就能同时启动所有节点、配好参数、搞定重映射，还能嵌套复用其他 Launch 文件。

## 前置概念

- ROS（机器人操作系统）：Launch 文件是 ROS 常用组件之一
- Node（节点）：Launch 文件的核心任务就是启动多个节点
- 参数服务器（Parameter Server）：Launch 文件可通过 `<param>` 向参数服务器写入全局变量

## 推导到 / 关联到

- catkin 工作空间：Launch 文件通常放在功能包的 `launch/` 目录下
- TF（坐标变换）：Launch 文件中常用 `static_transform_publisher` 发布静态变换
- Gazebo 仿真环境：常通过 Launch 文件启动 Gazebo 并加载机器人模型

## 易混概念

- Shell 脚本（`.sh`）：Shell 脚本按顺序执行命令，无法感知 ROS 节点状态；Launch 文件作为 ROS 原生工具，能自动管理 rosmaster 启动、节点崩溃重启（`respawn`）、节点依赖（`required`）等 ROS 特有的生命周期
- `<param>` vs `<arg>`：`<param>` 写入 ROS 参数服务器（全局，节点代码可读）；`<arg>` 是 Launch 文件内部局部变量（节点代码不可读），用于文件级重构

## 典型例子

- 双乌龟同步运动：`<launch> <node pkg="turtlesim" type="turtlesim_node" name="sim"/> <node pkg="turtlesim" type="turtle_teleop_key" name="key" output="screen"/> </launch>` — 两龟订阅同一话题同步运动
- `<node>` 三要素：`pkg`（功能包）、`type`（可执行文件）、`name`（节点运行名）
- `<include>` 嵌套复用：`<include file="$(find pkg_name)/launch/file.launch" />`
