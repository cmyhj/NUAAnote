---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [7]
first_seen: 2026-04-14
prerequisites: [move_base 导航框架, AMCL 自适应蒙特卡洛定位, GMapping 算法]
related: [move_base 导航框架, AMCL 自适应蒙特卡洛定位, GMapping 算法, SLAM（即时定位与地图构建）]
contrasts: [move_base 导航框架, SLAM（即时定位与地图构建）]
---

# ROS Navigation Stack

## 定义

ROS Navigation Stack（ROS 导航栈）是 ROS 中一套用于**移动机器人自主导航**的功能包集合，包含地图服务、定位、路径规划、运动控制、代价地图等模块。以 move_base 为核心节点，整合 AMCL（定位）、costmap_2d（代价地图）、global_planner（全局规划）和 local_planner（局部规划）等组件。

## 直觉理解

好比手机地图导航 App 的"全家桶"——导航栈不是单一功能，而是一整套系统：地图（Map） + 定位（我在哪）+ 全局路径规划（怎么去）+ 局部避障（躲开障碍物）+ 速度控制（以多快速度走）。每个组件可以单独替换升级，但整套系统协同工作。

## 前置概念

- move_base：导航栈的核心调度节点
- AMCL：提供定位能力
- 代价地图（Costmap）：在占用网格地图基础上叠加机器人半径的膨胀区域

## 推导到 / 关联到

- 导航栈的完整流程：GMapping 建图 → 保存地图 → 加载地图 + AMCL 定位 → move_base 导航
- recovery_behaviors：导航栈内置的卡住自救策略（旋转、后退、清理代价地图）
- map_server：提供地图的保存和加载功能

## 易混概念

- Navigation Stack vs move_base：move_base 是导航栈的"大脑"和调度中心；Navigation Stack 包含 move_base 在内的全部功能包集合
- Navigation Stack vs 导航算法：导航栈是一个集成框架，内部可替换不同的全局规划器（A*/Dijkstra）和局部规划器（DWA/TEB）
- 2D 导航栈 vs 3D 导航栈：ROS Navigation Stack 标准版解决 2D 平面移动机器人的导航；3D 导航（如无人机）需额外框架

## 典型例子

TurtleBot3 完整导航流程：
1. 建图：`roslaunch turtlebot3_slam turtlebot3_slam.launch`（GMapping）
2. 保存地图：`rosrun map_server map_saver -f ~/map`
3. 导航：`roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=~/map.yaml`
4. RViz 中指定目标点 → 导航栈自动完成全局路径规划 + 局部避障 + 到达目标
