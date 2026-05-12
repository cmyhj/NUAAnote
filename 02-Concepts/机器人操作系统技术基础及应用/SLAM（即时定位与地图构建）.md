---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [7]
first_seen: 2026-04-14
prerequisites: [TF（坐标变换）, ROS Topic（话题通信）]
related: [GMapping 算法, AMCL 自适应蒙特卡洛定位, move_base 导航框架]
contrasts: [结构光扫描建图]
---

# SLAM（即时定位与地图构建）

## 定义

SLAM（Simultaneous Localization and Mapping，即时定位与地图构建）是指机器人在未知环境中，一边移动一边利用自身搭载的传感器构建环境地图，同时利用该地图估计自身位置的技术。

## 直觉理解

想象你被蒙上眼睛放进一个陌生的房间里，需要一边摸墙走路一边在脑中画地图，同时还得知道自己当前在房间的哪个位置。你每走一步，墙的相对位置会帮你修正"我在哪"的猜测，而"我在哪"的信息又帮你更准确地画出墙的位置——两个问题相互依赖、交替求解。

## 前置概念

- TF（坐标变换）：SLAM 涉及 laser→base→odom→map 的多层坐标变换链
- ROS Topic（话题通信）：SLAM 节点通过 Topic 订阅激光雷达数据和 TF，发布地图
- 里程计（Odometry）：编码器估算的机器人位移，作为 SLAM 的初始位姿估计

## 推导到 / 关联到

- GMapping 算法：ROS 中最常用的 2D SLAM 实现，基于粒子滤波
- AMCL 自适应蒙特卡洛定位：已知地图后的机器人定位问题，SLAM 的后置步骤
- move_base 导航框架：SLAM 建图后的路径规划与导航
- 占用网格地图（Occupancy Grid Map）：SLAM 的输出形式

## 易混概念

- SLAM vs 定位：SLAM = 建图 + 定位同时进行；定位（如 AMCL）假设地图已知，只做位置估计
- SLAM vs 结构光扫描：SLAM 强调机器人在运动过程中实时计算；结构光扫描通常需要静态环境
- 2D SLAM vs 3D SLAM：2D SLAM 输出平面栅格地图（GMapping）；3D SLAM 输出三维点云地图（如 LOAM）

## 典型例子

- TurtleBot3 + GMapping：在 Gazebo 仿真环境中，机器人搭载激光雷达扫描房间，生成二维占用网格地图
- 扫地机器人：本质就是一个低成本 SLAM 系统，用激光雷达或视觉传感器实时建图并规划清扫路径
