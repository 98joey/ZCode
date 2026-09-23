# Agent 源码学习范式 v2

> 源码是标本，不是毛坯房。  
> 问题是导航，实验是手段，重建是检验，设计能力才是最终成果。

这份文档用于约束我在研究 Agent 前端、Client、RPC、Server、Runtime 等源码时的学习方式。

目标不是“把某个项目学完”，也不是把开源项目改造成自己的项目，而是：

> **通过观察、假设、实验、破坏、重建和迁移，逐步形成独立设计 Web Agent 系统的能力。**

---

# 一、核心原则

整个学习过程遵循两个核心循环。

## 1. 基础认知循环

```text
观察 → 假设 → 微改 → 验证 → 复原 → 总结
```

重点不是“看源码”，而是不断修正自己对系统的理解。

---

## 2. 深度学习循环

对于真正值得深挖的核心机制：

```text
Concept → Hypothesis → Trace → Break → Rebuild → Transfer → Design
```

即：

1. **Concept**：这个机制解决什么问题？
2. **Hypothesis**：在读源码之前，我猜它是怎么工作的？
3. **Trace**：真实调用链是什么？
4. **Break**：如果故意破坏它，会发生什么？
5. **Rebuild**：使用同技术栈重新实现一个最小版本
6. **Transfer**：如果值得迁移，再用目标技术栈重新实现
7. **Design**：形成未来自己系统里的设计判断

不是所有章节都必须完整走完这条链。

> **Stop 是合法结果。**

---

# 二、先判断：这个机制值得研究到什么深度？

源码学习最大的风险之一，是把所有机制都当成同等重要。

必须先做价值分级。

---

## S 级：核心机制

这些机制直接决定未来是否有能力独立设计 Web Agent。

原则上研究到 **Level 5**。

典型包括：

```text
Streaming
RPC / Event Protocol
Session
Tool Call
Tool Approval
Reconnect / Recovery
Workspace
Agent State Synchronization
```

推荐流程：

```text
Concept
  ↓
Hypothesis
  ↓
Trace
  ↓
Break
  ↓
Reproduction Playground
  ↓
Transfer Playground
  ↓
Design
```

---

## A 级：重要机制

这些机制对完整 Agent 产品很重要，但未必值得双栈重写。

通常研究到 **Level 3～4**。

例如：

```text
State Projection
Terminal
Diff
Persistence
Task Lifecycle
Command Routing
Error Handling
Optimistic Update
```

推荐流程：

```text
Concept
  ↓
Trace
  ↓
Break
  ↓
必要时 Rebuild
  ↓
Stop
```

---

## B 级：外围机制

这些代码可能对项目本身重要，但对“Agent System Design”学习价值有限。

通常研究到 **Level 1～2** 即可。

例如：

```text
主题
i18n
普通设置页面
营销页
分享页
常规表单
普通 UI Components
```

推荐流程：

```text
快速理解
  ↓
确认无 Agent 特异性
  ↓
Stop
```

---

## 裁剪原则

每次开始研究前问：

> 这个机制对我未来自建 Web Agent 有什么意义？

如果回答不出来，不进入深度学习流程。

不要为了源码覆盖率学习。

---

# 三、Chapter 0：先建立实验室

在任何源码精读之前，必须先完成：

# Bootstrap & Observability

原则：

> **不能观察的系统，不适合做源码实验。**

---

## 1. 能安装

完成：

```text
clone
package manager install
env
build dependencies
```

确认依赖可以稳定安装。

---

## 2. 能运行

必须跑通：

```text
Web
Server
Agent Runtime
必要的 Auth / Model
```

最低要求：

> 能稳定完成一次真实 Agent 对话。

---

## 3. 能复现

必须有一个固定测试 Prompt。

例如：

```text
读取 package.json，并告诉我项目名称
```

后续所有实验尽量使用同一个输入。

这样才能比较行为差异。

---

## 4. 能观察

建立前端实验必需的观察能力。

### Browser DevTools

重点使用：

```text
Network
WebSocket Frames
Console
Sources
Performance
Application
```

---

### React DevTools

用于观察：

```text
Component Tree
Props
State
Render
```

如果项目状态管理支持 DevTools，也应开启。

---

### 日志插桩

允许在 Lab 中增加：

```ts
console.log()
console.trace()
performance.mark()
event logger
store subscriber
```

重点观察：

```text
Event 什么时候产生？
Event 什么时候进入 Client？
什么时候写 Store？
什么时候触发 Render？
```

---

## 5. 建立 Baseline

在开始修改前记录：

```text
一次正常请求产生了哪些 network 请求？
哪些 websocket frames？
哪些 store mutation？
哪些组件 render？
```

这是所有 Break 实验的对照组。

---

# 四、七点源码学习范式

## 1. 永远保留一个不可污染的基线

`main` 只用于跟踪 upstream。

不要直接在原始源码上长期实验。

推荐使用：

```text
Original
Lab
Playground
```

例如：

```text
zcode/
zcode-lab-stream/
zcode-lab-approval/
agent-study/
```

使用：

```bash
git worktree add ../zcode-lab-stream -b lab/message-stream
```

实验失败后：

```bash
git worktree remove ../zcode-lab-stream
git branch -D lab/message-stream
```

原则：

> **源码可以破坏，但基线不能污染。**

---

## 2. 每次研究一个“问题”，而不是一个“目录”

不要：

```text
今天看 packages/ui
明天看 packages/client
```

应该问：

```text
用户点击 Send 后，第一个状态变化在哪里？

Message Delta 如何进入 Store？

Session 的 source of truth 在哪里？

WebSocket 断开后如何恢复？

Tool Approval 是同步阻塞，还是异步状态机？
```

目录只是线索。

问题才是导航。

---

## 3. 读代码之前，先下注

不要让源码直接把答案喂给自己。

先写自己的假设。

例如：

> 我认为 Server 会产生有序 Event，Client 只负责 transport，Store 根据 Event 构建 UI projection。

然后再追踪源码。

记录：

```text
Hypothesis
Observed
Difference
Updated Model
```

原则：

> **源码阅读不是接受答案，而是修正自己的系统模型。**

---

## 4. 修改必须是“微创实验”

学习源码时，不做顺手重构。

避免：

```text
重构目录
改代码风格
换状态管理
重写 UI
顺手增加功能
```

应该进行目的明确的小实验。

例如：

### Streaming

```text
人为延迟 Delta 500ms
```

### Ordering

```text
交换两个 Event 顺序
```

### Reconnect

```text
主动断 WebSocket
```

### Tool Approval

```text
强制 Reject
```

### Store

```text
监听所有 mutation
```

### Session

```text
删除 Client Cache
```

原则：

> **修改不是为了优化系统，而是为了暴露系统设计。**

---

## 5. 实验结束后，必须能回到原版

一章结束后：

```bash
git diff
```

最好为空。

应该留下的是：

```text
架构图
调用链
实验记录
最小实现
设计结论
```

而不是一堆已经无法解释的 patch。

---

## 6. 每个核心机制都要有最小实现，但要区分两种 Playground

这是 v2 的重要修正。

---

# 五、双 Playground 模型

## Playground A：Reproduction Playground

目的：

> **验证我是否真的理解原项目。**

原则：

> 尽量使用和原项目相同的技术栈。

例如 ZCode 使用：

```text
React
Zustand
WebSocket / RPC
```

那么 Reproduction Playground 也尽量使用：

```text
React
Zustand
WebSocket
```

这样可以减少变量。

否则很难判断差异来自：

```text
Agent 机制
```

还是：

```text
Framework / State Library
```

---

## Playground B：Transfer Playground

目的：

> **验证这个机制如何迁移到我未来真正使用的技术栈。**

例如目标系统使用：

```text
Vue
Pinia
Node
WebSocket
```

那么可以在完成 Reproduction 之后再实现：

```text
React / Zustand 原型
        ↓
理解机制
        ↓
Vue / Pinia 迁移
```

原则：

> **先同栈复刻，再异栈迁移。**

---

## 并不是每章都需要 Transfer

只有满足以下条件之一才进入 Transfer：

```text
未来系统明确会使用
存在技术栈迁移价值
存在框架无关的架构问题
```

例如：

```text
Streaming        → 值得
Session          → 值得
RPC              → 值得
Tool Approval    → 值得

Theme            → 不值得
普通 Form        → 不值得
i18n             → 通常不值得
```

---

# 六、三层代码模型

整个项目始终区分三类代码。

## Layer 1：Original

原始源码。

回答：

> 他们到底怎么做的？

原则：

```text
Read Only
Follow Upstream
No Experiments
```

---

## Layer 2：Lab

用于：

```text
插桩
破坏
修改
实验
验证
```

回答：

> 为什么要这样做？

以及：

> 改掉会发生什么？

可以大胆破坏。

---

## Layer 3：Playground

自己重新实现。

包含：

```text
Reproduction Playground
Transfer Playground
```

回答：

> 离开原项目，我到底会不会做？

---

整体关系：

```text
Original
   │
   │ Observe / Trace
   ▼
Lab
   │
   │ Break / Experiment
   ▼
Understanding
   │
   ├───────────────┐
   ▼               ▼
Reproduction     Stop
Playground
   │
   │ Worth transferring?
   ▼
Transfer Playground
   │
   ▼
My Architecture
```

---

# 七、固定学习仓库结构

学习资产必须有固定落点。

推荐：

```text
agent-study/
│
├── PRINCIPLES.md
│
├── README.md
│
├── roadmap.md
│
├── original/
│   └── zcode/
│
├── labs/
│   ├── streaming/
│   ├── rpc/
│   ├── session/
│   └── tool-approval/
│
├── notes/
│   ├── streaming/
│   │   ├── README.md
│   │   ├── architecture.md
│   │   ├── trace.md
│   │   └── experiments.md
│   │
│   ├── rpc/
│   └── session/
│
├── playgrounds/
│   ├── reproduction/
│   │   ├── streaming-react/
│   │   └── session-react/
│   │
│   └── transfer/
│       ├── streaming-vue/
│       └── session-vue/
│
└── decisions/
    ├── web-agent-architecture.md
    ├── session-design.md
    └── event-protocol.md
```

---

## 各目录职责

### original/

第三方源码。

不保存自己的实验逻辑。

---

### labs/

实验代码。

允许脏。

允许失败。

允许删除。

---

### notes/

学习过程中的事实记录。

重点记录：

```text
架构
Trace
Experiment
Finding
```

---

### playgrounds/reproduction/

同栈最小实现。

验证：

> 我是否真的理解原机制？

---

### playgrounds/transfer/

目标技术栈实现。

验证：

> 如何迁移到自己的系统？

---

### decisions/

这是最重要的长期资产之一。

这里只记录：

> **我最终决定自己的系统怎么设计。**

不要把：

```text
ZCode 怎么做
```

和：

```text
我准备怎么做
```

混在一起。

---

# 八、四类核心沉淀资产

每章原则上只长期保留四类资产。

---

## 1. 架构图

例如：

```text
Browser
   ↓
Agent Client
   ↓
RPC
   ↓
Gateway
   ↓
Runtime
```

---

## 2. 调用链

例如：

```text
sendMessage()
   ↓
session.send()
   ↓
rpc.invoke()
   ↓
server handler
   ↓
agent.run()
```

---

## 3. 实验记录

例如：

```text
实验：
人为延迟 Delta 500ms

现象：
UI 顺序没有改变

原因：
Message 顺序由 sequenceId 决定

结论：
网络到达时间不是最终顺序依据
```

---

## 4. 最小实现

例如：

```text
streaming-react
streaming-vue
mini-session
mini-rpc
```

---

# 九、AI 在源码学习中的角色

这是 v2 的核心规则之一。

原则：

> **AI 可以替我劳动，但不能替我形成判断。**

---

## AI 可以做什么？

适合交给 AI 的任务：

```text
搜索 Symbol
寻找文件
建立初始 Source Map
生成 rg / grep 命令
寻找调用点
整理机械调用关系
搭实验脚手架
插入日志
生成测试数据
比较 Diff
总结大段样板代码
检查是否遗漏路径
```

这些工作的共同特点：

> 机械成本高，但不是核心学习价值。

---

## AI 不应该替我做什么？

必须自己完成：

```text
提出问题
定义研究目标
决定研究深度
读源码前下注
判断 Source of Truth
判断设计动机
设计 Break 实验
解释实验结果
判断 Trade-off
决定是否值得迁移
形成自己的架构结论
```

---

## 错误使用方式

直接问：

> 帮我分析 ZCode 的 Streaming 架构。

然后把 AI 输出当成学习成果。

这很容易产生：

```text
信息获取成功
学习过程失败
```

---

## 推荐使用方式

先自己写：

```text
Hypothesis:

我认为 Server 生成 Event，
Client 只负责 transport，
Store 根据 Event 构造 Message Projection。
```

然后让 AI：

> 找出 Message Delta 从 WebSocket 进入 Store 的所有关键文件和函数，不要解释设计思想。

接着：

```text
自己 Trace
自己判断
自己设计实验
```

最后再让 AI：

> 检查我的 Trace 是否遗漏重要分支。

---

## AI 的理想定位

```text
AI
│
├── Research Assistant
├── Code Navigator
├── Experiment Assistant
├── Boilerplate Generator
└── Reviewer
```

而不是：

```text
Teacher who gives final answer
```

真正的目标是：

> **把时间从机械劳动中释放出来，用在判断和设计上。**

---

# 十、源码学习中的危险信号

## 危险信号 1

> 这个代码不够优雅，我顺手重构一下。

说明正在从学习滑向开发。

---

## 危险信号 2

> 这个 UI 不好看，我先改一下。

如果与当前问题无关，立即停止。

---

## 危险信号 3

> 这个目录还有很多文件没看。

不要为了覆盖率读源码。

---

## 危险信号 4

> 我大概看懂了，不需要实验。

至少设计一个 Break。

---

## 危险信号 5

> 这个机制我看懂了，不需要重新实现。

对于 S 级机制，至少完成 Reproduction。

---

## 危险信号 6

> 我每章都必须做到 Level 5。

错误。

先检查研究等级。

---

## 危险信号 7

> 这个知识以后可能有用，所以先研究一下。

“以后可能有用”不是深挖理由。

---

## 危险信号 8

> Agent 已经帮我分析完了，我读一下总结就行。

说明 AI 正在替代学习过程。

---

## 危险信号 9

> 仓库已经改乱了，懒得继续看。

删除 Lab。

不要整理废墟。

---

# 十一、每章固定模板

```markdown
# Chapter N - Topic

## 0. Research Level

S / A / B

目标 Level：

---

## 1. Why

为什么值得研究？

它和未来 Web Agent 有什么关系？

---

## 2. Questions

本章必须回答哪些问题？

---

## 3. Hypothesis

在阅读源码前，我认为它如何工作？

---

## 4. Observation Setup

如何观察？

- DevTools
- WebSocket
- Store
- Logs
- React DevTools

---

## 5. Source Map

关键文件有哪些？

只列必要文件。

---

## 6. Trace

完整调用链是什么？

---

## 7. Break

设计 1～3 个实验。

### Experiment 1

Hypothesis:

Action:

Expected:

Observed:

Conclusion:

---

## 8. Reproduction

是否需要同栈最小实现？

如果需要：

目标：

范围：

不实现什么：

---

## 9. Transfer

是否值得迁移到目标技术栈？

如果不值得：

Stop Reason:

如果值得：

目标：

---

## 10. Design

如果让我设计自己的系统：

哪些设计保留？

哪些设计不采用？

哪些设计只适用于 ZCode？

---

## 11. Takeaways

最终只保留 3～5 条结论。
```

---

# 十二、学习等级评价标准

不要用：

```text
读了多少文件
看了多少行源码
学习了多少小时
```

衡量进度。

使用下面的 Level。

---

## Level 1：Explain

我能解释：

> 这个机制为什么存在？

---

## Level 2：Trace

我能完整说明：

> 数据 / 控制流经过哪些关键模块？

---

## Level 3：Break

我知道：

> 改坏哪里会出现什么现象？

---

## Level 4：Rebuild

我能够：

> 用同技术栈重新实现核心机制。

---

## Level 5：Transfer & Design

我能够：

```text
迁移到自己的技术栈
判断设计 Trade-off
设计自己的实现
解释为什么没有照抄原项目
```

---

# 十三、停止规则

每个课题都必须允许停止。

---

## Stop Rule 1

发现：

> 没有明显 Agent 特异性。

停止。

---

## Stop Rule 2

发现：

> 只是框架层常规实现。

停止。

---

## Stop Rule 3

达到目标 Level。

停止。

---

## Stop Rule 4

继续研究的边际收益已经很低。

停止。

---

## Stop Rule 5

和未来目标系统无关。

停止。

---

原则：

> **学习不是完整遍历，而是价值驱动搜索。**

---

# 十四、最终目标

我的目标不是：

> 成为一个非常懂 ZCode 或 DeepSeek Harness 源码的人。

而是：

> **能够独立设计和实现 Web Agent Frontend、Agent Client、RPC、Gateway、Session、Tool Interaction、Runtime Interaction 等核心系统。**

最终学习路径：

```text
观察
  ↓
提出问题
  ↓
下注
  ↓
Trace
  ↓
Break
  ↓
Reproduce
  ↓
Transfer
  ↓
Design
```

但必须始终记住：

```text
              是否值得？
                  │
        ┌─────────┴─────────┐
        │                   │
       Yes                  No
        │                   │
      深挖                 Stop
```

---

# 十五、时时勤拂拭

当源码越来越复杂、仓库越来越乱、注意力越来越分散时，重新问自己：

> **我现在研究的问题是什么？**

> **它属于 S、A 还是 B？**

> **目标 Level 是多少？**

> **我的 Hypothesis 是什么？**

> **我是否真的能观察这个系统？**

> **我做了什么 Break 实验？**

> **实验告诉了我什么？**

> **这个机制值得 Rebuild 吗？**

> **值得 Transfer 吗？**

> **AI 是在帮我劳动，还是在替我思考？**

> **这个设计对我未来的 Web Agent 有什么意义？**

如果回答不了：

```text
Stop
↓
回到 Baseline
↓
重新定义问题
```

---

# 十六、最后的约束

牢记：

> **源码是标本，不是毛坯房。**

> **不能观察的系统，不适合做源码实验。**

> **不是所有知识都值得获得同样深度的理解。**

> **先同栈复刻，再异栈迁移。**

> **AI 可以替我劳动，但不能替我形成判断。**

> **Stop 是一种成功。**

> **问题是导航，实验是手段，重建是检验，设计能力才是最终成果。**
