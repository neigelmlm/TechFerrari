---
title: OpenClaw Overall Architecture Technical Analysis
date: 2026-03-24
author: neigelmlm
description: OpenClaw Overall Architecture Technical Analysis
tags:
    OpenClaw
cover: cover.png
---
 From the dimensions of "capability highlights" and "design philosophy", the six core capability modules and top-level design principles of the OpenClaw system have been refined , answering the question of "what are the core advantages and capabilities of this system". It breaks down the system's core capabilities into six modules, namely:

1. Gateway Control Plane : The central scheduling core of the system, which uses a single WebSocket server to centrally manage sessions and events across the entire system;
2. **Multi-channel Notification ** System : Solve the problem of multi-platform message access, and be compatible with 15+ mainstream Notify Platforms through a unified abstraction layer;
3. Plug-in Extension : The extensible base of the system, which implements independent plug-in package management based on the Monorepo architecture and supports flexible expansion;
4. Cross-platform Client : The entry point for users to interact with the system, covering multiple types of terminals such as CLI, desktop, and mobile.
5. AI ** Agent Engine** : The core of the system's AI capabilities, implementing multi-model support and intelligent processing based on the Pi Agent RPC model;
6. Local-first Design : The security and data foundation of the system, which achieves autonomous control of data and sessions through local storage to safeguard privacy.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGRiYzBiZTMzMzIxODM5OWM5YjFiOTU5NWE3OWQxNGZfS1NyTFl3WmdCOVlWd2Jpd1NVNkwyZThVYTJJYkNWQkZfVG9rZW46RTR6d2JnTlBGb3hndk94RnJpOGNuWWZ3bkVmXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

2. ## Seven-layer architecture layer

 From the dimension of "Technology Implementation, Data Flow", it disassembles the seven-layer technical architecture of OpenClaw , answering the questions of "what is the technical structure of this system" and "what is the complete process of data from initiation to implementation". It vertically disassembles the system into 7 layers according to the complete link of "User Input → System Processing → Data Storage", with each layer having its own responsibilities and upstream and downstream linkages:

1. Client Layer : The entry point for user interaction, corresponding to the "Cross-platform Client" in the first diagram;
2. Gateway Control Plane : The scheduling hub of the entire system, fully corresponding to the "Gateway Control Plane" in the first diagram, it receives client instructions and manages whole-link sessions and events;
3. Channel Access Layer : The unified entry for multi-platform messages, corresponding to the "Multi-channel Notify System" in the first diagram, enabling standardized access to various third-party Notify Platforms;
4. **Routing and Processing Layer ** : The "transportation hub" for message flow, responsible for message routing and distribution, permission filtering, and rule processing;
5. AI ** Agent Layer** : The core of intelligent processing, fully corresponding to the "AI Agent Engine" in the first diagram, responsible for large model invocation, tool invocation, and session context management;
6. Tools and Services Layer : Capability Support Layer, providing basic tool capabilities such as browser, media processing, speech conversion, and scheduled tasks for AI processing and upper-layer business;
7. Data Storage Layer : The foundation for data implementation, corresponding to the "Local-First Design" in the first diagram, responsible for the local persistent storage of session, configuration, cache, and vector data.

The first one is the "horizontal capability slice", which presents the six most core highlight capabilities of the system side by side, enabling users to quickly understand the core advantages of the system at a glance; the second one is the "vertical process link", which breaks down the system into seven layers according to the data flow sequence, clearly explaining how each module is connected and collaborates to complete a full business process.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MTYwODUzODc3MTQ4OTY5ZmZkZTg5NTJiZGIzYzRiMWFfV0JwMm1MbVM2a1k4MHpWUFVSNFp6MjBTSE9qS0ZIV1dfVG9rZW46WTFJcGJHWVhobzdJRXZ4MkV6b2NtbkRNblNkXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

3. ## Technology Stack

| Technical Layering                                                                                                                 | Core Technology Components                                                      | corresponds to the functional architecture layer                  |
| ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| ---------------------------------------------------------------------------------------------------------------------------------- |                                                                                 |                                                                   |
| Application Layer                                                                                                                  | CLI (Commander)、Control UI (LitElement)、iOS/macOS (SwiftUI)、Android (Kotlin) | Client Layer                                                      |
| -                                                                                                                                  | -                                                                               | -                                                                 |
| Communication Layer                                                                                                                | `WebSocket RPC (Request/Response/Event Frames)`                               | Gateway Control Plane Communication Base                          |
| Core Service Layer                                                                                                                 | `Gateway Server、ACP Layer、Agent Runtime (pi-mono)`                          | Gateway Control Plane + AI Agent Layer                            |
| Channel Adaptation Layer                                                                                                           | Plugin SDK, Core Channel, Extended Channel                                      | Channel Access Layer + Plug-in Extension                          |
| Data Persistence Layer                                                                                                             | JSON Files, SQLite (Optional), LanceDB (Vector Memory)                          | Data Storage Layer                                                |
| External Service Layer                                                                                                             | OpenAI API、Claude API、Tailscale、Playwright                                   | AI Agent Layer + Tools and Services Layer (External Dependencies) |
| Operating Environment                                                                                                              | Node.js 22+、TypeScript、pnpm、Vitest、tsdown、Oxlint                           | Underlying operating environment of the entire system             |

4. ## Complete Business Closed-loop Flow Diagram

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MjhiMzc1ZGViNDE5YTViYjk4MzlkOTIzZmQyYzI4NzBfVnhsZTlqTnZsNFJ4UTE3aUtRWGUxd3pmcjdLUndBc25fVG9rZW46RHJuM2JRMEtzb1ZZNWR4dlJGOGNzaEk5bjVXXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

This diagram is  ** the "data flow and whole-link business process dimension" during runtime ** , which is completely different from the previous static architecture, capability, and technology stack breakdowns. It focuses on  ** the dynamic execution process ** , and the core question it answers is:  ** the complete lifecycle, data flow, and inter-module call and collaboration relationships of a user message / a scheduled task from triggering to final completion of the response within OpenClaw ** .

It connects the previously static architecture modules into a complete and executable process. The core transfer link is divided into two major trigger entry points and five core steps, forming a complete closed loop:

1. Two major trigger starting points (the beginning of the process)

   1. Passive Trigger: Original messages sent by users from external Notify Platforms such as WhatsApp and Telegram;
   2. Active Trigger: System-built scheduled tasks (Cron Jobs), heartbeat loops, and actively initiated scheduling tasks.
2. Core Hub: Gateway Control PlaneAll triggered events/messages are aggregated to the Gateway/Router Core Gateway:

   1. External messages: First, perform format standardization through Channel Adapters, and then uniformly connect to the Access GateWay;
   2. The gateway is responsible for message enqueuing, priority management, and task assignment, and distributes processing tasks to the Agent runtime environment;
   3. It is also the convergence point of the final response, responsible for returning the results processed by AI back to the external channels via the original path.
3. Intelligent Execution Core: Agent Runtime EnvironmentUndertakes tasks assigned by the gateway and completes core intelligent processing:

   1. Read and write session context, configuration, and historical conversations from the storage layer, and obtain the background information required for inference;
   2. Call large language models (LLMs) (such as GPT/Claude) to complete thinking reasoning and instruction understanding;
   3. Invoke toolkits/skill sets on demand to perform extended actions such as browser operations and media processing.
4. Data Base: Storage and Memory Layer provides whole-link data support for Agent operation, including Agent configuration, persona instructions, conversation history, and vector semantic retrieval capabilities, corresponding to the local-first design of the previous architecture.
5. Response Closed LoopThe final result processed by the Agent is postbacked to the Gateway, then through the channel adapter, the standardized response is converted into the format of the corresponding platform and returned to the user, thus completing a full interaction process.
6. ## Agent Intelligent Agent Core Execution Link

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=Mjk3ZDNlOThiMjYzZWVmZmZiYjFhMjlhN2U2NWJjYWFfbXE4WXdTZkNtUHlHOG1sRVF4eTAwN0hHdWt6TDEzcTRfVG9rZW46VllCbGJZYks5b2FES3F4aHVZUmNveHRYbnRjXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. **Channels: Connect to various Notify Platforms, providing a unified messaging interface. Support: Feishu, DingTalk, WhatsApp, Telegram, Discord, Slack, ** SMS , etc., enabling seamless integration between AI and users' daily communication.

   1. Why does open claw need a message entry layer instead of creating one on its own?
      Telegram, Discord, etc. are already mature chat platforms, with built-in:

      1. Account System, User Management, Message Push, Rich Media (Images / Files / Emoticons)
      2. Cross-platform (Mobile / PC / Web), Secure Encryption, Compliance Review
         **If OpenClaw develops its own ** IM **: **
      3. Developing the Client, Server, and Push Link from scratch requires extremely high costs
      4. Unable to quickly reach existing users, making promotion difficult
         By using a "Channel Adapter" for a thin layer of adaptation, Agent can directly run in the IM that users are already using,  ** Out Of The Box ** .
2. Gateway (大门) : A Node.js daemon based on WebSocket, running on localhost:18789, which manages sessions, routes requests, authenticates identities, manages message channels, tools, and events. All messages are routed, authenticated, and session-managed through this.
3. Is it because Openclaw has connected to many message entry points or because AIAgent requires Gateway Server to process messages from different users?
   AI** Agent inherently requires a Gateway to manage users and sessions**

   Regardless of whether OpenClaw connects to 1 or 100 message entrances, as long as it is a system with ** multiple users, multiple sessions, and multiple Agents** , it must have a Gateway to perform three tasks:

   1. User Identification + Conversation Isolation : Each user's chat is an independent conversation, and there should be no cross-talk. For example, if you chat with it about code and your friend chats with it about stock trading, Gateway must remember "who is who and where the conversation has reached" and accurately deliver messages to the corresponding conversations.
   2. **Queuing + Rate Limiting ** : Agent/LLM has limited computing power and cannot process hundreds of messages simultaneously. Gateway uses queues (Lanes) to queue requests and protect the backend from being overwhelmed.
   3. Unified Format + Decoupling : Convert user messages into a unified format that Agents can understand. Agents only focus on "thinking + tool invocation" without worrying about where the messages come from.
4. Agent (Brain) : Connects to LLMs (Claude, GPT, Ollama, etc.), has a dedicated persona, and is responsible for understanding the context intention, formulating step-by-step plans, and deciding which tools or skills to call.
5. Skills (Hands and Feet) : A series of Skills&Tools defined using SKILL.md, which are the hands and feet for Agents to interact with the real world, supporting file operations, browser control, API calls, etc.
6. Nodes (Sensors/Terminals) : Small agents running on user-side devices (mobile phones, laptops, Raspberry Pi, desktops), which can provide local capabilities such as camera, geolocation, or system notifications.
7. ## Summary

First, look at the feature map : understand what OpenClaw "wants to achieve".

Looking at the ** hierarchical diagram ** : know what it "looks like" and how the modules are divided;

Looking at the technical specification : understand "what technologies are used and how it is built".

Looking at the data flow diagram : understand how data flows when it "runs".

Finally, take a look at ** the Agent Link Diagram ** : understand how its "most core AI brain works internally"

2. # `Gateway` Central Control Plane

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ODliMDdkYWNhYzFlNmJmZWMwNWRiMGU2MTVlOGUzZDBfQUtLcU9EcVM3VzQ1YUFDc0hEcWNkYzNoMGFGWDRscGpfVG9rZW46QVpBY2I2YmNRb25KdFN4QVIxTWNNVm1HbkRlXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

How does the Gateway work?

OpenClaw Gateway is a long-running daemon process (a professional term in programming for a "long-running background program") that, once started, does not terminate after execution but remains running on the computer/server until manually stopped or a failure occurs. It is built on Node.js (Node.js is not a programming language but**an environment for running JavaScript ** code ) and uses WebSocket communication technology (a communication method that allows "two-way real-time chat"). It mainly does the following things:

1. IM ** Platform Adaptation** : The message formats of each IM platform vary, and Gateway will unify external messages into the standard internal Message events model.
2. Authentication and Authorization:
   1. **Identity Authentication ** : Verify the legal identity of the requester (e.g., based on JWT, OAuth2.0, or platform token verification);
   2. Permission Verification (  ACL  **) ** : Based on role (RBAC) or resource (ABAC), determine whether the user has permission to access the target Agent;
   3. Route Distribution : Forward the requests that have passed the verification to the corresponding Agent instances through the routing engine.
3. Conversation Routing: The routing logic determines the target conversation based on three key dimensions:
   1. User Identifier (e.g., `user_id`) : Uniquely identifies the sender of the message;
   2. Workspace Identifier (e.g., `workspace_id`) : Isolate different business scenarios or organizational units;
   3. Conversation context (e.g., `session_id` + historical message sequence): Maintains the continuity of the conversation.
4. **Lane Queue **: As a concurrency control layer, it assigns an independent "queue lane" to each session, ensuring:
   1. Messages in the same conversation are processed in order (to avoid out-of-order processing);
   2. The processing of different conversations is isolated from each other (to avoid race conditions, such as simultaneously modifying the state of the same conversation);
   3. The system can still operate stably under high concurrency scenarios (such as group chat spamming, batch @).
5. Session Management: Lifecycle and State Maintenance of Sessions
   1. **Lifecycle ** : Creation of Session (start of a new conversation), maintenance (updating status during message processing), and recycling (releasing resources after the conversation ends);
   2. Context Cache : Use Redis or in-memory cache to store session context (such as historical messages, current intent) to avoid frequent database queries;
   3. State Policy : Defines rules for truncation (e.g., discarding early messages when the number of context tokens exceeds the threshold), archiving (storing long-inactive sessions in the database), and reconstruction (restoring the context when the user initiates a conversation again).
6. Agent Equipment and Invocation : Before the Agent executes a task, complete the assembly of the runtime context (dependency injection):
7. Inject Historical State : Load the cached Session history into the Agent's context;
8. Inject available skills : Find the currently callable Skill modules of the Agent according to business rules;
9. **Injection Tool Set ** : Executes Tool Policy (such as allowlist, permission verification) to determine the tools currently available for the Agent (such as API calls, local scripts).
10. Remote Agent Tool Execution : When an Agent needs to invoke tools on a remote node (such as taking photos on a mobile device or executing scripts on an edge device):
11. Callback Approval : Agent first initiates a tool call request to Gateway, and Gateway performs security checks (such as permissions, audit logs);
12. Command Issuance : After approval, the Gateway issues commands to the corresponding remote nodes via RPC (e.g., gRPC) or WebSocket;
13. Result Postback : After the remote node completes execution, it returns the result to the Gateway, which then forwards it to the Agent.
14. Tool Safety Approval is the second core safety gate of the system, which conducts secondary safety verification and approval interception for Agent's tool calls and high-risk operation executions, and also undertakes the verification of remote tool call requests from the execution layer Agent, strictly preventing unauthorized and high-risk operations, and achieving closed-loop control of execution behaviors.
15. Node Registry is the "address book" for all system nodes, responsible for managing the registration, status maintenance, and addressing scheduling of all connected device nodes and execution nodes, recording the basic information, online status, and available capabilities of nodes, and also handling the access registration of device nodes.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MWFkZWI2ZTBhNjA0OTAyNDgzNzQ2ZTBmMzQyNDY2MGFfR0phTkRXMkpnSkYwbVJxRDJyeTFwYVF2NzhZNFkybDdfVG9rZW46TmdKNmI3MW83b05kb214bzl3SWNRTWNrbldlXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. ## `Channel Adapters`

IM platforms such as WhatsApp, Telegram, WeChat, DingTalk, Discord, etc., are multiple incompatible communication systems:

1. **Different Protocols ** : HTTP, WebSocket, Long Connection, Private Protocol.
2. **Different message models ** : semantics of text, rich media, reference/reply, thread, channel, group, @.
3. Authentication Differences : Token, QR Code Login, Callback Signature, Permission Hierarchy.
4. **Different rate-limiting strategies ** : Some platforms limit by user, some by robot, and some by interface.

**OpenClaw Gateway, on the other hand, addresses the issue of "uniformity" **and provides a unified access method. It brings several engineering benefits:

* Conversations are inherently continuous : Consistent context can be maintained across platforms.
* **The state no longer splits ** : There is no need to store a copy for each platform.
* Behavior is predictable : All decision-making logic is centralized in one place.
* **Auditable and replayable ** : In case of problems, faults can be located.

**The Gateway will become a critical path in the system. If not well-designed, it is prone to becoming a "bottleneck/single point", requiring supporting observability and disaster recovery capabilities. **

In terms of specific implementation,  each Channel in OpenClaw is implemented as an independent plug-in extension package and must implement a unified set of ChannelPlugin interfaces . It should be noted that it does not implement the interface by inheriting a BaseChannel base class, but rather by combining capabilities through a set of Adapters interfaces. This allows for "only a small number of adapters must be implemented, and the rest are optional capabilities."

Therefore, the feature sets of different channels vary greatly. For example, Telegram supports polls and threads, while WhatsApp does not; Discord has the concepts of servers and roles, whereas iMessage does not.

How to adapt the core message processing logic to these differences?

OpenClaw's approach is to have Channel inform itself of the supported capabilities through capabilities declarations. Then, the core code determines subsequent actions by checking attributes such as capabilities.threads/polls.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YmQ2Zjk3OThhMTBlMDc4MWMyNDRmODJlMGE1NmE5ZTZfZGZEOUlWVU1VZVV6Sm5YU0dKZnNFRHM4OVBoemtNdE5fVG9rZW46QzdiQ2JLdDJQb2t2dDZ4bzNReGNNbWdwbnpiXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

2. ## `WebSocket` Communication Protocol

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YTAyNjliZTY4NGRiOGFmZGUxYzg0NzlmMjRiZTgwNDBfZktwdHFPNWFHekg1QkZIWWEzQlhweU1YSUgwS1FRZHhfVG9rZW46V0lQWGJvYWZvb2FNbHZ4Sk1ZQmNBUnVqbmpkXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. ### Gateway Core Communication Foundation
2. Communication Protocol : Use WebSocket (instead of HTTP) - WebSocket is a full-duplex real-time communication protocol, where both the client and server can actively send messages bidirectionally, making it suitable for components such as CLI/TUI/GUI that require real-time interaction.
3. Communication Object : External components such as Gateway ↔ CLI (Command Line Interface), TUI (Terminal User Interface), GUI (Graphical User Interface), Node, Channel, etc.
4. Connection Address : Fixed as `ws://127.0.0.1:18789` (127.0.0.1 is the local loopback address, 18789 is the port that Gateway listens on, `ws://` is the identifier for the WebSocket protocol).
5. Data Carrier : All messages are transmitted via WebSocket message frames, and the content must be in JSON** format** (lightweight, easy to parse, and a common format for communication between front-end and back-end/components).
6. ### Three Core Message Formats

During the communication process, there are only three types of message frames, corresponding to the two interaction modes of "Request - Response" and "Active Push" respectively

1. Request Format (RequestFrame) : Client → Gateway (Active Request), `{type: "req", id, method, params}`
2. Response Format (ResponseFrame) : Gateway → Client (Reply to Request), `type: "res", id, ok, payloaderror`
3. Event Format (EventFrame) : Gateway → Client (Active Push), `{type: "event", event, payload, seq?, stateVersion?}`
4. Supported Event Types :
5. `Agent`: Agent-related events (such as agent online/offline, status change, configuration update)
6. `chat`: Chat-related events (such as new messages, message retractions, chat end / creation)
7. `presence`: Online status events (such as a node/user going online/offline, busy/idle)
8. `health`: Health status events (health check results of Gateway or components, resource utilization alerts)
9. `heartbeat`: Heartbeat event (maintains the connection, confirms both parties are online, and prevents disconnection)
10. `cron`: Scheduled task events (triggered by scheduled task execution, task completion/failure)
11. `tick`: Clock tick event (periodic time synchronization, such as pushing time once per second)
12. `shutdown`: Shutdown event (notification that the Gateway or component is about to shut down, allowing the Client to perform cleanup)

When establishing a connection, the Client performs a handshake, and the first frame must be req:connect. The Gateway will res(ok) to inform the Client of the currently supported RPC methods and event list, and the Client will perform operations accordingly.`type` distinguishes the frame type,`id` associates req/res frames,`event` identifies the type of the pushed event,`ok/payload/error` indicates the response result.

暂时无法在飞书文档外展示此内容

3. ## WebSocket Network Mode

| Mode                                                                                    | Bind Address                                  | Auth Required             | Use Case                          |
| --------------------------------------------------------------------------------------- | --------------------------------------------- | ------------------------- | --------------------------------- |
| --------------------------------------------------------------------------------------- |                                               |                           |                                   |
| loopback                                                                                | 127.0.0.1                                     | Optional (recommended)    | Local-only access, SSH tunnels    |
| -                                                                                       | -                                             | -                         | -                                 |
| lan                                                                                     | 0.0.0.0 (all interfaces)                      | Yes (token or password)`` | Direct LAN/WAN access without VPN |
| tailnet                                                                                 | Tailscale IP (100.64.0.0/10)                  | Yes (token or password)   | Tailscale mesh network            |
| auto                                                                                    | Falls back through loopback → tailnet → lan | If non-loopback           | Automatic best-effort             |
| custom                                                                                  | gateway.host value                            | Yes (token or password)   | Specific IP binding               |

* Loopback Mode : Auth is optional but recommended. By default, Onboarding will generate a token.
* Non-loopback mode : lan/tailnet/custom enforces the use of Auth. If gateway.auth.token or gateway.auth.password is not set, the Gateway will refuse to start.

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

4. ## Session Routing

After the Gateway receives a Message from the Channel, the Gateway's Session Router will be responsible for routing the Message to a Session of an Agent based on the Message Session Key. This ensures the accurate matching of the Message and the Session.

1. ### Session Key
2. View Session Key: By default, each Agent has at least one main Session. Additionally, Sessions are also used for communication between Agents in Subagents and Multi-Agents.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=OTZmYjZkOGRlY2JiOWNlMjlmYjllZDVhZDNhNzNmYWJfcXUyRVpLZXdwRUtsTkdzRjNnQW1nRjlRS0NjMlRJRmpfVG9rZW46SUR1WGJud0lRb0Rpck94UGFQOGN0SnZ2bmI5XzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. The same Agent can also create a new Session:

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MDE2M2RlMjIzZWFjNWE4Y2U4MTljYTMzMTZlZjk1YzFfMHlsaE9hR0l0clJvMGJaallERGdBejNJeUl1Q0ZvcmpfVG9rZW46UEFXNmJGNmpnb083bXp4Z21KQ2NzbWlpbm5kXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

2. ### Agent Binding of Messages

In real-world scenarios, the binding relationship between messages and Agents can be extremely complex, for example:

* Feishu Group A uses the "Research Assistant" Agent
* Feishu Group B uses the "Operation and Maintenance Robot" Agent
* The same user uses the "default assistant" in a private chat
* Members of a certain role group on Discord have exclusive Agents

To handle these complex scenarios, OpenClaw has designed a set of hierarchical priority message bindings mechanisms . Through the configuration of bindings, it is determined which Agent will process each message, with priorities (from high to low):

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=OGM4ZDUxY2M4NjRkOWM4NjVhYWNjYWQwZTAwZTIxNWNfQUZVSWRqSEVqcUt2Nk95RTQwbkF2bVNrUFhibFVlaWpfVG9rZW46V1dqemJyQmhyb0pPVWt4akYyM2NBRktNbnJmXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. peer : Precisely bind the Agent to a specific group or user.

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

2. peer.parent : Thread messages inherit the binding relationship of the parent message. For example: all replies under a Discord topic remain bound to the same Agent; conversation continuity in a Slack thread.

```JSON
主消息（群组A） → 绑定到 "研究助手"
     └─ 回复消息 1 → 自动继承 "研究助手"
         └─ 回复消息 2 → 继续继承 "研究助手"
```

3. team : Bind the Agent to a specific team or organization.

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

4. Account : The same platform may have multiple Bots, which are bound to different Agents.

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

5. channel : The last fallback rule, bound to the default Agent.

```JSON
   {
     "type": "route",
     "agentId": "default-assistant",
     "match": {
       "channel": "feishu"  // 所有飞书消息
     }
   }
```

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTFhNmUzM2Y0MzM5NjZlM2Y1ZGY5NTdjMzY2ZmY5ZGZfbmQwbXNtb202bDBuQnplU1FGOGd3aTdUT3dJQUt2VjJfVG9rZW46VmxtZ2JRdTllb3lPTUN4MmwyRWMwOENUbndiXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

3. ### Session Isolation Strategy

An Agent may serve multiple users simultaneously, so the isolation granularity of Sessions directly affects the user experience. For example, User A: Book me a flight ticket for tomorrow morning; User B: What's the weather like today? Agent: I just booked a flight ticket for User A... (Chaos!); User C: Where's my order? (The Agent doesn't know whose order it is).

Therefore, we need to consider a question: when the same Agent serves multiple users simultaneously, should we share a single context or provide each user with an independent context? In response, OpenClaw supports multiple Session isolation strategies.

* main (default, single-user mode) : All messages share a single session, which is the default choice prioritizing single-user, personal use.

```JSON
  {
     "agents": {
       "defaults": {
         "sessionIsolation": "main"
       }
     }
   }
```

* per-channel-peer (recommended, multi-user mode): Each chat object (group or user) has an independent conversation. In multi-user scenarios, it is recommended to use this strategy.

```JSON
   {
     "agents": {
       "defaults": {
         "sessionIsolation": "per-channel-peer"
       }
     }
   }
```

* **per-channel (platform-level isolation) ** : One session per IM platform.
* per-sender (user-level isolation) : One session per sender.

Based on the isolation strategy of Session, OpenClaw will determine in which Session the Message should be processed.

Additionally, OpenClaw also supports cross-platform identity linking. For example, the identities of the same user on Telegram and WhatsApp can be bound to the same UserID, thereby merging Messages from both ends into the same Session to achieve cross-platform contextually continuous conversations.

5. ## Agent Equipment Loading

In OpenClaw, an Agent is essentially a ReAct loop, rather than a long-term background daemon that operates independently of tasks.

It doesn't know which conversation it is in, which tools to use, or which capabilities it can access. All this "equipment" is assembled by the Gateway before assigning tasks to the Agent , including: loading information related to Session Store, Skill, and Tool-Policy.

* Session Store : `~/.openclaw/agents/main/sessions/`, which records the session information of the Agent.
* Skills : `~/.openclaw/workspace/skills`, which records the skills installed by the Agent.
* Tool Policy : Records the tool whitelist and blacklist of the Agent.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDk1MjE1MjFlNWUyZWVmMjZmYTFjYzc3ZGU1NTgwZDFfcXFmNlowOWdMS0tSc1JrSlc2QnhIQmhpREpaZnJibWJfVG9rZW46UUhpSmJFc3dtb3UwUjd4NDVabmNWOWxEbkpjXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. ### Complete whole-link message processing
2. Users send conversation messages through various message channels and enter the system;
3. The Router of the gateway receives messages, matches the rules, and then distributes them to the specified target Agent;
4. The gateway synchronization accomplishes three tasks: loading session state via Session Store, injecting available skills via Skills Registry, and loading security verification rules via Tool Policy, all of which are passed to the downstream Agent;
5. The Agent layer constructs a System Prompt based on the information injected by the gateway to initialize the behavioral rules of the large model;
6. Initiate the ReAct core loop: Feed the user message + system prompt + historical memory to the LLM for reasoning and thinking;
7. If the LLM determines that no tool invocation is required, it directly generates the final answer, the process ends, and the result is returned to the user;
8. If the LLM determines that a tool needs to be invoked, it generates a tool invocation instruction and passes it to the Tool Runner for execution;
9. After Tool Runner completes tool execution, it returns the results to Agent Loop and feeds them back to LLM for the next step of reasoning;
10. The cycle of "thinking - acting - result feedback" continues to execute until the task is completed, and the LLM generates the final answer and returns it to the user;
11. Throughout the process, Memory continuously updates conversation and task memory, Session Store synchronously updates session status, and Tool Policy verifies the security of tool invocation throughout.
12. ### `Session` Loading

During the Session Store (Session Loading) phase, Gateway mainly loads session.jsonl. Gateway looks up sessions.json via SessionKey to obtain the current SessionId, and then finds the corresponding session.jsonl based on the SessionId and loads it into the Agent.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTkzOTdkMjI4Mzk3NDg1ZWRjNjFlMDRjMmFkNDk3M2JfNE54NnpsNmRhRmZJVzNock1iNFduaWNtVEFZajd3TnlfVG9rZW46VzVUZWJOUm5Tb1NQSVJ4NGFDWmNmM0Z0bjlnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

The conversation history of the session is stored in the session.jsonl file, serving as the agent's short-term memory, and will be periodically reset for new sessions. The first line of session.jsonl is the Session Header, with the following structure, which records the SessionId.

```JSON
{
    type: "session",
    id: "session-uuid",           // sessionId
    cwd: "/path/to/workspace",    // 工作目录
    timestamp: 1234567890,
    parentSession?: "parent-id"   // 可选，fork 时存在
}
```

The content following mainly includes conversation records, structured as follows, which records the time and content of the conversation.

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

3. ### `Skills` Loading

OpenClaw Skills has 3 installation levels, with loading priorities (from high to low) as follows:

1. Workspace skills : `~/.openclaw/workspace-xxx/skills`, Agent level, only effective within the Agent.
2. User skills : `~/.openclaw/skills`, user level, shared by all Agents.
3. Bundled skills : Built-in skills distributed with the npm installation package of OpenClaw.

There are two reasons for stratification: 1) To avoid having too many Skills, which would reduce matching accuracy; 2) To avoid having too many Skills occupying the context window.** Because in actual use, we need to distinguish between "basic general Skills" and "Agent-specific Skills".**

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDVjYjQ1ZmJmNmVjYzhkODU2NzY2Njc4MWZmYWMwODFfYjUzemE3aTFoUEdRZGk5cXJIZWFoSnNvREZEcnRnTE9fVG9rZW46R0R5bWJhejdPb3JhdmR4MldXZGNjSnpIblpnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

When the Agent starts, it scans the above Skills and builds a `Skills Snapshot` (snapshot) , which includes:

1. Filtered `Skills List`
2. Formatted text fragments, including `Skill name` and `Description`, are used to inject into `System Prompt`.
3. Regarding relevant environment variables, Skill can declare its dependencies on command paths and so on.

It is worth noting that Skills Snapshot will only be built in the first round of a New Session, and subsequent rounds of conversation within the same Session will reuse it without scanning again. Therefore, if the Skills List is updated, it is recommended to use it after a New Session.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ODMzNGU0Yjg5ZDEyZTZiY2I0MjJkNTJiYjU1OTk0OTdfaFNoZktZRzhQNGdjZFVwNkw3QzRCMG0xNUhJSUdqYTFfVG9rZW46QlFNTWJtQjE4b1J4Z1F4Rko4MmNJZlZkbmpjXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

OpenClaw comes with a large number of Skills (49+), including:

* Apple 生态 ：Notes、Reminders、Things 3、Bear Notes 等。
* Google Workspace ：Gmail、Calendar、Drive、Docs、Sheets（通过 gog CLI）等。
* Communication Tools : Slack, iMessage, Twitter/X, Discord, etc.
* Smart Home : Philips Hue, Sonos, Eight Sleep, etc.
* Development Tools : GitHub CLI, Claude Code subprocess, Whisper transcription, etc.

It is worth noting that since the installation of Skills is extremely simple, it can very easily lead to the inflation and low-quality spread of Skills. Therefore, it is recommended to regularly use high-level models to scan Skills and clean up low-quality and unnecessary ones.

4. ### `Tool-policy` Loading

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MzIxNzg5MTRlYWY2OWFjM2UzNzE1ZTllOWUxYjVjOWVfZ1h1NzFCQld2WWJQckF2Smt3dUQwdWpMb3ZudVNhaUpfVG9rZW46UkRhTGJXWk80b08wZjF4aDZXd2NYRWNWbldlXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

As shown in the figure above, `Tool-policy` is the core security component of the Gateway in the Open Claw architecture, providing enterprise-level AI Agents with identity-based access control list (ACL) capabilities for tool invocation. By replacing the soft constraints of Agent-level prompts with pre-engineered constraints at the gateway layer, it enables zero-trust, dynamic, and fine-grained management and control of Agent tool permissions.

1. #### Core Design Features of Tool-policy
2. **Gateway layer pre-control, not Agent runtime control **

   1. Control timing: Complete permission filtering before the Agent equips tools
   2. Implementation Effect:Unauthorized tools will not be loaded into the Agent execution environment, preventing unauthorized access from the source, and completely avoiding privilege escalation caused by prompt injection and uncontrollable Agent behavior.
3. Identity-based dynamic allowlist generation mechanism
   After the message flows into the Gateway, the gateway relies on ** multi-dimensional identity identifiers such as ChannelId, UserId, and GroupId ** to ** calculate and generate in real-time a dedicated tool allowlist for this session **, with permissions strongly bound to the session scenario:

   1. Family Group: Only open `read`/`sessions_list`/`sessions_send` basic messaging tools
   2. Working Group: Open `exec`/`browser` and other high-level capability tools
   3. Open Discord Server: Retain only read-only tools to implement minimum privilege control
4. #### Zero Trust Security Architecture

Tool-policy adheres to the zero-trust security architecture , and reconstructs the Agent permission management logic:

1. Agent is not trusted by defaultBased on the premise that "Agent itself does not have the ability to self-manage permissions", it controls risks throughthe principle of least privilegeand the core measure is not to grant tool invocation permissions unless necessary.
2. **Centralized control of permissions ** The permission to invoke Agent tools is not decided by the Agent itself, but is uniformly managed and distributed by the  ** Gateway Authorization Center ** , achieving centralized and standardized permission control.
3. Scenario-based real-time authorizationPermissions are not permanently bound to users/Agents. When users switch between scenarios such as groups and tasks, the Gateway immediately recalculates permissions, without cross-scenario permission reuse, thus preventing permission spread.
4. #### Tool-policy Eight-layer Permission Filtering Pipeline
5. Owner Identity Threshold : An identity level threshold that isolates certain high-risk operations before they enter the pipeline. For example, certain tools can only be invoked by the Owner of the channel.
6. Profile Role Template : A role template that defines the basic capability boundaries of a certain type of Agent. For example, a Coding Agent may not be allowed to use the web_search tool.
7. Provider Profile Model Adaptation Strategy : The same Agent adjusts its capability boundaries according to different LLMs. For example, when switching to Gemini, a certain tool needs to be removed.
8. Global Organization Policy : Unified rules at the organizational level, to which all Agents are bound. For example, all Agents within the organization are prohibited from operating remote devices (Nodes).

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

5. Agent Individual Strategy : Customization of tool permissions refined to individual Agents. For example, a certain analysis Agent can only read files and search, but cannot call the exec tool.

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

6. Group / Channel Scenario Strategy : The same Agent has different tool capabilities in different channels/groups. Family groups, work groups, and public groups have different permissions. For example, requests in the company announcement group are not allowed to use the exec tool; however, the R&D group can.

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

7. **Sandbox Hard Isolation Strategy ** : The hard boundary of a secure isolation environment that cannot be bypassed by upper layers. For example, the sandbox environment only allows read tools but does not allow the use of write tools.

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

8. Subagent Strategy : Systematic protection against "recursive explosion" at the framework level. For example, subagents are not allowed to use the spawn tool to recursively generate subagents.

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

OpenClaw Predefined Toolset:

| Group                                              | Contains tools                                                                     |
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

6. ## Session Concurrency Control

The session control layer of the Gateway, used to control the session concurrency strategy between the Gateway and the Agent.

* Queue mode: Prevents message explosion (in high-frequency scenarios) and optimizes user experience.
* Session lane: Prevents session conflicts (file concurrency, state competition) and ensures data consistency.

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

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=Yjc0OWQ2ZjM2ZmM0YmQ0MTY3ZmU0NGQ3ZGNjYTk4MzBfZ01GWlRQeWlpZjFIWXJZRm5MUVNtUHBMbEJZOERYbldfVG9rZW46RDhSd2JmcTZjb1pXQzF4bHZDTGMwNlB6bjFnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. ### `Queue Mode` (Queue Mode)

When we use IM and OpenClaw for conversations, the situation of "high-frequency message input" often occurs, such as inputting "Hello", "I'd like to ask", "How to install a new Skill for OpenClaw", "Can you help me install it" all at once...

In response to this,  when multiple messages arrive rapidly, Gateway provides a control strategy for high-frequency message input.

* **Followup Mode: **Process messages one by one in FIFO order, where each message is treated as an Agent task to receive a complete response before starting to process the next message.

```JSON
   消息1 ──> 任务1 ──┐
   消息2 ──> 任务2 ──┼─> session:abc (lane) ──> 顺序执行3次
   消息3 ──> 任务3 ──┘
```

* Collect (搜集) Mode : Wait within a time window (e.g., 2s), aggregate the messages arriving within the window into one, and then assign it as a task to the Agent.

```JSON
   消息1 ─┐
   消息2   ├──> [合并] ──> 1个任务 ──> session:abc (lane) ──> 执行1次
   消息3 ─┘
```

* Steer Mode : New messages within a time window will be routed to the currently running previous round of Agent Loop; if it fails, wait for the next round of Loop.For example, during the gap after a certain LLM API call in the ReAct Loop ends, append the new message to the message stream to participate in the next round of LLM inference.

```Plain
   消息1 ──> [正在运行] (直接注入，不进lane)
   消息2 ──> 如果当前空闲 → 立即执行
            如果忙碌     → 退化为 followup → 进 lane
```

* Interrupt Mode : After a new message arrives within the time window, it directly interrupts the current operation, clears the queue, discards the response, and restarts from the new message.

It is worth noting that if OpenClaw is executing a very long task (such as searching ten web pages) and you want it to change direction midway, you can only wait for it to finish. After switching to steer mode, the new messages you send will be injected in real-time into the task the AI is currently processing, and it will immediately adjust its direction without having to wait.

```JSON
{
  "messages": {
    "queue": {
      "mode": "steer"
    }
  }
}
```

2. ### Lane Concurrency Control

Lane is the concurrency control mechanism of Gateway. Different types of tasks run in independent Lanes, and each Lane has its own concurrency limit:

* main lane : Global default lane, handling tasks of the main Agent (default concurrency 4).
* subagent lane : Handles subagent tasks (default concurrency 8).
* session:[key] lane : Separated by the New session button, ensuring that each session has only one active operation (default concurrency 1).
* cron lane : Scheduled task.
* Telegram-specific lane : answer and reasoning, used for different types of content streams.

3. ### Session Lane (Session Lane)

First, we need to distinguish  the core differences between "ordinary RESTful API" and "conversational Agent" :

1. Stateless HTTP + RESTful API :
2. This type of interface (such as APIs for checking weather or orders) is "stateless" - each request is independent, and the server does not need to remember the information from the previous request. Therefore, it can  ** handle high concurrency in parallel ** : for example, if 1000 users check the weather simultaneously, the server can handle these 1000 requests concurrently without interference.
3. Conversational Agents (such as intelligent customer service, AI chatbots) :
4. This type of service is "stateful" -- the Agent must remember the context of the entire conversation (for example, if you first ask "What's the weather like in Beijing tomorrow?" and then ask "What about Shanghai?", the Agent should know that "What about Shanghai?" is a follow-up to the previous weather question).
5. And the time required for the Agent to process a single message: it needs to go through the process of "Thinking → Action → Observation" (for example, AI needs to first understand the problem, call tools, and generate a response), and this process may take several hundred milliseconds or even seconds.

If the "parallel approach" used to handle RESTful APIs is directly applied to handle conversations, serious problems will arise:

* Scenario example: You sent message A to the Agent, and the Agent is currently taking time to process A; at this time, you sent message B. If the server processes A and B in parallel, the following will occur:
  * Context window confusion: The Agent may incorrectly mix the content of B into the processing flow of A, resulting in answers that are off-topic;
  * State confusion: For example, A has not finished processing yet, but B has already modified the state of the conversation, ultimately causing the logic of the entire conversation to be completely disordered.
* This  "conflict caused by parallel processing of multiple messages in the same conversation" is "conversation race condition".

The "Session Lane" function of Gateway is designed to address this issue , and its core logic can be understood through the analogy of "bank windows":

| Concept                                                                                                                                                                        | Popular metaphor                       | Specific Logic                                                                                                                                                                                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |                                        |                                                                                                                                                                                                        |
| Session                                                                                                                                                                        | A complete conversation of a user      | For example, the entire process from the start to the end of your chat with Agent is a Session, which has a unique identifier.                                                                         |
| -                                                                                                                                                                              | -                                      | -                                                                                                                                                                                                      |
| Lane (车道)                                                                                                                                                                    | Exclusive window assigned to this user | The Gateway assigns a unique Lane (which can be understood as a "dedicated processing channel") to each Session.                                                                                       |
| Serial Queue                                                                                                                                                                   | Queue in front of the window           | Each Lane corresponds to a "serial queue" in memory - tasks in the queue must follow the "first come, first served" principle, with one task being processed only after the previous one is completed. |

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NTMyM2YwYjM2MmZmMGFlNWIzMmYzZjFiYzkwMTI1OTRfNjBUcVIzSE03NkZ5VmJKMEcyc2lHVUNQWGEyc1pTSGVfVG9rZW46WHBJN2JjSkxJb1RjVHp4QzZkYmN4ZmpWbkRnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ODk1ZGVkMDVkNjhhZmIyYTEyZThhODQ4ZmI4NWVjNDJfNHZWR2JtMmV0QnVUcUJGQzlGTDFVZTN2YVpxbkV3bE5fVG9rZW46WVcyeWJVTGhhb1NGUUJ4aXRNbGNBZENrbkFkXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

**For stateful Agent conversations, "task serialization within the same session" is the basic solution to ensure consistency. ** Do not easily parallelize multiple rounds of conversations for the same user, especially when the workflow involves write operations or even human participation.

暂时无法在飞书文档外展示此内容

7. ## Configure Hot Reload

Gateway has designed a sophisticated configuration hot reload mechanism,  with the core principle being: perform Hot Module Replacement whenever possible, and only restart when necessary . Gateway provides 4 reload modes, which are configured via gateway.reload.mode:

| Mode                                                  | Behavior                                                                                    |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| ----------------------------------------------------- |                                                                                             |
| off                                                   | Completely disabled, any changes are ignored                                                |
| -                                                     | -                                                                                           |
| hot                                                   | Only Hot Module Replacement, changes requiring a restart are ignored (no automatic restart) |
| restart                                               | Any change triggers a Gateway restart                                                       |
| hybrid (default)                                      | Perform Hot Module Replacement if possible, and restart if necessary                        |

When openclaw.json changes, Gateway does not simply perform a "full reload", but instead evaluates each configuration path individually to determine how to handle it:

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MjU0YWJlNmJmNWVhYjk5ODM4MTE4OTdkNGI4NTVmYTlfaDlVUlUzQlpNb29CNElVVG9heUJ0dFJjRGNDR0Q5eWlfVG9rZW46RUNDY2JRMXJZbzBlc1p4anprSWN5TG5tblFnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

Each rule specifies two attributes: kind (Hot Module Replacement / Requires Restart / No Action Required) and actions (specific Hot Module Replacement actions, such as restarting a certain module). For example:

```JSON

{ prefix: "hooks.gmail", kind: "hot", actions: ["restart-gmail-watcher"] },
// → 改了 hooks.gmail.model，只重启 Gmail 监听器

{ prefix: "hooks", kind: "hot", actions: ["reload-hooks"] },
// → 改了其他 hook 配置，重载 hook 模块

{ prefix: "cron", kind: "hot", actions: ["restart-cron"] },
// → 改了定时任务配置，只重启 cron 调度器
```

If the scheduled task configuration is modified here, only the cron scheduler will be restarted.

8. ## Data Stream

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmIzNWEzMGViZDMzNzllNzZiZGI0MTFmMWI2OGI5YTRfNklHZFBZcGtFZHczeDdqV3ZzSjBRY0NPc2xpd3ZGcFpfVG9rZW46T0U3cGIwdFYwb3pKZmV4ZUhXN2NDb1dObkJ5XzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

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

OpenClaw Channels is responsible for connecting multiple IM (Instant Messaging Platforms) to the central Gateway, communicating messages through the WebSocket protocol, and translating them into the unified standard format of the Gateway.

OpenClaw Channels adopts a Plugin-based architecture design, where all Notify Platforms are Channel plugins. They are only responsible for matters at the "access layer", including: login, sending and receiving messages, and protocol adaptation (converting platform events into standard events). Meanwhile, the responsibilities are handled by Gateway, including: state management, business decision-making, business decision-making, permissions, and scheduling strategies, etc. It can be seen that Channel is lightweight and driver-layered, and integrating a new Notify Platform simply means adding one more plugin.

The plugin's workflow is actually quite simple yet highly effective:

1. The plugin receives external events
2. Convert to Standard Event
3. Submitted to Gateway
4. Gateway processes and generates a response
5. The plugin converts the response back to the platform format and sends it out

Below is an example of accessing two accounts on the Feishu platform:

1. Core Function: Configure the enablement status, message reception strategy, connection method, and multi-account authentication information of the Feishu channel.
2. Key fields:`enabled`controls the channel switch,`groupPolicy/dmPolicy`respectively manages group chat/one-on-one chat message rules,`appId/appSecret`is the core authentication information for Feishu applications.
3. Flexible Expansion:`accounts`supports multi-account configuration, which can meet the scenarios where different Lines of Business use different Feishu applications.

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

4. # Agents Intelligent Agents

OpenClaw supports the multi-Agent architecture, where the main intelligent agent is called the Main Agent, which is a streamlined and efficient programming agent. The OpenClaw Agent adopts the Agent Loop operating mode, processes user messages, executes tool calls, and feeds the results back to the LLM, looping until the LLM generates a response without tool calls.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmIwMGFiOGNkNzA4MDk0ODczOTJmY2Q2YWE3MTRhNDJfaGVkc2J3QTVuUGs4SGpjbmhHU3UwRU5EZnB2RjJnbGdfVG9rZW46TGREdGJFTkdlb1BHRHd4VW80Y2M4SXhpbm5nXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. ## Agent Runner

`Agent Runner` is an Agent execution engine extended from the `Pi Agent Core` (Author: Mario Zechner) Agent framework, responsible for coordinating model inference, tool execution, and session management for all Agent interactions.

Before invoking the LLM, Agent Runner will perform `Context Engineering` (context engineering) tasks, including:

1. LLM Parser : Dynamically selects the LLM most suitable for the current task.
2. System Prompt Builder : Dynamically assemble System Prompts based on currently enabled Skills (such as browser automation, file access, etc.). Dynamic loading and construction can save a lot of context windows.
3. Conversation History Loader : Reads persisted memory data from the locally stored Workspace directory.
4. Context Window Guard : When the Context Windows approach the Max token limit of the LLM OpenAPI (e.g., 80%), the Agent will automatically compress to ensure the conversation does not interrupt. However, the compressed Context Windows are likely to lose critical conversation records.

Only after preparing the final Prompt set will the Agent Runner call the LLM OpenAPI to send a request and pass the result returned by the LLM to the next layer of the Agentic Loop for processing. ** It can be seen that in the Agent Runner stage, optimizing token usage and effectively compressing Context Windows become one of the important considerations. **

2. ## Context Window GuardContext Window Management

The context window of the LLM API is limited (e.g., 128k). As the conversation of the multi-turn dialogue agent lengthens and Tools, Skills, etc. are added, the Prompt Size is likely to exceed the limit, rendering the LLM API unavailable.

Therefore, Agent Runner implements `Context Window Guard`, which conducts multi-stage defense before the LLM API is called but after the Prompt is constructed:

1. Perform `token size` check by threshold.
2. If the check is passed, but  a context overflow is still triggered during operation, then automatic compression (using LLM to perform a structured summary of the previous conversation history) is initiated.
3. If compression fails, this system will attempt to reset the session and prompt the user.

It is worth noting that context compression comes at a cost, as it leads to information loss and summary bias, while resetting the session may also affect continuity. In enterprise scenarios, unique summary strategies, protection of key facts (such as order numbers/amounts/dates), and traceable logs can be configured as needed to minimize deviations after compression.

**Context window management is part of Context Engineering, and a mature Agent system should use a multi-layered strategy of "pre-check + post-remedy" to minimize the likelihood of LLM errors being the final errors users encounter. **

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YTMyM2UzZDI2NWRhYTkyMmNjM2VkMWFlNGJiYzAyMWNfNzNxTDVQWHEwbGdrMHpTNjdlZVlzOGo3ZU53NmlZcFBfVG9rZW46V1dTOWJEVzlvb0hadEN4TkhHT2NVZXdublRmXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

The above figure shows  ** the Token lifecycle management and overflow fault tolerance process of OpenClaw Gateway for LLM context window constraints ** , which, through ** hierarchical strategies of pre-check, graded alert, model degradation, summary compression, and session reset **, maximizes the continuity of conversation context while ensuring service availability.

1. #### Prompt Construction and Token Pre-verification Phase

* Input : User Request + Historical Conversation Context → Construct Complete Prompt
* Core Logic : Perform pre-verification based on the model's context window (example: 32k Tokens):
* Token ≥ 32k : Directly enter the request distribution process (expected to trigger context overflow)
* 16k ≤ Token < 32k : Record WARN-level log alerts, mark "context approaching capacity limit", and continue request distribution
* Token < 16k : Throws `FailoverError`, triggers **model degradation chain **(switches to a backup model with a smaller context window) , then performs request distribution

2. #### Request Distribution and Result Determination Phase

* **Request Distribution ** :  Submit the Prompt to the Target LLM Service
* Result Branch :
* **Request successful ** : Directly return the model-generated result, and the process terminates
* Context Overflow : EnterOverflow Fault Tolerance Processing Branch

3. #### Context overflow fault tolerance handling phase

* First overflow determination :
* Non-first overflow : directly return an error message and execute session reset , terminating the current conversation link
* First overflow : Trigger LLM-driven context summary compression
  * Extract structured summaries from earlier messages in historical conversations, replace the original long text with summaries, and reduce Token usage
* Compression Result Determination :
* **Compression successful ** : Reconstruct the Prompt based on the compressed context and re-enter the request distribution process
* Compression failed : Return error message and execute session reset , terminate the current conversation link

2. ### LLM API Thinking Level Classification

Some tasks require in-depth thinking and reasoning from the LLM, while others only need quick answers. Therefore, OpenClaw Agent has adopted a thinking grading mechanism, which defines 6 increasing thinking levels to correspond to tasks of different complexities:

| Level                                                           | Examples of applicable scenarios                                 |
| --------------------------------------------------------------- | ---------------------------------------------------------------- |
| --------------------------------------------------------------- |                                                                  |
| `off`                                                         | "Hello", check time, simple translation                          |
| -                                                               | -                                                                |
| `minimal`                                                     | Quick Q&A, Format Conversion, Simple Summary                     |
| `low`                                                         | Default. Everyday conversations, writing emails, code completion |
| `medium`                                                      | Code Debugging, Multi-step Analysis, and Solution Comparison     |
| `high`                                                        | Architecture Design, Complex Algorithms, Long Text Reasoning     |
| `xhigh`                                                       | Only applicable to `specific models such as gpt-5.2-codex`     |

It should be noted that different LLMs vary greatly in their support for "thinking levels". Some support multiple levels of thinking, some only have on/off for thinking, and some do not support it at all. Therefore, OpenClaw maps internal levels to the corresponding LLM API parameters based on the selected LLM.

When a message enters the system, the thinking level will be determined according to a certain priority:

 Priority Decrease : The system checks in the order of "inline instruction → Session specification → Agent configuration → model fallback" sequentially. Once a match is found at a certain level, it directly adopts that thinking level and no longer checks further down.

**The internal connection command **refers to  **embedding a system instruction directly in the content of the business message sent by the user ** , so that the system can temporarily execute specific logic when processing this message (here it specifies the "thinking level"). The instruction is not sent separately, but "embedded in the same message".

 Granularity Control :

* Inline Instruction:Single Message Level Fine-grained control, suitable for scenarios where temporary high-intensity/low-intensity thinking is required.
* Session Specification:** Session-level ** persistent control, suitable for scenarios where the entire conversation requires a unified level of thinking intensity.
* Agent / Model Configuration:** Global-level ** basic control, serving as the default fallback strategy.

| Priority                | Source                                                       | Example                                                         | Effective Scope      |
| ----------------------- | ------------------------------------------------------------ | --------------------------------------------------------------- | -------------------- |
| ① Inline Instruction   | Message contains `/think high`                             | `Help me optimize this code /think high`                      | This message         |
| ②Session Specification | Send separately `/think medium`                            | Send pure `/think medium`(no content)                         | Subsequent messages  |
| ③Agent Configuration   | `agents.defaults.thinkingDefault`Configuration             | `openclaw.json`configuration                                  | Global Default       |
| ④ Model fallback       | Check the model information directory to determine the level | If the model supports Reasonning, otherwise fallback to `off` | Global Configuration |

Additionally, if an error like "The current model does not support this level of thinking" is returned by the LLM API during runtime, the system will parse the error message and automatically retry with a lower level, trying to avoid direct task failure as much as possible.

3. ### LLM API Failover

Using a single LLM poses a risk of "single point of failure", and OpenClaw Agent supports Multi-Models. It also implements a two-layer fault tolerance mechanism:

* Inner Authentication Rotation : Multiple API Keys within the same LLM automatically switch, continuously trying the next Key that is not in the cooldown period (exponential backoff mechanism), until success or all Keys have been tried.
* Outer model degradation : Automatically fall back to the next candidate LLM.

It should be noted that model degradation may lead to differences in LLM capabilities and even affect the stability of tool invocation. When enterprises implement it, they often need to clarify model stratification and their respective capability base lines (which tasks allow degradation, which require strong models, to which model degradation is allowed, etc.), and record degradation logs for easy troubleshooting and auditing.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MmI3N2Y5OWNhMTQ5YjcwOTEzZDBmOGRjNDg3MmZjMTFfNE5LMkw2dUpKRVdCTUxmQVlTSEo2NVhCdFlrWU1FbFhfVG9rZW46T1pTZWJwejVHb3hKd294ZjBBWWNlQkFCbmxoXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

The above figure shows ** the **** "dual-layer high-availability model scheduling architecture" ** of OpenClaw Gateway **, which consists of ** an inner authentication rotation ** and ** an outer model degradation chain . The goal is to ensure that user requests can continue to be processed and to improve system availability and fault tolerance in failure scenarios such as API Key exhaustion and model service unavailability.

1. #### Inner layer: Key Rotation

The inner layer is responsible for ** multi-key load balance and failover of a single model **:

* Configure a set of API Keys for each model (e.g., `Key A → Key B → Key C`) and use them in a sequential rotation.
* When the current Key is exhausted due to quota depletion, rate limiting, or authentication failure, the system automatically switches to the next Key and retries.
* Only whenall keys of the model are exhausted / invalidatedwill the outer "model degradation chain" be triggered, switching to the next alternative model.

2. #### Outer layer: Model Failover Chain

The outer layer is responsible for ** cross-model service degradation and failover **, attempting models in order of preset priority :

1. Preferred Model : Prioritize the use of `Claude Sonnet`, and ensure its availability through the rotation of inner `Key A/B/C`.
2. Level 1 Downgrade : If all Keys of `Claude Sonnet` are exhausted, it will automatically downgrade to `Gemini Pro` and use its corresponding `Key D/E` for rotation.
3. Secondary Degradation : If all keys of `Gemini Pro` are also exhausted, continue to degrade to `gpt-4o`.
4. Successful Termination : As long as any model successfully processes the request, a response is returned and the process ends.
5. ## Agentic Loop

OpenClaw Agentic Loop is based on the open-source project Pi SDK (https://github.com/badlogic/pi-mono). A Loop consists of 4 core stages:

1. Context Assembly
2. Model Inference
3. Tool Execution
4. Reply Dispatch

```Plain
提问 → 思考 → 规划 → 行动 → 观察 → 思考 → 行动 → 等待 → 检查  → 纠错  ... → 完成
```

4. ## Agent's workspace

OpenClaw supports the multi-Agent architecture, where each Agent has its own dedicated data such as LLM, configuration, session, Workspace, Memory, etc., as shown in the figure below.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NmQ1ZDFiOTExZWE5MmY5ZDQ2NWU5NWEwZDM3MDkyNmFfbFNhS3hMcllISDVKcFZKSlNVdGkyWUtVUUZqR29uNVVfVG9rZW46U2pmcmJ0bFdVb0Z4Mkd4REF4b2NHNGhLbjRnXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

This diagram shows ** the file directory and data storage architecture of the OpenClaw system **, with the core being to split the  system into two major storage domains: ** Global State Directory ** and ** Agent Workspace **, clearly demarcating the responsibility boundaries between  "system infrastructure configuration" and "Agent business logic definition". The following will break it down in detail by module.

1. ### Upper layer: Global State Directory (Global State Directory)

The root directory is `~/.openclaw`, which is the ** system-level global data root directory ** of OpenClaw,  storing public configurations, authentication information, and persistent states across workspaces and agents, and is the infrastructure for system operation , decoupled from the business logic of individual agents.

1. Global Basic Configuration

   1. `Global openclaw.json`: The core global configuration file for OpenClaw instances, which is the core parameter definition file for system startup and operation, covering the configuration of core capabilities such as default model, workspace, session management, concurrency control, sub-agent scheduling, and security sandbox, and determines the basic runtime behavior of OpenClaw instances.

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
2. Agent Configuration Directory**`agents/<agent-id>/agent`**stores the system-level runtime configuration for a single Agent, associated with two core files:

   1. `auth-profiles.json (API Keys)`: Authentication configuration file, which stores API keys, authentication credentials, and service provider configurations for LLM calls, and is the core authentication information for the system to call large models.
   2. `models.json (Automatically discovered models)`: Model metadata file, which stores the list of available large models automatically identified by the system, model capability parameters, and invocation configuration, and is the core data source for model scheduling and degradation chains.
3. Agent Session Directory**`agents/<agent-id>/sessions`**stores the full session persistence data of a single Agent, with the core output being `*.jsonl transcription records`:

   1. Persist in jsonl format  the full context, session state, and execution logs of each round of conversation, supporting capabilities such as session recovery, historical backtracking, and conversation auditing.
4. ### Lower layer: Workspace (Agent workspace)

The root directory is `~/<agent-workspace>`, which is  ** the business definition working directory for a single Agent ** , storing 8 files

These 8 files, from the capability boundary `AGENTS.md`, identity positioning (`IDENTITY.md`, personality traits (`SOUL.md`, user adaptation `USER.md`, security control `TOOLS.md`, timing capability (`HEARTBEAT.md`, initialization guidance (`BOOTSTRAP.md`, long-term memory `MEMORY.md`, jointly construct the complete personality and capability system of the Agent from 8 dimensions.

Among them, `AGENTS.md` defines the boundaries of capabilities, `SOUL.md` injects the soul, `TOOLS.md` demarcates the restricted areas, which are the three core elements.

5. ## Agent's core configuration file (lower layer: Workspace)

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

| Configuration File Name                                                                                                                                                                                                                                                                  | Is it mandatory? | Core Role                                  | Supplementary Explanation                                                                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |                  |                                            |                                                                                                                                                                                                                           |
| `AGENTS.md`                                                                                                                                                                                                                                                                            | is               | Define the code of conduct for Agent       | Defining the corebehavioral boundariesof Agents, such as ensuring safety, anthropomorphic conversation, maintaining memory, etc., is the core code of conduct for Agent behavior, and it is recommended to read carefully |
| -                                                                                                                                                                                                                                                                                        | -                | -                                          | -                                                                                                                                                                                                                         |
| `IDENTITY.md`                                                                                                                                                                                                                                                                          | is               | Define Agent's identity information        | Set theoccupation/roletags of the Agent, such as programming expert, designer, engineer, etc., which determine the core competency positioning of the Agent                                                               |
| `SOUL.md`                                                                                                                                                                                                                                                                              | is               | Define the personality traits of the Agent | Injecting Agent'spersonality attributes, such as cheerful, humorous, calm, etc., determines Agent's dialogue style and emotional expression                                                                               |
| `USER.md`                                                                                                                                                                                                                                                                              | is               | Define the information of the owner (user) | Recordthe user's personalized information, such as name, hobbies, habits, etc., so that the Agent can precisely adapt to the user's needs                                                                                 |
| `TOOLS.md`                                                                                                                                                                                                                                                                             | is               | Define the permission control for Tool     | Configurethe allowlist/blocklist for tool invocation, defining the scope of tools that the Agent can use or is prohibited from using, for security management and control                                                 |
| `HEARTBEAT.md`                                                                                                                                                                                                                                                                         | Optional         | Define scheduled task configuration        | Configure the Agent'speriodic execution tasks, such as periodically checking specified content, pushing reminders, etc., and configure them as needed                                                                     |
| `BOOTSTRAP.md`                                                                                                                                                                                                                                                                         | is               | First-time startup boot configuration      | Only takes effect when the Agent is started for the first time, used to complete the onboarding process                                                                                                                   |
| `MEMORY.md`                                                                                                                                                                                                                                                                            | is               | Long-Term Memory Document (RAG Source)     | As the data source for Agent's long-term memory, it supports RAG (Retrieval Augmented Generation) capabilities and retains long-term conversation/knowledge memory                                                        |

Notably, Agent is not a resident process but a transient instance, with each conversation being a complete "load-execute-destroy" cycle. That is, each conversation loads the above content to build the basic System Prompt.

1. ### `AGENTS.md`

When an Agent starts, it reads AGENTS.md to define how the Agent operates, including:

1. Where files are stored and how memory works.
2. What tools are available and when to use them.
3. When to speak and when to remain silent.
4. When to report and when to handle independently.
5. How to use the Task Control Center, such as task updates, comments, etc.

Without AGENTS.md, the performance of the Agent cannot remain consistent.

2. ### `SOUL.md`

Personality is important, and professional Agents with a clear identity can produce focused and high-quality outputs. The SOUL file contains names and roles, for example:

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

6. ## `Agent`'s `Memory` System
7. ### Centralized Memory Storage

Different from the design of Claude Code, Cursor, etc., which use Project directories and sessions to isolate context,  OpenClaw is your personal assistant, so its Memory can mix and store the context of all your conversations with it without the need for isolation.

* Benefit : It always remembers everything you've talked about in multiple conversations and won't forget due to creating a new conversation. For example: You asked it to organize your emails in Telegram in the morning, write a report in Slack in the afternoon, and schedule tomorrow's agenda in WhatsApp in the evening, and it remembers it all. It's just like a personal assistant.
* Disadvantage : It consumes a large number of tokens because it inputs all the context into the LLM at once.

2. ### Long-term Memory Refinement

OpenClaw has designed a very ingenious, Markdown file-based persistent memory system, which is also stored in the Workspace directory:

* `memory`: The directory contains daily work memories, saved in files named YYYY-MM-DD.md, used for real-time review of the conversation process, with timeline and context.
* `MEMORY.md`: Used to store the refined "essence" of long-term memory, such as key preferences and conclusions extracted through deduplication, categorization, and extraction from daily working memory, which remain valid over the long term.

More specifically, OpenClaw automatically reviews the recent working memory every certain period (e.g., 1 day), summarizes the work content once, and refines valuable information into the long-term memory file MEMORY.md.

The hierarchical design of memory and MEMORY.md facilitates the Agent's recollection when needed in the future. For example, one month later, OpenClaw can still recall (retrieve) what you did at that time.

3. ### Refresh memory before conversation compression

`Agent Context Window Guard` Before compressing the context, to avoid the loss of critical information, a memory refresh will be performed. This is equivalent to giving the Agent an opportunity to write critical information into memory.

暂时无法在飞书文档外展示此内容

The above figure shows  the full closed-loop process of "session initiation → memory retrieval → dialogue generation → memory writing → memory persistence before context compression" , with the core being to enable memory to deeply participate in the dialogue lifecycle while ensuring information integrity and memory traceability.

| Module                                                                                                                | Core Responsibilities                                                                                                  |
| --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| --------------------------------------------------------------------------------------------------------------------- |                                                                                                                        |
| `System Prompt`                                                                                                     | Inject system-level instructions, load workspace boot files, and initialize long-term memory                           |
| -                                                                                                                     | -                                                                                                                      |
| `Agent Loop`                                                                                                        | Agent Main Loop, coordinating conversation flow, tool invocation, model inference, and memory writing                  |
| `memory_search`                                                                                                     | Memory retrieval tool, responsible for memory index construction, query, and precise segment extraction                |
| `File System`                                                                                                       | Persistent storage layer, stores `MEMORY.md`(long-term memory) and `memory/YYYY-MM-DD.md`(temporal working memory) |

1. **Session Start ** : Load the workspace boot configuration (such as `BOOTSTRAP.md `), and load the full text `MEMORY.md `at one time during session initialization, inject the long-term essence memory into the initial dialogue context, and lay a memory foundation for subsequent interactions.
2. **Inject retrieval instruction ** : `System Promp` t injects a mandatory instruction into `Agent Loop`: "Before answering a question, call `memory_search`", ensuring that  Agent must retrieve historical memory before processing user queries , avoiding cross-session forgetting and ensuring conversation coherence.
3. Memory Retrieval and Extraction :
4. Using the user's question as the query term, call ` memory_search` (query) to initiate memory retrieval; if a memory index is found to be missing or corrupted during retrieval, the index will be automatically rebuilt before matching relevant memories;
5. After matching relevant memory,  call `memory_get ()` to precisely extract the specified memory file (such as `memory/2026-02-07.md`) , the memory segment within the specified line range, and inject this segment into the conversation context for the model to use.
6. Model responds and writes to memory :
7. The model generates responses by combining retrieved memory fragments and the current user question;
8. Write the key information of this session (such as new user requirements, task progress, core conclusions, etc.) into the daily chronological memory file (`memory/YYYY-MM-DD.md`), and at the same time, the `memory_search` module performs incremental indexing on the newly added content and updates the retrieval data.
9. **Threshold trigger memory persistence ** : When the `Context Window Guard `detects that the amount of context Token is close to the compression threshold, it first triggers the `memory flush `(memory persistence) mechanism: the key memory that is not persisted in the current session is written to the memory file corresponding to the date; after completion of persistence, context compression is performed to avoid key memory loss caused by discarding the old context.

* Core Process: ** Initialize and Load Memory → Force Retrieve Instructions → Precise Retrieve and Extract → Generate Response and Update Memory → Persist Memory Before Compression ** ;

4. ### `MEMORY.md` VS `session.jsonl`

| Dimension                                                                                                                                      | Session Store                                           | Memory & MEMORY.md                                                      |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------------------- |
| ---------------------------------------------------------------------------------------------------------------------------------------------- |                                                         |                                                                         |
| Storage Location                                                                                                                               | `~/.openclaw/agents/[agentId]/sessions/`              | `~/.openclaw/workspace/memory/`和 `~/.openclaw/workspace/MEMORY.md` |
| -                                                                                                                                              | -                                                       | -                                                                       |
| Owner                                                                                                                                          | `Gateway`                                             | `Agent`                                                               |
| Usage                                                                                                                                          | Store all conversations in a short-term session         | Stores long-term memory and daily working memory                        |
| File Format                                                                                                                                    | `JSONL`                                               | `Markdown`                                                            |
| Content Type                                                                                                                                   | Complete conversation history (User/Assistant messages) | Manual summary, long-term memory, reference information                 |
| Access Method                                                                                                                                  | Gateway Automatic Reading and Management                | Agent reads through `memory_search`/`memory_get`.                   |
| Session Isolation                                                                                                                              | One file per session key (independent)                  | Global sharing (accessible to all sessions)                             |
| Size Control                                                                                                                                   | Automatically cleaned up via `session.maintenance`    | Manual Management                                                       |
| Search Support                                                                                                                                 | Semantic search is not supported                        | Supports `memory_search`semantic retrieval                            |
| Vector Index                                                                                                                                   | Not supported                                           | Supports hybrid search of embedding vectors + BM25                      |

`Session Store` short-term memory has a `reset` mechanism: after reset, a suffix x.jsonl.reset.date will be added to x.jsonl.

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

The long-term memory in MEMORY.md has a Flush mechanism:

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

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NTE3MTM4ZGRhNWZhZDg1NzVkNWJjNGI2NDc2MzZjMjRfY2k0MHZUd3AxV2RhaGZiYmYyT0p1ZVdlbWFHMnV6UDlfVG9rZW46Tkxsa2JQYW14b2Y2bm54dUdnTmNTbmdsbnlmXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

The figure above shows  ** the complete startup and initialization process of OpenClaw Agent ** , which is mainly divided into the "synchronous startup phase" and the "asynchronous background phase". The following is a detailed breakdown for you:

1. #### Top-level entry: Agent startup

This is the starting point of the entire process, and after the Agent is launched, it will ** concurrently trigger two core initialization tasks that are synchronous and executed immediately **:

* Bootstrap Loading (Configuration and Memory Loading)
* Session Initialization (Session Persistence)

2. #### Synchronous Initialization Phase (executed immediately upon startup)
3. Bootstrap Loading (Synchronous, Immediate): Responsible for injecting ** initial configuration and basic memory ** into the Agent, serving as the source of the Agent's "personality" and "knowledge":

   1. Load File:
      * `AGENTS.md`: Instructions and Capability Definitions for Agents
      * `SOUL.md`: Agent's character design, style, and code of conduct
      * `USER.md`: User's basic information and preferences
      * `MEMORY.md`: Long-term essential memory (main conversation memory)
      * `memory/today.md` / `memory/yesterday.md`: Recent chronological memory
   2. Final Action: Inject all loaded content ** into the System Prompt **, enabling the Agent to have complete context awareness from the start of the conversation.
4. Session Initialization (Synchronous, Immediate): Responsible for ** session persistence and history management** , ensuring traceability of conversations:

   1. Core Operations:
      * Read or create `a session log file in.jsonl format`
      * Write `session header` (session metadata, such as ID, start time, etc.)
      * Start tracking the complete conversation history
5. #### Asynchronous background phase (starts after initialization is completed, non-blocking)
6. Memory Index (Lazy Loading, Asynchronous)
   This is  ** the underlying support for memory retrieval ** , using the "lazy loading" design (only truly executed during the first memory retrieval):

   1. Trigger condition: first call `memory_search` (memory retrieval interface)
   2. Index building process:
      * Scan `MEMORY.md` and all `memory/*.md` memory files
      * Chunk the content into approximately 400 tokens
      * Generate vector embeddings for each chunk
      * Stored in SQLite database (Path: `~/.openclaw/state/memory/<agentId>.sqlite`)
   3. File monitoring mechanism:
      * Monitor `file changes in MEMORY.md` and `memory/**/*.md`
      * When it changes, it is marked as "Dirty Data", and 1500ms debounce is used to avoid frequent reconstruction
      * Subsequent automatic synchronization updates the index
7. Session Monitoring (Background, Asynchronous)
   Responsible for real-time synchronization of conversation logs and memory indexes , ensuring the timeliness of memory retrieval:

   1. Monitoring Object: The `.jsonl` log file corresponding to the session
   2. Threshold for triggering synchronization:
      * File Incremental Byte Count `deltaBytes > 100KB`
      * or number of incremental messages `deltaMessages > 50`
   3. Triggered Action: Execute "Index Synchronization" to update newly generated session content into the memory index, enabling subsequent retrieval to obtain the latest conversation information
8. ### Cross-`Session` Long-Term Memory Recall

`MEMORY.md` is shared across `Session`. When it needs to be used across `Session`, "memory recall" needs to be executed, which is actually `RAG`. `RAG` process:

1. Keyword exact match + semantic vector similarity match;
2. Select the top_K most relevant chunks;
3. Inject into the current Prompt.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YzUwZTIzMDNkMmRhNTI5Mjk5ZDlhNDU5NTIwZjg5ZjdfbjVaeG1laHlKZjV4OTNnMXhkMDdJTFRjaTh0RkdHS0JfVG9rZW46TkxXbGJYOURTbzhpWGt4RDlUaWNpQUpmbmFlXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

7. ## `Subagent` (Subagent)

When dealing with complex and long-running tasks, if the main Agent is made to wait synchronously, it will severely block user interaction. Therefore, the Subagent mechanism is introduced to break down long-running complex tasks and execute them in the background.

Currently, the LLM will "automatically" instruct the Main Agent to spawn one or more asynchronous `Subagents` via the `sessions_spawn` tool based on the actual situation.

```Plain
If a task is more complex or takes longer, spawn a sub-agent...
```

1. Derivation : The main Agent derives `Subagent` for complex tasks and sets it to background/Asynchronous Mode.
2. Notice : The main Agent first replies to the user with "Task has been started...", and then ends the current session.
3. Callback : `Subagent` posts back the results after task completion, which are then aggregated by the main `Agent ` and pushed to the user.

暂时无法在飞书文档外展示此内容

`Subagent` is like a "temporary worker", responsible for completing a certain subtask. Therefore, it is necessary to define clear boundaries for Subagent, which is responsible for execution and should not have management authority. So, for security reasons `Subagent` will not inherit all the capabilities of the main Agent, but instead adopt the strategy of "default withdrawal of sensitive and high-risk tools". For particularly dangerous tools, a secondary check will be added to ensure that they cannot be bypassed.

It is worth noting that `the necessity of deriving Subagent` is currently derived through LLM suggestions, so misjudgment is also possible. Additionally, attention should be paid to the overhead of `Subagent`, which will introduce more issues related to session management, result aggregation, failure fallback, and concurrency control (especially when multiple Subagents are running in parallel). Therefore, in practice, some timeout and retry strategies are usually required to make this mechanism operate more stably.

Configure the concurrency number and Tool-Policy of Sub-agents:

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

8. ## `Multi-Agent`(Multi-Agent)

First, we need to understand that in the design of OpenClaw,`Multi-Agent` and `Subagent` have fundamental differences :

| Dimension                                                                                                                                                                                                                                                                                                                                                            | Multi-Agent                                                                                                                                                                                                        | Sub-agents                                                                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |                                                                                                                                                                                                                    |                                                                                                                       |
| Application Scenarios                                                                                                                                                                                                                                                                                                                                                | Multiple completely independent agents can run side by side, either without interfering with each other or collaborating with one another. Use case: isolation for multi-user/multi-role/multi-workflow scenarios. | The main agent generates sub-agents to execute tasks. Use cases: task splitting, parallel processing, background work |
| -                                                                                                                                                                                                                                                                                                                                                                    | -                                                                                                                                                                                                                  | -                                                                                                                     |
| Conversation Hierarchy                                                                                                                                                                                                                                                                                                                                               | Lateral isolation (agent:main:whatsapp:...)                                                                                                                                                                        | Vertical nesting (agent:main:subagent:[uuid])                                                                         |
| Session Storage                                                                                                                                                                                                                                                                                                                                                      | `~/.openclaw/agents/[agentId]/sessions/`                                                                                                                                                                         | Manage from the parent agent's runtime context                                                                        |
| Tool Strategy                                                                                                                                                                                                                                                                                                                                                        | Static configuration is in `agents.list[].tools`                                                                                                                                                                 | Default restricted (excluding `session tools`)                                                                      |
| Session Key Format                                                                                                                                                                                                                                                                                                                                                   | `agent:[agentId]:[mainKey]`                                                                                                                                                                                      | `agent:[agentId]:subagent:[uuid]`                                                                                   |
| Creation Method                                                                                                                                                                                                                                                                                                                                                      | Automatically created when the Gateway starts                                                                                                                                                                      | Dynamically created via `sessions_spawn`tool                                                                        |
| Lifecycle                                                                                                                                                                                                                                                                                                                                                            | Independent and complete lifecycle                                                                                                                                                                                 | Depends on the operation of the parent agent                                                                          |
| Queue lane                                                                                                                                                                                                                                                                                                                                                           | `main/subagent/cron`等                                                                                                                                                                                           | dedicated `subagent lane`                                                                                           |
| Concurrency Control                                                                                                                                                                                                                                                                                                                                                  | via `agents.defaults.maxConcurrent`                                                                                                                                                                              | 通过 `agents.defaults.subagents.maxConcurrent`                                                                      |
| Default Concurrency                                                                                                                                                                                                                                                                                                                                                  | main default 4                                                                                                                                                                                                     | subagent default 8                                                                                                    |
| Nested Support                                                                                                                                                                                                                                                                                                                                                       | Not applicable                                                                                                                                                                                                     | Supports maxSpawnDepth (default 1, maximum 2)                                                                         |
| Tool Access                                                                                                                                                                                                                                                                                                                                                          | Any tool can be configured                                                                                                                                                                                         | Default exclusion of session tools (when depth ≥ 2)                                                                  |

As can be seen, when we need to execute completely unrelated and highly diverse task scenarios in OpenClaw, we can create Multi-Agents to achieve complete isolation in terms of personality, memory, and context among multiple Agents. For example: I need a daily life Agent, a programming Agent, a writing Agent, and a fitness Agent.

1. ### Create a new Agent

The instructions for adding a new Agent are very simple:

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

In this way, we have 2 Agents. Next, we need to use `bindings` configuration to `bind Agents` to different `Channel / accountId` to achieve conversation isolation. As shown in the following configuration:

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

Note: The prerequisite is that we need to create 2 bots in the Feishu Open Platform.

Check the configuration effect:

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

2. ### Use the specified Agent

There are 3 ways we can specify the Agent for use:

1. Select Agent via IM Bot, as Channel and Agent have a one-to-one correspondence.
2. Select an Agent via the GUI. As shown in the figure below, after selecting an Agent, go to Chat to start a conversation.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NjM2MGE2Njg4NTdjYjQyOWMxMDc2Njg5ZGFlZDMwOTJfZEdnQ2hhV1ozem1xdk9uVHhHaEJyOXhMZjBJczNiMDZfVG9rZW46VG5KRWJIMTQzb3hoRVd4TGgxbGNIOUtjbmdlXzE3NzQ0MzUxNDM6MTc3NDQzODc0M19WNA)

1. Select an Agent via TUI.

```JSON
/agents               # 查看 agents 清单
/agent main          # 切换到 main
/agent claudecode    # 切换到 claudecode
```

3. ### Collaboration between Agents

After creating multiple Agents, we may wish to have them collaborate to leverage the advantages of the Multi-Agent architecture.

OpanClaw provides 3 ways of collaboration among Agents:

1. `sessions_send` **, `Agent-to-Agent Messaging` (direct communication) method **: Send messages to an existing session, provided that both parties have a session and the `tools.agentToAgent` feature needs to be enabled. When communication between Agents needs to be written into the other party's `Memory`, it is recommended to use the `sessions_send` tool to send messages to each other.

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

2. `sessions_spawn` **, `Sub-Agent` ** Mode :  Treat an Agent as a subagent of another Agent . By re-engaging the subagent and passing in the task, it can complete the delivery. In task distribution scenarios, sessions_spawn is recommended.

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

3. Shared**`Workspace`**** ( **** File Sharing ** ** ) ** : Although each agent has its own independent workspace, it can also share specific directories.

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

Notably,`Agent-to-Agent` is based on Session, but as we mentioned earlier, `Session.jsonl` is short-term memory, so to avoid forgetting, we also need to maintain long-term memory in AGENTS.md.

```Markdown
## Agent-to-Agent Messaging

当你需要其他角色的意见或具体执行时：
- **日常生活问题和任务** → `@main` 处理主人的日常生活
- **编程问题和任务** → `@claudecode` 获取专业的编程技术支持
```

Additionally, when we configure multiple Agents with both `Agent-to-Agent` and `Sub-Agent` collaboration modes, how does OpenClaw decide which mode to use? In fact, it is the LLM that makes the decision based on the scenario. `Agent-to-Agent` is used for collaborative scenarios, `Sub-Agent` is used for delegation scenarios.

9. ## `Heartbeat & Cron`

`Heartbeat`&`Cron` are the foundation for implementing OpenClaw's active notification sending:

* Heartbeat : Periodic heartbeat check, in the main session, periodically (default 30 minutes) triggers the Agent to read `HEARTBEAT.md`, and autonomously executes the routine tasks defined therein (such as receiving messages, sending reports), and replies `HEARTBEAT_OK` when there are no tasks.
* Cron : Scheduled task execution, triggered at a precise time (e.g., sending a report punctually at 9:00 every day).

https://docs.openclaw.ai/zh-CN/automation/cron-vs-heartbeat

| Usage Scenarios                                                                                | Type                       | Features/Description                           |
| ---------------------------------------------------------------------------------------------- | -------------------------- | ---------------------------------------------- |
| ---------------------------------------------------------------------------------------------- |                            |                                                |
| Check the inbox every 30 minutes                                                               | Heartbeat                  | Capable of batch processing and context-aware  |
| -                                                                                              | -                          | -                                              |
| Send the report punctually at 9:00 AM every day                                                | Timed Task - Isolated Mode | Requires precise timing                        |
| Monitor upcoming events in the calendar                                                        | Heartbeat                  | Naturally suitable for periodic sensing        |
| Run weekly in-depth analysis                                                                   | Timed Task - Isolated Mode | Independent task, different models can be used |
| Remind me in 20 minutes                                                                        | Timed Task - Main Session  | Precisely timed one-time task                  |
| Backend Project Health Check                                                                   | Heartbeat                  | is carried on the existing cycle               |

1. ### Configure `Heartbeat`
2. Periodic Configuration

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

2. Write `HEARTBEAT.md`: Every time a heartbeat is triggered, this file will be read once.

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

2. ### Configure `Cron`

Precisely time-triggered Cron is often used for point-based reporting:

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

* Explicitly specify --to "user:XXX" (your Feishu ID)
* Explicitly specify --account default
* Explicitly specify --channel feishu

3. ### Configure message source

OpenClaw needs to be combined with specific Skills and message sources (such as Brave Search, Zhipu Internet Search MCP, etc.) to implement web search and improve information quality.

5. # `Nodes` Terminals
6. ## Nature of Nodes

Nodes is the unified abstract encapsulation of OpenClaw for** physical devices (iOS/Android/macOS)**, an abstraction layer for hardware terminals.

* For developers / upper-level agents: There's no need to worry about whether the underlying system is iOS or Android. Simply call the unified API (such as `camera.snap`), and OpenClaw will handle device differences for you.
* For the device: it needs to install a lightweight ** Node Client App ** (which can be understood as a "device-side agent"), and this App is the bridge for OpenClaw to interact with hardware —— for example, when you call `camera.snap`, in fact, after the Node Client App receives the instruction, it calls the camera API of the mobile phone to complete the photo capture.

Currently supported Node types include:

1. macOS Node : system.run (execute command), system.notify (notification), Canvas/Camera.
2. iOS Node : Canvas, voice wake-up, camera photo/video recording, screen recording, voice trigger.
3. Android Node : Canvas, voice interaction, camera, screenshot, SMS integration (optional).

Local tools will be executed directly within the Agent module, but if remote Nodes are involved, they must go through the Gateway. The system will implement two levels of checks, and only after passing both checks will commands be issued to remote devices.

* Allowlist , for example, a certain iOS node only allows the camera.snap tool.
* Manual approval , for example, requires manual confirmation when calling sms.send.

2. ## Communication Protocol between Nodes and Gateway

* Transport : Gateway WebSocket (LAN/Tailscale/SSH Tunnel)
* Discovery : node.list / node.describe enumeration capabilities
* Execute : node.invoke runs local operations on the device
* Commands : camera.snap/camera.clip (take photo/record video), screen.record, location.get, notifications

3. ## Communication process between Nodes and Gateway

* Capability Statement : When a new Node connects for the first time, it requires manual approval to be trusted; when connecting, it must inform the Gateway of the commands and capabilities it supports, which also serves as the basis for subsequent tool security verification.
* Disconnection Cleanup : When a Node disconnects, the Gateway will automatically reject all incomplete remote calls to prevent the Agent's tool requests from hanging for an extended period.
* Timeout mechanism : Each remote call has a default timeout of 30 seconds. After the timeout, it automatically fails and is cleaned up.

暂时无法在飞书文档外展示此内容

If the Gateway is running on a Linux server, but you want to use a Mac's camera to take photos.

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

[OpenClaw System Architecture Analysis_openclaw Architecture - CSDN Blog](https://blog.csdn.net/evilstar2015/article/details/157617589?ops_request_misc=elastic_search_misc&request_id=0ee0e5d60705817091e6440c0a7318fe&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-157617589-null-null.142^v102^pc_search_result_base1&utm_term=OpenClaw%20%E6%9E%B6%E6%9E%84&spm=1018.2226.3001.4187)

https://zhuanlan.zhihu.com/p/2015645820083533188

https://zhuanlan.zhihu.com/p/2011197075765884377

https://zhuanlan.zhihu.com/p/2004505448938767308
