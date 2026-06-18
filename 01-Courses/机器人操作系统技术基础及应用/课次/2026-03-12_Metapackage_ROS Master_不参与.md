# 机器人操作系统技术基础及应用 — 第8节 (2026-03-12)

**教师**: 鞠锋 | **教室**: 10203 | **周次**: 第2周

---

## 本节主线

延续第1/2章内容，重点深入 **ROS 文件系统**（metapackage 与普通 package 的区别）→ **ROS 开源社区层次结构** → **ROS 三大核心通信机制**（Topic / Service / Parameter），并以对比表格总结 Topic 与 Service 的差异。结尾布置了作业格式要求。

---

## 时间轴

| 时间 | 话题 |
|------|------|
| 00:05–01:00 | 课堂引入（ASR噪声大，内容不连贯） |
| 01:36–04:02 | 回顾常见 ROS 命令（rosrun, roslaunch, roscd, roscp, rosdep 等），强调考试会考"写出5个命令并解释作用" |
| 04:16–05:28 | Metapackage（元功能包）与普通 package 的区别：① 有 `<export>` 标签；② 没有 `<build_depend>`（只有运行时依赖 `run_depend` / `exec_depend`） |
| 05:30–06:50 | 现场演示查看 navigation metapackage 的 package.xml |
| 06:56–08:49 | 用 `roscd` 跳转、`gedit` 查看 metapackage 清单；新版 ROS 用 `exec_depend` 替代 `run_depend` |
| 09:06–12:51 | ROS 开源社区层次：Distribution → Software Repositories → Metapackages → Packages → Nodes / Messages / Services |
| 12:51–14:41 | 过渡到核心通信机制：分布式通信是 ROS 最核心技术，三种机制：Topic（话题）、Service（服务）、Parameter（参数管理） |
| 14:41–17:56 | Topic 通信概览：Publish-Subscribe 模型，Talker 与 Listener 示例 |
| 17:56–24:00 | **Topic 通信 7 步骤详解**（核心考点） |
| 24:00–24:37 | 强调 7 步中前 5 步用 RPC，后 2 步用 TCP — **考试必考** |
| 25:09–26:47 | ROS Master 的作用：仅负责建立连接，不参与数据传输；连接建立后可关闭 Master |
| 26:47–32:29 | **Service 通信 5 步骤详解**（比 Topic 少 2 步，无发布者-订阅者间的 RPC 协商） |
| 32:29–34:00 | **Parameter 管理机制 3 步骤**：全部用 RPC，无 TCP |
| 34:00–35:50 | 参数管理三步：`setParam` → Master 保存 → `getParam` → Master 返回 |
| 35:51–41:30 | Topic vs Service 对比表格：异步/同步、发布订阅/客户端服务器、有/无缓冲区、实时性、节点关系等 |
| 41:30–43:30 | 第二章总结 + **作业格式说明** |
| 43:30–48:35 | 课堂演示：学生刘志远演示 Linux 命令（`cd`、`tab` 补全、箭头翻历史）；尝试下载仿真包失败（疑似网络问题） |
| 48:35–50:03 | 预习第三章：用 C++ 实现 Topic 和 Service 通信 |
| 51:35–54:57 | 课后问答：竞赛相关（校赛 5 月、需上传报告、1/3 出线名额） |

---

## 关键概念

| 概念 | 说明 |
|------|------|
| **Metapackage** | 一种特殊的 package，只打包不编译；有 `<export>` 标签，无 `<build_depend>`，仅有运行时依赖 |
| **ROS Master** | 节点管理器，负责 Topic/Service 的注册与匹配，但**不参与**实际数据传输 |
| **Topic** | 异步、发布-订阅模型；一对多/多对多通信；有缓冲区；实时性弱 |
| **Service** | 同步、客户端-服务器模型；一对多（服务端唯一）；无缓冲区；实时性强 |
| **Parameter Server** | 由 ROS Master 管理的全局变量存储；仅用 RPC 通信（3 步），无 TCP |
| **RPC** | Remote Procedure Call，用于 Topic/Service 建立连接阶段的前几步 |
| **TCP** | Topic 后 2 步 / Service 后 2 步的**实际数据传输**协议 |

---

## 要点详述

### 1. 常见 ROS 命令（考试要求）

**[01:57–04:02]**

> 考试题型：写出 5 个常见命令并解释作用 / 给出功能写命令名 / 给出英文写解释。

| 命令 | 作用 |
|------|------|
| `rosrun` | 运行功能包中的可执行文件 |
| `roslaunch` | 启动 `.launch` 启动文件 |
| `roscd` | 跳转到功能包目录 |
| `roscp` | 复制功能包中的文件 |
| `rosdep` | 自动安装功能包依赖 |

### 2. Metapackage vs 普通 Package

**[04:16–05:28]**

| 区别 | 普通 Package | Metapackage |
|------|-------------|-------------|
| `<export>` 标签 | ❌ 无 | ✅ 有，内含 `<metapackage />` |
| `<build_depend>` | ✅ 有编译依赖 | ❌ 无 |
| 依赖类型 | 编译 + 运行时 | 仅有运行时依赖（`run_depend` / `exec_depend`） |

新版 ROS 中 `run_depend` 已更名为 `exec_depend`。

### 3. Topic 通信 7 步骤（考试核心）

**[17:56–24:37]**

```
Talker (Publisher)              ROS Master               Listener (Subscriber)
      │                              │                          │
① ──►│─── register publisher ──────►│                          │
      │    (advertise, RPC)          │                          │
      │                              │◄──── ② subscribe ───────│
      │                              │    (subscribe, RPC)      │
      │                              │                          │
      │                              │── ③ lookup & send ─────►│
      │                              │    talker addr (RPC)     │
      │                              │                          │
      │◄──── ④ connect request ──────│─────────── (RPC) ────────│
      │    (TCP as param, RPC)       │                          │
      │── ⑤ confirm + send TCP ────►│─────────── (RPC) ────────│
      │    addr (RPC)                │                          │
      │                              │                          │
      │◄──── ⑥ TCP connect ─────────│──────────────────────────│
      │                              │                          │
      │── ⑦ data transfer ─────────►│─────────── (TCP) ────────│
```

**关键要点：**
- **步骤 ① ~ ⑤**：使用 **RPC** 协议（建立连接阶段）
- **步骤 ⑥ ~ ⑦**：使用 **TCP** 协议（实际数据传输阶段）
- Master 仅在 ①~③ 参与，此后可关闭而不影响已建立的通信
- 启动顺序无强制要求

### 4. Service 通信 5 步骤

**[26:47–32:29]**

```
Server (Talker)                   ROS Master               Client (Listener)
      │                              │                          │
① ──►│─── register service ────────►│                          │
      │    (advertiseService, RPC)   │                          │
      │                              │◄──── ② lookup ──────────│
      │                              │    (lookupService, RPC)  │
      │                              │── ③ return addr ──────►│
      │                              │    (RPC)                 │
      │                              │                          │
      │◄──── ④ service request ─────│────────── (TCP) ─────────│
      │── ⑤ service response ──────►│────────── (TCP) ─────────│
```

- 比 Topic 少 2 步（省略了发布者与订阅者之间的 RPC 协商）
- **步骤 ① ~ ③**：RPC；**步骤 ④ ~ ⑤**：TCP
- 服务端唯一，客户端可多个

### 5. Parameter 管理 3 步骤

**[32:29–35:50]**

```
Node (Talker)                     ROS Master                Node (Listener)
      │                              │                          │
① ──►│─── setParam(key, value) ────►│                          │
      │    (RPC)                     │                          │
      │                              │── ② getParam(key) ─────►│
      │                              │    (RPC)                 │
      │                              │── ③ return value ──────►│
      │                              │    (RPC)                 │
```

- **全部 3 步都用 RPC**，无 TCP
- Parameter Server 完全由 ROS Master 管理

### 6. Topic vs Service 对比

**[35:51–41:30]**

| 对比维度 | Topic | Service |
|---------|-------|---------|
| 同步性 | **异步** | **同步** |
| 通信模型 | **发布-订阅** | **客户端-服务器** |
| 底层协议 | TCP/UDP | TCP/UDP |
| 反馈机制 | ❌ 无反馈 | ✅ 有应答 |
| 缓冲区 | ✅ 有 | ❌ 无 |
| 实时性 | 弱 | 强 |
| 节点关系 | 多对多 | 服务端唯一，客户端多 |
| 适用场景 | 不断更新、少量逻辑的数据传输 | 数据量小、强逻辑处理的数据交换 |

**6 个关键术语的对应关系不要搞混：**

- **Topic** → **Publish / Subscribe** → **异步**
- **Service** → **Client / Server** → **同步**

### 7. 作业格式要求

**[42:01–43:30]**

- 每人一个文件，模板发在群中
- 文件内包含：**班级、学号、姓名**
- 每次作业另起一页：标题为"第X次作业" + 日期 + 章号
- 先抄题目，再写解答（可能含编程）
- **不要分多个文件**，一个人一个文件内多页
- 最终转 **PDF** → 汇总给班长 → 班长发教师邮箱
- 计入平时分；认真做对考试有帮助

---

## 作业 / 考试 / 通知

| 类型 | 内容 |
|------|------|
| **考试** | ① 写 5 个常见 ROS 命令并解释作用 ② Metapackage 判断（有无 export、有无 build_depend） ③ Topic 7 步骤（哪几步用 RPC / 哪几步用 TCP / 画出图 / 写出每步作用） ④ Service 5 步骤 ⑤ 6 个术语对应关系（Topic-发布订阅-异步 / Service-客户端服务器-同步） |
| **作业** | 后续 1–2 次作业。格式见上。最终交 PDF。 |
| **通知** | ① 回去尝试安装仿真包，有结果反馈 ② 下节课进入第三章（Topic / Service 的 C++ 实现） |

---

## 待核对

以下为 ASR 识别可能不准确之处，需对照录音/课件确认：

| 时间 | 原文（ASR） | 疑点 |
|------|------------|------|
| 00:05–01:00 | 整段"成果的会议"、"党委去"、"我们的朋友" | ASR 完全混乱，内容不明 |
| 03:14 | "俄罗斯lunch" | 应为 **roslaunch** |
| 03:18 | "lunch点lunch文件" | 应为 **launch 文件** |
| 05:06 | "软体配对" / "标配" | 前者可能为 **runtime dependency**，后者可能为 **build dependency** |
| 06:51 | "gji" | 应为 **gedit** |
| 15:33–15:40 | "300 400多" / "pop球" | 可能为 **publisher** / **subscriber** 的模糊识别 |
| 17:04 | "操可注册" | 应为 **Talk 注册** |
| 23:08 | "丁院长" | 可能为 **Listener** 的误识别 |
| 23:28, 24:00 | "GDP" | 应为 **TCP** |
| 34:35 | "c program" | 应为 **setParam** |
| 35:14 | "get" | 应为 **getParam** |
| 45:37–46:00 | "谁知变"、"回到我的后面" | Linux 命令演示，具体指令被噪声覆盖 |
| 51:35–54:57 | 整段课后问答 | ASR 质量差，关于竞赛、成绩等话题内容模糊 |

---

## 回看建议

- **[01:57–04:02]** 命令复习 — 必考，建议整理完整列表
- **[04:16–05:28]** Metapackage 区别 — 概念性考点，注意 `export` 和 `build_depend` 缺省
- **[17:56–24:37]** Topic 7 步 — **最重要考点**，建议能默画流程图并标注每步协议类型
- **[26:47–32:29]** Service 5 步 — 与 Topic 对比记忆，注意仅 5 步的原因
- **[32:29–35:50]** Parameter 3 步 — 简单但容易被忽略
- **[35:51–41:30]** 对比表格 — 综合理解两种机制的差异