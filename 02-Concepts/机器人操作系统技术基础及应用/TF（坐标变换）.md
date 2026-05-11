---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [4]
first_seen: 2026-03-31
prerequisites: [ROS（机器人操作系统）]
related: [URDF（统一机器人描述格式）, Launch 文件, RViz]
contrasts: [参数服务器]
---

# TF（坐标变换）

## 定义

TF（Transform）是 ROS 中的坐标变换框架，使用**树形数据结构**管理多坐标系间的变换关系，带时间缓冲，支持在任意时间点完成点、向量等坐标变换。

## 直觉理解

好比**导航地图上的"你在哪"**——世界坐标系是地图原点，机器人中心、机械臂末端、摄像头各自都有一个坐标系。TF 自动计算"摄像头拍到的物体"在"世界坐标系"中的位置，你只需说"我要知道 A 相对于 B 的坐标"，TF 帮你算好。

## 前置概念

- ROS（机器人操作系统）：TF 是 ROS 常用组件之一
- 坐标系与坐标变换基础：平移（x, y, z）+ 旋转（yaw/pitch/roll 或四元数）

## 推导到 / 关联到

- URDF（统一机器人描述格式）：URDF 定义机器人各 Link 的相对关系，运行时由 TF 发布者广播实时坐标变换
- Launch 文件：`static_transform_publisher` 可在 Launch 文件中配置，用于发布固定坐标系间的变换
- RViz：三维可视化工具，依赖 TF 数据来正确显示各传感器和机器人部件的空间位置
- SLAM：即时定位与地图构建，核心输出就是机器人坐标系与世界坐标系的 TF 变换

## 易混概念

- 参数服务器（Parameter Server）：存储静态配置值（如传感器型号）；TF 是动态坐标系变换关系，随机器人的运动实时更新
- URDF：描述机器人的"静态"结构（Link-Joint 层级）；TF 描述"运行时"各坐标系间的实时变换，URDF 的 Joint 状态通过 TF 发布器转为实际坐标变换

## 典型例子

- 乌龟跟随例程（`turtle_tf`）：安装 `ros-noetic-turtle-tf` 后运行，两只乌龟中第二只自动跟随第一只。TF monitor 查看坐标系发布频率（~100Hz），TF echo 实时计算两龟坐标变换
- TF 工具族：`rosrun tf tf_monitor`（监测发布状态）、`rosrun tf tf_echo frame1 frame2`（实时坐标变换）、`rqt_tf_tree`（图形化 TF 树）
- 两步操作：① Broadcaster 广播坐标变换关系 → ② Listener 接收并缓存，通过 `lookupTransform()` 查询
