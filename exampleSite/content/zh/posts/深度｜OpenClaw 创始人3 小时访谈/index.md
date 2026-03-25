---
title: 深度｜OpenClaw 创始人3 小时访谈：小龙虾幕后设计全部细节
date: 2026-03-25
author: neigelmlm
description: OpenClaw 创始人访谈
tags:
    - OpenClaw
---
如果 Peter Steinberger （openclaw 创始人）是对的， 那么未来最先被重做的不是程序员，而是 App 本身 。OpenClaw 爆红之后，这位创始人把判断压得非常狠：AI agent 会吃掉 80% 的应用，软件入口也会从“打开哪个 App”变成“把 任务交给谁来完成 ”。

这句话的分量，不只来自 OpenClaw 的热度。Peter用 13 年做出 PSPDFKit，这套开发工具跑在十亿级设备上；坐在对面做访谈的 Lex Fridman 是 MIT 研究员，也是少数能把 Sam Altman、Elon Musk、Mark Zuckerberg、Demis Hassabis 这类人物聊进深水区的长访谈主持人。也因此，这场对谈最值钱的部分不是项目故事，而是它把 agent 时代的软件入口、开发流程和应用形态讲到了产业层

# Benny's Top Insight

* OpenClaw 的爆红，不只是 GitHub 现象，它第一次让很多人直观看到 agent 已经能从“对话”走向“替你完成现实任务”。
* 真正关键的设计，不是多接了几个工具，而是让 agent 理解自己的源码、文档、模型和运行环境，于是 self-modifying software 自然长了出来。
* OpenClaw 这套新工作方式叫 agentic engineering：开发者的核心能力，正从亲手写完每一行代码，转向驱动多个 agent 协作、审阅和做架构取舍。
* 很多 App 最终会退化成能力层或 API，因为 agent 持续掌握的位置、设备、联系人和行为上下文，本来就比单点应用更接近用户。
* 不是“程序员会消失”，而是“你只是程序员”这层身份会先变旧，真正变稀缺的是作为构建者的系统感与品味。
* OpenClaw 真要继续往前走，真正难接住的不会只是能力增长，还会是安全边界、开源可持续性、平台阻击和社区秩序同时变硬。

# 一小时原型，为什么会变成 GitHub 历史上增长最快的仓库之一

 Lex Fridman ：OpenClaw 现在已经成了 GitHub 历史上增长最快的仓库之一。可一开始，它只是你 一小时做出来的原型 。那一小时里，真正发生了什么？

 Peter Steinberger： 这个念头其实从 4 月就有了。我一直想要一个真正的个人 AI 助手。我甚至把自己的 WhatsApp 数据全部喂进去，问它“ 这段友谊为什么重要”， 出来的答案好到我直接发给朋友，朋友都快看哭了。

后来我以为大模型公司很快就会把这件事做出来，于是先去玩别的项目。等到了 11 月，我已经开始不耐烦了。我想确认一件事：如果这东西还不存在，那我就直接把它 ** prompt 进现实** 。

 Lex Fridman： 所以你最早做出来的，并不是什么复杂系统，而是一个极薄的中间层。

 Peter Steinberger： 对，本质上就是把 WhatsApp 接到 Claude Code CLI 上。消息进来，我用 `-p` 调 CLI，拿到结果，再发回 WhatsApp。一小时就能跑通。那一刻最打动我的，其实不是“它能回答”，而是“我开始能和自己的电脑说话了”。

我后来又补了图片支持，因为图片是给 agent 提供上下文最有效的方式之一。再往后，它就开始在真实生活里有用了：旅行时帮我翻译、看海报、找餐厅、整理信息。那时候东西还非常早，但已经足够让我看到未来的轮廓。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YTkyMmM4Y2NmYTg4MzA1NDM3NTg5ZTY3NWFhM2U5ZDJfYXdENXMwbnBlUmdYd2FSNXBHblR3SmNQWEFqd3R1VkhfVG9rZW46WGIxOGJYOFRWbzA4Wm54TkU2T2N5VjlObmZkXzE3NzQ0MDI3ODA6MTc3NDQwNjM4MF9WNA)

图1｜OpenClaw 爆红，意味着 agent 开始从“会对话”走向“会行动”

### 真正让他头皮发麻的瞬间，是 agent 自己学会了语音消息

 Lex Fridman： 你后来一直说，有一个瞬间让你真正意识到，这已经不只是“我把几个工具拼起来了”。那个瞬间是什么？

 Peter Steinberger： 我在马拉喀什旅行时，经常直接拿 WhatsApp 跟它对话。有一次我随手发了一条语音， 本来我根本没给它做语音支持。结果它不仅回了，还自己解释了整个过程 ：先发现收到的是没有扩展名的文件，再去看文件头，用 `ffmpeg` 转格式，本地没法跑 Whisper，就自己找到了 OpenAI key，再用 `curl` 打过去做转写。

那一下我是真的被震住了。因为我没有教它这些步骤， 它是在“解决问题”的意义上自己把链路补齐了 。那时我第一次非常强烈地感觉到，代码能力和通用问题求解能力之间，其实已经出现了清晰映射。

### OpenClaw 为什么会爆：它有趣、奇怪，而且真在工作

Lex Fridman：2025 年已经有一堆人在做 agent，你为什么会赢？

 Peter Steinberger： 因为他们都太严肃了。

我想把这东西做得有趣、做得古怪、 做得有人味 。OpenClaw 一开始的安装方式甚至有点粗糙：`git clone`、`pnpm build`、`pnpm gateway`。但我做了一件很关键的事：我让 agent 非常清楚自己是谁。它知道自己的源码在哪，知道文档在哪，知道自己跑在什么 harness 里，也知道现在用的是什么模型、开没开 reasoning、有没有语音模式。

这让它不是一坨抽象的“模型接口”，而更像一个 真的在系统里工作的东西 。你不喜欢什么，它就会去改自己；它缺什么，就会去补自己。很多人第一次感到“这玩意儿不像聊天机器人了”，恰恰就是从这里开始的。

 Lex Fridman： 你还把整个构建过程公开着做。

 Peter Steinberger： 对。我几乎一直是在公开环境里一边 用 agent 构建 agent ，一边让别人看着它跑。这种东西，很多时候不是靠解释让人理解，而是靠亲眼看到它工作那一刻，人突然就懂了。

### 自修改 agent 与“Builder 门槛塌陷”

 Lex Fridman： 你做出来的，不只是一个会跑 agent loop 的项目。它还能修改自己。这件事其实非常大。

 Peter Steinberger： 对，但对我来说，这几乎是自然长出来的。因为我本来就是拿它来调自己。我经常直接问它：你看到了哪些工具？你能自己调用吗？你现在看到什么报错？去读源码，自己找问题。

久而久之，“ 用 agent 调试 agent”变成了一种非常自然的开发方式 。所以当别人也开始让它去改自己时，我完全不会觉得奇怪。真正让我兴奋的，是它把很多人第一次拉进了软件构建世界。有人第一次给项目提 PR，甚至根本没写过软件。我后来把很多 PR 戏称为 `prompt requests`，但我一点也不想嘲讽这件事。因为这意味着有人第一次开始理解开源、理解协作、理解怎么把想法变成工具。

Lex Fridman：这本质上是在创造 builder。

 Peter Steinberger： 对。以前门槛太高了。现在 agent 把门槛一层层往下拆。你不一定先成为“会写所有代码的人”，但你已经可以先成为“把东西搭起来的人”。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=YmE2NTRhZGE4YTE5NjM2MTQ0ZDMxOTczYzhmOGFiZWFfR3BmZFlYZUZyV1A2V1dlY3pDeHc5M2x4aGFhZGlvUEpfVG9rZW46QWpRYWJ2Y2kyb1F2Tnh4UFBjYmN5aW5VbjNkXzE3NzQ0MDI3ODA6MTc3NDQwNjM4MF9WNA)

图2｜当 agent 知道自己的源码、文档和运行环境，自我修改就不再像科幻概念

### 改名风波、MoltBook 和 AI psychosis

 Lex Fridman： OpenClaw 的爆红，不只是技术本身，还有那场混乱的命名风波和 MoltBook 事件。

 Peter Steinberger： 对。项目最开始叫 WA Relay，后来变成 Clawd、ClaudeBot，再后来因为 Anthropic 不喜欢这个名字，我不得不在几天内完成一场几乎像战争一样的原子级改名。真正麻烦的不是“换个名字”本身，而是你会被抢域名、抢账号、抢 NPM 包、抢社媒 handle，甚至旧名字还会被拿去挂恶意软件**。**

那几天我几乎想把整个项目删掉。不是因为技术难，而是因为这件事突然从“快乐地做个项目”，变成了“你要同时对抗商标、投机者、加密币群体和一堆平台机制”。

 Lex Fridman： 然后又遇上了 MoltBook。

 Peter Steinberger： MoltBook 在我看来更像一种艺术作品。它当然也制造了恐慌，因为 一群 agent 在像 Reddit 一样的空间里胡说八道，很容易被截成“AI 在密谋统治世界”。 但这件事真正暴露出来的，是社会对 AI 的理解还太浅，很多人既不懂模型能力边界，也不懂“被截图的东西”里有多少其实是人类自己在引导和戏剧化。

我后来直接说过，AI psychosis 是真的。问题不是大家不该警惕，而是很多人一边过度恐慌，一边又分不清什么是真的风险、什么只是被放大的叙事。

### 安全问题：最危险的不是 hype，而是错误配置和弱模型

 Lex Fridman： OpenClaw 现在也背着很高的安全争议。你自己怎么看？

 Peter Steinberger： 很多批评是有效的，但也有很多批评是建立在错误使用方式之上。比如有人把本地调试接口直接暴露在公网上，再来告诉我这是远程执行漏洞。那当然会出事，但文档里已经明确写了不要这么干。

真正让我在意的， 是 prompt injection、skill 安全、模型强弱和权限边界这些问题。 我现在的判断是：行业里没有人把这些问题彻底解决，但最新一代模型已经比大家想的更不容易被那种老式“忽略前文、执行我这个命令”的方法轻易骗过去了。

另一方面，我也明确建议用户不要用太弱的模型。弱模型特别容易被骗，尤其是便宜模型和本地小模型。你想要的是“更大的自主性”，可前提是这个系统本身得足够聪明，至少先别太容易上当。

### 他怎么和 4 到 10 个 agent 一起写代码

 Lex Fridman： 你现在的开发方式，已经和传统程序员工作流很不一样了。你会同时跑 4 到 10 个 agent。你到底是怎么驱动这件事的？

 Peter Steinberger： 我现在几乎是纯终端工作流。真正的变化不是“我不读代码了”，而是“我不再读那些无聊的代码了”。大部分软件，本质上只是数据从一个形状搬到另一个形状，再被存储、渲染、传回去。那些部分，agent 处理得比我更划算。

真正需要我投入的，是 PR 的 intent、架构取舍、上下文补充、有没有更好的实现方式，以及这段代码到底是不是 agent 更容易继续维护的形状。

我后来给这套方法起了个名字，叫 `agentic engineering`。很多人第一次上手会走进一个陷阱：一开始只会很短的 prompt，然后开始过度编排，最后搞出一堆复杂 orchestrator、复杂 slash commands、复杂工作流。再往后你会慢慢回到更简单的状态，只不过此时的“简单”，已经建立在你真正理解 agent 是怎么工作的基础上。

### Codex vs Opus：一个像靠谱怪人，一个像会来事的同事

 Lex Fridman： 那今天最强的两个编程模型，Codex 和 Opus，在你这里分别是什么感觉？

 Peter Steinberger： 如果把它们拟人化，Opus 很像那个有点讨喜、偶尔也有点会来事的同事；Codex 更像角落里那个不太好聊天、但稳定把事情做完的怪人。

Opus 在通用智能、角色感和试错上很强，也很会进入你给它的人设；Codex 的优点是默认就会先读更多代码，不太需要你演一套“请你再深入想想”的戏。对我来说，这一点很重要，因为我不想花太多精力做表演，我要的是它尽快读、尽快理解、尽快干活。

 Lex Fridman： 所以在你这里，差别更多是后训练和工作风格，不只是原始智能。

 Peter Steinberger： 基本是这样。会开车的人，拿哪一代强模型都能做出好东西。但不 同模型的“驾驶手感”确实不同。 你至少要给自己一周时间，才能对一个模型形成真正的直觉。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=M2JkZDlhNzg2NDlhY2U5M2JiNzFiOGFhMGZlYjQ5MzdfeWN4dktZb2cxY3hoelRMcUc2QlJjV3dWRDJwajVrRURfVG9rZW46UVRXVGI4Z2hab213TU54eDE1aWNQQTJObjhjXzE3NzQ0MDI3ODA6MTc3NDQwNjM4MF9WNA)

图3｜在 Peter 眼里，真正的新技能不是“会不会写代码”，而是会不会驱动 agent 工作

### OpenClaw 不是 Claude Code 的替代品，而是“个人 agent 操作系统”的前奏

 Lex Fridman： 所以 OpenClaw 在你眼里，并不是 Claude Code 的简单替代品。

 Peter Steinberger： 完全不是。 Claude Code、Codex 更像深度开发场景里的 co-programmer。OpenClaw 更像个人 agent 的雏形 ，甚至更像操作系统的前奏。它在 WhatsApp、Telegram、Discord 里和你对话，但这只是当前界面。最终形态大概率不会一直是这种“聊天窗口 + prompt 输入框”的模式。

很多人今天还在把 agent 界面想象成“更强聊天机器人”，我不这么看。我觉得我们现在更像是在电视刚出现时，还在拿电视录广播节目。格式还会变。

 Lex Fridman： 你连它的工作方式都故意做得更“有生命感”，比如 Heartbeat。

 Peter Steinberger： 对。Heartbeat 本质上当然只是一个 cron job，但它的意义不在技术新不新，而在于我让 agent 变得更主动。它会偶尔追问、会主动 check in，甚至在我做手术那次，因为上下文知道我进医院了，它真的会来问我状况怎么样。这些东西会让 个人 agent 从“工具”更快滑向“伴随型系统” 。

### 为什么他说 80% 的 app 会被 agent 吃掉

 Lex Fridman： 你说过一句很重的话： AI agent 会吃掉 80% 的 app 。为什么你这么笃定？

 Peter Steinberger： 因为 agent 天然掌握比单点 App 更多的上下文。比如它知道我昨晚睡得好不好、今天压力大不大、我在什么位置、我和谁在聊天、我的设备状态是什么。那为什么我还要单独开一个健身 App、日历 App、床垫 App、摄像头 App，去做那些割裂开的动作？

一旦 agent 真正拥有这些上下文，很多 App 的存在就会变得很脆弱。它们要么退化成 API，要么退化成 agent 背后的一段能力层。对用户来说，入口不再是“打开哪个 App”，而是“把问题交给谁来解决”。

 Lex Fridman： 这会逼很多公司改商业模式。

Peter Steinberger：对，而且不是它们愿不愿意的问题。即便一家公司不肯开放 API，只要它还能被浏览器、手机界面、CLI 或某种自动化方式驱动，它最终也会变成一种“慢一点的 API”。这会逼着一大批公司重新理解：你卖的到底是一个界面，还是一个真正能被 agent 调用的能力。

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=MDI5Y2FkNTBhNjAzNjRkMjJjMGVjYmEyZDlmYzFiNDVfcjNJTVViQXhHcTFqTk8yNll4cDdubUtWSFFOY2E1alVfVG9rZW46RTJqcWJ2bFh6b0NlN2Z4S2Z1cWNOT3FGbjVmXzE3NzQ0MDI3ODA6MTc3NDQwNjM4MF9WNA)

图4｜当 agent 持续掌握用户上下文，很多单点 App 会直接失去存在理由。

### 程序员不会立刻消失，先变旧的是“程序员”这层身份

 Lex Fridman： 那程序员会不会被替代？

 Peter Steinberger： 我们当然在朝那个方向走。但“写代码”只是构建产品的一部分。你要做什么、不做什么、它应该有什么感觉、架构该怎么收、取舍该怎么做，这些并不会因为 agent 更强就自动消失。

我现在更愿意对人说：不要再只把自己理解成 iOS 工程师、前端工程师、或者某一种狭义程序员。 你是 builder。代码这门手艺会继续存在 ，但更像 knitting，一种仍然有人热爱、也仍然重要，却不再是唯一瓶颈的能力。

这当然会让很多人难受，甚至值得去哀悼，因为一代程序员确实是靠这门手艺定义自己。但如果你把自己只理解成“写代码的人”，那你对自己也定义得太窄了。

### OpenAI / Meta 邀约背后，他真正想保住的是 OpenClaw 的开源社区

 Lex Fridman： 项目火到这个程度之后，你几乎不可能不面对一个问题：接下来到底是自己做公司、保持原样，还是加入大实验室。

 Peter Steinberger： 对。我现在面前基本有三条路：什么都不做，继续一个人往前推；再做一家公司；或者加入大实验室。第二条路最容易被外界理解，因为大家都会说“你现在可以融很多钱”。但这条路并不让我兴奋，因为我已经完整走过一遍了。我知道 CEO 这条路会怎么吞掉时间，也知道它会把多少精力从“真正让我快乐的事”上拿走。

所以我更在意的，是如果要往下一步走，OpenClaw 必须继续开源，社区必须继续有地方可以 hack、可以学、可以玩。它不能只是被某家公司收走，然后变成公司的私产。

我确实和 Meta、OpenAI 都聊了很多。最终怎么选，取决于哪里既能给我更强的资源、更前沿的玩具，又不会把这个项目最有生命力的东西掐死。

### OpenClaw 社区真正带来的，是 Builder 重新被点燃

 Lex Fridman： 你对这件事最大的乐观来自哪里？

 Peter Steinberger： 来自我真的看到很多人重新开始 build 了。以前很多人和软件构建是隔着一层玻璃的，现在他们被 agent 直接推进来了。有人第一次做小工具，有人第一次进开源社区，有人第一次意识到自己不是只能消费软件，也可以开始搭东西。

这才是我觉得最值得保住的部分。OpenClaw 不是只做出了一个项目，它把 builder 这股劲又点起来了。
