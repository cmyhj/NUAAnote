---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [5]
first_seen: 2026-04-02
prerequisites: [ROS Topic（话题通信）]
related: [RViz 可视化工具, rqt 可视化工具]
contrasts: [SSH 远程记录]
---

# rosbag 数据记录

## 定义

rosbag 是 ROS 中用于录制和回放话题消息数据的命令行工具。它可以将运行中的 ROS 话题数据记录到 `.bag` 文件中，之后离线回放时各话题按原始时间戳重新发布，实现"数据回放"功能。

## 直觉理解

好比机器人的"黑匣子"——在机器人运行过程中把所有传感器数据、控制指令都录下来，以后再放出来像"重播录像"一样看到当时机器人的全部感知与决策过程。

## 前置概念

- ROS Topic（话题通信）：rosbag 录制和回放的基本单位是话题
- 时间戳（Timestamp）：bag 文件按时间戳记录每条消息，回放时恢复原始时序

## 推导到 / 关联到

- RViz 可视化工具：录制 bag 后用 RViz 可离线回放可视化
- rqt 可视化工具：rqt_bag 插件提供 bag 文件的图形化浏览界面
- 调试与复现：录制真实机器人一次运行，可在仿真环境中反复回放调试算法

## 易混概念

- rosbag 录制 vs SSH 远程记录：rosbag 在机器人本体录制话题数据；SSH 通过远程登录操作录制过程但数据仍在机器人端
- rosbag 回放 vs 仿真运行：回放走"真实数据流程"但不涉及物理仿真；仿真在 Gazebo 中模拟物理世界生成数据

## 典型例子

```bash
# 录制所有话题
rosbag record -a

# 录制指定话题
rosbag record /scan /tf /odom

# 回放
rosbag play recorded.bag

# 回放并循环、以 0.5 倍速
rosbag play -r 0.5 -l recorded.bag
```
