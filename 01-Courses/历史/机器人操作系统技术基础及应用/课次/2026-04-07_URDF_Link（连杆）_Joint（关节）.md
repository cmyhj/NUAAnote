# 机器人操作系统技术基础及应用 · 第7节

| 课程 | 教师 | 节次 | 周次 | 日期 | 地点 |
|------|------|------|------|------|------|
| 机器人操作系统技术基础及应用 | 鞠锋 | 第7节 | 第6周 | 2026-04-07 16:15-17:05 | 10203 |

---

## 本节主线

本节围绕 **机器人的建模与仿真**（教材第六章）展开，核心内容是 **URDF（统一机器人描述格式）** 的语法与使用。通过一个移动机器人底盘案例，讲解 Link、Joint、Robot 等标签的配置方法，并演示了 `check_urdf` 和 `urdf_to_graphiz` 等辅助工具。

---

## 时间轴

| 时间 | 内容 |
|------|------|
| 01:34-03:16 | 课堂点名 |
| 03:16-05:52 | 教学进度说明 & 最终答辩时间提醒 |
| 05:59-08:00 | 进入正题：建模与仿真章节概述，URDF 简介 |
| 08:00-15:00 | **Link 标签**详解：外观、物理属性、碰撞检测 |
| 15:02-21:50 | **Joint 标签**详解：6种关节类型、语法 |
| 21:53-23:54 | **Robot 标签**：顶层组织 |
| 23:54-25:07 | **Gazebo 标签**：仿真额外配置 |
| 25:07-35:13 | 案例：移动机器人 URDF 模型（mrobot）文件结构与逐段解析 |
| 35:13-47:18 | 工具演示：`check_urdf` 检查、`urdf_to_graphiz` 可视化 |
| 47:18-51:35 | 案例代码回顾：base_link → left motor → left wheel 的关节链 |
| 51:35-54:59 | 课间休息 & 收尾 |

---

## 关键概念

| 概念 | 说明 |
|------|------|
| **URDF (Unified Robot Description Format)** | 统一机器人描述格式，ROS 中描述机器人模型的 XML 文件，扩展名为 `.urdf` |
| **Link（连杆）** | 描述机器人某个刚体部分的外观（尺寸、颜色、形状）和物理属性（质量、惯性矩阵），以及碰撞参数 |
| **Joint（关节）** | 连接两个 Link，描述运动学和动力学属性，包括运动位置、速度限制 |
| **Collision（碰撞检测）** | 单独设置的碰撞区域（通常大于可视区域），降低碰撞检测计算量，一般设为长方体 |
| **Gazebo** | 仿真环境中所需的额外配置标签（材料、插件、控制器等） |

---

## 要点详述

### 1. 教学进度 & 答辩提醒

- 本周（第6周）开始 **建模与仿真**，计划用一周（两次课）讲完，可能压缩
- 移动机器人章节缩减到 2 课时，后面安排 2 课时介绍 **ROS 2**
- **最终答辩** 分为两波：最后一周的周二组 / 周四组。距答辩仅剩约两周，需尽快完成综合性编程作业和PPT
- 禁止小组间雷同，鼓励做三维模型（如三维小乌龟）

### 2. URDF 基础

- **全称**（考试重点）：英文 *Unified Robot Description Format*，中文 **统一机器人描述格式**，缩写 **URDF**
- 基于 **XML** 格式，以标签（tag）组织配置
- 顶层标签为 `<robot>`，内部包含 `<link>` 和 `<joint>` 序列

### 3. Link 标签

```
<link name="link_name">
  <inertial>...</inertial>
  <visual>
    <origin xyz="..." rpy="..."/>
    <geometry>
      <cylinder length="..." radius="..."/>
    </geometry>
    <material name="...">
      <color rgba="..."/>
    </material>
  </visual>
  <collision>
    <geometry>...</geometry>
  </collision>
</link>
```

- `name` 属性：必填，唯一标识
- `inertial`：惯性参数（质量、惯性矩阵）
- `visual`：外观（原点坐标、几何形状、颜色）
  - 几何形状支持：`<cylinder>`、`<box>`、`<sphere>`、`<mesh>`（导入外部模型）
  - `material` / `color` 使用 RGBA 四个值（红/绿/蓝/透明度）
- `collision`：碰撞区域，形状通常简化（如长方体），比 visual 略大以降低计算量

### 4. Joint 标签 — 六种关节类型

| 类型 | 说明 |
|------|------|
| **continuous** | 连续旋转关节，无角度限制（如电风扇、轮子） |
| **revolute** | 有限角度旋转关节（如人类手臂关节，±180° 以内） |
| **prismatic** | 滑动关节，沿某一轴线平移，带位置极限 |
| **planar** | 平面关节，允许在平面内平移和旋转 |
| **floating** | 浮动关节，完全自由（6自由度），无限制 |
| **fixed** | 固定关节，不允许运动（调试时锁住关节用） |

```
<joint name="joint_name" type="continuous">
  <parent link="parent_link_name"/>
  <child link="child_link_name"/>
  <origin xyz="..." rpy="..."/>
  <axis xyz="..."/>
  <limit lower="..." upper="..." effort="..." velocity="..."/>
  <dynamics damping="..." friction="..."/>
</joint>
```

- **必填属性**：`name`、`type`
- **关键子标签**：`<parent>`、`<child>` — 定义连接关系
- `origin`：关节相对于父 link 的坐标变换
- `axis`：旋转/平移轴方向
- `limit`：极限值（上下限位置、速度、力矩）
- `dynamics`：物理属性（阻尼、摩擦力）

### 5. Robot 标签 — 顶层组织

```
<robot name="my_robot">
  <link name="base_link"> ... </link>
  <link name="left_wheel"> ... </link>
  <joint name="base_to_left_wheel" type="continuous"> ... </joint>
</robot>
```

- 整个 URDF 文件的根标签，`name` 为机器人名称
- 内部按顺序定义所有 Link 和 Joint
- 结尾必须有 `</robot>`（考试常见失分点）

### 6. URDF 功能包结构

```
your_robot_description/
├── urdf/          # 存放 .urdf 或 .xacro 文件
├── meshes/        # 存放引用的 3D 模型渲染文件
├── launch/        # 启动文件
└── config/        # Rviz 等配置文件
```

### 7. 案例：mrobot 移动机器人底盘

- **7 个 Link**：1 个底板（base_link）+ 2 个电机 + 2 个驱动轮 + 2 个万向轮
- **6 个 Joint**：连接电机的 `fixed` 关节 + 连接轮子的 `continuous` 关节
- 单位：**米**（默认）
- 颜色配置：`<material name="yellow">` 使用 RGBA（1 1 0 1）表示黄色不透明
- 电机相对底座绕 X 轴旋转 **90°（π/2）**

### 8. 辅助工具

- **`check_urdf`**：检查 URDF 文件语法，输出解析结果
  ```
  check_urdf mrobot.urdf
  ```
- **`urdf_to_graphiz`**：生成机器人结构拓扑图（PDF）
  ```
  urdf_to_graphiz mrobot.urdf
  ```
  输出两个文件：`mrobot.pdf`（结构图）、`mrobot.gv`（Graphviz 源文件）
  - 注意：工具图中椭圆代表 Joint，书中椭圆代表 Link，考试以教材为准

---

## 作业 & 考试通知

| 事项 | 说明 |
|------|------|
| **综合性编程作业** | 小组合作，尽快完成；禁止组间雷同；建议做三维模型加分 |
| **最终答辩** | 最后一周周二（较早）/ 周四（多两天）；需准备视频 + PPT |
| **期末考试** | URDF 全称（中/英文）、缩写、标签语法均为考点 |

---

## 待核对

- `<link>` 语法中，`name` 属性的引号格式（教师提到"回头确认"） @12:20
- `check_urdf` 工具需要提前安装 `liburdfdom-tools` @41:15

---

## 回看建议

- **08:00-15:00** — Link 标签语法与碰撞检测原理（考试重点）
- **15:02-21:50** — 六种 Joint 类型的区分（continuous vs revolute 常考）
- **25:07-35:13** — mrobot 案例整体结构（理解 Link-Joint 层级关系）
- **35:13-47:18** — `check_urdf` 和 `urdf_to_graphiz` 工具实操