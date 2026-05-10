---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [2]
first_seen: 2026-03-10
prerequisites: [ROS（机器人操作系统）]
related: [ROS Service（服务通信）, Launch 文件]
contrasts: [ROS Service（服务通信）, 参数服务器]
---

# ROS Topic（话题通信）

## 定义

Topic（话题）是 ROS 中基于**发布/订阅（Publish/Subscribe）**模型的异步通信方式。发布者（Publisher）将消息广播到话题上，一个或多个订阅者（Subscriber）接收该消息，双方互不知晓对方存在。

## 直觉理解

好比**电台广播**：电台（Publisher）只管向外广播信号，不管谁在听；听众（Subscriber）打开收音机调到相同频率（Topic）就能收到，不用知道电台在哪里——一对多、多对一、多对多都支持。

## 前置概念

- ROS（机器人操作系统）：ROS 的三层核心通信机制之一
- Node（节点）：ROS 中独立运行的进程，话题通信的发布者和订阅者都是节点
- Message（消息）：话题上传输的严格数据结构，支持嵌套和自定义

## 推导到 / 关联到

- ROS Service（服务通信）：对比——Topic 异步/多对多 vs Service 同步/一问一答
- rqt_graph：可视化节点与话题的关系图
- rosbag：录制和回放话题消息，用于离线调试

## 易混概念

- ROS Service（服务通信）：Topic 是异步、一对多/多对多、发布者和订阅者互不感知；Service 是同步、一对一（server 到多个 client）、Client 必须知道 Server 的存在
- 参数服务器（Parameter Server）：全局键值存储，非流式通信；Topic 是持续的数据流

## 典型例子

- turtlesim 中的 `/turtle1/cmd_vel` 话题：键盘控制节点（Publisher）发布速度指令，turtlesim_node（Subscriber）接收后移动乌龟
- 双乌龟同步运动演示：两个 `turtlesim_node` 订阅同一 `/turtle1/cmd_vel` 话题，接收同一键盘指令同步运动
