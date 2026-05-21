# 第3章 ROS编程基础

## 学习目标

- 掌握工作空间的创建、编译和覆盖机制
- 掌握功能包的创建和目录结构
- 能够编写 Publisher 和 Subscriber 的 C++ 代码
- 能够编写 Server 和 Client 的 C++ 代码
- 理解自定义话题消息和服务数据的定义方法
- 掌握 ROS 的命名空间类型及解析规则
- 了解分布式多机通信的配置方法

---

## 3.1 第一个例程——小乌龟仿真回顾

```bash
roscore                                # 启动 ROS Master
rosrun turtlesim turtlesim_node         # 启动仿真器节点
rosrun turtlesim turtle_teleop_key      # 启动键盘控制节点
```

小乌龟例程中：

- **话题**：`/turtle1/cmd_vel`（速度控制指令，类型 `geometry_msgs/Twist`）
- **服务**：`/spawn`（新增乌龟）、`/kill`（删除乌龟）、`/clear`（清除轨迹）
- **参数**：`turtlesim` 节点提供背景色等可配置参数

> 这个看似简单的例程，**蕴含了 ROS 最基础的原理和机制**——节点、话题、服务、参数。

---

## 3.2 工作空间与功能包

### 3.2.1 工作空间（Workspace）

工作空间是存放 ROS 工程开发相关文件的文件夹。Catkin 编译系统下的标准工作空间包含四个目录：

| 目录 | 名称 | 作用 |
|------|------|------|
| **src** | 代码空间 | 存放所有功能包的源码文件（**最常用**） |
| **build** | 编译空间 | 存放编译过程中的缓存和中间文件（自动生成） |
| **devel** | 开发空间 | 存放编译生成的可执行文件和环境变量脚本 |
| **install** | 安装空间 | 执行 `catkin_make install` 后生成，用于部署分发 |

**创建和编译工作空间：**

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace          # 初始化工作空间
cd ~/catkin_ws
catkin_make                    # 编译工作空间
source devel/setup.bash        # 设置环境变量
echo $ROS_PACKAGE_PATH         # 验证环境变量
```

> 每次打开新终端都需要重新 `source devel/setup.bash`。可以将此命令写入 `~/.bashrc` 使其在所有终端中自动生效。

### 3.2.2 功能包（Package）

功能包是 ROS 软件的基本单元，所有功能包必须平行放置在 `src/` 目录下，**不允许嵌套**。

**创建功能包：**

```bash
cd ~/catkin_ws/src
catkin_create_pkg <package_name> [depend1] [depend2] ...
```

示例：
```bash
catkin_create_pkg learning_communication std_msgs rospy roscpp
```

标准功能包目录结构：

| 目录/文件 | 存放内容 |
|-----------|----------|
| `src/` | C++ 源代码 |
| `scripts/` | Python 脚本 |
| `include/` | 头文件 (.h) |
| `launch/` | 启动文件 (.launch) |
| `msg/` | 自定义消息类型 (.msg) |
| `srv/` | 自定义服务类型 (.srv) |
| `config/` | 配置文件 |
| `package.xml` | 功能包清单（元信息，**必须**） |
| `CMakeLists.txt` | 编译规则（**必须**） |

### 3.2.3 工作空间的覆盖（⚠️ 考试必考）

ROS 允许多个工作空间并存。当设置新的工作空间环境变量后，其路径会追加到 `ROS_PACKAGE_PATH` 的**最前端**。

**覆盖规则：**
- ROS 从前往后查找功能包，找到第一个匹配即停止
- 不同工作空间可以存在同名功能包
- 同一工作空间内**不允许**存在同名功能包

```
ROS_PACKAGE_PATH:
  ~/my_ws/devel          ← 优先查找（最前端）
  /opt/ros/noetic/share  ← 系统默认
```

**优势**：可以在自己工作空间创建同名包"覆盖"系统包，无需修改源码。

**风险**：如果功能包 B 依赖同空间的 A，但前面的工作空间有同名的 A 覆盖了原来的 A，可能导致 B 调用错误的依赖。

**随堂例题 3-1**（工作空间与功能包）

1. 工作空间 4 个目录中，存放源代码的是？存放可执行文件的是？ A) src/devel  B) build/install  C) src/devel... 
2. 编译 ROS 工作空间的命令是？ A) `cmake`  B) `colcon build`  C) `catkin_build`  D) `catkin_make`
3. 创建功能包的正确命令是？ A) `catkin_make pkg pkg_name`  B) `catkin_create_pkg pkg_name [deps]`  C) `roscreate pkg pkg_name`
4. 工作空间覆盖的查找方式是？ A) 从后往前  B) 从前往后，找到即停  C) 全部查找选最新版本
5. 同一工作空间内是否允许同名功能包？ A) 允许  B) 不允许

> **答案**：1.A  2.D  3.B  4.B  5.B

---

## 3.3 话题编程：Publisher 与 Subscriber

### 3.3.1 Publisher 的实现流程

```
① 初始化 ROS 节点：ros::init(argc, argv, "talker")
② 创建节点句柄：ros::NodeHandle n
③ 注册 Publisher：n.advertise<msg_type>("topic_name", queue_size)
④ 设置循环频率：ros::Rate loop_rate(10)  // 10Hz
⑤ 发布消息：chatter_pub.publish(msg)
⑥ 处理回调：ros::spinOnce()
⑦ 按频率休眠：loop_rate.sleep()
```

**核心代码示例**（发布 "Hello World" 到 chatter 话题）：

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"

int main(int argc, char **argv) {
    ros::init(argc, argv, "talker");           // 初始化节点
    ros::NodeHandle n;                          // 创建句柄
    ros::Publisher pub = n.advertise<std_msgs::String>("chatter", 1000);
    ros::Rate loop_rate(10);                    // 10Hz

    int count = 0;
    while (ros::ok()) {
        std_msgs::String msg;
        std::stringstream ss;
        ss << "hello world " << count;
        msg.data = ss.str();
        pub.publish(msg);                       // 发布消息
        ros::spinOnce();                        // 处理回调
        loop_rate.sleep();                      // 按频率休眠
        ++count;
    }
    return 0;
}
```

### 3.3.2 Subscriber 的实现流程

```
① 初始化 ROS 节点：ros::init(argc, argv, "listener")
② 创建节点句柄：ros::NodeHandle n
③ 注册 Subscriber：n.subscribe("topic_name", queue_size, callback)
④ 进入循环等待：ros::spin()
⑤ 在回调函数中处理消息数据
```

**核心代码示例**（订阅 chatter 话题并打印）：

```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"

void chatterCallback(const std_msgs::String::ConstPtr& msg) {
    ROS_INFO("I heard: [%s]", msg->data.c_str());
}

int main(int argc, char **argv) {
    ros::init(argc, argv, "listener");
    ros::NodeHandle n;
    ros::Subscriber sub = n.subscribe("chatter", 1000, chatterCallback);
    ros::spin();    // 进入循环，等待消息到达
    return 0;
}
```

### 3.3.3 编译配置（CMakeLists.txt）

```cmake
# 设置需要编译的代码和生成的可执行文件
add_executable(talker src/talker.cpp)
add_executable(listener src/listener.cpp)

# 设置链接库
target_link_libraries(talker ${catkin_LIBRARIES})
target_link_libraries(listener ${catkin_LIBRARIES})

# 设置依赖（若涉及自定义消息）
add_dependencies(talker ${PROJECT_NAME}_generate_messages_cpp)
```

四个关键编译配置项：

| 配置项 | 作用 |
|--------|------|
| `include_directories` | 设置头文件相对路径 |
| `add_executable` | 设置编译的代码和生成的可执行文件名 |
| `target_link_libraries` | 设置链接库 |
| `add_dependencies` | 设置编译依赖 |

### 3.3.4 运行

```bash
cd ~/catkin_ws
catkin_make
source devel/setup.bash
roscore                                    # 终端1
rosrun learning_communication talker       # 终端2
rosrun learning_communication listener     # 终端3
```

**随堂例题 3-3**（Publisher/Subscriber 代码填空）

补全以下 Publisher 的关键代码行：

```cpp
#include "ros/ros.h"
int main(int argc, char **argv) {
    // ① 初始化 ROS 节点，命名为 "talker"
    _________________________________________

    ros::NodeHandle n;
    // ② 向 Master 注册 Publisher，向话题 "chatter" 发布 String，缓冲区 1000
    ros::Publisher pub = _________________________________________;

    ros::Rate loop_rate(10);
    while (ros::ok()) {
        std_msgs::String msg;
        pub.______________;   // ③ 发布消息
        ros::spinOnce();      // ④ 处理回调
        loop_rate._________;  // ⑤ 按频率休眠
    }
    return 0;
}
```

CMakeLists.txt 中编译声明：
```cmake
______________(talker src/talker.cpp)     // ⑥ 声明可执行文件
______________(talker ${catkin_LIBRARIES}) // ⑦ 设置链接库
```

> **答案**：① `ros::init(argc, argv, "talker");`  ② `n.advertise<std_msgs::String>("chatter", 1000);`  ③ `publish(msg)`  ④ (已给出)  ⑤ `sleep()`  ⑥ `add_executable`  ⑦ `target_link_libraries`

---

## 3.4 自定义话题消息

### 3.4.1 定义 .msg 文件

在功能包 `msg/` 目录下创建，例如 `Person.msg`：

```
string name
uint8  sex
uint8  age

uint8 unknown = 0
uint8 male    = 1
uint8 female  = 2
```

### 3.4.2 编译配置

**package.xml** 中添加：
```xml
<build_depend>message_generation</build_depend>
<run_depend>message_runtime</run_depend>
```

**CMakeLists.txt** 中修改：
```cmake
find_package(catkin REQUIRED COMPONENTS
  roscpp rospy std_msgs message_generation  # 添加 message_generation
)

add_message_files(FILES Person.msg)
generate_messages(DEPENDENCIES std_msgs)

catkin_package(
  CATKIN_DEPENDS message_runtime ...
)
```

---

## 3.5 服务编程：Server 与 Client

### 3.5.1 自定义服务数据（.srv）

在功能包 `srv/` 目录下创建，例如 `AddTwoInts.srv`：

```
int64 a
int64 b
---
int64 sum
```

`---` 上方为请求数据，下方为应答数据。

### 3.5.2 Server 的实现流程

```
① 初始化 ROS 节点
② 创建 Server 实例：n.advertiseService("service_name", callback)
③ 循环等待服务请求，进入回调函数
④ 在回调函数中完成服务功能的处理，反馈应答数据
⑤ 返回 true
```

**核心代码示例：**

```cpp
#include "ros/ros.h"
#include "learning_communication/AddTwoInts.h"

bool add(learning_communication::AddTwoInts::Request  &req,
         learning_communication::AddTwoInts::Response &res) {
    res.sum = req.a + req.b;
    ROS_INFO("request: x=%ld, y=%ld", (long int)req.a, (long int)req.b);
    ROS_INFO("sending back response: [%ld]", (long int)res.sum);
    return true;
}

int main(int argc, char **argv) {
    ros::init(argc, argv, "add_two_ints_server");
    ros::NodeHandle n;
    ros::ServiceServer service = n.advertiseService("add_two_ints", add);
    ROS_INFO("Ready to add two ints.");
    ros::spin();
    return 0;
}
```

### 3.5.3 Client 的实现流程

```
① 初始化 ROS 节点
② 创建 Client 实例：n.serviceClient<srv_type>("service_name")
③ 实例化服务数据类型，填充 request
④ 发布服务请求：client.call(srv)——阻塞等待应答
⑤ 处理 response
```

**核心代码示例：**

```cpp
#include "ros/ros.h"
#include "learning_communication/AddTwoInts.h"

int main(int argc, char **argv) {
    ros::init(argc, argv, "add_two_ints_client");
    ros::NodeHandle n;
    ros::ServiceClient client = n.serviceClient<learning_communication::AddTwoInts>("add_two_ints");

    learning_communication::AddTwoInts srv;
    srv.request.a = atoll(argv[1]);
    srv.request.b = atoll(argv[2]);

    if (client.call(srv)) {
        ROS_INFO("Sum: %ld", (long int)srv.response.sum);
    } else {
        ROS_ERROR("Failed to call service");
        return 1;
    }
    return 0;
}
```

### 3.5.4 编译与运行

```bash
# 编译
add_executable(server src/server.cpp)
target_link_libraries(server ${catkin_LIBRARIES})
add_executable(client src/client.cpp)
target_link_libraries(client ${catkin_LIBRARIES})

# 运行
rosrun learning_communication server
rosrun learning_communication client 3 5   # 输出 Sum: 8
```

---

## 3.6 ROS 中的命名空间

### 3.6.1 四种命名类型

| 类型 | 格式 | 示例 | 解析为全局名称（假设默认 ns 为 `/wg`，节点为 `node1`） |
|------|------|------|------|
| **基础名称** | 无前缀 | `name` | `/wg/name` |
| **全局名称** | `/` 开头 | `/global/name` | `/global/name`（不变化） |
| **相对名称** | 不含 `/` 开头 | `relative/name` | `/wg/relative/name` |
| **私有名称** | `~` 开头 | `~private/name` | `/wg/node1/private/name` |

> 相对名称解析基于**节点的命名空间**（不含节点名本身），私有名称则用**节点的全局名称**作为命名空间。

### 3.6.2 设置默认命名空间

1. **命令行参数**：`__ns:=default-namespace`
2. **launch 文件**：`<node ns="sim1" ...>`
3. **环境变量**：`export ROS_NAMESPACE=default-namespace`

### 3.6.3 命名重映射

在启动节点时改名，避免命名冲突：

```bash
rosrun rospy_tutorials talker chatter:=/wg/chatter
```

> **注意**：ROS 的命名解析在命名重映射**之前**发生。

**随堂例题 3-2**（命名空间与多机通信）

1. 命名空间 `~`（波浪号）表示什么类型？ A) 全局名称  B) 相对名称  C) 私有名称  D) 基础名称
2. 节点 `/wg/node1` 内部使用相对名称 `node2`，解析结果是？ A) `/node1/node2`  B) `/wg/node2`  C) `/node2`  D) `/wg/node1/node2`
3. 分布式通信中，从机需要设置哪个环境变量指向主机的 Master？ A) `ROS_HOSTNAME`  B) `ROS_MASTER_URI`  C) `ROS_ROOT`  D) `ROS_PACKAGE_PATH`
4. launch 文件中启动节点的 `<node>` 标签三个必须属性是？ A) pkg, type, name  B) pkg, file, node  C) src, exec, id

> **答案**：1.C  2.B  3.B  4.A

---

## 3.7 分布式多机通信

ROS 允许节点分布在多台机器上运行，系统中只能有一个 Master。

### 配置步骤

**1. 确保所有计算机在同一网络**，使用 `ifconfig` 查看 IP，用 `/etc/hosts` 添加对方的 IP 和主机名。

**2. 在从机上设置 Master 位置：**

```bash
export ROS_MASTER_URI=http://主机的IP:11311
```

**3. 测试：**

```bash
# 主机
roscore
rosrun turtlesim turtlesim_node

# 从机
rostopic list              # 应能看到主机的话题列表
rostopic pub -r 10 /turtle1/cmd_vel geometry_msgs/Twist ...  # 控制主机上的乌龟
```

> 建立连接后关掉 Master 不影响已有连接的数据传输，但新节点无法加入。

---

## 本章小结

1. **工作空间**四个目录：src → build → devel → install
2. **功能包**创建：`catkin_create_pkg`，平行放置不嵌套
3. **工作空间覆盖**：`ROS_PACKAGE_PATH` 从前往后查找，优先匹配最前面的路径
4. **Publisher**：init → 注册 Publisher → publish → spinOnce → sleep
5. **Subscriber**：init → subscribe（含回调函数） → spin
6. **Server**：init → advertiseService → 回调函数处理并返回 true
7. **Client**：init → 创建 Client 实例 → 填充 request → call（阻塞等待）
8. **命名空间**四种：基础、全局、相对、私有
9. **多机通信**：设置 `ROS_MASTER_URI` 指向主机即可

## 课后练习

1. 编写 Publisher 节点发布速度指令，控制小乌龟做圆周运动。
2. 编写 Subscriber 节点订阅小乌龟的位姿话题并实时打印其位置。
3. 编写服务 Server 和 Client，实现运算 `f = (a + b) * c`。写出完整的 srv 文件、Server 代码、Client 代码和终端命令。
4. 简述工作空间覆盖机制，并说明其利弊。
5. 节点 `/robot1/sensor` 内使用私有名称 `~rate`，解析后的全局名称是什么？
