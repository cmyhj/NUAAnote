---
type: concept
course: 机器人操作系统技术基础及应用
chapter: [3]
first_seen: 2026-03-17
prerequisites: [ROS（机器人操作系统）]
related: [Launch 文件, ROS Topic（话题通信）, ROS Service（服务通信）]
contrasts: [普通文件夹]
---

# catkin 工作空间

## 定义

catkin 工作空间（Workspace）是 ROS 中用于组织、编译和管理功能包的标准目录结构，包含四层子目录：`src`（代码空间）、`build`（编译空间）、`devel`（开发空间）和 `install`（安装空间）。

## 直觉理解

好比**一个标准化的建筑工程现场**——`src` 是设计图纸和原材料仓库（源代码），`build` 是施工的临时脚手架（编译中间文件），`devel` 是建好的毛坯房（可直接运行），`install` 是精装修交付版（打包安装到系统路径）。

## 前置概念

- ROS（机器人操作系统）：工作空间是 ROS 工程开发的基础设施
- catkin 编译系统：ROS 的编译系统，取代了旧版 rosbuild，自动处理依赖和编译流程
- Package（功能包）：ROS 软件的基本组织单元，平行放在 `src/` 下

## 推导到 / 关联到

- Launch 文件：通常存放在功能包内的 `launch/` 目录下
- ROS Topic（话题通信）和 ROS Service（服务通信）：在功能包中编写发布/订阅/服务的 C++/Python 代码
- Workspace Overlay（工作空间覆盖）：多个工作空间的同名功能包由 `ROS_PACKAGE_PATH` 的前后顺序决定查找优先级——每年必考

## 易混概念

- 普通文件夹：普通文件夹没有编译系统和环境变量配置（无 `setup.bash`）；catkin 工作空间经过 `catkin_make` 初始化后，`devel/` 目录下生成 `setup.bash`，`source` 后才能被 ROS 识别
- build vs devel：`build/` 存放编译中间产物（临时文件），`devel/` 存放最终可执行文件和环境配置脚本（开发时用）；`install/` 是通过 `make install` 生成的交付版

## 典型例子

- 标准创建流程：`mkdir -p catkin_ws/src` → `cd catkin_ws/src` → `catkin_init_workspace` → `cd ..` → `catkin_make` → `source devel/setup.bash`
- 功能包创建：`catkin_create_pkg learning_communication std_msgs roscpp rospy` — 创建依赖 `std_msgs`、`roscpp`、`rospy` 的功能包
- `catkin_make`：编译命令，是考试必考命令
