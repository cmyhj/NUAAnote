# 第6章 SLAM 与自主导航

> 对应教材第9章

## 学习目标

- 理解 SLAM（即时定位与地图构建）的基本概念
- 掌握常用深度传感器的类型与特点
- 掌握 gmapping SLAM 建图流程
- 理解 AMCL 蒙特卡洛定位原理
- 掌握 move_base 导航框架的构成
- 了解 TF 变换在导航中的关键作用

---

## 6.1 理论基础

### 6.1.1 什么是 SLAM

**SLAM（Simultaneous Localization and Mapping，即时定位与地图构建）** 由 Smith、Self 和 Cheeseman 于 1988 年提出。

SLAM 描述为：**机器人在未知环境中从未知位置开始移动，移动过程中根据位置估计和地图进行自身定位，同时建造增量式地图。**

### 6.1.2 什么是自主导航

自主导航：在机器人工作空间中，根据定位导航系统找到一条**从起始状态到目标状态、可以避开障碍物的最优路径**。

SLAM 生成的地图是自主导航的主要蓝图。

### 6.1.3 深度感知传感器对比

| 传感器 | 优点 | 缺点 | 应用 |
|--------|------|------|------|
| **激光雷达** | 精度高、响应快、数据量小、可实时 SLAM | 成本高（进口高端雷达 1 万+） | 扫地机器人、自动驾驶 |
| **单目摄像头** | 简单、适用性强 | 静止时无法测距，复杂度高 | 低成本 SLAM |
| **双目摄像头** | 静止也能感知距离 | 标定复杂、运算量大 | 机器人视觉定位 |
| **RGB-D 摄像头** | RGB + 深度信息，成本低，多功能 | 视野窄、盲区大、噪声大 | 室内服务机器人主流方案 |

---

## 6.2 SLAM 建图

### 6.2.1 ROS 中的 SLAM 功能包

| 功能包 | 维度 | 传感器 | 适用场景 |
|--------|------|--------|----------|
| **gmapping** | 二维 | 激光雷达 | 室内小场景，计算快 |
| hector | 二维 | 激光雷达 | 不需要里程计 |
| cartographer | 二维 | 激光雷达 | 大场景，Google 出品 |
| rgbslam | 三维 | RGB-D 摄像头 | 室内三维重建 |
| ORB_SLAM | 三维 | 单目/双目/RGB-D | 学术界常用 |

### 6.2.2 gmapping 建图流程

gmapping 基于**粒子滤波**实现二维占据栅格 SLAM：

1. 订阅 `/scan`（激光数据）和 `/tf`（里程计到激光的坐标变换）
2. 实时构建并输出 `/map`（占据栅格地图）
3. 在未知环境中遥控机器人遍历所有区域
4. 使用 `map_saver` 保存地图

**保存地图：**
```bash
rosrun map_server map_saver -f my_map
# 生成 my_map.pgm（图像文件）和 my_map.yaml（元数据文件）
```

---

## 6.3 自主导航

### 6.3.1 ROS Navigation 栈四步流程

```
gmapping SLAM建图 → map_server保存/加载地图 → amcl定位 → move_base路径规划
```

### 6.3.2 map_server —— 地图服务

- `map_saver`：将 SLAM 构建的 `/map` 话题保存为 `.pgm` + `.yaml` 文件对
- `map_server` 节点：加载地图，发布到 `/map` 话题供导航使用

### 6.3.3 amcl —— 自适应蒙特卡洛定位

AMCL（Adaptive Monte Carlo Localization）基于**粒子滤波**实现机器人在已知地图上的全局定位：

- 订阅 `/scan`（激光数据）和 `/tf`（坐标变换）
- 将激光扫描与静态地图匹配
- 估计 `map → odom` 变换，消除里程计累积漂移
- 初始位姿可通过 rviz 的 `2D Pose Estimate` 手动给出

### 6.3.4 move_base —— 路径规划核心

move_base 是导航栈的核心动作执行器，接收目标位姿后协调两个规划器：

```
                    ┌──────────────┐
  /map (静态地图) →  │ global_planner│ → 全局路径 (大尺度)
                    └──────────────┘
                    ┌──────────────┐
  /scan (传感器)  → │ local_planner │ → 局部轨迹 (避障)
                    └──────────────┘
                           ↓
                      /cmd_vel (速度指令)
```

**两层代价地图：**

| 代价地图 | 数据来源 | 作用 |
|----------|----------|------|
| global_costmap | 静态地图膨胀障碍物 | 全局路径规划 |
| local_costmap | 实时传感器数据 | 局部避障与轨迹优化 |

### 6.3.5 TF 变换链在导航中的关键作用

导航栈要求完整的 TF 变换链，任何缺失都将导致导航失败：

```
map → odom → base_footprint → base_link → laser
 ↑       ↑         ↑              ↑          ↑
地图  里程计/AMCL  机器人投影    机器人中心   激光雷达
原点  估计漂移    到地面        坐标系       坐标系
```

> AMCL 负责修正 `map → odom` 的漂移，其余变换由 URDF 模型和 robot_state_publisher 发布。

### 6.3.6 导航核心话题汇总

| 话题 | 方向 | 说明 |
|------|------|------|
| `/scan` | 输入 | 激光雷达数据 |
| `/odom` | 输入 | 里程计数据 |
| `/tf` | 输入 | 坐标变换（map→odom→base_link→laser） |
| `/map` | 输入 | 静态占据栅格地图 |
| `/cmd_vel` | 输出 | 最终速度控制指令 |
| `/move_base_simple/goal` | 输入 | rviz 2D Nav Goal 设定的目标位姿 |

---

## 本章小结

1. **SLAM**：机器人在未知环境中同时定位与建图
2. **gmapping**：基于粒子滤波的二维激光 SLAM，输出 `/map` 话题
3. **map_server**：`map_saver` 保存地图（.pgm + .yaml），`map_server` 加载地图
4. **amcl**：基于粒子滤波的全局定位，修正 `map → odom` 漂移
5. **move_base**：global_planner（全局路径）+ local_planner（局部避障）→ 输出 `/cmd_vel`
6. **TF 变换链**必须完整：map → odom → base_link → laser
7. 深度传感器：激光雷达（精度高/成本高）、RGB-D（多功能/成本低）

**随堂例题 6-1**（SLAM / AMCL / move_base）

1. SLAM 的正确全称含义是？ A) Synchronized Laser and Motion  B) Simultaneous Localization and Mapping  C) Sequential Location and Mapping
2. AMCL 的全称和核心算法是？ A) Automatic Mapping，卡尔曼滤波  B) Adaptive Monte Carlo Localization，粒子滤波  C) Advanced Motion Control，PID控制
3. 保存 SLAM 建图结果的命令是？ A) `rosrun map_server map_save`  B) `rosrun map_server map_saver -f mymap`  C) `rosrun gmapping save_map`
4. move_base 协调哪两个规划器？ A) path_planner/speed_planner  B) global_planner/local_planner  C) long_planner/short_planner
5. SLAM 导航四步流程（按顺序填空）：\_\_\_\_（建图）→ \_\_\_\_（保存地图）→ \_\_\_\_（定位）→ \_\_\_\_（路径规划）
6. 导航必备 TF 链路是？ A) laser→base_link→odom→map  B) map→odom→base_link→laser  C) base_link→map→odom→laser

> **答案**：1.B  2.B  3.B  4.B  5.gmapping/map_server/amcl/move_base  6.B

## 课后练习

1. 简述 SLAM 的定义和核心思想。
2. 激光雷达、单目摄像头、双目摄像头、RGB-D 摄像头各有什么优缺点？
3. gmapping SLAM 建图的基本流程是什么？
4. AMCL 在导航中的具体作用是什么？它修正了什么 TF 变换？
5. move_base 包含哪两个规划器？各自的功能是什么？
6. ROS 导航栈要求哪些 TF 变换必须完整？缺失了某个变换会导致什么后果？
