# 第4章 ROS 中的常用组件

## 学习目标

- 掌握 launch 启动文件的编写和使用
- 理解 TF 坐标变换的原理与使用方法
- 掌握 rqt 工具箱中各工具的用途
- 了解 rviz 三维可视化平台的功能
- 了解 Gazebo 仿真环境
- 掌握 rosbag 数据记录与回放

---

## 4.1 launch 启动文件

### 4.1.1 为什么需要 launch 文件

当系统中的节点数量不断增加时，每个节点开一个终端逐个 `rosrun` 会非常麻烦。**launch 文件**可以一次性启动多个节点，同时自动启动 ROS Master，并实现节点参数配置。

### 4.1.2 基本语法

launch 文件采用 XML 格式，根元素为 `<launch>`：

```xml
<launch>
    <!-- 启动一个节点 -->
    <node pkg="turtlesim" type="turtlesim_node" name="sim1" />
    <node pkg="turtlesim" type="turtlesim_node" name="sim2" />
</launch>
```

**`<node>` 标签**的三个必填属性：

| 属性 | 说明 |
|------|------|
| `pkg` | 节点所在的功能包名称 |
| `type` | 可执行文件名 |
| `name` | 运行时的节点名称（覆盖节点内 `init()` 指定的名称） |

**常用可选属性：**

| 属性 | 说明 |
|------|------|
| `output="screen"` | 标准输出打印到终端屏幕（默认输出到日志文件） |
| `respawn="true"` | 节点停止时自动重启 |
| `required="true"` | 必要节点——该节点终止时，launch 中其他节点也被终止 |
| `ns="namespace"` | 为节点内的相对名称添加命名空间前缀 |
| `args="arguments"` | 节点需要的输入参数 |

### 4.1.3 参数设置

**`<param>`**：设置 ROS 系统运行参数（parameter），存储在参数服务器中：
```xml
<param name="output_frame" value="odom"/>
```

**`<rosparam>`**：批量从 YAML 文件加载参数：
```xml
<rosparam file="$(find pkg)/config/params.yaml" command="load" ns="local_costmap"/>
```

**`<arg>`**：launch 文件内部的局部变量（argument），与 ROS 节点内部无关：
```xml
<arg name="robot_name" default="turtlebot"/>
<param name="robot" value="$(arg robot_name)"/>
```

> `<param>` 和 `<arg>` 的区别：前者是 ROS 系统的全局参数（parameter，运行时节点可通过 `ros::param::get()` 获取），后者是 launch 文件内部变量（argument，仅限 launch 文件使用）。

### 4.1.4 重映射 `<remap>`

不修改原有功能包代码，只将接口名称重映射（要求数据类型相同）：

```xml
<remap from="/turtlebot/cmd_vel" to="/cmd_vel"/>
```

### 4.1.5 嵌套复用 `<include>`

复用一个已有的 launch 文件：
```xml
<include file="$(dirname)/other.launch"/>
```

---

## 4.2 TF 坐标变换

### 4.2.1 什么是 TF

TF（Transform Frame）是 ROS 中管理多个坐标系之间实时变换关系的核心库。它使用**树形数据结构**，根据时间缓冲并维护坐标系之间的变换关系。

机器人系统中常见的坐标系：`world`（世界）、`base_link`（机器人本体）、`laser`（激光雷达）、`camera`（摄像头）、`gripper`（夹爪）等。

TF 的两个核心操作：
- **广播（Broadcast）**：向系统发布坐标系之间的变换关系
- **监听（Listen）**：接收并查询需要的坐标变换

### 4.2.2 TF 调试工具

| 工具 | 功能 |
|------|------|
| `tf_monitor` | 打印 TF 树中所有坐标系的发布状态 |
| `tf_echo <source> <target>` | 查看指定坐标系之间的实时变换关系 |
| `static_transform_publisher` | 发布两个坐标系之间的静态变换 |
| `view_frames` | 生成 PDF 文件可视化整个 TF 树 |
| `rqt_tf_tree` | 运行时可视化 TF 树 |

### 4.2.3 乌龟跟随例程

通过启动两只乌龟，一只跟随另一只运动，展示了 TF 的核心应用：

```bash
roslaunch turtle_tf turtle_tf_demo.launch
```

系统中存在三个坐标系（world → turtle1, world → turtle2），通过 TF 获得 turtle2 相对于 turtle1 的变换，计算角速度和线速度完成跟随控制。

### 4.2.4 TF 广播器与监听器

**广播器核心代码：**
```cpp
tf::TransformBroadcaster br;
tf::Transform transform;
transform.setOrigin(tf::Vector3(msg->x, msg->y, 0.0));
transform.setRotation(tf::Quaternion(0, 0, 0, 1));
br.sendTransform(tf::StampedTransform(transform, ros::Time::now(),
                                        "world", "turtle_name"));
```

**监听器核心代码：**
```cpp
tf::TransformListener listener;
tf::StampedTransform transform;
listener.waitForTransform("/turtle2", "/turtle1", ros::Time(0), ros::Duration(3.0));
listener.lookupTransform("/turtle2", "/turtle1", ros::Time(0), transform);
```

---

## 4.3 Qt 工具箱（rqt）

rqt 是 ROS 基于 Qt 架构的后台图形工具套件：

| 工具 | 功能 | 命令 |
|------|------|------|
| **rqt_console** | 图形化显示和过滤 ROS 日志消息（info/warn/error） | `rqt_console` |
| **rqt_graph** | 图形化显示当前 ROS 系统计算图 | `rqt_graph` |
| **rqt_plot** | 二维数值曲线绘制工具 | `rqt_plot` |
| **rqt_tf_tree** | 可视化 TF 变换树 | `rqt_tf_tree` |
| **rqt_reconfigure** | 动态参数配置 | `rosrun rqt_reconfigure rqt_reconfigure` |

---

## 4.4 rviz 三维可视化平台

rviz 是 ROS 的三维可视化工具，可以实时显示机器人模型、传感器数据、路径规划等。

### 4.4.1 核心概念

- **Display（显示项）**：通过添加不同类型的 Display 订阅 ROS 话题，在三维视图中渲染
- **Fixed Frame（固定参考系）**：所有物体变换到这个坐标系下渲染，通常设为 `base_link` 或 `map`

### 4.4.2 常用 Display

| Display | 订阅数据 | 渲染内容 |
|---------|----------|----------|
| RobotModel | `/robot_description` | URDF 机器人模型 |
| LaserScan | `/scan` | 激光雷达数据点云 |
| Image | `/camera/image_raw` | 摄像头图像 |
| Path | 路径话题 | 导航路径轨迹 |
| Map | `/map` | 占据栅格地图 |
| TF | TF 广播 | 彩色坐标轴 |
| PointCloud2 | `/points` | 三维点云 |

### 4.4.3 rviz 与 Gazebo 的分工

- **Gazebo**：物理仿真，产生数据（激光、图像、里程计）
- **rviz**：数据可视化，展示感知范围、路径和运动状态
- 二者协同形成"仿真-感知-显示"的调试闭环

---

## 4.5 Gazebo 仿真环境

Gazebo 是一个功能强大的开源三维物理仿真平台。

### 4.5.1 核心特点

| 特点 | 说明 |
|------|------|
| 动力学仿真 | 支持 ODE、Bullet、SimBody、DART 多种物理引擎 |
| 三维可视化 | 逼真的光线、纹理、阴影 |
| 传感器仿真 | 支持传感器数据仿真及噪声模拟 |
| 可扩展插件 | 用户自定义插件扩展功能 |
| TCP/IP 传输 | 后台仿真与前台图形通过网络通信，支持远程仿真 |
| 云仿真 | 支持在 Amazon、Softlayer 等云端运行 |

### 4.5.2 构建仿真环境

1. **直接插入模型**：从 Gazebo 模型库选择并放置模型
2. **Building Editor**：手动绘制环境（菜单 Edit → Building Editor）

---

## 4.6 rosbag 数据记录与回放

rosbag 是 ROS 内置的数据记录与回放工具，核心价值在于**场景复现**。

### 4.6.1 三个核心命令

**记录数据：**
```bash
mkdir ~/bagfiles && cd ~/bagfiles
rosbag record -a                    # -a 记录所有话题
rosbag record /scan /tf /odom       # 记录指定话题
```

**查看信息：**
```bash
rosbag info <bagfile>
```

**回放数据：**
```bash
rosbag play <bagfile>               # 正常回放
rosbag play <bagfile> -r 2          # 2倍速回放
rosbag play <bagfile> --clock       # 发布模拟时钟
```

### 4.6.2 典型工作流

1. Gazebo 仿真或实机运行中，用 rosbag 记录 `/scan`、`/odom`、`/tf`、`/cmd_vel` 等话题
2. 离线阶段重放 bag 文件，调整算法参数
3. 对比不同参数配置下的表现，无需重复运行完整仿真

---

## 本章小结

1. **launch 文件**：XML 格式，`<launch>` 根元素 + `<node>` 启动节点，支持参数 `<param>`/`<arg>`、重映射 `<remap>`、嵌套 `<include>`
2. **TF**：树形结构管理坐标变换，广播（Broadcaster）→ 监听（Listener），调试工具包括 `tf_echo`、`view_frames`
3. **rqt**：日志 `rqt_console` + 计算图 `rqt_graph` + 数据绘图 `rqt_plot`
4. **rviz**：三维可视化平台，通过 Display 插件订阅话题渲染数据
5. **Gazebo**：三维物理仿真引擎，与 rviz 协同工作
6. **rosbag**：`record -a`（记录所有）→ `info`（查看）→ `play`（回放）

**随堂例题 4-1**（launch / TF / rviz / rosbag）

1. launch 文件中 `<node>` 标签的三个必须属性是？ A) pkg, type, name  B) pkg, file, node  C) src, exec, id  D) path, name, type
2. TF 的核心功能是？ A) 编译功能包  B) 管理机器人各坐标系之间的变换关系  C) 3D可视化  D) 数据记录
3. RViz 中最容易出错但至关重要的全局配置是？ A) Global Frame  B) Fixed Frame  C) World Frame  D) Base Frame
4. 导航必备 TF 链路是？ A) laser→base_link→odom→map  B) map→odom→base_link→laser  C) base_link→map→odom→laser
5. 记录所有话题数据的 rosbag 命令是？ A) `rosbag save -a`  B) `rosbag capture all`  C) `rosbag record -a`  D) `rosbag collect --all`
6. 对比 RViz 和 Gazebo 的区别（功能、物理引擎、传感器处理三个维度）。

> **答案**：1.A  2.B  3.B  4.B  5.C  6.RViz是可视化工具(无物理引擎)，Gazebo是物理仿真引擎(有重力/碰撞/摩擦)

## 课后练习

1. 编写一个 launch 文件，同时启动两个 turtlesim 仿真器节点，并设置不同的命名空间。
2. TF 中广播器和监听器的作用分别是什么？
3. `rqt_graph` 和 `rqt_plot` 分别有什么用途？
4. rviz 的 Fixed Frame 设置有什么作用？设置不当会出现什么问题？
5. 使用 rosbag 记录小乌龟运动的数据，关闭仿真器后播放 bag 文件复现运动轨迹。
