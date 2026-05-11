---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [7]
first_seen: 2026-04-14
prerequisites: [AMCL 自适应蒙特卡洛定位, 占用网格地图, TF（坐标变换）]
related: [SLAM（即时定位与地图构建）, ROS Navigation Stack, GMapping 算法]
contrasts: [AMCL 自适应蒙特卡洛定位, GMapping 算法]
---

# move_base 导航框架

## 定义

move_base 是 ROS Navigation Stack 的核心节点，负责接收目标位置（goal），结合地图、定位信息和传感器数据，规划全局路径并发布线速度和角速度指令驱动机器人移动到目标点。它内部集成了**全局路径规划**（Global Planner）和**局部路径规划**（Local Planner）两层架构。

## 直觉理解

好比你在手机上用地图导航去一个陌生商场——地图 App（全局规划器）先规划出一条从当前位置到商场的路线（A→B→C→D），但在实际驾驶中遇到修路堵车时，导航会实时调整局部路线（局部规划器），绕开障碍最终到达目的地。move_base 就是机器人的"自动驾驶导航系统"。

## 前置概念

- AMCL：为 move_base 提供当前机器人在地图中的精确位姿
- 占用网格地图：move_base 的全局路径规划基于已知栅格地图
- 代价地图（Costmap）：在原始地图上叠加障碍物膨胀区域，防止机器人碰撞

## 推导到 / 关联到

- ROS Navigation Stack：move_base 是导航栈的核心调度节点
- 全局规划器（Global Planner）：通常用 Dijkstra / A* 算法在全局地图上规划路径
- 局部规划器（Local Planner）：通常用 DWA（Dynamic Window Approach）实时避障
- 恢复行为（Recovery Behaviors）：卡住时的自救策略（旋转、后退、重新规划）

## 易混概念

- move_base vs AMCL：AMCL 回答"我在哪"（定位）；move_base 回答"我怎么去那里"（导航）
- 全局路径 vs 局部路径：全局路径 = 基于已知地图的长期路线规划（计算慢、更新频率低）；局部路径 = 基于实时传感器数据的短期避障（计算快、更新频率高）
- 代价地图 vs 占用网格地图：占用网格地图是静态的 SLAM 输出；代价地图是 move_base 在占用网格上叠加机器人半径膨胀后的动态导航用地图

## 典型例子

ROS 中运行 move_base 导航：
```
roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=/path/to/map.yaml
```
在 RViz 中用 "2D Nav Goal" 按钮点击目标点 → move_base 自动规划路径并驱动机器人沿路行驶、避开障碍物、最终到达目标。
