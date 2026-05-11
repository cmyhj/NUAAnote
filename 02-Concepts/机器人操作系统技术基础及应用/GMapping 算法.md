---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [7]
first_seen: 2026-04-14
prerequisites: [SLAM（即时定位与地图构建）, TF（坐标变换）]
related: [AMCL 自适应蒙特卡洛定位, ROS Navigation Stack]
contrasts: [卡尔曼滤波 SLAM, Hector SLAM]
---

# GMapping 算法

## 定义

GMapping 是一种基于 **粒子滤波（Particle Filter / Rao-Blackwellized Particle Filter）** 的 2D SLAM 算法，已集成于 ROS 的 `gmapping` 功能包。它的核心思想是用一组带权重的粒子代表机器人可能的位姿，每个粒子维护一张独立的栅格地图，通过观测更新权重后重采样。

## 直觉理解

想象有 100 个你被放进同一个陌生房间，每个人一边走动一边猜测自己在哪并画自己的地图。有些人猜得准（他们的地图和别人重合度高），就获得更高"可信度"；猜得差的就被淘汰。下一代人基于上一代最可信的几位来继续探索——这是粒子滤波的"优胜劣汰"思想。

## 前置概念

- SLAM：GMapping 是 SLAM 问题的具体实现算法
- 粒子滤波（Particle Filter）：用离散样本（粒子）近似概率分布，适用于非线性非高斯系统
- TF 坐标变换：GMapping 同时需要激光雷达→基座、基座→里程计、地图→里程计的变换

## 推导到 / 关联到

- AMCL：GMapping 解决建图+定位，AMCL 解决定位（地图已知）
- 占用网格地图：GMapping 的输出——每个格子标记为"障碍物"或"空闲"
- ROS Navigation Stack：GMapping 为导航栈提供地图输入

## 易混概念

- GMapping vs Hector SLAM：GMapping 依赖里程计输入；Hector SLAM 仅依赖激光扫描匹配，不依赖里程计，但对激光雷达帧率要求高
- 粒子滤波 vs 卡尔曼滤波：粒子滤波适用于非线性非高斯分布（任意分布），计算量大；卡尔曼滤波假设线性和高斯分布，计算量小

## 典型例子

ROS 中运行 GMapping：
```
rosrun gmapping slam_gmapping scan:=/scan
```
输入：`/tf`（坐标变换）+ `/scan`（激光雷达数据）
输出：`/map`（占用网格地图）+ `/map_metadata`（地图元数据）
考试高频点：GMapping 的**三个输入（深度信息 + IMU + 里程计）** 和 **一个输出（占用网格地图）**
