# 第1章 ROS简介

## 学习目标

- 理解 ROS 的定义及其作为"元操作系统"的含义
- 掌握 ROS 的五大特点及其意义
- 了解 ROS 的发展历史与版本命名规律
- 能够完成 ROS Noetic 的安装与环境配置
- 掌握基本的 Linux 命令行操作
- 能够运行小乌龟仿真例程并理解其背后的 ROS 机制

---

## 1.1 ROS 是什么

### 核心定义

**ROS（Robot Operating System，机器人操作系统）是一个运行在 Linux 之上的开源元操作系统（Meta-OS）。**

它不是 Windows 或 Linux 那样的传统操作系统——ROS 无法直接运行在裸机上，必须依托 Linux（通常是 Ubuntu）。ROS 的本质是一个**中间件 / 软件框架**，为机器人开发提供：

- 硬件抽象与底层设备控制
- 进程间消息传递（通信机制）
- 功能包管理与软件复用
- 开发工具与库函数（可视化、仿真、调试）

**ROS 解决的核心问题：避免"重复发明轮子"。** 机器人软件开发中，大量时间被消耗在重复构建底层基础设施（传感器驱动、进程间通信等），ROS 将这些通用功能标准化，让开发者专注于上层智能算法。

> **考试重点**：ROS 版本名称 **Noetic**，对应 Ubuntu **20.04**（代号 Focal Fossa）。往年考试中很多学生写不出自己使用的 ROS 版本。

**随堂例题 1-1**（ROS 定义与版本）

1. ROS 的全称是？ A) Real-time Operating System  B) Robot Operating System  C) Robotic Operation Software  D) Remote Operating Standard
2. ROS 最准确的定义是？ A) 底层操作系统  B) 运行在 Linux 之上的元操作系统  C) 桌面操作系统  D) 嵌入式实时操作系统
3. ROS 设计要解决的核心问题是？ A) 硬件成本过高  B) "重复发明轮子"  C) Linux 不稳定  D) C++ 效率低
4. ROS Noetic 对应的 Ubuntu 版本是？ A) 16.04  B) 18.04  C) 20.04  D) 22.04

> **答案**：1.B  2.B  3.B  4.C

---

### 1.1.1 ROS 的起源与发展

| 年份 | 事件 |
|------|------|
| 2007 | 斯坦福大学提出 Personal Robot 项目，试图消除"重复发明轮子" |
| 2008 | Willow Garage 公司接手项目，ROS 正式诞生 |
| 2010 | ROS 1.0（Box Turtle）正式发布，此后以海龟类型命名 |
| 2011 | TurtleBot 发布——廉价、开源的 ROS 入门机器人 |
| 2013 | Willow Garage 解散，OSRF（开源机器人基金会）接管 |
| 2017 | OSRF 更名为 Open Robotics；ROS 2.0（Ardent）发布 |
| 2020 | ROS Noetic 发布——ROS 1 的最终 LTS 版本，支持至 2025 年 |

**标志性机器人：**

- **PR2**：全功能研究机器人，售价约 40 万美元，两条手臂各 7 个关节，配备高清摄像头、激光雷达、惯性测量单元
- **TurtleBot**：低成本教学机器人（几千元），本课程实验课使用
- **百度 Apollo**：基于 ROS 的自动驾驶开放平台

**ROS 的应用领域**：轮式机器人、人形机器人、工业机器人、农业机器人、四足机器人、水下机器人、无人机、医疗手术机器人等。NASA 的 VIPER 月球探测器也基于 ROS 2 和 Gazebo。

---

### 1.1.2 ROS 的设计目标

**提高机器人研发中的软件复用率。** ROS 被设计为分布式结构，每个功能模块可单独设计、编译，运行时以松散耦合方式结合。

---

### 1.1.3 ROS 的五大特点（⚠️ 考试高频）

| # | 特点 | 说明 |
|---|------|------|
| **1** | **点对点设计** | 分布式网络，每个进程以节点形式运行，可分布于多台主机。节点间通过 TCP/IP 通信，分散实时计算压力 |
| **2** | **多语言支持** | 语言弱相关的框架结构。使用中立的接口定义语言描述消息，编译时生成目标语言代码。支持 C++、Python、Java、Lisp 等 |
| **3** | **架构精简、集成度高** | 模块化设计，每个功能节点可单独编译。统一的消息接口便于移植复用。集成了 OpenCV、PCL 等大量开源库 |
| **4** | **组件化工具包丰富** | 3D 可视化 rviz、物理仿真 Gazebo、Qt 工具箱 rqt、数据记录 rosbag 等 |
| **5** | **免费并且开源** | BSD 许可证，允许商业和非商业使用、修改和分发 |

**随堂例题 1-2**（五大特点）

1. 以下哪项**不是** ROS 的五大特点之一？ A) 点对点设计  B) 需要商业许可  C) 多语言支持  D) 组件化工具包丰富
2. "点对点设计"的核心含义是？ A) 所有节点在同一台主机  B) 节点间通过 TCP/IP 网络分布式通信  C) 使用共享内存通信  D) 每个节点直接连接硬件
3. ROS 1 的致命架构缺陷是？ A) 不支持 Python  B) Master 节点单点故障  C) 无法运行在 Ubuntu 上  D) 不支持话题通信

> **答案**：1.B  2.B  3.B

---

## 1.2 如何安装 ROS

### 1.2.1 操作系统与 ROS 版本的选择

ROS 主要支持 Ubuntu。ROS 与 Ubuntu 的版本一一对应：

- 2013 年以前：一年发布两次（4 月和 10 月）
- 2013 年起（Hydro 开始）：每年 5 月发布一次——5 月 23 日是"世界乌龟日"
- **LTS 版本**（偶数年发布）：提供 5 年支持
- **非 LTS 版本**：提供 2 年支持

本课程使用 **ROS Noetic** + **Ubuntu 20.04 (Focal Fossa)**。

### 1.2.2 安装方式选择

| 安装方式 | 特点 |
|----------|------|
| 虚拟机安装 | 简单，适合初学者和偶尔使用者。硬件支持一般，运行速度较慢 |
| 硬盘安装（双系统） | 复杂，硬件支持好，运行速度快，适合有经验的开发者 |

**虚拟机安装建议**：虚拟硬盘至少 30-40GB，不要装在 C 盘。

### 1.2.3 安装步骤概览

**1. 配置 Ubuntu 软件源**
确保 restricted、universe、multiverse 三种软件源已启用。

**2. 添加 ROS 软件源**
```bash
# 使用国内镜像（推荐中国科学技术大学 USTC）
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'

# 或清华大学镜像
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```

**3. 添加密钥**
```bash
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

**4. 安装 ROS**
```bash
sudo apt update
sudo apt install ros-noetic-desktop-full  # 推荐：桌面完整版
```

三种安装版本对比：

| 版本 | 包含内容 | 适用场景 |
|------|----------|----------|
| Desktop-Full | 核心功能 + 通用库 + rviz + Gazebo + rqt | 推荐，功能最全 |
| Desktop | 核心功能 + 通用库 + rviz + rqt | 精简版，无 Gazebo |
| ROS-Base | 仅核心功能包 + 构建工具 + 通信机制 | 嵌入式 / 最小系统 |

**5. 初始化 rosdep**
```bash
sudo rosdep init
rosdep update
```
> 若 rosdep init 因网络原因失败，可使用国内的 rosdepc 替代：
> ```bash
> sudo apt-get install python3-pip
> sudo pip install rosdepc
> sudo rosdepc init
> rosdepc update
> ```

**6. 设置环境变量**
```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

**7. 验证安装**
```bash
roscore  # 启动 ROS Master，若无报错则安装成功
```

### 1.2.4 第一个 ROS 例程——小乌龟仿真

```bash
# 终端1：启动 ROS Master
roscore

# 终端2：启动小乌龟仿真器
rosrun turtlesim turtlesim_node

# 终端3：启动键盘控制节点
rosrun turtlesim turtle_teleop_key
```

按键盘方向键即可控制小乌龟移动。

> 小乌龟例程虽然简单，但**蕴含了 ROS 最基础的原理和机制**——节点、话题、消息，是理解 ROS 的入口。

---

## 1.3 Linux 系统基础

### 1.3.1 操作系统与 Linux

- **操作系统（OS）**：管理和控制计算机硬件与软件资源的程序，是运行在裸机上的最基本系统软件
- **Linux**：自由开源的类 Unix 操作系统，1991 年由 Linus Torvalds 编写
- **Linux 发行版** = Linux 内核 + GNU 软件 + 安装工具 + 其他自由/专有软件
- **Ubuntu** 是目前使用最广泛的 Linux 发行版之一，也是 ROS 官方支持最好的系统

### 1.3.2 基本命令（必须掌握）

| 命令 | 功能 | 示例 |
|------|------|------|
| `ls` | 列出当前目录内容 | `ls` |
| `cd` | 切换工作目录 | `cd test`、`cd ..`、`cd ~` |
| `pwd` | 显示当前绝对路径 | `pwd` |
| `mkdir` | 创建目录 | `mkdir test2` |
| `mv` | 移动或重命名文件/目录 | `mv test3 ..`、`mv old new` |
| `cp` | 复制文件或目录 | `cp a.txt b.txt` |
| `rm` | 删除文件或目录 | `rm file.txt` |
| `touch` | 创建空文件/更新时间戳 | `touch newfile.txt` |
| `sudo` | 以超级用户权限执行命令 | `sudo apt install ...` |

### 1.3.3 重要概念

- **Home 目录**：当前用户的个人文件夹，用 `~` 表示，完整路径为 `/home/用户名`
- **根目录 `/`**：Linux 文件系统最高层
- **绝对路径**：以 `/` 开头，从根目录完整描述位置
- **相对路径**：以当前位置为参考，`..` 表示上一级，`.` 表示当前
- **终端（Terminal）**：Linux 最重要的交互方式，快捷鍵 `Ctrl+Alt+T`

> 实用的终端技巧：按**方向键上**可翻出上一条执行过的命令。

**随堂例题 1-3**（安装与命令）

1. 安装 ROS Noetic 桌面完整版的命令是？ A) `sudo apt install ros-kinetic-desktop`  B) `sudo apt install ros-noetic-desktop-full`  C) `sudo apt install ros-melodic-desktop`  D) `sudo apt install ros-noetic-ros-base`
2. 启动 ROS Master 的命令是？ A) `rosrun master`  B) `rosstart`  C) `roscore`  D) `rosmaster`
3. 环境变量配置文件中需要添加哪一行？ `source ______`
4. 在终端中，`cd ..` 的作用是？`~` 代表什么？
5. `mv` 命令除了移动文件外，还可以实现什么功能？

> **答案**：1.B  2.C  3.`/opt/ros/noetic/setup.bash`  4.返回上一级目录；Home 目录  5.重命名

---

## 本章小结

1. ROS 是运行在 Linux 之上的**元操作系统/中间件**，不是传统操作系统
2. ROS 的**五大特点**：点对点设计、多语言支持、架构精简、组件化工具包丰富、免费开源
3. ROS 1.0 于 **2010** 年发布，**Noetic** 是 ROS 1 的最终 LTS 版本（2020 年发布）
4. 安装 ROS Noetic 需要 Ubuntu 20.04，推荐使用 Desktop-Full 安装版本
5. **roscore** 是启动 ROS Master 的命令，必须先于所有其他节点启动
6. 小乌龟例程使用 `rosrun turtlesim turtlesim_node` 和 `turtle_teleop_key` 启动
7. 必须掌握 6 个基本 Linux 命令：`ls`、`cd`、`pwd`、`mkdir`、`mv`、`cp`

## 课后练习

1. ROS 的五大特点是什么？简要说明每个特点的含义。
2. 从 2007 年到 2020 年，简述 ROS 发展史上的 5 个关键时间节点及对应事件。
3. ROS Noetic 对应哪个版本的 Ubuntu？LTS 的含义和支持时长是什么？
4. 简述 ROS 三种安装版本（Desktop-Full、Desktop、ROS-Base）的区别。
5. 在终端中启动小乌龟仿真器并控制其运动，用 `rqt_graph` 查看节点关系图。
6. 写出前 5 个必备 Linux 命令及其功能。
7. 解释以下 Linux 路径的含义：`~`、`/`、`..`、`/home`。
