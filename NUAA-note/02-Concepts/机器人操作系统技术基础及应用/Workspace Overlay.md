---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [3]
first_seen: 2026-03-17
prerequisites: [catkin 工作空间, ROS（机器人操作系统）]
related: [catkin 工作空间, ROS Topic（话题通信）]
contrasts: [catkin 工作空间, 全局安装]
---

# Workspace Overlay

## 定义

Workspace Overlay（工作空间叠加）是 ROS catkin 编译系统中的一种环境管理机制，允许**多个工作空间按优先级堆叠**——后 sourced 的工作空间中的功能包会覆盖先 sourced 的相同名称功能包，从而实现"不修改原有工作空间，即可测试新版本"的沙盒式开发。

## 直觉理解

好比电脑的"环境变量 PATH"——先装的软件在 C:\Program Files，后装的在 C:\Apps，系统找程序时从第一个路径开始找，如果第一个路径有同名程序就直接用，不再往下找。ROS 的 Workspace Overlay 同理：你可以在 `devel_ws` 装了普通版功能包，然后在 `test_ws` 装了修改版，source test_ws 后系统自动优先用修改版。

## 前置概念

- catkin 工作空间：ROS 功能包的编译和运行环境，包含 src/build/devel/install 四目录
- source 命令：`source devel/setup.bash` 将工作空间环境变量加入当前 shell
- 功能包（Package）：ROS 软件的最小单元

## 推导到 / 关联到

- 编译环境管理：Overlay 机制使得开发者无需修改系统级工作空间即可测试新包
- Package Path：`echo $ROS_PACKAGE_PATH` 查看当前所有已 source 的工作空间（按优先级排序）
- CMakeLists.txt 配置：Overlay 自动处理依赖，无需手动指定依赖包路径

## 易混概念

- Overlay vs 直接覆盖安装：Overlay 不影响原始工作空间目录，只是运行时路径优先级；直接覆盖安装会物理替换原有文件
- Overlay vs chroot/容器：Overlay 只影响 ROS 包查找路径，不影响系统库和其他工具
- 环境叠加顺序：最后 source 的优先级最高——`source A/setup.bash && source B/setup.bash` 后，B 覆盖 A

## 典型例子

- 多版本测试：`~/ros_ws` 中安装了官方 kobuki 驱动，`~/kobuki_dev_ws` 中自己修改了 kobuki 驱动。source `kobuki_dev_ws` 后，ROS 使用修改版驱动
- 典型叠加链：机器人本体工作空间（底层驱动）→ 算法工作空间（导航/感知）→ 应用工作空间（业务逻辑）
