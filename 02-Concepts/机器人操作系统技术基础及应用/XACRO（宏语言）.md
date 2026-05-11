---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [6]
first_seen: 2026-04-07
prerequisites: [URDF（统一机器人描述格式）]
related: [Launch 文件, Gazebo 仿真环境]
contrasts: [URDF（统一机器人描述格式）, Launch 文件]
---

# XACRO（宏语言）

## 定义

XACRO（XML Macros）是 URDF 的宏语言扩展，通过在标准 URDF 语法基础上引入常量定义、数学表达式和宏函数，消除 URDF 文件中的重复代码，提高模型文件的模块化和可维护性。XACRO 文件在使用前需通过 `xacro` 工具转换为标准 URDF。

## 直觉理解

URDF 好比用"纯文本"写代码——每个轮子、每条关节都要手写一遍所有标签；XACRO 好比给 URDF 加上了"函数和变量"——定义一次宏，传不同参数就能生成多个相似部件，比直接写 URDF 省力得多。

## 前置概念

- URDF（统一机器人描述格式）：XACRO 是 URDF 的超集/预处理工具，最终输出仍是 URDF
- XML：XACRO 基于 XML 语法

## 推导到 / 关联到

- Launch 文件：Launch 文件中可直接调用 XACRO 文件（通过 `xacro` 解析后加载）
- Gazebo 仿真环境：复杂机器人模型通常用 XACRO 编写，再导入 Gazebo 仿真

## 易混概念

- XACRO vs URDF：URDF = 最终模型描述；XACRO = 带宏的中间模板，需预处理转换
- XACRO vs Launch 文件：XACRO = 简化模型文件编写；Launch 文件 = 启动多个节点/配置
- `<xacro:property>` vs `<arg>`：XACRO 属性在预处理阶段完成替换；Launch 文件的 `<arg>` 在运行时生效

## 典型例子

```xml
<!-- 定义轮子宏 -->
<xacro:macro name="wheel" params="name prefix">
  <link name="${prefix}_wheel">
    <visual><geometry><cylinder radius="0.1" length="0.05"/></geometry></visual>
  </link>
  <joint name="${prefix}_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="${prefix}_wheel"/>
  </joint>
</xacro:macro>

<!-- 调用宏生成左右轮 -->
<xacro:wheel name="left" prefix="left"/>
<xacro:wheel name="right" prefix="right"/>
```
编译步骤：`xacro model.xacro > model.urdf`
