---
title: OpenClaw 总体架构技术拆解
date: 2026-03-24
author: neigelmlm
description: OpenClaw 总体架构技术拆解
tags:
    OpenClaw
cover: cover.png
---
 从「能力亮点、设计理念」的维度，提炼了 OpenClaw 系统的 6 大核心能力模块与顶层设计原则 ，回答的是「这个系统有什么核心优势、核心能力是什么」的问题。它把系统的核心能力拆成了 6 个模块，分别是：

1. Gateway 控制平面 ：系统的中枢调度核心，用单一 WebSocket 服务器统一管理全系统的会话与事件；
2. 多渠道消息系统 ：解决多平台消息接入问题，通过统一抽象层兼容 15 + 主流消息平台；
3. 插件化扩展 ：系统的扩展性底座，基于 Monorepo 架构实现独立插件包管理，支持灵活扩展；
4. 跨平台客户端 ：用户与系统交互的入口，覆盖 CLI、桌面端、移动端多类终端；
5. AI ** 代理引擎** ：系统的 AI 能力核心，基于 Pi Agent RPC 模式实现多模型支持与智能处理；
6. 本地优先设计 ：系统的安全与数据底座，通过本地存储实现数据、会话的自主可控，保障隐私。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YTgyMmRlYzUxMDNmMjhmZGViODM5N2ZlMWU3ZDViNzJfSW5WeEE1OGtaemh2eXJSaEhPZkRZcEZBWk1WcVZMYkFfVG9rZW46RTR6d2JnTlBGb3hndk94RnJpOGNuWWZ3bkVmXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

2. ## 七层架构层

 从「技术实现、数据流转」的维度，拆解了 OpenClaw 的七层技术架构 ，回答的是「这个系统的技术结构是什么、数据从发起到落地的完整流程是怎样的」的问题。它按照「用户输入→系统处理→数据存储」的完整链路，把系统纵向拆成了 7 个层级，每层各司其职、上下游联动：

1. 客户端层 ：用户交互的入口，对应第一张图的「跨平台客户端」；
2. Gateway 控制平面 ：全系统的调度中枢，完全对应第一张图的「Gateway 控制平面」，承接客户端指令，管理全链路的会话与事件；
3. 渠道接入层 ：多平台消息的统一入口，对应第一张图的「多渠道消息系统」，实现各类第三方消息平台的标准化接入；
4. 路由与处理层 ：消息流转的 “交通枢纽”，负责消息的路由分发、权限过滤、规则处理；
5. AI ** 代理层** ：智能处理的核心，完全对应第一张图的「AI 代理引擎」，负责大模型调用、工具调用、会话上下文管理；
6. 工具与服务层 ：能力支撑层，为 AI 处理和上层业务提供浏览器、媒体处理、语音转换、定时任务等基础工具能力；
7. 数据存储层 ：数据落地的底座，对应第一张图的「本地优先设计」，负责会话、配置、缓存、向量数据的本地持久化存储。

第一张是「横向的能力切片」，把系统最核心的 6 个亮点能力并列展示，让用户一眼看懂系统的核心优势；第二张是「纵向的流程链路」，把系统按数据流转的顺序拆成七层，讲清楚了各个模块如何串联协作、完成一次完整的业务处理。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YjY1OTY0NDhlYjMzMGEzZmJiOTdiNDRkNTIyMGZmZTVfVHlrSWQ4eHRKZkFDRklTNWNHNWV1b1QyMWRhUFFVcTdfVG9rZW46WTFJcGJHWVhobzdJRXZ4MkV6b2NtbkRNblNkXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

3. ## 技术栈

| 技术分层                                                                                              | 核心技术组件                                                                    | 对应功能架构层                       |
| ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------ |
| ----------------------------------------------------------------------------------------------------- |                                                                                 |                                      |
| 应用层                                                                                                | CLI (Commander)、Control UI (LitElement)、iOS/macOS (SwiftUI)、Android (Kotlin) | 客户端层                             |
| -                                                                                                     | -                                                                               | -                                    |
| 通信层                                                                                                | `WebSocket RPC (Request/Response/Event Frames)`                               | Gateway 控制平面通信底座             |
| 核心服务层                                                                                            | `Gateway Server、ACP Layer、Agent Runtime (pi-mono)`                          | Gateway 控制平面 + AI 代理层         |
| 通道适配层                                                                                            | Plugin SDK、核心通道、扩展通道                                                  | 渠道接入层 + 插件化扩展              |
| 数据持久层                                                                                            | JSON Files、SQLite（可选）、LanceDB（向量记忆）                                 | 数据存储层                           |
| 外部服务层                                                                                            | OpenAI API、Claude API、Tailscale、Playwright                                   | AI 代理层 + 工具与服务层（外部依赖） |
| 运行环境                                                                                              | Node.js 22+、TypeScript、pnpm、Vitest、tsdown、Oxlint                           | 全系统底层运行环境                   |

4. ## 完整的业务闭环流转图

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NmE2ODkwYzNhNGYwZDI0ZjNiYmJjNTk5Y2E5OTU5ZWJfY3ZyRldpT1dLTldlM3pLTWlscXFWWUlyZ2J5dUozTGpfVG9rZW46RHJuM2JRMEtzb1ZZNWR4dlJGOGNzaEk5bjVXXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

这张图是 运行时的「数据流与业务全链路流转维度」 ，和之前几张静态的架构、能力、技术栈拆解完全不同，它聚焦的是 动态的执行流程 ，核心回答的是： 一条用户消息 / 一个定时任务，从触发到最终完成响应，在 OpenClaw 里完整的生命周期、数据流向、模块间的调用与协作关系 。

它把之前静态的架构模块，串成了一个可运行的完整流程，核心流转链路分为两大触发入口、五大核心环节，形成完整闭环：

1. 两大触发起点（流程的开端）

   1. 被动触发：用户从 WhatsApp、Telegram 等外部消息平台发送的原始消息；
   2. 主动触发：系统内置的定时任务（Cron Jobs）、心跳循环，主动发起的调度任务。
2. 核心中枢：Gateway 控制平面所有触发的事件 / 消息，全部汇总到 Gateway/Router 核心网关：

   1. 外部消息：先通过 Channel Adapters 做格式标准化，统一接入网关；
   2. 网关负责消息入队、优先级管理、任务分配，把处理任务下发给 Agent 运行环境；
   3. 同时也是最终响应的收口，负责把 AI 处理后的结果，原路返回给外部渠道。
3. 智能执行核心：Agent 运行环境承接网关分配的任务，完成核心的智能处理：

   1. 从存储层读写会话上下文、配置、历史对话，获取推理所需的背景信息；
   2. 调用 LLM 大模型（GPT/Claude 等）完成思维推理、指令理解；
   3. 按需调用工具集 / 技能集，完成浏览器操作、媒体处理等扩展动作。
4. 数据底座：存储与记忆层为 Agent 运行提供全链路的数据支撑，包括 Agent 配置、人设指令、对话历史、向量语义检索能力，对应之前架构的本地优先设计。
5. 响应闭环Agent 处理完成的最终结果，回传给 Gateway 网关，再通过渠道适配器，把标准化响应转换为对应平台的格式，返回给用户，完成一次完整的交互流程。
6. ## Agent 智能体核心执行链路

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YWE4ZGIyMTA3YjVlMjg3YmQ1MDUzYjEwNDcxMDNlNDVfQ1Z0emxlY2pHS2ZVbXV2V1ZwM0FuQTR6Z2R2U25RNkpfVG9rZW46VllCbGJZYks5b2FES3F4aHVZUmNveHRYbnRjXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. Channels（通道） ：连接各种消息平台，提供统一消息接口。支持：飞书、钉钉、WhatsApp、Telegram、Discord、Slack、SMS 等，让 AI 与用户的日常通信无缝对接。
2. 为什么 open claw 需要一个消息入口层？而不是自己做一个？
   Telegram、Discord 等已经是成熟的聊天平台，自带：

   1. 账号体系、用户管理、消息推送、富媒体（图片 / 文件 / 表情）
   2. 跨端（手机 / PC / 网页）、安全加密、合规审核
      如果 OpenClaw 自己做 IM：
   3. 需要从零开发客户端、服务器、推送链路，成本极高
   4. 无法快速触达已有用户，推广难度大
      用「渠道适配器」做一层薄薄的适配，就能让 Agent 直接跑在用户已经在用的 IM 里， 开箱即用 。
3. Gateway（大门） ：基于 WebSocket 的中央控制平面，运行在 localhost:18789 之上的一个 Node.js 守护进程，管理会话、路由请求、身份鉴权、消息通道、工具和事件。所有消息经由此路由、管理认证和会话。
4. 是因为 Openclaw 接了很多消息入口还是因为 AIAgent 就需要 Gateway Server 来处理不同的用户的消息？
   AI Agent 天生就需要 Gateway 来管理用户与会话

   不管 OpenClaw 接 1 个还是 100 个消息入口，只要是多用户、多会话、多 Agent 的系统，就必须有 Gateway 做三件事：

   1. 认人 + 会话隔离 ：每个用户的聊天是独立会话，不能串线。比如你和它聊代码、朋友和它聊炒股，Gateway 必须记住 “谁是谁、聊到哪了”，把消息精准送到对应会话。
   2. 排队 + 限流 ：Agent/LLM 算力有限，不能同时处理几百条消息。Gateway 用队列（Lane）让请求排队，保护后端不被冲垮。
   3. 统一格式 + 解耦 ：把用户消息转成 Agent 能看懂的统一格式。Agent 只专心做 “思考 + 工具调用”，不用管消息来自哪里。
5. Agent（大脑） ：连接 LLM（Claude、GPT、Ollama 等），有专门的人设，负责理解上下文意图、制定分步计划、决定要调用哪些工具或技能。
6. Skills（手脚） ：使用 SKILL.md 进行定义的一系列 Skills&Tools，是 Agent 和现实世界互动的手脚，支持文件操作、浏览器控制、API 调用等。
7. Nodes（传感器/终端） ：运行在用户端设备（手机、笔记本、Raspberry Pi，台式机）的小智能体，可以提供摄像头、地理位置或系统通知等本地能力。
8. ## 总结

先看 特征图 ：知道 OpenClaw 「想做成什么样」；

再看 分层图 ：知道它「长什么样、模块怎么分」；

再看 技术表 ：知道它「用什么技术、怎么搭出来」；

再看 数据流图 ：知道它「跑起来时，数据怎么流转」；

最后看 Agent 链路图 ：知道它「最核心的 AI 大脑，内部是怎么干活的」

2. # `Gateway` 中央控制平面

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NTc5MjQ2YmVlNGE3NmE5NjNiODA1NmEzYTU2M2QwZjZfTjhUbjNKcVZRN0hZcHdqa0FpdUZDV3lvbFhkdkdOamlfVG9rZW46QVpBY2I2YmNRb25KdFN4QVIxTWNNVm1HbkRlXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

Gateway 的工作原理？

OpenClaw Gateway 是一个长期运行（这个程序启动后不会执行完就关掉，一直待在电脑 / 服务器里运行，除非手动停止或出故障）的守护进程（编程里对 “后台长期运行程序” 的专业叫法），基于 Node.js 构建（Node.js 不是编程语言，而是 运行 JavaScript 代码的环境 ），采用 WebSocket 通信技术（一种允许 “双向实时聊天” 的通信方式）。主要做以下件事：

1. IM 平台适配 ：各 IM 平台的消息格式不一样，Gateway 会把外部消息统一成标准的内部 Message events 模型。
2. 认证与鉴权：
   1. 身份认证 ：验证请求方的合法身份（如基于 JWT、OAuth2.0 或平台 Token 校验）；
   2. 权限校验（ACL） ：基于角色（RBAC）或资源（ABAC）判断用户是否有访问目标 Agent 的权限；
   3. 路由分发 ：通过路由引擎将校验通过的请求转发至对应的 Agent 实例。
3. 会话路由：路由逻辑依赖三个关键维度确定目标会话：
   1. 用户标识 （如 `user_id`）：唯一标识消息发送者；
   2. 工作区标识 （如 `workspace_id`）：隔离不同业务场景或组织单元；
   3. 会话上下文 （如 `session_id` + 历史消息序列）：维护对话的连续性。
4. 车道队列（Lane Queue）：作为并发控制层，为每个会话分配独立的 “队列车道”，确保：
   1. 同一会话的消息按顺序处理（避免乱序）；
   2. 不同会话的处理相互隔离（避免竞态条件，如同时修改同一会话的状态）；
   3. 系统在高并发（如群聊刷屏、批量 @）下仍能稳定运行。
5. 会话管理：会话的生命周期与状态维护
   1. 生命周期 ：Session 的创建（新对话开始）、维护（消息处理中更新状态）、回收（对话结束后释放资源）；
   2. 上下文缓存 ：用 Redis 或内存缓存存储会话上下文（如历史消息、当前意图），避免频繁查库；
   3. 状态策略 ：定义截断（如上下文 Token 超过阈值时丢弃早期消息）、归档（长期不活跃的会话存入数据库）、重建（用户再次发起对话时恢复上下文）的规则。
6. Agent 装备与调用 ：在 Agent 执行任务前，完成运行上下文的组装（依赖注入）：
7. 注入历史状态 ：将缓存的 Session 历史加载到 Agent 的上下文中；
8. 注入可用技能 ：根据业务规则查找 Agent 当前可调用的 Skill 模块；
9. 注入工具集合 ：执行 Tool Policy（如白名单、权限校验），确定 Agent 当前可使用的工具（如 API 调用、本地脚本）。
10. 远程 Agent 工具执行 ：当 Agent 需要调用远程节点的工具（如手机端拍照、边缘设备执行脚本）时：
11. 回调审批 ：Agent 先向 Gateway 发起工具调用请求，Gateway 进行安全校验（如权限、审计日志）；
12. 命令下发 ：审批通过后，Gateway 通过 RPC（如 gRPC）或 WebSocket 将命令下发至对应远程节点；
13. 结果回传 ：远程节点执行完成后，将结果返回 Gateway，再由 Gateway 转发给 Agent。
14. 工具安全审批系统的第二道核心安全闸门，针对 Agent 的工具调用、高危操作执行二次安全校验与审批拦截，同时承接执行层 Agent 的远程工具调用请求校验，严防越权、高危操作，实现执行行为的闭环管控。
15. 节点注册表全系统节点的 “通讯录”，负责管理所有接入的设备节点、执行节点的注册、状态维护、寻址调度，记录节点的基础信息、在线状态、可用能力，同时承接设备节点的接入注册。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=Njc0MzVkODhmNTUyMWNjOWNhMzc5N2FhNTlhNjVjZGJfSHVkVDFkYW9pWnJObG1FUlRKcWw0MDhOV1hLVmlFSTJfVG9rZW46TmdKNmI3MW83b05kb214bzl3SWNRTWNrbldlXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. ## `Channel Adapters`

WhatsApp、Telegram、微信、钉钉、Discord 等等 IM 平台是多个互不兼容的通信系统：

1. 协议不同 ：HTTP、WebSocket、长连接、私有协议。
2. 消息模型不同 ：文本、富媒体、引用/回复、线程、频道、群组、@ 的语义。
3. 鉴权不同 ：token、扫码登录、回调签名、权限分级。
4. 限流策略不同 ：有的平台按用户限，有的按机器人限，有的按接口限。

 而 OpenClaw Gateway 则解决了 “统一性” 的问题 ，提供了一套统一的接入方式。它带来几个工程化的好处：

* 会话天然连续 ：跨平台也能保持一致的上下文。
* 状态不再分裂 ：不需要每个平台各存一份。
* 行为可预测 ：所有决策逻辑集中在一个地方。
* 可审计、可回放 ：出了问题能定位故障。

Gateway 会成为系统关键路径，设计不好容易变 “瓶颈/单点”，需要配套可观测性与容灾能力。

具体实现方面，OpenClaw 中的每一个 Channel 都被实现为一个独立的插件扩展包，必须实现一套统一的 ChannelPlugin 接口。需要注意的是，它不是通过继承一个 BaseChannel 基类的方式来实现接口，而是通过一组 Adapters（适配器）接口来组合能力。这样可以实现 “只有少量适配器必须实现，其余是可选能力。"

因此，不同渠道的功能集差异很大，比如 Telegram 支持投票和线程，WhatsApp 不支持；Discord 有服务器和角色概念，而 iMessage 没有。

如何让核心消息处理逻辑适配这些差异？

OpenClaw 的做法是让 Channel 通过 capabilities 声明告知自己支持什么功能。然后核心代码通过检查 capabilities.threads/polls 等属性来决定后续的行为。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NWNjNzAxMDg1MTViMWJiNzJmNzMzMmY1ZDNiZmU5ZGFfaHdrR3BWVTA4THFnZ081MlQ2czFkeko2NDV2NVlZb2JfVG9rZW46QzdiQ2JLdDJQb2t2dDZ4bzNReGNNbWdwbnpiXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

2. ## `WebSocket `通信协议

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YzFmMjU4YjAxYTY4NTk1M2VhMTA3YTExZDczYTI1YWZfTGdEdmNtclJEM01lVHpsNzVKeDg4dmgwQmdwdENLTUlfVG9rZW46V0lQWGJvYWZvb2FNbHZ4Sk1ZQmNBUnVqbmpkXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. ### Gateway 核心通信基础
2. 通信协议 ：采用 WebSocket（而非 HTTP）—— WebSocket 是全双工的实时通信协议，客户端和服务端能双向主动发消息，适合 CLI/TUI/GUI 这类需要实时交互的组件。
3. 通信对象 ：Gateway ↔ CLI（命令行界面）、TUI（终端界面）、GUI（图形界面）、Node（节点）、Channel（通道）等外部组件。
4. 连接地址 ：固定为 `ws://127.0.0.1:18789`（127.0.0.1 是本地回环地址，18789 是 Gateway 监听的端口，`ws://` 是 WebSocket 协议的标识）。
5. 数据载体 ：所有消息都通过 WebSocket 消息帧传输，且内容必须是 JSON 格式（轻量、易解析，是前后端 / 组件间通信的常用格式）。
6. ### 三种核心消息格式

通信过程中只有三类消息帧，分别对应「请求 - 响应」和「主动推送」两种交互模式

1. 请求格式（RequestFrame） ：客户端 → Gateway（主动请求），`{type:"req", id, method, params}`
2. 响应格式（ResponseFrame） ：Gateway → 客户端（回复请求），`type:"res", id, ok, payloaderror`
3. 事件格式（EventFrame） ：Gateway → 客户端（主动推送），`{type:"event", event, payload, seq?, stateVersion?}`
4. 支持事件类型 ：
5. `agent`：代理相关事件（如代理上线 / 下线、状态变更、配置更新）
6. `chat`：聊天相关事件（如新消息、消息撤回、聊天结束 / 创建）
7. `presence`：在线状态事件（如某个节点 / 用户上线 / 离线、忙碌 / 空闲）
8. `health`：健康状态事件（Gateway 或组件的健康检查结果、资源占用告警）
9. `heartbeat`：心跳事件（维持连接，确认双方在线，防止连接断开）
10. `cron`：定时任务事件（定时任务执行触发、任务完成 / 失败）
11. `tick`：时钟滴答事件（周期性时间同步，比如每秒推送一次时间）
12. `shutdown`：关闭事件（Gateway 或组件即将关闭的通知，客户端可做收尾）

客户端在建立连接时会进行一次握手，首帧必须为 req:connect。Gateway 会 res(ok) 告知客户端当前支持的 RPC 方法与事件列表，客户端据此执行操作。`type` 区分帧类型，`id` 关联 req/res 帧，`event` 标识推送的事件类型，`ok/payload/error` 表示响应结果。

暂时无法在飞书文档外展示此内容

3. ## WebSocket 网络模式

| Mode                                                                                    | Bind Address                                  | Auth Required             | Use Case                          |
| --------------------------------------------------------------------------------------- | --------------------------------------------- | ------------------------- | --------------------------------- |
| --------------------------------------------------------------------------------------- |                                               |                           |                                   |
| loopback                                                                                | 127.0.0.1                                     | Optional (recommended)    | Local-only access, SSH tunnels    |
| -                                                                                       | -                                             | -                         | -                                 |
| lan                                                                                     | 0.0.0.0 (all interfaces)                      | Yes (token or password)`` | Direct LAN/WAN access without VPN |
| tailnet                                                                                 | Tailscale IP (100.64.0.0/10)                  | Yes (token or password)   | Tailscale mesh network            |
| auto                                                                                    | Falls back through loopback → tailnet → lan | If non-loopback           | Automatic best-effort             |
| custom                                                                                  | gateway.host value                            | Yes (token or password)   | Specific IP binding               |

* loopback 模式 ：Auth 是可选的，但推荐使用，默认情况下，Onboarding 会生成一个 token。
* non-loopback 模式 ：lan/tailnet/custom 强制使用 Auth，未设置 gateway.auth.token 或者 gateway.auth.password 的话，Gateway 会拒绝启动。

```JSON
  "gateway": {
    "port": 18987,
    "mode": "local",
    "bind": "loopback",
    "auth": {
      "mode": "token",
      "token": "XXX"
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    },
    "nodes": {
      "denyCommands": [
        "camera.snap",
        "camera.clip",
        "screen.record",
        "calendar.add",
        "contacts.add",
        "reminders.add"
      ]
    }
  },
```

4. ## 会话路由

当 Gateway 从 Channel 接收到 Message 后，Gateway 的 Session Router 会根据 Message Session Key 负责将 Message 路由到某个 Agent 的 Session 中。以此来保证 Message 和 Session 的准确匹配。

1. ### Session Key
2. 查看 Session Key：默认的，每个 Agent 至少一个 main Session。此外，在 Subagent 和 Multi-Agent 中还会用于 Agent 之间通信的 Session。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NDVlMTZiYjZiMmY2YWUwZDM2M2M1NWI5ZTg2ZDgxYjNfZ0Zla0N5d2x4R3ZHTDlVb1N3UEVWV0V3bzQ5VE81U25fVG9rZW46SUR1WGJud0lRb0Rpck94UGFQOGN0SnZ2bmI5XzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. 同一个 Agent 也可以创建新建一个 Session：

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NjUxZDQxMjZlODkwZDFmYzFhNDhjNjU2Y2EzNGEzMGNfcVloN2h3Z3Z0c2sya01RYU9NMG9oNmszY2wybDFPTTJfVG9rZW46UEFXNmJGNmpnb083bXp4Z21KQ2NzbWlpbm5kXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

2. ### 消息的 Agent 绑定

现实场景中，消息和 Agent 的绑定关系会非常复杂，例如：

* 飞书群组 A 使用 “研究助手” Agent
* 飞书群组 B 使用 “运维机器人” Agent
* 同一个用户在私聊中使用 “默认助手”
* Discord 某个角色组的成员拥有专属 Agent

为了处理这些复杂情况，OpenClaw 设计了一套 分层优先级消息绑定（bindings）机制 。通过 bindings 配置完成，决定每条消息由哪个 Agent 处理，优先级（从高到低）：

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NzViMjUxODMzYWUxOGJiNWQ2NDdlYmY1ZTUyZTYzYjNfQ3o1cUJZQmZ4ZEg3RHJJZ0J5cHN2Q2d3N1d4RnFOU0ZfVG9rZW46V1dqemJyQmhyb0pPVWt4akYyM2NBRktNbnJmXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. peer ：精确绑定 Agent 到特定群组或用户。

```JSON
   {
     "type": "route",
     "agentId": "research-assistant",
     "match": {
       "channel": "feishu",
       "peer": "XXX"  // 飞书群组 A
     }
   }
```

2. peer.parent ：线程消息继承父消息的绑定关系。例如：Discord 话题下的所有回复保持绑定同一个 Agent；Slack 线程中的对话连续性。

```JSON
主消息（群组A） → 绑定到 "研究助手"
     └─ 回复消息 1 → 自动继承 "研究助手"
         └─ 回复消息 2 → 继续继承 "研究助手"
```

3. team ：绑定 Agent 到特定的团队或组织。

```JSON
 {
     "type": "route",
     "agentId": "devops-bot",
     "match": {
       "channel": "discord",
       "team": "engineering-team"  // 工程团队
     }
   }
```

4. account ：同一平台可能有多个 Bot，绑定到不同的 Agent。

```JSON
 {
     "type": "route",
     "agentId": "main",
     "match": {
       "channel": "feishu",
       "accountId": "default"  // 飞书账号 A
     }
   },
   {
     "type": "route",
     "agentId": "claudecode",
     "match": {
       "channel": "feishu",
       "accountId": "claudecode"  // 飞书账号 B
     }
   }
```

5. channel ：最后的兜底规则，绑定到默认 Agent。

```JSON
   {
     "type": "route",
     "agentId": "default-assistant",
     "match": {
       "channel": "feishu"  // 所有飞书消息
     }
   }
```

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MjhiYjIzMGVlYTA4MjhmYWM1YWI4NTc3MDg2NGEyY2RfMmo0M20wb05VdWNNdzdDUDlSUjc1RHZuRjI1T0EzcHFfVG9rZW46VmxtZ2JRdTllb3lPTUN4MmwyRWMwOENUbndiXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

3. ### Session 隔离策略

一个 Agent 可能同时服务多个用户，因此 Session 的隔离粒度会直接影响用户体验。比如户 A：帮我订明天早上的机票；用户 B：今天气怎么样？ Agent：我刚帮用户 A 订机票了...（混乱！）； 用户 C：我的订单呢？（Agent 不知道谁的订单）。

所以，我们需要考虑一个问题，当同一个 Agent 同时服务多个用户时，应该采用共享一个上下文，还是每人独立一份上下文？对此，OpenClaw 支持多种 Session 隔离策略。

* main（默认，单用户模式） ：所有消息共享一个会话，这是单用户、个人使用优先的默认选择。

```JSON
  {
     "agents": {
       "defaults": {
         "sessionIsolation": "main"
       }
     }
   }
```

* per-channel-peer（推荐，多用户模式）：每个聊天对象（群组或用户）独立会话，在多用户场景下，建议使用该策略。

```JSON
   {
     "agents": {
       "defaults": {
         "sessionIsolation": "per-channel-peer"
       }
     }
   }
```

* per-channel（平台级隔离） ：每个 IM 平台一个会话。
* per-sender（用户级隔离） ：每个发送者一个会话。

根据 Session 的隔离策略，OpenClaw 会判断 Message 应该在哪个 Session 中处理。

此外，OpenClaw 还支持跨平台身份链接。比如，同一个用户在 Telegram 与 WhatsApp 上的身份可以被绑定到同一个 UserID，从而把两端的 Message 合并到同一个 Session 中，实现跨平台的上下文连续对话。

5. ## Agent 装备加载

在 OpenClaw 中，Agent 本质上是一个 ReAct 循环，而不是一个长期驻留后台、独立于任务的守护进程（Daemon）。

它不知道自己处于哪个会话、该使用哪些工具、能访问哪些能力。所有这些 “装备”，都由 Gateway 在交给 Agent 任务前完成组装，包括：需要将 Session Store、Skill、Tool-Policy 相关的信息加载。

* Session Store ：`~/.openclaw/agents/main/sessions/`，该目录下记录了 Agent 的会话信息。
* Skills ：`~/.openclaw/workspace/skills`，记录了 Agent 安装的技能。
* Tool Policy ：记录了 Agent 的工具黑白名单。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NDJiNTNjYWJkODRiMzk0ZTkxY2NmNjljMWJkNTcxOTZfcUxhSDVDMEhNREFSbmk3QTk3Z0d0eTVVeTFVQkxUbDhfVG9rZW46UUhpSmJFc3dtb3UwUjd4NDVabmNWOWxEbkpjXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. ### 完整的消息处理全链路
2. 用户从各类消息渠道发送对话消息，进入系统；
3. 网关的 Router 接收消息，匹配规则后分发到指定的目标 Agent；
4. 网关同步完成三件事：通过 Session Store 加载会话状态、通过 Skills Registry 注入可用技能、通过 Tool Policy 加载安全校验规则，全部传递给下游 Agent；
5. Agent 层基于网关注入的信息，构建 System Prompt，初始化大模型的行为规则；
6. 启动 ReAct 核心循环：将用户消息 + 系统提示词 + 历史记忆，交给 LLM 做推理思考；
7. 若 LLM 判断无需调用工具，直接生成最终回答，流程结束，结果返回给用户；
8. 若 LLM 判断需要调用工具，生成工具调用指令，交给 Tool Runner 执行；
9. Tool Runner 完成工具执行后，把结果返回给 Agent Loop，再次喂给 LLM 做下一步推理；
10. 「思考 - 行动 - 结果反馈」的循环持续执行，直到任务完成，LLM 生成最终回答返回给用户；
11. 全程中，Memory 持续更新对话与任务记忆，Session Store 同步更新会话状态，Tool Policy 全程校验工具调用的安全性。
12. ### `Session` 加载

Session Store（会话加载）环节，主要是 Gateway 会将 session.jsonl 进行加载。Gateway 通过 SessionKey 查找 sessions.json 获取当前 SessionId，再根据 SessionId 找到对应的 session.jsonl 加载到 Agent 中。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MTBiZmYxODI4MzBhZGQ5ZGU1OWJkYTYzYWE5YTJiMWVfN3F2MWtzVmtLa2E4TGtWQXhxeHdWR0Q2NllrVHpWOEdfVG9rZW46VzVUZWJOUm5Tb1NQSVJ4NGFDWmNmM0Z0bjlnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

session.jsonl 文件中保存在该 Session 的对话历史，作为 Agent 的短期记忆，会周期性的进行 reset 新会话。session.jsonl 的第一行是 Session Header，结构如下，记录了 SessionId。

```JSON
{
    type: "session",
    id: "session-uuid",           // sessionId
    cwd: "/path/to/workspace",    // 工作目录
    timestamp: 1234567890,
    parentSession?: "parent-id"   // 可选，fork 时存在
}
```

后面的内容主要包括对话记录，结构如下，记录了对话的时间和内容。

```JSON
{
"type": "message",
"id": "XXX",
"parentId": "XXX",
"timestamp": "XXX",
"message": {
"role": "toolResult",
"toolCallId": "XXX",
"toolName": "read",
"content": [{
"type": "text",
"text": "XXXX"
                }],
"isError": false,
"timestamp": XXX
        }
}
```

3. ### `Skills` 加载

OpenClaw Skills 有 3 个安装级别，加载优先级（从高到低）为：

1. Workspace skills ：`~/.openclaw/workspace-xxx/skills`，Agent 级别，只在 Agent 中生效。
2. User skills ：`~/.openclaw/skills`，用户级别，所有 Agent 共享。
3. Bundled skills ：内置技能，随 OpenClaw 的 npm 安装包分发。

分层的原因有 2 个：1）避免 Skills 太多从而降低了匹配准确率；2）避免 Skills 太多占用了上下文窗口。因为在实际使用时，我们需要区分 “基础通用能力 Skills” 和 “Agent 专属能力 Skills”。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=OTFlYzQyNjk3ZWU2MWM5MmM2MjQzMTVkNzkzODkzYjVfTlNNRWx0SjE1UHhOeEZxRHFQdUVZMWcxMnloTkFEb29fVG9rZW46R0R5bWJhejdPb3JhdmR4MldXZGNjSnpIblpnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

Agent 启动时会扫描上述 Skills，并构建一个 ` Skills Snapshot`（快照），其中包含：

1. 过滤后的 `Skills List`
2. 格式化后的文本片段，包括 `Skill name` 和 ` Description`，用来注入 `System Prompt`。
3. 相关的环境变量，Skill 可以声明自己依赖的命令路径等。

值得注意的是，Skills Snapshot 只会在 New Session 的第一轮构建，之后的同一 Session 的多轮对话都会复用，不会再次扫描。所以如果更新了 Skills List，建议 New Session 之后使用。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MzI3MGEyZWZkY2MzOTM3ZGFhMjk5Mjk1NDk5ODllNjZfQVFZRVdDR2RRNlhYdVFYbWhrck51b2tGSWxUQng2VExfVG9rZW46QlFNTWJtQjE4b1J4Z1F4Rko4MmNJZlZkbmpjXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

OpenClaw 内置了大量的 Skills（49+ 个），包括：

* Apple 生态 ：Notes、Reminders、Things 3、Bear Notes 等。
* Google Workspace ：Gmail、Calendar、Drive、Docs、Sheets（通过 gog CLI）等。
* 通信工具 ：Slack、iMessage、Twitter/X、Discord 等。
* 智能家居 ：Philips Hue、Sonos、Eight Sleep 等。
* 开发工具 ：GitHub CLI、Claude Code 子进程、Whisper 转录等。

值得注意的是，由于 Skills 的安装实在过于简单，非常容易导致 Skills 膨胀、低质扩散的情况。所以推荐定期使用高阶模型对 Skills 进行扫描，清理低质量和不必要的 Skills。

4. ### `Tool-policy` 加载

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2M2YzcyODUxMGNmMTUxYmZkYmU2YjI2Njk5M2NiYjhfbUc4STgwMXpqSVdRSHV5VG9nUFFWT2VWWFpVMXBBVkdfVG9rZW46UkRhTGJXWk80b08wZjF4aDZXd2NYRWNWbldlXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

如上图所示，`Tool-policy` 是 Open Claw 架构中 Gateway 核心安全组件，为企业级 AI Agent 提供基于身份的工具调用访问控制列表（ACL） 能力，通过网关层前置工程化约束替代 Agent 层提示词软约束，实现 Agent 工具权限的零信任、动态化、细粒度管控。

1. #### Tool-policy 核心设计特性
2. 网关层前置管控，非 Agent 运行时管控

   1. 管控时机：Agent 装备工具前完成权限过滤
   2. 实现效果：无权限工具不会被加载至 Agent 执行环境，从源头杜绝越权调用，彻底规避提示词注入、Agent 行为不可控导致的权限逃逸。
3. 基于身份的动态白名单生成机制
   Message 流入 Gateway 后，网关依据** ChannelId、UserId、GroupId** 等多维度身份标识，实时计算并生成本次会话专属的工具可用白名单，权限与会话场景强绑定：

   1. 家庭群组：仅开放 `read`/`sessions_list`/`sessions_send` 基础消息类工具
   2. 工作群组：开放 `exec`/`browser` 等高阶能力工具
   3. 公开 Discord 服务器：仅保留只读类工具，实现最小权限管控
4. #### 零信任安全架构

Tool-policy 遵循 零信任安全架构 ，重构 Agent 权限管控逻辑：

1. Agent 默认不可信以 “Agent 本身不具备权限自管控能力” 为前提，通过权限最小化原则防控风险，核心手段为非必要不授予工具调用权限。
2. 权限中心化管控 Agent 工具调用权限不由自身决策，统一由 Gateway 授权中心 集中管控、统一下发，实现权限管控的中心化、标准化。
3. 场景化实时授权权限不与用户 / Agent 永久绑定，用户切换群组、任务等场景时，Gateway 立即重新计算权限，无跨场景权限复用，杜绝权限扩散。
4. #### Tool-policy 八层权限过滤管道
5. Owner 身份门槛 ：身份级别门槛，在进管道之前就隔绝某些高危操作。比如：某些 tool 仅能由渠道的 Owner 调用。
6. Profile 角色模板 ：角色模板，定义某类 Agent 的基本能力边界。比如：一个 Coding  Agent 可能不允许使用 web_search 工具。
7. Provider Profile 模型适配策略 ：同一 Agent 根据不同的 LLM 调整能力边界。比如：切换到 Gemini 时某个工具要被移除。
8. 全局组织策略 ：组织级别统一规则，所有 Agent 都受约束。比如：组织内所有 Agent 都不能操作远程设备（Nodes）。

```JSON
{
     tools: {
       // 禁用特定工具
       deny: ["browser"],

       // 只允许特定工具
       allow: ["exec", "read", "write"],

       // 使用预定义工具组
       profile: "messaging", // minimal | coding | messaging | full

       // 按提供商限制
       byProvider: {
         "openai/gpt-5.2": { allow: ["group:fs"] }
       }
     }
   }
```

5. Agent 单体策略 ：精细化到单个 Agent 的工具权限定制。比如：某个分析 Agent 只能读文件和搜索，但不能调用 exec 工具。

```JSON
  {
     agents: {
       list: [
         {
           id: "support",
           tools: {
             profile: "messaging",  // 继承全局 profile 后进一步限制
             deny: ["browser"],
             byProvider: {
               "openai/gpt-5.2": { profile: "minimal" }
             }
           }
         }
       ]
     }
   }
```

6. 群组 / 渠道场景策略 ：同一 Agent 在不同渠道/群组中有不同的工具能力。家庭群、工作群、公开群有不同的权限。比如：公司公告群里的请求不允许用 exec 工具；但研发群可以。

```JSON
   {
     bindings: [
       { agentId: "family", match: { channel: "whatsapp", accountId: "family", peer: { kind: "group", id: "family-group" } } },
       { agentId: "work", match: { channel: "whatsapp", accountId: "work", peer: { kind: "group", id: "work-group" } } },
       { agentId: "stranger", match: { channel: "whatsapp", accountId: "public", peer: { kind: "group", id: "public-group" } } }
     ],
     agents: {
       list: [
         {
           id: "family",
           tools: { allow: ["read", "sessions_list"] },
         },
         {
           id: "work",
           tools: { allow: ["read", "exec", "write"] },
         },
         {
           id: "stranger",
           tools: { deny: ["exec", "write", "browser"] }
         }
       ]
     }
   }
```

7. Sandbox 沙箱硬隔离策略 ：安全隔离环境的硬边界，不可被上层绕过。比如沙箱环境只允许 read 工具，但不允许使用 write 工具。

```JSON
   {
     tools: {
       sandbox: {
         tools: {
           // 沙盒会话允许的工具
           allow: [
             "exec",
             "process",
             "read",
             "write",
             "edit",
             "apply_patch",
             "sessions_list",
             "sessions_history",
             "sessions_send",
             "sessions_spawn",
             "session_status"
           ],
           // 沙盒会话禁用的工具
           deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"]
         }
       }
     }
   }
```

8. Subagent 子代理策略 ：框架层面防“递归爆炸”的系统性保护。比如：subagent 不允许使用 spawn 工具再递归产生 subagent。

```JSON
 {
     agents: {
       defaults: {
         subagents: {
           // 子代理默认工具策略
           maxConcurrent: 8,
           maxSpawnDepth: 2,
           tools: {
             allow: ["read", "exec"],
             deny: ["browser", "gateway"]
           }
         }
       }
     }
   }
```

OpenClaw 预定义工具组：

| 组别                                               | 包含工具                                                                           |
| -------------------------------------------------- | ---------------------------------------------------------------------------------- |
| -------------------------------------------------- |                                                                                    |
| `group:runtime`                                  | `exec, bash, process`                                                            |
| -                                                  | -                                                                                  |
| `group:fs`                                       | `read, write, edit, apply_patch`                                                 |
| `group:sessions`                                 | `sessions_list, sessions_history, sessions_send, sessions_spawn, session_status` |
| `group:memory`                                   | `memory_search, memory_get`                                                      |
| `group:web`                                      | `web_search, web_fetch`                                                          |
| `group:ui`                                       | `browser, canvas`                                                                |
| `group:automation`                               | `cron, gateway`                                                                  |
| `group:messaging`                                | `message`                                                                        |
| `group:nodes`                                    | `nodes`                                                                          |

6. ## 会话并发控制

Gateway 的会话控制层，用于控制和 Agent 之间的会话并发策略。

* Queue mode：防止消息爆炸（高频场景），优化用户体验。
* Session lane：防止会话冲突（文件并发、状态竞争），保证数据一致性。

```JSON
 {
     messages: {
       queue: {
         mode: "collect",
         debounceMs: 1000,    // 防抖延迟
         cap: 20,              // 每会话最大排队数
         drop: "summarize",    // 溢出策略
       }
     },
     agents: {
       defaults: {
         maxConcurrent: 4,     // main lane 并发上限
         subagents: {
           maxConcurrent: 8,   // subagent lane 并发上限
         }
       }
     }
   }
```

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=OTY2NmQ0ODI1MTk4NTkxZmY0MmI2YzVhNzQ1NTgyNzRfVE50UDQ2Zmw0NDdwbThrRllHbkd3V0NHN0VXcmJiZW1fVG9rZW46RDhSd2JmcTZjb1pXQzF4bHZDTGMwNlB6bjFnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. ### `Queue Mode`（队列模式）

当我们使用 IM 和 OpenClaw 进行对话时，经常会出现 “高频消息输入” 的情况，比如一口气输入 “你好”、“我想问下”、“如何给 OpenClaw 安装新的 Skill”、“能帮我安装吗”...。

对此，当多条消息快速到达时，Gateway 提供了高频消息输入的控制策略。

* Followup（跟进）模式 ：按 FIFO 顺序逐条处理，每条消息都作为一次 Agent 任务获得一次完整回复，然后再开始处理下一条消息。

```JSON
   消息1 ──> 任务1 ──┐
   消息2 ──> 任务2 ──┼─> session:abc (lane) ──> 顺序执行3次
   消息3 ──> 任务3 ──┘
```

* Collect（搜集）模式 ：在时间窗内（比如 2s）先等待，把窗口内到达的消息聚合成一条，再作为任务交给 Agent。

```JSON
   消息1 ─┐
   消息2   ├──> [合并] ──> 1个任务 ──> session:abc (lane) ──> 执行1次
   消息3 ─┘
```

* Steer（转向）模式 ：在一个时间窗内的新消息会被路由到正在运行的上一轮 Agent Loop；如果失败，则等待下一轮 Loop。例如在 ReAct Loop 中的某一次 LLM API 调用结束后的间隙，将新消息追加到消息流后，从而参与到下一轮的 LLM 推理中。

```Plain
   消息1 ──> [正在运行] (直接注入，不进lane)
   消息2 ──> 如果当前空闲 → 立即执行
            如果忙碌     → 退化为 followup → 进 lane
```

* Interrupt（打断）模式 ：时间窗内的新消息到来后直接中断当前运行、清空队列并丢弃回复，从新消息重新开始。

值得注意的是。如果 OpenClaw 正在执行一个很长的任务（比如搜索十个网页），你中途想让它换个方向，只能干等它做完。改成 steer 模式后，你发的新消息会实时注入到 AI 正在处理的任务中，它会立刻调整方向，不用干等。

```JSON
{
  "messages": {
    "queue": {
      "mode": "steer"
    }
  }
}
```

2. ### Lane（车道）并发控制

Lane 是 Gateway 的并发控制机制，不同类型的任务运行在独立的 Lane 中，每个 Lane 有自己的并发上限：

* main lane ：全局默认 lane，处理 main Agent 的任务（默认并发 4）。
* subagent lane ：处理 subagent 的任务（默认并发 8）。
* session:[key] lane ：按 New session 按钮分隔，保证每个会话只有一个活跃运行（默认并发 1）。
* cron lane ：定时任务。
* Telegram 特定 lane ：answer 和 reasoning，用于不同类型的内容流。

3. ### Session Lane（会话车道）

首先要区分「普通 RESTful API」和「对话式 Agent」的核心差异：

1. 无状态 HTTP + RESTful API ：
2. 这类接口（比如查天气、查订单的 API）是「无状态」的——每个请求独立，服务器不用记住上一次请求的信息。因此可以 高并发并行处理 ：比如 1000 个用户同时查天气，服务器能同时处理这 1000 个请求，互不干扰。
3. 对话式 Agent（比如智能客服、AI 对话机器人） ：
4. 这类服务是「有状态」的——Agent 必须记住整个对话的上下文（比如你先问「北京明天天气」，再问「上海呢」，Agent 要知道「上海呢」是承接上一句的天气问题）。
5. 且 Agent 处理单条消息需要耗时：要走「思考（Thinking）→ 行动（Action）→ 观察（Observation）」的流程（比如 AI 要先理解问题、调用工具、生成回答），这个过程可能需要几百毫秒甚至几秒。

如果直接用处理 RESTful API 的「并行方式」处理对话，会出现严重问题：

* 场景举例：你给 Agent 发了消息 A，Agent 正在耗时处理 A；此时你又发了消息 B。如果服务器并行处理 A 和 B，就会出现：
  * 上下文窗口混乱：Agent 可能把 B 的内容错误混入 A 的处理流程，导致回答驴唇不对马嘴；
  * 状态混乱：比如 A 还没处理完，B 已经修改了会话的状态，最终整个会话的逻辑完全错乱。
* 这种「同一个会话的多条消息并行处理导致的冲突」，就是「对话竞态」。

Gateway（网关）的「Session Lane」功能就是为了解决这个问题，核心逻辑可以用「银行窗口」的比喻理解：

| 概念                                                                                                           | 通俗比喻                 | 具体逻辑                                                                                           |
| -------------------------------------------------------------------------------------------------------------- | ------------------------ | -------------------------------------------------------------------------------------------------- |
| -------------------------------------------------------------------------------------------------------------- |                          |                                                                                                    |
| Session                                                                                                        | 一个用户的一次完整对话   | 比如你和 Agent 从开始聊天到结束，这整个过程就是一个 Session，有唯一标识。                          |
| -                                                                                                              | -                        | -                                                                                                  |
| Lane（车道）                                                                                                   | 给这个用户分配的专属窗口 | Gateway 为每个 Session 分配一个唯一的 Lane（可以理解为「专属处理通道」）。                         |
| 串行队列                                                                                                       | 窗口前的排队队伍         | 每个 Lane 对应内存里的一个「串行队列」——队列里的任务必须「先来后到」，一个处理完才能处理下一个。 |

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTYyMDIxYmQwNGYyNTBlZjEyNGM3NGQwZWM5ZDM3NDVfbXhmSE96SjJVQlhoTDlEMFhIenZEOGJmRld4ckF2UUlfVG9rZW46WHBJN2JjSkxJb1RjVHp4QzZkYmN4ZmpWbkRnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ODAxNWJjMzIzYTJiZDQ2N2MyNDYwOWUxZTZlMTVjMGZfYXdiTGtoQVNGSzRmcnQxOVFLbGZ4RkFCTU55SnNnVTlfVG9rZW46WVcyeWJVTGhhb1NGUUJ4aXRNbGNBZENrbkFkXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

 对有状态 Agent 对话来说，“同一会话的任务串行性” 是保证一致性的基本方案 。不要轻易并发同一用户的多轮对话，尤其当工作流还涉及写操作甚至人类参与时。

暂时无法在飞书文档外展示此内容

7. ## 配置热重载

Gateway 设计了一套精细的配置热重载机制，核心原则是：能热更新的就热更新，必须重启的才重启。 Gateway 提供 4 种重载模式，通过 gateway.reload.mode 进行配置：

| 模式                               | 行为                                           |
| ---------------------------------- | ---------------------------------------------- |
| ---------------------------------- |                                                |
| off                                | 完全禁用，任何变更都忽略                       |
| -                                  | -                                              |
| hot                                | 仅热重载，需要重启的变更被忽略（不会自动重启） |
| restart                            | 任何变更都触发 Gateway 重启                    |
| hybrid（默认）                     | 能热更新就热更新，需要重启就重启               |

当 openclaw.json 发生变化时，Gateway 并不会简单地 “全部重载”，而是对每一个配置路径逐一判断应该如何处理：

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NDdhMjQyZTFjOTg3ZjU3YjJjNmY2NTg5MjUzY2I2Y2RfS0h3S1FFRzZFWUIxVEkxd0RyamhRdndUSnpTSGswNTRfVG9rZW46RUNDY2JRMXJZbzBlc1p4anprSWN5TG5tblFnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

每条规则指定两个属性：kind（热更新 / 需重启 / 无需处理）和 actions（具体的热更新动作，如重启某个模块）。比如：

```JSON

{ prefix: "hooks.gmail", kind: "hot", actions: ["restart-gmail-watcher"] },
// → 改了 hooks.gmail.model，只重启 Gmail 监听器

{ prefix: "hooks", kind: "hot", actions: ["reload-hooks"] },
// → 改了其他 hook 配置，重载 hook 模块

{ prefix: "cron", kind: "hot", actions: ["restart-cron"] },
// → 改了定时任务配置，只重启 cron 调度器
```

这里如果修改了定时任务配置，则只会重启 cron 调度器。

8. ## 数据流

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NGE3YzdiNzZkYmQxZWFkY2U0ZWRiNzFkYjI0NzY0M2ZfTXpnUmRLTWxqbmRzQUJaclluUWM5UWZyWllOV01WdVBfVG9rZW46T0U3cGIwdFYwb3pKZmV4ZUhXN2NDb1dObkJ5XzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

```Bash
外部消息 
  → 渠道层 (WhatsApp/Telegram/etc.)
    → 频道元数据解析 (channelId, peerId, guildId)
      → 路由解析 (resolveAgentRoute)
        → 6 级路由优先级匹配
        → 7 级配置匹配
          → 多层访问控制检查
            → Allowlist 白名单
            → Mention Gating (群组)
            → Command Gating (控制命令)
              → 会话激活/创建
                → SessionKey 解析 (agentId + scope + peerId)
                  → 消息入队 (双车道)
                    → sessionLane: 单会话串行
                    → globalLane: 全局并发控制
                      → Agent 执行 (runEmbeddedPiAgent)
                        → Auth Profile 选择/轮转
                        → Thinking Level 应用
                        → 上下文溢出检查
                        → 工具调用 (如果需要)
                        → 流式响应生成
                          → 响应处理
                            → 类型转换 (适配频道)
                            → 分块发送 (长消息)
                            → 重试机制
                              → 渠道发送
                                → 会话持久化
                                  → 用户收到回复
```

3. # Channels

OpenClaw Channels 负责将多个 IM（即时通讯平台）接入到中央的 Gateway，以 WebSocket 协议进行消息的收发通信，并翻译为 Gateway 的统一标准格式。

OpenClaw Channels 采用了 Plugin 插件化的架构设计，所有的消息平台都是 Channel 插件。只负责 “接入层” 的事情，包括：登录、收发消息、协议适配（把平台事件转换成标准事件）。而负责的事情则交由 Gateway 负责，包括：状态管理、业务决策、业务决策、权限与调度策略等。可见 Channel 是轻量化的、驱动层化的，接入一个新的消息平台，只是多一个插件。

插件工作流程其实很朴素，但很有效：

1. 插件接收外部事件
2. 转换为标准事件
3. 提交给 Gateway
4. Gateway 处理并生成响应
5. 插件把响应转回平台格式发出去

下面是一个 Feishu 平台中的 2 个账户接入的示例：

1. 核心作用：配置飞书渠道的启用状态、消息接收策略、连接方式和多账号认证信息。
2. 关键字段：`enabled`控制渠道开关，`groupPolicy/dmPolicy`分别管控群聊 / 单聊消息规则，`appId/appSecret`是飞书应用的核心鉴权信息。
3. 灵活扩展：`accounts`支持多账号配置，可满足不同业务线使用不同飞书应用的场景。

```JSON

  "channels": {
    "feishu": {
      "enabled": true,
      "groupPolicy": "open",
      "dmPolicy": "pairing",
      "allowFrom": [
        "*"
      ],
      "connectionMode": "websocket",
      "accounts": {
        "default": {
          "appId": "XXX",
          "appSecret": "XXX",
          "domain": "feishu"
        },
        "claudecode": {
          "appId": "XXX",
          "appSecret": "XXX",
          "domain": "feishu"
        }
      }
    }
  },
```

4. # Agents 智能体

OpenClaw 支持 multi-Agent 架构，主智能体称之为 Main Agent，是一个精简高效的编程智能体。 OpenClaw Agent 采用 Agent Loop 运行模式，处理用户消息、执行工具调用、将结果反馈给 LLM，循环直到 LLM 生成无工具调用的响应为止。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MmIxZDc0MTAyNTEwZDE1ZGJhYzM5ZGU2NTQ0ZTdlMDRfQ0ZoS1VjcnI5bkZwSHFmS0x1cnNGYldVQ0J3MUJndmtfVG9rZW46TGREdGJFTkdlb1BHRHd4VW80Y2M4SXhpbm5nXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. ## 智能体运行器（Agent Runner）

`Agent Runner`是基于 `Pi Agent Core`（作者：Mario Zechner）这个 Agent 框架基础之上扩展而来的 Agent 执行引擎，负责协调所有 Agent 交互的模型推理、工具执行和会话管理。

在调用 LLM 之前，Agent Runner 会进行 `Context Engineering`（上下文工程） 的工作，包括：

1. LLM 解析器 ：动态选择最适用于当前任务的 LLM。
2. System Prompt 构建器 ：根据当前启用的 Skills（如浏览器自动化、文件访问等）动态组装 System Prompt，动态加载和构建可以剩下很多上下文窗口。
3. 会话历史加载器 ：从本地存储的 Workspace 目录下读取持久化的记忆数据。
4. 上下文窗口卫士（Context Window Guard） ：当 Context Windows 接近 LLM OpenAPI 的 Max token 限制时（如 80%），Agent 就会自动压缩，以此来确保对话不中断。但是压缩过后的 Context Windows 很可能会丢失关键对话记录。

准备好最终的 Prompt 集合后，Agent Runner 才会调用 LLM OpenAPI 发送请求，并将 LLM 返回的结果交由下一层 Agentic Loop 来处理。可见在 Agent Runner 环节，token 的用量优化，以及 Context Windows 的有效压缩就成为重要的考量之一。

2. ## Context Window Guard 上下文窗口管理

在 LLM API 的上下文窗口是有限（例如 128k）。多轮对话 Agent 随着会话变长，再叠加 Tools、Skills 等，Prompt Size 很可能会超限，导致 LLM API 不可用。

所以 Agent Runner 实现了 `Context Window Guard`，在构建 Prompt 完成但尚未调用 LLM API 之前，进行多阶段的防卫：

1. 按阈值做 `token size` 检查。
2. 如果通过了检查，但运行过程中仍触发上下文溢出的话，则启动自动压缩（使用 LLM 将前面的会话历史进行结构化摘要）。
3. 如果压缩失败，则此系统会尝试重置会话，并提示用户。

值得注意的是，上下文的压缩是有代价的，它会带来信息损失与摘要偏差，而重置会话也可能影响连续性。企业场景里可以根据需要配置独特的摘要策略、关键事实保护（如订单号/金额/日期）、以及可追溯的日志等，尽量避免压缩后的跑偏。

上下文窗口管理是 Context Engineering 的一部分，成熟的 Agent 系统要用 “事前检查 + 事后补救” 的多层策略，尽量别让 LLM 报错成为用户看到的最终错误。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjUwYTExZTEzY2NmNGJiYzZmM2IzZmJhMTAwMTA2Nzdfd2dDdlBQcnl4eGNoNEtPSEI5S3pGQW9zNzV1Y1R4M21fVG9rZW46V1dTOWJEVzlvb0hadEN4TkhHT2NVZXdublRmXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

上图是  OpenClaw Gateway 针对 LLM 上下文窗口约束的 Token 生命周期管理与溢出容错流程 ，通过预检查、分级告警、模型降级、摘要压缩、会话重置的分层策略，在保障服务可用性的同时，最大化对话上下文的连续性。

1. #### Prompt 构建与 Token 预校验阶段

* 输入 ：用户请求 + 历史对话上下文 → 构建完整 Prompt
* 核心逻辑 ：基于模型上下文窗口（示例为 32k Token）执行预校验：
* Token ≥ 32k ：直接进入请求分发流程（预期将触发上下文溢出）
* 16k ≤ Token < 32k ：记录 WARN 级日志告警，标记「上下文接近容量上限」，继续请求分发
* Token < 16k ：抛出 `FailoverError`，触发模型降级链（切换至更小上下文窗口的备用模型），再执行请求分发

2. #### 请求分发与结果判定阶段

* 请求分发 ：将 Prompt 提交至目标 LLM 服务
* 结果分支 ：
* 请求成功 ：直接返回模型生成结果，流程终止
* 上下文溢出（Context Overflow） ：进入溢出容错处理分支

3. #### 上下文溢出容错处理阶段

* 首次溢出判定 ：
* 非首次溢出 ：直接返回错误提示并执行 会话重置 ，终止当前对话链路
* 首次溢出 ：触发 LLM 驱动的上下文摘要压缩
  * 对历史对话中较早的消息进行结构化摘要提取，用摘要替换原始长文本，降低 Token 占用
* 压缩结果判定 ：
* 压缩成功 ：基于压缩后的上下文重建 Prompt，重新进入请求分发流程
* 压缩失败 ：返回错误提示并执行 会话重置 ，终止当前对话链路

2. ### LLM API 思考等级划分

有些任务是需要 LLM 深度思考和推理，但有些任务只需要快速回答即可。所以 OpenClaw Agent 采用了思考分级机制，定义了 6 个递增的思考等级，用来对应不同复杂度的任务：

| 等级                                       | 适用场景举例                     |
| ------------------------------------------ | -------------------------------- |
| ------------------------------------------ |                                  |
| `off`                                    | "你好"、查时间、简单翻译         |
| -                                          | -                                |
| `minimal`                                | 快问快答、格式转换、简单总结     |
| `low`                                    | 默认。日常对话、写邮件、代码补全 |
| `medium`                                 | 代码 Debug、多步分析、方案对比   |
| `high`                                   | 架构设计、复杂算法、长文推理     |
| `xhigh`                                  | 仅限 `gpt-5.2-codex`等特定模型 |

需要注意的是，不同 LLM 对 “思考等级” 的支持差异很大。有的支持多档位的思考级别、有的只有思考的开/关、有的则完全不支持。因此 OpenClaw 会根据所选的 LLM，把内部等级映射到对应的 LLM API 参数。

当一条消息进入系统后，思考等级会按一定优先级确定：

 优先级递减 ：系统按「内联指令 → Session 指定 → Agent 配置 → 模型兜底」的顺序依次检查，一旦匹配到某一级别，就直接采用该思考等级，不再向下检查。

内联指令（Inline Command） 是指在 用户发送的业务消息内容中，直接嵌入一条系统指令 ，让系统在处理这条消息时，临时执行特定逻辑（这里就是指定「思考等级」）。指令不是单独发送，而是「嵌在同一条消息里」。

 粒度控制 ：

* 内联指令：单消息级 精细控制，适合临时需要高强度 / 低强度思考的场景。
* Session 指定：会话级 持久控制，适合整段对话需要统一思考强度的场景。
* Agent / 模型配置：全局级 基础控制，作为默认兜底策略。

| 优先级         | 来源                                    | 示例                                    | 生效范围 |
| -------------- | --------------------------------------- | --------------------------------------- | -------- |
| ①内联指令     | 消息中带 `/think high`                | `帮我优化这段代码 /think high`        | 本条消息 |
| ②Session 指定 | 单独发送 `/think medium`              | 发送纯 `/think medium`（无内容）      | 后续消息 |
| ③Agent 配置   | `agents.defaults.thinkingDefault`配置 | `openclaw.json`中配置                 | 全局默认 |
| ④模型兜底     | 检查模型信息目录来决定等级              | 如果模型支持 Reasonning，否则 →`off` | 全局配置 |

另外，如果运行时遇到 LLM API 返回类似 “当前模型不支持该思考等级” 的错误，系统会解析错误信息并自动降档重试 ，尽量避免任务直接失败。

3. ### LLM API 故障转移

使用单一 LLM 存在 “单点故障” 风险，OpenClaw Agent 支持 Multi-Models。并实现了 2 层容错机制：

* 内层认证轮转 ：同一个 LLM 内的多个 API Key 实现自动切换，不断尝试下一个不在冷却期内的 Key（指数退避机制），直到成功或所有 Key 都试完。
* 外层模型降级 ：自动降级到下一个候选 LLM。

需要注意，模型降级可能带来 LLM 能力差异，甚至影响工具调用的稳定性。企业落地时往往需要明确模型分层与各自能力基线（哪些任务允许降级、哪些必须强模型、允许降级到哪个模型等），并记录降级日志，便于排查与审计。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YjAyNzk4ZmNhNDM5ZDI3NDgzMjhjN2MxODczMmIyZTJfclA5UThjeW5aTk5xYk85dklMWUtaNlgyUkZYUTNadGJfVG9rZW46T1pTZWJwejVHb3hKd294ZjBBWWNlQkFCbmxoXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

上图是  OpenClaw Gateway 的 「双层高可用模型调度架构」，由内层认证轮转和外层模型降级链组成，目标是在 API Key 耗尽、模型服务不可用等故障场景下，保证用户请求能持续被处理，提升系统可用性与容错能力。

1. #### 内层：认证轮转（Key Rotation）

内层负责单个模型的多 Key 负载均衡与故障转移：

* 为每个模型配置一组 API Key（如 `Key A → Key B → Key C`），按顺序轮转使用。
* 当当前 Key 因额度耗尽、限流或鉴权失败时，系统自动切换到下一个 Key 重试。
* 只有当该模型的所有 Key 都耗尽 / 失效时，才会触发外层的「模型降级链」，切换到下一个备选模型。

2. #### 外层：模型降级链（Model Failover Chain）

外层负责跨模型的服务降级与故障转移，按预设优先级依次尝试模型：

1. 首选模型 ：优先使用 `Claude Sonnet`，并通过内层 `Key A/B/C` 轮转保证其可用性。
2. 一级降级 ：若 `Claude Sonnet` 所有 Key 耗尽，则自动降级到 `Gemini Pro`，并使用其对应的 `Key D/E` 轮转。
3. 二级降级 ：若 `Gemini Pro` 所有 Key 也耗尽，则继续降级到 `gpt-4o`。
4. 成功终止 ：只要任意一个模型成功处理请求，就返回响应，流程结束。
5. ## 智能体处理循环（Agentic Loop）

OpenClaw Agentic Loop 基于开源项目 Pi SDK（https://github.com/badlogic/pi-mono）。 一个 Loop 中包含了 4 个核心阶段：

1. 上下文组装（Context Assembly）
2. 模型推理（Model Inference）
3. 工具执行（Tool Execution）
4. 回复分发（Reply Dispatch）

```Plain
提问 → 思考 → 规划 → 行动 → 观察 → 思考 → 行动 → 等待 → 检查  → 纠错  ... → 完成
```

4. ## Agent 的工作区

OpenClaw 支持 multi-Agent 架构，每个 Agent 有自己专属的 LLM、配置、会话、Workspace、Memory 等数据。如下图所示。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ODM1NmRiMWJkYjE3YTZmODdlNGI1MTdjYmQ1MzgwNDFfdzhNNG96UWQ1SXNEd0w0cFJHQldxYjNsWEd6bERyckZfVG9rZW46U2pmcmJ0bFdVb0Z4Mkd4REF4b2NHNGhLbjRnXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

这张图是OpenClaw 系统的文件目录与数据存储架构，核心是把系统拆分为「全局状态目录」和「Agent 工作区」两大存储域，清晰划分了「系统基础设施配置」和「Agent 业务逻辑定义」的职责边界，下面分模块详细拆解。

1. ### 上层：Global State Directory（全局状态目录）

根目录为 `~/.openclaw`，是 OpenClaw 的 系统级全局数据根目录 ，存放跨工作区、跨 Agent 的公共配置、鉴权信息、持久化状态，是系统运行的基础设施，和单个 Agent 的业务逻辑解耦。

1. 全局基础配置

   1. `全局 openclaw.json`：OpenClaw 实例的核心全局配置文件，是系统启动、运行的核心参数定义文件，覆盖了默认模型、工作区、会话管理、并发控制、子智能体调度、安全沙箱等核心能力的配置，决定了 OpenClaw 实例的基础运行行为。

   ```JSON
   {
     "agents": {
       "defaults": {
         "model": {
           "primary": "zai/glm-4.7"
         }
       },
       "models": {
         "zai/glm-4.7": {
           "alias": "GLM"
         }
       },
       "workspace": "/Users/fanguiju/.openclaw/workspace",
       "compaction": {
         "mode": "safeguard"
       },
       "maxConcurrent": 4,
       "subagents": {
         "maxConcurrent": 8
       },
       "sandbox": {
         "mode": "off",
         "workspaceAccess": "rw",
         "scope": "agent",
         "docker": {
           "image": "openclaw-sandbox-browser:bookworm-slim",
           "workdir": "/workspace",
           "readOnlyRoot": false,
           "network": "bridge",
           "capDrop": [],
           "pidsLimit": 256,
           "memory": "2g",
           "cpus": 2,
           "binds": []
         }
       }
     }
   }
   ```
2. **Agent 配置目录 ****`agents/<agent-id>/agent`**存放单个 Agent 的系统级运行配置，关联两个核心文件：

   1. `auth-profiles.json (API Keys)`：认证配置文件，存放 LLM 调用的 API Key、鉴权凭证、服务商配置，是系统调用大模型的核心认证信息。
   2. `models.json (自动发现的模型)`：模型元数据文件，存放系统自动识别的可用大模型列表、模型能力参数、调用配置，是模型调度、降级链的核心数据来源。
3. **Agent 会话目录 ****`agents/<agent-id>/sessions`**存放单个 Agent 的全量会话持久化数据，核心产出是 `*.jsonl 转写记录`：

   1. 以 jsonl 格式持久化每一轮对话的全量上下文、会话状态、执行日志，支持会话恢复、历史回溯、对话审计等能力。
4. ### 下层：Workspace（Agent 工作区）

根目录为 `~/<agent-workspace>`，是 单 Agent 的业务定义工作目录 ，存放8 个文件

这 8 个文件从能力边界 `AGENTS.md`，身份定位（`IDENTITY.md` ，人格特质（`SOUL.md`，用户适配 `USER.md`，安全管控 `TOOLS.md`，定时能力（`HEARTBEAT.md`，初始化引导（`BOOTSTRAP.md`，长期记忆 `MEMORY.md`8 个维度，共同构建了 Agent 的完整人格与能力体系。

其中 `AGENTS.md` 定义能力边界、`SOUL.md` 注入灵魂、`TOOLS.md` 划定禁区 是核心三要素。

5. ## Agent 的核心配置文件（下层：Workspace）

```JSON

~/.openclaw/workspace
├── AGENTS.md          # 智能体的描述和提示词
├── HEARTBEAT.md       # 系统健康检查清单
├── IDENTITY.md        # OpenClaw 智能体的身份和人设
├── SOUL.md            # 智能体个性定义
├── TOOLS.md           # 可用工具参考
├── USER.md            # 用户偏好/上下文
├── MEMORY.md          # 长期策划记忆（可选）
├── memory/            # 持久化记忆目录
│   ├── 2026-01-28.md  # 每日笔记
│   └── 2026-01-29.md
```

| 配置文件名称                                                                                                                                           | 是否必选 | 核心作用               | 补充说明                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ---------------------- | ------------------------------------------------------------------------------------------------------ |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ |          |                        |                                                                                                        |
| `AGENTS.md`                                                                                                                                          | 是       | 定义 Agent 的行为准则  | 明确 Agent 的核心行为边界，例如保障安全、拟人化回话、维护记忆等，是 Agent 行为的核心规范，推荐仔细阅读 |
| -                                                                                                                                                      | -        | -                      | -                                                                                                      |
| `IDENTITY.md`                                                                                                                                        | 是       | 定义 Agent 的身份信息  | 设定 Agent 的职业/角色标签，例如编程专家、设计师、工程师等，决定 Agent 的核心能力定位                  |
| `SOUL.md`                                                                                                                                            | 是       | 定义 Agent 的人格特征  | 注入 Agent 的性格属性，例如开朗、幽默、沉稳等，决定 Agent 的对话风格与情感表现                         |
| `USER.md`                                                                                                                                            | 是       | 定义主人（用户）的信息 | 记录用户的个性化信息，例如称呼、爱好、习惯等，让 Agent 能精准适配用户需求                              |
| `TOOLS.md`                                                                                                                                           | 是       | 定义 Tool 的权限控制   | 配置工具调用的白名单/黑名单，划定 Agent 可使用/禁止使用的工具范围，用于安全管控                        |
| `HEARTBEAT.md`                                                                                                                                       | 可选     | 定义定时任务配置       | 配置 Agent 的定期执行任务，例如定期检查指定内容、推送提醒等，按需配置                                  |
| `BOOTSTRAP.md`                                                                                                                                       | 是       | 首次启动引导配置       | 仅在 Agent 第一次启动时生效，用于完成初始化引导（onboarding）流程                                      |
| `MEMORY.md`                                                                                                                                          | 是       | 长期记忆文档（RAG 源） | 作为 Agent 长期记忆的数据源，支撑 RAG（检索增强生成）能力，保留长期对话/知识记忆                       |

值得注意的是，Agent 并不是一个常驻的进程，而是一个瞬态实例，每个对话都是一次完整的 “加载-执行-销毁” 循环。即：每次对话都会加载上述内容，用于构建基础的 System Prompt。

1. ### `AGENTS.md`

Agent 启动时都会读取 AGENTS.md，定义 Agent 如何操作，包括：

1. 文件存储在哪里以及内存如何工作。
2. 有哪些工具以及何时使用它们。
3. 何时说话、何时保持安静。
4. 何时上报、何时独立处理。
5. 如何使用任务控制中心，例如任务更新、评论等。

没有 AGENTS.md，那 Agent 的表现无法保持一致。

2. ### `SOUL.md`

个性很重要，具有明确身份的专业 Agent 能产生专注、高质量的输出。 SOUL 文件中包含姓名和角色，例如：

```JSON
姓名：Petra
角色：SEO和关键词研究专家

## 个性
数据驱动的SEO专家。痴迷于搜索意图和用户行为。
用关键词、搜索量和排名难度思考。
从不推荐没有数据支持的目标关键词。
始终考虑从认知到转化的完整客户旅程。

## 你擅长什么
- 关键词研究和搜索意图分析
- 竞争对手内容差距分析
- 页面SEO优化和技术SEO审计
- 内部链接策略和网站架构
- 跟踪排名和识别优化机会

## 你在乎什么
- 搜索可见性和有机流量增长
- 将内容与实际用户搜索行为匹配
- 通过内容集群建立主题权威
- 数据支持的决策而非直觉
```

6. ## `Agent` 的 `Memory`系统
7. ### 集中保存记忆

和 Claude Code、Cursor 等使用 Project 目录和会话来隔离上下文的设计不同。OpenClaw 是你的个人助手，所以它的 Memory 中可以混合保存你与它的所有会话的上下文，而不需要隔离。

* 好处 ：你们在多个会话中聊过的内容它都始终记得，不会因为创建了新会话而忘记。例如：你上午在 Telegram 里让它帮你整理邮件，下午在 Slack 里让它写个报告，晚上在 WhatsApp 里让它安排明天的日程，它全都记得。就像是私人助手一样。
* 坏处 ：特别消耗 token，因为它会将所有上下文一股脑的都输入到 LLM 中。

2. ### 长期记忆提炼

OpenClaw 设计了一个非常巧妙的、基于 Markdown 文件的持久化记忆系统。同样存放在  Workspace 目录下：

* `memory`：目录下是每日的工作记忆。用命名为 YYYY-MM-DD.md 的文件保存，用于实时回顾会话过程，带由时间线和上下文。
* `MEMORY.md`：用于保存提炼后的 “精华” 长期记忆，比如从每日工作记忆中去重、归类、抽取出的关键偏好与结论，长期有效。

更具体的，OpenClaw 每隔一段时间（如 1 天）就会自动 review 最近的 memory 工作记忆，总结一次工作内容，并将有价值的信息提炼到 MEMORY.md 长期记忆文件里面去。

memory 和 MEMORY.md 的分层设计，便于 Agent 在以后有需要的时候进行回忆。例如：一个月后，OpenClaw 依旧可以在回忆（检索）到你当时做过什么。

3. ### 会话压缩前刷新记忆

`Agent Context Window Guard` 压缩上下文之前，为了避免关键信息的缺失，会进行一下记忆刷新。相当于给了 Agent 一次把关键信息写入记忆的机会。

暂时无法在飞书文档外展示此内容

上图展示了「会话启动→记忆检索→对话生成→记忆写入→上下文压缩前记忆持久化」的全闭环流程，核心是让记忆深度参与对话生命周期，同时保障信息不丢失、记忆可追溯。

| 模块                                                                        | 核心职责                                                                                |
| --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| --------------------------------------------------------------------------- |                                                                                         |
| `System Prompt`                                                           | 注入系统级指令、加载工作区引导文件、初始化长期记忆                                      |
| -                                                                           | -                                                                                       |
| `Agent Loop`                                                              | Agent 主循环，统筹对话流程、工具调用、模型推理与记忆写入                                |
| `memory_search`                                                           | 记忆检索工具，负责记忆索引构建、查询、精确片段提取                                      |
| `File System`                                                             | 持久化存储层，保存 `MEMORY.md`（长期记忆）和 `memory/YYYY-MM-DD.md`（时序工作记忆） |

1. 会话启动 ：加载工作区引导配置（如 `BOOTSTRAP.md`），并在会话初始化时一次性加载 `MEMORY.md`全文，将其中的长期精华记忆注入初始对话上下文，为后续交互奠定记忆基础。
2. 注入检索指令 ：`System Promp`t 向 ` Agent Loop` 注入强制指令：「回答问题前，调用 ` memory_search`」，确保 Agent 处理用户查询前必须检索历史记忆，避免跨会话遗忘，保障对话连贯性。
3. 记忆检索与提取 ：
4. 以用户问题为查询词，调用 ` memory_search` (query) 发起记忆检索；若检索时发现记忆索引缺失或损坏，会自动重建索引后再匹配相关记忆；
5. 匹配到相关记忆后，调用 ` memory_get ()` 精确提取指定记忆文件（如 `memory/2026-02-07.md`）、指定行范围的记忆片段，将该片段注入对话上下文供模型使用。
6. 模型回答并写入记忆 ：
7. 模型结合检索到的记忆片段和当前用户问题生成回答；
8. 将本次会话的关键信息（如用户新需求、任务进展、核心结论等）写入当日时序记忆文件（`memory/YYYY-MM-DD.md`），同时 `memory_search` 模块对新增内容做增量索引，更新检索数据。
9. 阈值触发记忆持久化 ：当 `Context Window Guard` 检测到上下文 Token 量接近压缩阈值时，先触发 ` memory flush`（记忆持久化）机制：将当前会话中未持久化的关键记忆写入对应日期的记忆文件；完成持久化后再执行上下文压缩，避免因丢弃旧上下文导致关键记忆丢失。

* 核心流程： 初始化加载记忆→强制检索指令→精准检索提取→生成回答并更新记忆→压缩前持久化记忆 ；

4. ### `MEMORY.md` VS `session.jsonl`

| 维度                                                                                                                                   | Session Store                              | Memory & MEMORY.md                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ | ----------------------------------------------------------------------- |
| -------------------------------------------------------------------------------------------------------------------------------------- |                                            |                                                                         |
| 存储位置                                                                                                                               | `~/.openclaw/agents/[agentId]/sessions/` | `~/.openclaw/workspace/memory/`和 `~/.openclaw/workspace/MEMORY.md` |
| -                                                                                                                                      | -                                          | -                                                                       |
| 所有者                                                                                                                                 | `Gateway`                                | `Agent`                                                               |
| 用途                                                                                                                                   | 存储短期的某个会话中的所有对话             | 存储长期记忆和每日工作记忆                                              |
| 文件格式                                                                                                                               | `JSONL`                                  | `Markdown`                                                            |
| 内容类型                                                                                                                               | 完整对话历史（User/Assistant 消息）        | 人工总结、长期记忆、参考信息                                            |
| 访问方式                                                                                                                               | Gateway 自动读取和管理                     | Agent 通过 `memory_search`/`memory_get`读取                         |
| 会话隔离                                                                                                                               | 每个 session key 一个文件（独立）          | 全局共享（所有会话都可访问）                                            |
| 大小控制                                                                                                                               | 通过 `session.maintenance`自动清理       | 手动管理                                                                |
| 搜索支持                                                                                                                               | 不支持语义搜索                             | 支持 `memory_search`语义检索                                          |
| 向量索引                                                                                                                               | 不支持                                     | 支持嵌入向量 + BM25 混合搜索                                            |

`Session Store` 短期记忆具有 `reset`机制：reset 后会在 x.jsonl 添加后缀 x.jsonl.reset.date。

```JSON
   {
     session: {
       reset: {
         mode: "daily",        // 每日 4:00 AM（Gateway 主机时间）
         atHour: 4,            // 自定义小时
         idleMinutes: 120      // 空闲 2 小时后过期
       }
     }
   }
```

MEMORY.md 长期记忆具有 Flush 机制：

```JSON
   {
     agents: {
       defaults: {
         compaction: {
           memoryFlush: {
             enabled: true,
             softThresholdTokens: 4000,
             systemPrompt: "会话接近压缩。现在存储持久记忆。",
             prompt: "将任何长期笔记写入 memory/YYYY-MM-DD.md；如果没有要存储的内容，回复 NO_REPLY。"
           }
         }
       }
     }
   }
```

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=Mjk2ZDc1MzY3MzhiMmUwNTZhMjQxZDg2MjVlZGQyMzNfS1hHc3p2cFZiT2hUZWJLY3BBNGJWUWw0R1dqYzhtRzVfVG9rZW46Tkxsa2JQYW14b2Y2bm54dUdnTmNTbmdsbnlmXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

上图展示了  OpenClaw Agent 的完整启动与初始化流程 ，核心分为「同步启动阶段」和「异步后台阶段」，下面为你详细拆解：

1. #### 顶层入口：Agent 启动

这是整个流程的起点，Agent 启动后会并行触发两个同步、立即执行的核心初始化任务：

* Bootstrap 加载（配置与记忆加载）
* Session 初始化（会话持久化）

2. #### 同步初始化阶段（启动时立即执行）
3. Bootstrap 加载（同步，立即）：负责为 Agent 注入 初始配置与基础记忆 ，是 Agent 「人格」和「知识」的来源：

   1. 加载文件：
      * `AGENTS.md`：Agent 的指令与能力定义
      * `SOUL.md`：Agent 的人设、风格与行为准则
      * `USER.md`：用户的基础信息与偏好
      * `MEMORY.md`：长期精华记忆（主会话记忆）
      * `memory/今天.md` / `memory/昨天.md`：近期时序记忆
   2. 最终动作：将所有加载内容注入到系统提示词（System Prompt），让 Agent 在对话一开始就具备完整的上下文感知。
4. Session 初始化（同步，立即）：负责 会话的持久化与历史管理 ，保证对话可追溯：

   1. 核心操作：
      * 读取或创建 `.jsonl` 格式的会话日志文件
      * 写入 `session header`（会话元数据，如 ID、启动时间等）
      * 开始追踪完整的对话历史
5. #### 异步后台阶段（初始化完成后启动，非阻塞）
6. Memory 索引（懒加载，异步）
   这是 记忆检索的底层支撑 ，采用「懒加载」设计（只有第一次检索记忆时才真正执行）：

   1. 触发条件：首次调用 `memory_search`（记忆检索接口）
   2. 索引构建流程：
      * 扫描 `MEMORY.md` 和所有 `memory/*.md` 记忆文件
      * 将内容按约 400 token 分块
      * 为每个分块生成向量嵌入（embeddings）
      * 存入 SQLite 数据库（路径：`~/.openclaw/state/memory/<agentId>.sqlite`）
   3. 文件监听机制：
      * 监听 `MEMORY.md` 和 `memory/**/*.md` 的文件变化
      * 变化时标记为「dirty（脏数据）」，并通过 1500ms 防抖避免频繁重建
      * 后续自动同步更新索引
7. Session 监听（后台，异步）
   负责 会话日志与记忆索引的实时同步 ，保证记忆检索的时效性：

   1. 监听对象：会话对应的 `.jsonl` 日志文件
   2. 触发同步的阈值：
      * 文件增量字节数 `deltaBytes > 100KB`
      * 或 增量消息数 `deltaMessages > 50`
   3. 触发动作：执行「索引同步」，将新产生的会话内容更新到记忆索引中，让后续检索能获取到最新对话信息
8. ### 跨 `Session` 长期记忆召回

`MEMORY.md` 是跨 `Session` 共享，当跨 `Session` 需要使用时需要执行 “记忆召回”，实际上就是 `RAG`。 `RAG` 流程：

1. 关键词精确匹配 + 语义向量相似度匹配；
2. 取最相关的 top_K 个 chunk；
3. 注入到当前 Prompt。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NzU3NDY5MmJlNGY5OGVhNmExY2Y4ZTc1OGFmZjEzOGVfUms5eGx3QnFoZWIwTnplSDdqSm1zNlEyUDhRa2JGcjZfVG9rZW46TkxXbGJYOURTbzhpWGt4RDlUaWNpQUpmbmFlXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

7. ## `Subagent`（子代理）

面对复杂、长时间的任务，如果让主 Agent 同步等待就会严重阻塞用户交互，所以引入了 Subagent 机制，把长时间的复杂任务拆出去后台执行。

目前，LLM 会根据实际情况 “自行” 指示 Main Agent 通过 ` sessions_spawn` 工具派生一个或多个异步的 `Subagent`。

```Plain
If a task is more complex or takes longer, spawn a sub-agent...
```

1. 派生 ：主 Agent 为复杂任务派生 `Subagent`，并设置为后台/异步模式。
2. 通知 ：主 Agent 先回复用户 “任务已启动…”，然后结束当前会话。
3. 回调 ：`Subagent` 任务完成后回传结果，由主 `Agent `汇总并推送给用户。

暂时无法在飞书文档外展示此内容

`Subagent` 就像是一个 “临时工”，负责把某个子任务做完。因此需要给 Subagent 明确的边界，它负责执行，不应该具有管理权。所以，出于安全的考虑 `Subagent` 不会继承主 Agent 的全部能力，而是采用 “默认收回敏感与高危工具” 的策略。对特别危险的工具，还会再叠加二次检查，确保无法绕过。

值得注意的是，`Subagent` 的派生的必要性，目前主 Agent 是通过 LLM 建议来派生的，所以也有可能会误判。另外也要注意 `Subagent` 的开销，它会引入更多的会话管理、结果汇总、失败回退与并发控制问题（尤其是多个 Subagent 并行时）。因此实践上通常要配套一些超时、重试策略，让这套机制运行更稳定。

配置 Sub-agents 的并发数量和 Tool-Policy：

```JSON
   {
     agents: {
       defaults: {
         subagents: {
           maxConcurrent: 8,
           maxSpawnDepth: 2,
           tools: {
             allow: ["read", "exec"],
             deny: ["browser", "nodes"]
           }
         }
       }
     }
   }
```

8. ## `Multi-Agent`（多智能体）

首先我们需要知道，在 OpenClaw 的设计中，`Multi-Agent` 和 `Subagent` 有着本质的区别：

| 维度                                                                                                                                                                            | Multi-Agent                                                                                    | Sub-agents                                                        |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |                                                                                                |                                                                   |
| 应用场景                                                                                                                                                                        | 多个完全独立的智能体并排运行，可以互不干扰，或者相互协作。用于：多用户/多角色/多工作流的隔离。 | 主 agent 生成子代理来执行任务。用于：任务拆分、并行处理、后台工作 |
| -                                                                                                                                                                               | -                                                                                              | -                                                                 |
| 会话层级                                                                                                                                                                        | 横向隔离（agent:main:whatsapp:...）                                                            | 纵向嵌套（agent:main:subagent:[uuid]）                            |
| 会话存储                                                                                                                                                                        | `~/.openclaw/agents/[agentId]/sessions/`                                                     | 从父 agent 运行上下文管理                                         |
| 工具策略                                                                                                                                                                        | 静态配置在 `agents.list[].tools`                                                             | 默认受限（不包含 `session tools`）                              |
| Session key 格式                                                                                                                                                                | `agent:[agentId]:[mainKey]`                                                                  | `agent:[agentId]:subagent:[uuid]`                               |
| 创建方式                                                                                                                                                                        | Gateway 启动时自动创建                                                                         | 通过 `sessions_spawn`工具动态创建                               |
| 生命周期                                                                                                                                                                        | 独立的完整生命周期                                                                             | 依赖于父 agent 的运行                                             |
| 队列 lane                                                                                                                                                                       | `main/subagent/cron`等                                                                       | 专用的 `subagent lane`                                          |
| 并发控制                                                                                                                                                                        | 通过 `agents.defaults.maxConcurrent`                                                         | 通过 `agents.defaults.subagents.maxConcurrent`                  |
| 默认并发数                                                                                                                                                                      | main 默认 4                                                                                    | subagent 默认 8                                                   |
| 嵌套支持                                                                                                                                                                        | 不适用                                                                                         | 支持 maxSpawnDepth（默认 1，最大 2）                              |
| 工具访问                                                                                                                                                                        | 可配置任何工具                                                                                 | 默认排除 session tools（depth ≥ 2 时）                           |

可见，当我们需要在 OpenClaw 中执行完全不相干且差异巨大的任务场景时，我们可以创建 Multi-Agent，以此来实现多个 Agent 之间的个性、记忆、上下文方面的完全隔离。例如：我需要一个日常生活 Agent、一个编程 Agent、一个写作 Agent、一个健身 Agent。

1. ### 创建新的 Agent

添加一个新的 Agent 的指令非常简单：

```Bash
# ========== 第一步：查看当前已配置的所有 agent（AI 代理） ==========
# openclaw agents list：核心查询命令，列出所有已配置的智能代理及其详细信息
$ openclaw agents list

# 命令输出解析（带注释）：
Agents:
- main (default)               # 默认的 agent 实例，名称为 main
  Workspace: ~/.openclaw/workspace          # main agent 的工作目录（存储运行时数据）
  Agent dir: ~/.openclaw/agents/main/agent  # main agent 的配置文件存放目录
  Model: zai/glm-4.7                        # main agent 默认使用的 AI 模型为 glm-4.7
  Routing rules: 0                          # 无自定义路由规则（路由规则用于指定不同场景用哪个 agent）
  Routing: default (no explicit rules)      # 路由方式为默认（未配置自定义规则）
# 系统提示：路由规则用于映射渠道/账号/对等端到指定 agent；--bindings 参数可查看完整规则
Routing rules map channel/account/peer to an agent. Use --bindings for full rules.
# 系统提示：渠道状态仅反映本地配置，--probe 参数可查询实时健康状态
Channel status reflects local config/creds. For live health: openclaw channels status --probe.

# ========== 第二步：新增一个名为 claudecode 的 agent ==========
# openclaw agents add [agent名称]：添加新的 agent 实例
# claudecode：自定义的新 agent 名称，可根据业务场景命名（比如按模型/功能命名）
$ openclaw agents add claudecode

# ========== 第三步：再次查询，验证新 agent 是否添加成功 ==========
$ openclaw agents list

# 命令输出解析（带注释）：
Agents:
- main (default)               # 原有默认 agent，配置信息无变化
  Workspace: ~/.openclaw/workspace
  Agent dir: ~/.openclaw/agents/main/agent
  Model: zai/glm-4.7
  Routing rules: 0
  Routing: default (no explicit rules)
- claudecode                   # 新增的 agent 实例，名称为 claudecode
  Workspace: ~/.openclaw/workspace-claudecode  # 自动生成独立工作目录（避免与 main 冲突）
  Agent dir: ~/.openclaw/agents/claudecode/agent  # 独立配置目录
  Model: zai/glm-5                        # 新 agent 默认使用 glm-5 模型（与 main 区分）
  Routing rules: 1                        # 自动生成 1 条基础路由规则（新增 agent 自带）
```

这样我们就拥有了 2 个 Agent，接下来需要使用 `bindings`配置将 `Agents`绑定到不同的 `Channel / accountId` 中，以此来实现对话隔离。如下配置所示：

```JSON
// 2 个 Feishu accounts
   {
     channels: {
       feishu: {
         enabled: true,
         groupPolicy: "open",
         dmPolicy: "pairing",
         allowFrom: ["*"],
         connectionMode: "websocket",
         accounts: {
           default: {
             // Main agent 的飞书机器人
             appId: "XXX",
             appSecret: "YYY",
             domain: "feishu"
           },
           claudecode: {
             // Claudecode agent 的飞书机器人
             appId: "XXX",
             appSecret: "YYY",
             domain: "feishu"
           }
         }
       }
     },
// 每个 agent 用一个 feishu account
     bindings: [
       {
         agentId: "main",
         match: { channel: "feishu", accountId: "default" }
       },
       {
         agentId: "claudecode",
         match: { channel: "feishu", accountId: "claudecode" }
       }
     ]
   }
```

注：前提是我们需要在飞书开放平台里面创建好 2 个机器人。

查看配置效果：

```YAML
$ openclaw agents list --bindings
Agents:
- main (default)
  Workspace: ~/.openclaw/workspace
  Agent dir: ~/.openclaw/agents/main/agent
  Model: zai/glm-5
  Routing rules: 1
  Routing: default (no explicit rules)
  Routing rules:
    - feishu accountId=default
- claudecode
  Workspace: ~/.openclaw/workspace-claudecode
  Agent dir: ~/.openclaw/agents/claudecode/agent
  Model: zai/glm-5
  Routing rules: 1
  Routing rules:
    - feishu accountId=claudecode

$ openclaw channels status
- Feishu claudecode: enabled, configured, running
- Feishu default: enabled, configured, running
```

2. ### 使用指定的 Agent

有 3 种方式我们可以指定 Agent 进行使用：

1. 通过 IM 机器人来选择 Agent，因为 Channel 和 Agent 是一一对应的。
2. 通过 GUI 来选择 Agent。如下图，选择 Agent 后到 Chat 进行对话。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YzJhZjZmYjYwNjNlZGViMjY0ZjNmM2FkYWI5MTI0NjJfMEx1YWhQV2dPVXJqeEVKenk1bDRlWncwVFdIZnVsdkhfVG9rZW46VG5KRWJIMTQzb3hoRVd4TGgxbGNIOUtjbmdlXzE3NzQ0MzUzMjk6MTc3NDQzODkyOV9WNA)

1. 通过 TUI 来选择 Agent。

```JSON
/agents               # 查看 agents 清单
/agent main          # 切换到 main
/agent claudecode    # 切换到 claudecode
```

3. ### Agents 之间的协作

在创建了多个 Agents 之后，我们可能会希望让他们协作起来，发挥出 Multi-Agent 架构的优势。

OpanClaw 提供了 3 种 Agents 之间的协作方式：

1. `sessions_send`  ，  `Agent-to-Agent Messaging` （直接通信）方式：向已经存在的 session 发消息，前提是双方都有 session，且需要启用 `tools.agentToAgent` 特性。当 Agent 之间的通信需要被写入对方的 `Memory` 时，建议使用 ` sessions_send` 工具互相发送消息。

```JSON
   {
     tools: {
       "sessions": {
         "visibility": "all"   // <- 这个特别容易漏，记得一定要配上了，用于将 A 的会话写入到 B 的 session.jsonl 中
       }
       agentToAgent: {
         enabled: true,           // 启用 agent 间通信
         allow: ["main", "claudecode"]  // 允许通信的 agent 对
       }
     }
   }
```

2. `sessions_spawn`  ， `Sub-Agent` ** 方式** ：将 Agent 当做是另一个 Agent 的 Subagent。通过调起 subagent + 传入任务的方式，让其完成交付。任务派发场景中建议使用 sessions_spawn。

```JSON
   {
     "agents": {
       "list": [
         {
           "id": "main",
           "subagents": {
             "allowAgents": ["claudecode"]  // ← 新增：允许 main 调用 claudecode
           }
         },
         {
           "id": "claudecode",
           "name": "claudecode",
           "workspace": "/Users/fanguiju/.openclaw/workspace-claudecode",
           "agentDir": "/Users/fanguiju/.openclaw/agents/claudecode/agent",
           "model": "zai/glm-5"
         }
       ]
     }
   }
```

3. **共享 ** `Workspace`  （ 文件共享 ） ：虽然每个 agent 有独立的 workspace，但也可以共享特定目录。

```JSON
   {
     agents: {
       list: [
         {
           id: "main",
           workspace: "~/.openclaw/workspace",
         },
         {
           id: "claudecode",
           workspace: "~/.openclaw/workspace-claudecode",
           // 共享特定目录
           binds: ["~/Projects:/projects:ro"]  // 只读挂载
         }
       ]
     }
   }
```

值得注意的是，`Agent-to-Agent` 是基于 Session 的，但我们前面说过 `Session.jsonl` 是短期记忆，所以为了避免遗忘，我们还需要在 AGENTS.md 中进行长期记忆。

```Markdown
## Agent-to-Agent Messaging

当你需要其他角色的意见或具体执行时：
- **日常生活问题和任务** → `@main` 处理主人的日常生活
- **编程问题和任务** → `@claudecode` 获取专业的编程技术支持
```

另外，当我们给多个 Agent 即配置了 ` Agent-to-Agent` 也配置了 `Sub-Agent` 协作模式，那 OpenClaw 如何决断使用哪种方式呢？实际上是由 LLM 根据场景来进行决策的，`Agent-to-Agent` 用于协作场景，`Sub-Agent `用于委派场景。

9. ## `Heartbeat & Cron`

`Heartbeat` & `Cron` 是实现 OpenClaw 主动发送通知的基础：

* Heartbeat ：周期性心跳检查，在主会话中，周期性（默认 30 分钟）触发 Agent 读取 `HEARTBEAT.md`，并自主执行其中定义的例行任务（如收信、发报告），无任务时回复 `HEARTBEAT_OK`。
* Cron ：定时性任务执行，精确时间触发（如每天 9:00 准时发报告）。

https://docs.openclaw.ai/zh-CN/automation/cron-vs-heartbeat

| 使用场景                                                    | 类型            | 特点/说明                  |
| ----------------------------------------------------------- | --------------- | -------------------------- |
| ----------------------------------------------------------- |                 |                            |
| 每30分钟检查收件箱                                          | 心跳            | 可批量处理，具备上下文感知 |
| -                                                           | -               | -                          |
| 每天上午9点准时发送报告                                     | 定时任务-隔离式 | 需要精确定时               |
| 监控日历中即将到来的事件                                    | 心跳            | 天然适合周期性感知         |
| 运行每周深度分析                                            | 定时任务-隔离式 | 独立任务，可使用不同模型   |
| 20分钟后提醒我                                              | 定时任务-主会话 | 精确定时的一次性任务       |
| 后台项目健康检查                                            | 心跳            | 搭载在现有周期上           |

1. ### 配置 `Heartbeat`
2. 周期性配置

```YAML
{
  agents: {
    defaults: {
      heartbeat: {
        every: "1h",    // 每 1 小时醒一次
        target: "last", // 有事汇报时，将消息投递到最近活跃的 Channel
        activeHours: { start: "09:00", end: "23:00" } // 只在这个时段开启心跳（省 token）
      }
    }
  }
}
```

2. 写 `HEARTBEAT.md`：每次心跳触发，都会读一遍这个文件。

```Markdown
# HEARTBEAT.md

## 定期检查
- OpenClaw 领域热点追踪，包括：各大云厂商、开源社区方面的咨询。重点关注：OpenClaw 的安全方案、企业应用场景、竞品分析、社区版本新特性更新等方面的消息。
- 进行中且未完成的工作进展

## 主动汇报原则
- 发现热点 → 快速响应，提出观点，立即汇报
- 有数据洞察 → 主动分享
- 进行中工作 → 主动推进
```

2. ### 配置 `Cron`

精确时间触发的 Cron 常用于点发汇报：

```SQL
openclaw cron add \
   --name "AI 行业早报" \
   --cron "0 9 * * *" \
   --tz "Asia/Beijing" \
   --agent main \
   --session isolated \
   --thinking high \
   --announce \
   --channel feishu \
   --account default \
   --to "user:XXX" \
   --message "早报时间到，整理昨天一天的 OpenClaw 行业消息推送、热点追踪。包括：各大云厂商、开源社区方面的咨询。重点关注：OpenClaw 的安全方案、企业应用场景、竞品分析、社区版本新特性更新等方面的消息。"
```

* 明确指定 --to "user:XXX"（你的飞书 ID）
* 明确指定 --account default
* 明确指定 --channel feishu

3. ### 配置消息源

OpenClaw 需要结合具体的 Skills 和消息源（例如 Brave Search，智普联网搜索 MCP 等等）来实现网页搜索以及信息质量的提升。

5. # `Nodes` 终端
6. ## Nodes 本质

Nodes 是 OpenClaw 对物理设备（iOS/Android/macOS） 的统一抽象封装，硬件终端的抽象层。

* 对开发者 / 上层 Agent 来说：不用关心底层是 iOS 还是 Android，只需要调用统一的 API（比如 `camera.snap`），OpenClaw 会帮你处理设备差异。
* 对设备来说：需要安装一个轻量的  Node Client App （可以理解为 “设备端代理”），这个 App 是 OpenClaw 与硬件交互的桥梁 —— 比如你调用 `camera.snap`，其实是 Node Client App 接收到指令后，调用手机的摄像头 API 完成拍照。

目前支持的 Nodes 类型包括：

1. macOS Node ：system.run（执行命令）、system.notify（通知）、Canvas/摄像头。
2. iOS Node ：Canvas、语音唤醒、摄像头拍照/录像、屏幕录制、语音触发。
3. Android Node ：Canvas、语音交互、摄像头、屏幕截图、短信集成（可选）。

本地工具会在 Agent 模块内部直接执行，但如果涉及远程 Nodes，则必须通过 Gateway。系统会采用两道关卡，只有通过这两道检查后，命令才会被下发到远程设备。

* 白名单 ，例如某个 iOS 节点只允许 camera.snap 工具。
* 人工审批 ，例如调用 sms.send 时需要人工确认。

2. ## Nodes 和 Gateway 之间的通信协议

* 传输 ：Gateway WebSocket（LAN/Tailscale/SSH 隧道）
* 发现 ：node.list / node.describe 枚举能力
* 执行 ：node.invoke 运行设备本地操作
* 命令 ：camera.snap/camera.clip（拍照/录像）、screen.record、location.get、notifications

3. ## Nodes 和 Gateway 之间的通信流程

* 能力声明 ：新 Node 首次连接时，需要人工审批才能被信任；连接时必须告诉 Gateway 自己支持哪些命令和能力，也是后续工具安全的校验依据。
* 断连清理 ：当 Node 断开连接时，Gateway 会自动拒绝所有尚未完成的远程调用，防止 Agent 的工具请求长时间挂住。
* 超时机制 ：每次远程调用默认 30 秒超时，超时后自动失败并清理。

暂时无法在飞书文档外展示此内容

如果 Gateway 跑在 Linux 服务器上，但你想用 Mac 的摄像头拍照。

```JSON
// 列出已连接的节点
{ "tool": "nodes", "action": "status" }

// 在 Mac 节点上执行命令
{ "tool": "nodes", "action": "run", "node": "office-mac", "command": ["echo", "Hello"] }

// 拍照
{ "tool": "nodes", "action": "camera_snap", "node": "iphone-1" }

// 屏幕录制
{ "tool": "nodes", "action": "screen_record", "node": "office-mac", "duration": "10s" }

// 获取位置
{ "tool": "nodes", "action": "location_get", "node": "iphone-1" }
```

[OpenClaw 系统架构分析_openclaw架构-CSDN博客](https://blog.csdn.net/evilstar2015/article/details/157617589?ops_request_misc=elastic_search_misc&request_id=0ee0e5d60705817091e6440c0a7318fe&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-157617589-null-null.142^v102^pc_search_result_base1&utm_term=OpenClaw%20%E6%9E%B6%E6%9E%84&spm=1018.2226.3001.4187)

https://zhuanlan.zhihu.com/p/2015645820083533188

https://zhuanlan.zhihu.com/p/2011197075765884377

https://zhuanlan.zhihu.com/p/2004505448938767308
