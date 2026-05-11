---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [7]
first_seen: 2026-04-14
prerequisites: [SLAM（即时定位与地图构建）, 占用网格地图]
related: [GMapping 算法, move_base 导航框架, ROS Navigation Stack]
contrasts: [GMapping 算法, 卡尔曼滤波定位]
---

# AMCL 自适应蒙特卡洛定位

## 定义

AMCL（Adaptive Monte Carlo Localization，自适应蒙特卡洛定位）是 ROS Navigation Stack 中用于**已知地图下的机器人定位**的概率算法。它使用自适应粒子滤波（KLD 采样），在已知地图中实时估计机器人的位姿（位置 + 姿态）。

## 直觉理解

想象你拿着一个陌生校园的地图，被蒙上眼睛放到校园某个角落。你只能通过走路和感受周围环境（传感器）来判断自己在地图上的位置。AMCL 就是这样一个过程——同时保持很多个"你在哪"的猜测（粒子），随着你移动和感知，逐渐淘汰错误的猜测，最终精准定位。

## 前置概念

- 占用网格地图：AMCL 需要一张已知的栅格地图作为定位参考
- 粒子滤波：用加权粒子集合近似位姿的概率分布
- 里程计：提供机器人相对位移的估计（不精确但连续可用）

## 推导到 / 关联到

- GMapping：先 GMapping 建图（SLAM），后 AMCL 定位（已知地图）
- move_base：AMCL 为 move_base 提供机器人当前位置，move_base 据此规划路径
- 自适应（KLD 采样）：粒子数量根据当前分布复杂度自适应调整——分布集中时少用粒子，分布分散时多用粒子

## 易混概念

- AMCL vs GMapping：GMapping 在未知环境中同时建图和定位；AMCL 在地图已知时只做定位
- AMCL 粒子滤波 vs 扩展卡尔曼滤波（EKF）：粒子滤波能表示多峰分布（多个可能位置）；EKF 假设单峰高斯分布
- 全局定位 vs 局部定位：AMCL 支持全局定位（不知道初始位置也能找到自己）；单纯里程计只能做局部跟踪

## 典型例子

- TurtleBot 导航：先运行 GMapping 建图保存，下次启动时加载地图运行 AMCL 实现自动导航
- AMCL 输入/输出：订阅 `/scan`（激光）+ `/tf`（里程计变换）+ `/map`（已知地图），发布 `/amcl_pose`（估计位姿）
- 绑架机器人问题（Kidnapped Robot Problem）：AMCL 能处理机器人被突然移动到另一位置的情况，重新收敛到正确位姿
