---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [6]
first_seen: 2026-04-07
prerequisites: [ROS（机器人操作系统）, XML 基础]
related: [Gazebo 仿真环境, Launch 文件]
contrasts: [XACRO]
---

# URDF（统一机器人描述格式）

## 定义

URDF（Unified Robot Description Format，统一机器人描述格式）是 ROS 中用于描述机器人模型的 **XML 格式文件**（扩展名 `.urdf`），通过 `<link>`（连杆）和 `<joint>`（关节）标签定义机器人的外观、物理属性和运动学结构。

## 直觉理解

好比**乐高积木说明书**：每个零件（Link）是什么形状、什么颜色、多重，零件之间怎么连接（Joint）、能怎么动——URDF 就是用 XML 写的"机器人积木搭建说明书"。

## 前置概念

- ROS（机器人操作系统）：URDF 是 ROS 建模与仿真章节的核心内容
- XML 基础：URDF 基于 XML 标签语法
- Link（连杆）：描述刚体部分的外观（几何、颜色）和物理属性（质量、惯性矩阵）
- Joint（关节）：连接两个 Link，定义运动学关系（六种类型）

## 推导到 / 关联到

- Gazebo 仿真环境：URDF 导入 Gazebo 后需额外添加 `<gazebo>` 标签（碰撞、惯性、控制器插件）
- XACRO：URDF 的宏语言扩展，支持常量、数学运算和文件包含，避免重复代码
- check_urdf / urdf_to_graphiz：URDF 语法检查和拓扑图生成工具
- ros_control：五层架构，通过传动配置连接 URDF 关节与实际控制器

## 易混概念

- XACRO：URDF 的简化版增强——XACRO 是"URDF 宏"，在编译时展开为纯 URDF，支持 `<xacro:property>` 常量和 `<xacro:macro>` 宏定义；URDF 不支持任何变量和运算
- SDF（Simulation Description Format）：Gazebo 原生使用的模型格式，比 URDF 更通用；URDF 侧重于运动学描述，部分仿真属性需要转换

## 典型例子

- mrobot 移动机器人底盘：7 个 Link（1 底板 + 2 电机 + 2 驱动轮 + 2 万向轮）+ 6 个 Joint（fixed 连接电机 + continuous 连接轮子）
- 六种 Joint 类型：`continuous`（无限制旋转，如轮子）、`revolute`（有限角度旋转，如手臂关节）、`prismatic`（滑动）、`planar`（平面）、`floating`（浮动）、`fixed`（固定）
