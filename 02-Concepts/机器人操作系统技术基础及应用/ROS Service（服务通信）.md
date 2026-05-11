---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [2]
first_seen: 2026-03-10
prerequisites: [ROS（机器人操作系统）]
related: [ROS Topic（话题通信）, ROS（机器人操作系统）]
contrasts: [ROS Topic（话题通信）]
---

# ROS Service（服务通信）

## 定义

Service（服务）是 ROS 中基于**客户端-服务器（Client/Server）**模型的同步通信方式。客户端发送请求（Request），服务器处理并返回应答（Response），一问一答、阻塞等待。

## 直觉理解

好比**打电话问客服**：你拨号（Client call）→ 客服接听并处理（Server）→ 客服给出答复（Response）——一个 Server 可以服务多个 Client，但每个 Call 都必须等回复才能继续。

## 前置概念

- ROS（机器人操作系统）：ROS 的三大通信机制之一
- Node（节点）：Service 的 Server 和 Client 都是节点
- srv 文件：自定义服务数据格式，以 `---` 分隔请求域和应答域

## 推导到 / 关联到

- ROS Topic（话题通信）：对比——Service 适合一次性操作（如复位乌龟），Topic 适合持续数据流（如速度指令）
- rosservice 命令行：`rosservice list` / `rosservice call` 等调试工具
- 自定义消息（.msg）：Service 比自定义消息考试更常见，因涵盖请求+应答两个数据域

## 易混概念

- ROS Topic（话题通信）：Topic 异步/广播/多对多 vs Service 同步/一问一答/一对一（一个 Server 对多个 Client）
- 参数服务器（Parameter Server）：全局变量存储，读写即用；Service 需编程定义接口（.srv 文件）并注册回调

## 典型例子

- turtlesim 中的 `/spawn` 服务：`rosservice call /spawn "{x: 0, y: 0, theta: 0, name: 't1'}"` 生成新乌龟
- `/clear` 服务：清除屏幕上的乌龟运动轨迹
- 加法器示例：Server 接收两个整数并返回和（`AddTwoInts.srv`：`int64 a` + `int64 b` → `---` → `int64 sum`）
