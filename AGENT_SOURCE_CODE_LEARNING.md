# Agent 源码学习范式

> 源码是标本，不是毛坯房。  
> 学习阶段允许切开、注射、染色、做实验，但每次实验之后，都要能立刻恢复到原始状态。

这份文档用于约束我在学习 Agent 前端、Client、RPC、Server、Runtime 等源码时的方式。

目标不是“把某个项目学完”，也不是把开源项目改造成自己的项目，而是：

> **通过观察、实验、破坏和重建，逐步形成自己设计 Web Agent 系统的能力。**

---

## 一、核心学习循环

整个源码学习过程遵循：

```text
观察 → 假设 → 微改 → 验证 → 复原 → 总结
```

进一步抽象为每个学习章节的固定流程：

```text
Concept → Trace → Break → Rebuild → Design
```

即：

1. **Concept**：先理解这个机制解决什么问题
2. **Trace**：沿真实调用链追踪源码
3. **Break**：故意破坏系统，观察它为什么需要这些设计
4. **Rebuild**：脱离原项目，自己实现一个最小版本
5. **Design**：提炼成未来自己系统里的设计原则

不要停留在“看懂了”。

真正的标准是：

> **离开原项目后，我是否仍然能够解释它、破坏它、重新实现它，并对自己的系统做设计判断。**

---

# 二、七点源码学习范式

## 1. 永远保留一个不可污染的基线

Fork 项目后，`main` 分支只负责跟踪 upstream。

不要在 `main` 上直接实验。

建议使用独立 branch 或 `git worktree`：

```text
zcode/                 ← 原版标本
zcode-lab-stream/      ← Streaming 实验
zcode-lab-approval/    ← Tool Approval 实验
my-agent-playground/   ← 自己的最小实现
```

例如：

```bash
git worktree add ../zcode-lab-stream -b lab/message-stream
```

实验失败、代码变乱、已经不想继续看时：

```bash
git worktree remove ../zcode-lab-stream
git branch -D lab/message-stream
```

重新开始即可。

原则：

> **源码可以破坏，但基线不能污染。**

---

## 2. 每次研究一个“问题”，而不是一个“目录”

不要这样学习：

```text
今天研究 packages/ui
明天研究 packages/client
后天研究 packages/server
```

应该这样学习：

```text
用户发送一条 Prompt 后，前端第一个发生变化的状态是什么？

Tool Call 从 Server Event 到 UI 是如何完成的？

Session 的 source of truth 在 Client 还是 Server？

WebSocket 断开以后，状态是如何恢复的？

Streaming Message 为什么不会出现乱序？
```

目录只是实现细节。

**问题才是导航。**

一个好的源码问题通常可以画成一条调用链：

```text
User Input
   ↓
UI Action
   ↓
Client
   ↓
RPC
   ↓
Server
   ↓
Agent Runtime
   ↓
Event
   ↓
Store
   ↓
UI Render
```

---

## 3. 读代码之前，先下注

不要一开始就相信源码。

先根据：

- 目录结构
- 类型定义
- 接口
- UI 表现
- 网络请求
- 日志

建立自己的假设。

例如：

> 我猜模型返回的 delta 会先进入 Agent Client，再被转换为 Message Event，最后更新 Zustand Store。

然后去源码里验证。

可能结果是：

```text
假设正确
```

也可能：

```text
Server 已经完成 event normalization
Client 只是负责 transport
Store 又做了一层 projection
```

这种“预测 → 验证”的学习方式，比逐行阅读更容易形成长期记忆。

原则：

> **源码阅读不是接受知识，而是不断修正自己的系统模型。**

---

## 4. 修改必须是“微创实验”

学习源码时，不要顺手开始产品开发。

不要：

```text
重构目录
换状态管理
统一代码风格
重写组件
增加复杂业务功能
```

应该做非常小、目的明确的实验。

例如研究 Streaming：

```text
把 delta 延迟 500ms
```

研究 Event Ordering：

```text
人为交换两个 event 的顺序
```

研究 WebSocket：

```text
主动断开连接
```

研究 Tool Approval：

```text
强制默认 reject
```

研究 Store：

```text
记录每一次 state mutation
```

研究 Session：

```text
删除某个 session cache
```

原则：

> **修改代码不是为了让系统更好，而是为了暴露系统为什么这样设计。**

---

## 5. 实验结束后，必须能够回到原版

一章结束以后，不要求留下一个“更好的 ZCode”。

理想状态甚至可以是：

```bash
git diff
```

最终为空。

真正应该留下的是：

- 架构图
- 调用链
- 实验结论
- 自己的实现
- 设计原则

如果某个实验非常有价值，可以单独保留：

```text
lab-result/message-stream
lab-result/tool-approval
lab-result/session-reconnect
```

但不要让实验代码逐渐侵蚀主仓库。

原则：

> **学习成果保存在认知中，而不是保存在一堆无法解释的 patch 里。**

---

## 6. 每一个课题，都必须有自己的最小实现

只修改别人的项目仍然不够。

真正理解一个机制以后，要离开原项目重新做一次。

例如学习 Agent Streaming 后：

```text
mini-agent-stream-demo
```

只使用：

```text
Vue
Pinia
WebSocket
Node.js
```

实现：

```text
Prompt
  ↓
Server
  ↓
Streaming Event
  ↓
Client
  ↓
Store
  ↓
UI
```

可能只有 200～500 行。

但它的价值往往高于继续阅读几万行源码。

因为：

> **看懂别人的代码是知识。**
>
> **脱离原代码重新实现，才开始变成能力。**

---

## 7. 每章只沉淀四类长期资产

不要写大量逐文件、逐函数笔记。

半年后你通常不会在意：

```text
foo.ts 第 137 行调用了 bar()
```

真正值得长期保存的是：

### 1. 架构图

例如：

```text
Browser
   ↓
Agent Client
   ↓
RPC
   ↓
Agent Gateway
   ↓
Runtime
```

### 2. 调用链

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

### 3. 实验记录

例如：

```text
实验：
人为延迟 delta 500ms

现象：
UI 正常

结论：
Message 顺序不是依赖网络到达时间，而是依赖 sequenceId。
```

### 4. 自己的最小实现

例如：

```text
playground/session
playground/streaming
playground/tool-call
playground/reconnect
```

这四类东西，才是真正属于自己的知识库。

---

# 三、三层代码模型

整个学习项目始终维护三种代码。

## Layer 1：Original

原始开源源码。

用于回答：

> **他们到底是怎么做的？**

尽量保持纯净。

---

## Layer 2：Lab

用于破坏、实验、插桩、修改的源码副本。

用于回答：

> **为什么要这么做？如果我改掉会发生什么？**

这里可以大胆破坏。

---

## Layer 3：Playground

完全由自己重新实现的小项目。

用于回答：

> **如果没有这份源码，我还会不会做？**

真正的能力主要在这一层形成。

---

三层关系：

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
   │ Rebuild
   ▼
Playground
   │
   │ Abstract
   ▼
My Architecture
```

---

# 四、不要追求“学完一个项目”

大型 Agent 项目不存在真正意义上的“学完”。

错误目标：

> 我要把 ZCode 所有源码看一遍。

正确目标：

```text
ZCode
  ↓
提取设计模式
  ↓
验证设计模式
  ↓
重新实现
  ↓
形成自己的 Web Agent 架构
```

例如研究 Session：

```text
ZCode Session
      ↓
为什么需要 Session？
      ↓
Session 保存什么？
      ↓
谁是 source of truth？
      ↓
Client / Server 如何同步？
      ↓
删掉缓存会怎样？
      ↓
自己实现 MiniSession
      ↓
未来 Workspace Session 如何设计？
```

学习对象不是 ZCode。

学习对象是：

> **Agent System Design。**

ZCode 只是当前使用的一份优秀标本。

---

# 五、源码学习中的危险信号

当出现下面这些情况时，应立即停下来检查自己的方向。

### 危险信号 1

> “这个代码写得不够优雅，我顺手重构一下。”

很可能已经从学习进入开发。

---

### 危险信号 2

> “这个 UI 不喜欢，我先改漂亮一点。”

与当前研究问题无关的修改立即停止。

---

### 危险信号 3

> “这个目录还有很多文件没看完。”

不要为了覆盖率读源码。

回到当前问题。

---

### 危险信号 4

> “我大概理解了，不需要实验。”

必须设计一个破坏性实验。

---

### 危险信号 5

> “这个机制我看懂了，不需要自己实现。”

至少做一个最小版本。

---

### 危险信号 6

> “仓库已经被我改乱了，懒得继续看。”

说明 Original / Lab 没有隔离。

删除 Lab，重新开始。

不要整理废墟。

---

# 六、每一章的固定模板

以后所有源码课程尽量使用同一套结构。

```markdown
# Chapter N - Topic

## 1. Concept
这个机制解决什么问题？

## 2. Questions
这章需要回答哪些问题？

## 3. Hypothesis
看源码之前，我认为它是如何工作的？

## 4. Trace
完整调用链是什么？

## 5. Source Map
涉及哪些关键文件？

## 6. Break
设计 2～3 个破坏性实验。

## 7. Findings
实验结果是什么？

## 8. Rebuild
自己实现一个最小版本。

## 9. Design
如果让我自己设计，我会怎么做？

## 10. Takeaways
最终沉淀 3～5 条架构原则。
```

---

# 七、最终评价标准

不要用：

```text
我看了多少文件
我读了多少行源码
我学习了多少小时
```

衡量进度。

真正的评价标准是：

### Level 1：Explain

我能解释这个机制为什么存在。

### Level 2：Trace

我能完整追踪它的调用链。

### Level 3：Break

我知道破坏哪里会出现什么问题。

### Level 4：Rebuild

我能脱离原项目实现一个最小版本。

### Level 5：Design

我能够判断：

```text
哪些设计值得保留
哪些是项目历史包袱
哪些只适用于当前规模
自己的系统应该怎么设计
```

达到 Level 5，这个章节才真正完成。

---

# 八、最终目标

我的目标不是：

> 成为一个非常懂 ZCode / DeepSeek Harness 源码的人。

而是：

> **能够独立设计和实现 Web Agent Frontend、Agent Client、RPC、Gateway、Session、Tool Interaction、Runtime Interaction 等核心系统的人。**

所以整个学习过程始终保持：

```text
看懂
  ↓
验证
  ↓
破坏
  ↓
重建
  ↓
抽象
  ↓
形成自己的设计能力
```

---

# 九、时时勤拂拭

当源码越来越复杂、仓库越来越乱、注意力越来越分散时，回到这几个问题：

> **我现在研究的问题是什么？**

> **我的假设是什么？**

> **我做了什么实验？**

> **这个实验告诉了我什么？**

> **脱离原项目，我能不能自己实现？**

> **这个设计对我未来的 Web Agent 有什么意义？**

如果回答不了，就说明已经偏离学习目标。

删除实验。

回到基线。

重新开始。

---

> **源码是标本，不是毛坯房。**
>
> **问题是导航，实验是手段，重建是检验，设计能力才是最终成果。**
