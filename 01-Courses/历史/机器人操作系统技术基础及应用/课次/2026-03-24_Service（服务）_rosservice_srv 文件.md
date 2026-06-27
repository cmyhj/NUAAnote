# 第8节 · ROS服务（Service）通信机制

## 本节主线

1. 回顾自定义消息 → 2. ROS Service 概念与命令行交互（turtlesim 实操） → 3. 自定义服务数据（srv 文件定义） → 4. 服务编程：Server 与 Client 代码结构与流程 → 5. 综合性大作业说明与分组协调

## 时间轴

| 时间段 | 内容 |
|--------|------|
| 00:00–00:06 | 调试小车、场地测试等课前闲聊 |
| 00:14–03:14 | 考研/保研建议：做科创（实物或仿真），ROS 仿真可作为复试经历；强调综合性大作业的重要性 |
| 03:16–04:46 | 回顾自定义消息：排查 tab/空格缩进问题，`rosmsg show` 查看自定义类型 |
| 04:57–05:12 | 自定义消息是往年考试必考内容 |
| 05:12–05:26 | 引入 Service（客户端-服务器）通信方式 |
| 05:27–06:08 | turtlesim 自带服务：无需编程，用命令行交互控制小乌龟 |
| 06:09–06:23 | 书上例子代码有问题，教师已更新（右上角版本正确） |
| 06:24–07:49 | `rosservice list` 列出 turtlesim 所有可用服务 |
| 07:50–08:57 | 逐项解释服务：`/clear`、`/kill`、`/spawn`、`/set_pen`、`/teleport_absolute`、`/teleport_relative` 等 |
| 08:59–11:12 | 演示 `/clear`：键盘控制乌龟移动画轨迹 → `rosservice call /clear` 清除轨迹 |
| 11:12–11:45 | 可用服务通信写控制节点作为大作业保底方案（但分数不高） |
| 11:53–18:43 | 演示 `/spawn`：查看服务格式 → 生成新乌龟（坐标、角度、命名）；坐标原点在左下角；角度单位为弧度 |
| 18:44–20:25 | 键盘控制只能控制第一只乌龟，生成的其他乌龟未绑定键盘 |
| 20:26–20:59 | 命令行交互可用于调试：先试效果再写 C++ 代码 |
| 21:00–21:32 | 自定义服务数据（srv）——考试比自定义消息更常见，因为涵盖消息内容并扩展 |
| 21:34–22:25 | srv 文件：位于功能包根目录下的 `srv/` 文件夹，文件名后缀 `.srv` |
| 22:26–24:55 | srv 文件格式：请求域 + `---` 分隔线 + 应答域；两个数据域（请求 & 应答） |
| 24:57–26:35 | 示例 `AddTwoInts.srv`：`int64 a` / `int64 b` → `---` → `int64 sum` |
| 26:40–28:49 | 编译配置：`package.xml` 依赖 `message_generation` / `message_runtime`；`CMakeLists.txt` 添加 `add_service_files()` |
| 28:54–30:24 | 服务编程正题：加法器示例——Server 接收 a、b，返回 sum |
| 30:25–41:42 | **Server 代码结构**：① 引入头文件 → ② `ros::init()` 初始化 → ③ 创建 `ros::ServiceServer` 实例（`nh.advertiseService("add_ints", callback)`）→ ④ `ros::spin()` 循环等待 → ⑤ 回调函数中接收 `request` & 填写 `response` |
| 41:43–50:05 | **Client 代码结构**：① `ros::init()` → ② 创建 `ros::ServiceClient` 实例（`nh.serviceClient<AddTwoInts>("add_ints")`）→ ③ 实例化 srv 变量，从命令行参数 `argv[1]` `argv[2]` 赋值 `request.a` `request.b` → ④ `client.call(srv)` 阻塞调用 → ⑤ 检查返回值，从 `srv.response.sum` 获取结果 |
| 50:06–51:12 | Client 比之前示例复杂：需判断命令行参数数量、定义服务实例并赋值、调用服务并处理结果 |
| 51:37–54:57 | 综合性大作业分组讨论：建议 4–5 人；教师后续协调；单人组较另类但非强制；个人可考虑带同学做文书工作 |

## 关键概念

| 概念 | 说明 |
|------|------|
| **Service（服务）** | ROS 中的另一种通信方式，客户端-服务器模型（Client-Server），含请求（Request）与应答（Response）两个数据域 |
| **rosservice** | 与 Service 交互的命令行工具：`rosservice list`、`rosservice call`、`rosservice type`、`rosservice info` |
| **srv 文件** | 自定义服务数据定义文件，存放在功能包 `srv/` 目录下，后缀 `.srv`，用 `---` 分隔请求域与应答域 |
| **ServiceServer** | 服务器端类，通过 `nh.advertiseService()` 注册，绑定回调函数，用 `ros::spin()` 等待请求 |
| **ServiceClient** | 客户端类，通过 `nh.serviceClient<Type>()` 创建，用 `client.call(srv)` 发起请求（阻塞） |
| **节点名 vs 服务名** | Server 与 Client 的节点名不同，但通信用的服务名称必须相同（由 ROS Master 配对） |

## 要点详述

### 1. 命令行调用服务（turtlesim 实操）

- **列出服务：** `rosservice list`
- **调用清除轨迹：** `rosservice call /clear` （需先启动 turtlesim_node 并用键盘画线）
- **生成乌龟：** 先 `rosservice type /spawn` 查看格式 → `rosservice call /spawn "{x: 0, y: 0, theta: 0, name: 't1'}"` — `x`、`y` 为坐标（原点在左下角），`theta` 为弧度（如 1.5 ≈ 90°, 3.14 ≈ 180°），`name` 为乌龟名
- **`rosservice call` 支持 tab 补全**，参数格式为 YAML 字典
- 键盘控制默认绑定第一只乌龟 `/turtle1`，`/spawn` 生成的其他乌龟不会自动绑定

### 2. 自定义服务数据（srv）

```
# AddTwoInts.srv
int64 a
int64 b
---
int64 sum
```

- **请求域**（`---` 上方）：客户端发送给服务器的数据
- **应答域**（`---` 下方）：服务器处理完成后返回给客户端的数据
- **文件位置：** `功能包根目录/srv/文件名.srv`
- **编译配置：** `CMakeLists.txt` 中添加 `add_service_files(FILES AddTwoInts.srv)`；`package.xml` 依赖 `message_generation` 与 `message_runtime`（与自定义消息相同）

### 3. Server 端编程要点

```cpp
// 回调函数签名
bool add(AddTwoInts::Request &req, AddTwoInts::Response &res) {
    res.sum = req.a + req.b;
    return true;
}
// 注册服务
ros::ServiceServer service = nh.advertiseService("add_ints", add);
ros::spin();
```

- 创建 `ServiceServer` 用 `advertiseService(服务名, 回调函数)`
- 回调函数通过 `req` 指针（request）获取请求数据，通过 `res`（response）填写应答数据
- 必须返回 `true` 表示处理成功
- `ros::spin()` 循环等待，收到请求时自动调用回调

### 4. Client 端编程要点

```cpp
ros::ServiceClient client = nh.serviceClient<AddTwoInts>("add_ints");
AddTwoInts srv;
srv.request.a = atoll(argv[1]);  // 字符串 → 长整型
srv.request.b = atoll(argv[2]);
if (client.call(srv)) {
    ROS_INFO("Sum: %ld", (long int)srv.response.sum);
} else {
    ROS_ERROR("Failed to call service add_ints");
}
```

- 创建 `ServiceClient` 用 `serviceClient<类型>(服务名)`，**不需要回调函数**
- 服务名必须与 Server 注册的名称一致
- `client.call(srv)` 是**阻塞调用**，返回 `true` 表示成功
- 通过 `srv.response.成员` 获取结果
- Client 需额外处理命令行输入：`atoll()` 将字符串转为长整型

### 5. 节点名 vs 服务名

| 名称 | Server | Client |
|------|--------|--------|
| 节点名 | `add_two_ints_server` | `add_two_ints_client` |
| 服务名（通信用） | `add_ints` | `add_ints`（必须一致） |

## 作业考试通知

- **自定义消息** 为往年必考题型，要求编写 `.msg` 文件
- **自定义服务** 考试出现频率更高（内容更全面：请求 + 应答）
- **综合性大作业：**
  - 建议尽早组队，4–5 人一组
  - 保底方案：使用 turtlesim 仿真器 + 自己写控制节点（但分数不高）
  - 教师提醒大作业也可作为复试经历
  - 课后报名，教师后续协调分组

## 待核对

- [ ] [ASR模糊] 03:16–03:34 中教师排查自定义消息缩进问题：原文 "敲了一个tab"、"敲了两个空子" → 实际应是缩进用了 tab 而非空格，导致编译报错
- [ ] [ASR模糊] 12:23–13:30 中命令格式讲解部分：`rosservice type /spawn` 管道符和参数描述因教师口述和复制粘贴问题，表达断断续续
- [ ] 书上旧版示例代码有错误，教师已更新——确认教材版本与教师提供的右上角版本一致
- [ ] 坐标原点位置：通过实验（在 (0,0) 生成乌龟只露出 1/4 个）确定在左下角，但教师表示"不记得是左下角还是左上角"（14:47–14:55）
- [ ] 键盘控制绑定问题：教师提到可能重启键盘节点会改变控制对象（19:58–20:25），但未进一步验证
- [ ] `CMakeLists.txt` 中 `add_service_files()` 的具体放置位置和格式可对照 `add_message_files()` 确认

## 回看建议

- **重点回看 05:12–11:12**：Service 概念引入 + turtlesim 命令行实操演示（最关键的第一印象）
- **重点回看 21:34–28:49**：srv 文件定义格式与编译配置（考试高频考点）
- **重点回看 30:25–42:00**：Service 编程——Server 和 Client 完整流程
- **快进跳过 00:00–03:14**：考研/保研闲聊，与大作业相关但非课堂技术内容
- **快进跳过 51:37–54:57**：分组协调，非技术内容