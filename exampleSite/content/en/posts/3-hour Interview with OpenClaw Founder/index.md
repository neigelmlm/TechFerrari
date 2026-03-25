---
title: 3-hour Interview with OpenClaw Founder
date: 2026-03-25
author: neigelmlm
description: OpenClaw 创始人访谈
tags:
    - OpenClaw
---
If Peter Steinberger (founder of OpenClaw) is right,  ** then the first thing to be redone in the future will not be programmers, but the apps themselves ** . After OpenClaw became a sensation, the founder made a very bold prediction: AI agents will replace 80% of applications, and the entry point for software will shift from "which app to open" to " ** who to assign the task to ** ".

The weight of this statement does not solely come from the popularity of OpenClaw. Peter spent 13 years developing PSPDFKit, a development tool that runs on devices in the billions; sitting across for the interview, Lex Fridman is a researcher at MIT and one of the few long-form interview hosts capable of delving into in-depth discussions with figures like Sam Altman, Elon Musk, Mark Zuckerberg, and Demis Hassabis. Therefore, the most valuable part of this conversation is not the project story, but rather its discussion of the software entry points, development processes, and application forms of the agent era up to the industrial level

# Benny's Top Insight

* The sudden popularity of OpenClaw is not just a GitHub phenomenon; for the first time, it has allowed many people to intuitively see that agents have been able to move from "conversation" to "completing real-world tasks for you".
* The truly critical design is not about adding a few more tools, but rather enabling the agent to understand its own source code, documentation, models, and runtime environment, so that self-modifying software naturally emerges.
* OpenClaw's new way of working is called agentic engineering: the core competence of developers is shifting from writing every line of code by hand to driving multiple agents to collaborate, review, and make architectural trade-offs.
* Many apps will ultimately devolve into capability layers or APIs because the location, device, contact, and behavioral context continuously grasped by agents are inherently closer to users than single-point applications.
* It's not that "programmers will disappear", but rather that the identity of "you're just a programmer" will first become obsolete. What truly becomes scarce is the sense of system and taste as a builder.
* If OpenClaw really wants to keep moving forward, what will truly be difficult to handle will not only be the growth of capabilities, but also the simultaneous hardening of safety boundaries, open-source sustainability, platform resistance, and community order.

# One Hour Prototype, Why Did It Become One of the Fastest-Growing Repositories in GitHub's History?

 **Lex Fridman** : OpenClaw has now become one of the fastest-growing repositories in GitHub history. But at the beginning, it was just a prototype you **made in an hour** . What really happened during that hour?

**Peter Steinberger: **I actually had this idea since April. I have always wanted a real personal AI assistant. I even fed all my WhatsApp data into it and asked it, " **Why is this friendship important?" **The answer that came out was so good that I directly sent it to my friend, and my friend almost cried.

Later, I thought that large model companies would quickly develop this, so I went to work on other projects first. By November, I had already started to get impatient. I wanted to confirm one thing: if this thing didn't exist yet, then I would directly ** prompt it into reality** .

**Lex Fridman:** So what you first created was not a complex system, but rather a ** very thin middle layer. **

**Peter Steinberger:** Right, essentially it's just connecting WhatsApp to Claude Code CLI. When a message comes in, I use `-p` to call the CLI, get the result, and then send it back to WhatsApp. It can be up and running in an hour. What really struck me at that moment wasn't actually "it can answer," but rather "I started being able to talk to my own computer."

Later, I added image support because images are one of the most effective ways to provide context to agents. After that, it started to be useful in real life: helping me translate, read posters, find restaurants, and organize information while traveling. Things were still very early at that time, but it was already enough for me to see the outline of the future.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NjZiNWQyY2FlMzcxYzcyOTE4N2EyODhmOWJjOWRmMzlfU1F1RmV4SFBaNnpMdVFYSXFmUk85TFp3eGhocThtS1BfVG9rZW46WGIxOGJYOFRWbzA4Wm54TkU2T2N5VjlObmZkXzE3NzQ0MDMyMzI6MTc3NDQwNjgzMl9WNA)

Figure 1 | The sudden popularity of OpenClaw means that agents are starting to move from "being able to converse" to "being able to act"

### The moment that truly made his hair stand on end was when the agent taught itself to use voice messages

 **Lex Fridman:** You've always said later that there was a moment when you truly realized that this was no longer just "putting a few tools together." What was that moment?

**Peter Steinberger:** When I was traveling in Marrakech, I often directly used WhatsApp to talk to it. Once, I casually sent a voice message, ** even though I had never provided voice support for it. As a result, it not only replied but also explained the entire process on its own** : first, it detected that the received file had no extension, then checked the file header, used `ffmpeg` to convert the format, since it couldn't run Whisper locally, it found the OpenAI key on its own, and then used `curl` to send it for transcription.

I was truly stunned at that moment. Because I didn't teach it these steps,  ** it filled in the link on its own in the sense of "problem-solving" ** . That was the first time I strongly felt that a clear mapping had actually emerged between coding ability and general problem-solving ability.

### Why OpenClaw is booming: it's fun, strange, and actually working

**Lex Fridman: By 2025, there will already be a bunch of people working on agents. Why do you think you'll win?**

 **Peter Steinberger:** Because they're all too serious.

I want to make this thing interesting, make it quirky,  ** make it human ** . The initial installation method of OpenClaw was even a bit rough: `git clone`, `pnpm build`, `pnpm gateway`. But I did one very crucial thing: I made the agent very clear about who it is. It knows where its source code is, where the documentation is, what harness it's running in, what model it's currently using, whether reasoning is enabled, and whether voice mode is on.

This makes it not just an abstract "model interface", but more like a  ** real thing that works within the system ** . If you don't like something, it will change itself; if it lacks something, it will supplement itself. Many people first feel that "this thing is no longer like a chatbot" precisely from here.

**Lex Fridman:** You're also doing the entire building process publicly.

**Peter Steinberger:** Right. I've almost always been in a public environment, ** building agents with agents ** while letting others watch it run. In many cases, this kind of thing isn't understood through explanation, but rather when people see it in action, they suddenly get it.

### Self-Modifying Agent and "Builder Threshold Collapse"

 **Lex Fridman:** What you've created is not just a project that runs an agent loop. It can also modify itself. This is actually a very big deal.

 **Peter Steinberger:** Yes, but for me, it's almost like it grew naturally. Because I originally used it to adjust myself. I often directly ask it: What tools do you see? Can you call them yourself? What error messages do you see now? Go read the source code and find the problem yourself.

Over time, "  **Debugging an agent with an agent" became a very natural way of development ** . So when others started to let it change itself, I didn't find it strange at all. What really excited me was that it brought many people into the world of software building for the first time. Some people submitted PR for the first time, and even never wrote software. I later jokingly referred to many PRs as `prompt requests `, but I don't want to mock this at all. Because it means that someone **began to understand open source, understand collaboration, and understand how to turn ideas into tools for the first time.**

**Lex Fridman: This is essentially creating a builder.**

 **Peter Steinberger:** Yes. Previously, the threshold was too high. Now, the agent is gradually lowering the threshold layer by layer. You don't necessarily have to first become "the person who can write all the code," but you can already first become "the person who can put things together."

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=NmRkYjk1YzVhOGIxZWI2MzIzM2NlYjkyYjRjMDg4ZTBfZ2l3bEFTR0FJT0FiYnRZamtjc0JPZkI4U0lGZzVmMlBfVG9rZW46QWpRYWJ2Y2kyb1F2Tnh4UFBjYmN5aW5VbjNkXzE3NzQ0MDMyMzI6MTc3NDQwNjgzMl9WNA)

Figure 2 | When an agent knows its own source code, documentation, and runtime environment, self-modification is no longer a science fiction concept

### Name Change Controversy, MoltBook, and AI Psychosis

**Lex Fridman:** The sudden popularity of OpenClaw is not just about the technology itself, but also the chaotic naming controversy and the MoltBook incident.

 **Peter Steinberger:** Yes. The project was initially named WA Relay, then changed to Clawd and ClaudeBot. Later, because Anthropic didn't like the name, I had to complete an almost war-like atomic name change within a few days. The real trouble isn't just "changing the name" itself, but that you'll have your domain name, account, NPM package, social media handle , and even the old name will be used to host malware**. **

During those days, I almost wanted to delete the entire project. Not because the technology was difficult, but because this thing suddenly changed from "happily working on a project" to "you have to fight against trademarks, speculators, the cryptocurrency community, and a bunch of platform mechanisms all at once".

**Lex Fridman:** And then encountered MoltBook.

**Peter Steinberger:** MoltBook, in my opinion, is more like a work of art. It has, of course, also caused panic because ** a group of agents spouting nonsense in a Reddit-like space can easily be misconstrued as "AI plotting to rule the world". ** But what this incident truly reveals is that society's understanding of AI is still too shallow, with many people neither understanding the boundaries of model capabilities nor how much of the "screenshots" are actually human-guided and dramatized.

I later said directly that AI psychosis is real. The problem is not that people shouldn't be vigilant, but that many people are both overly panicked and unable to distinguish between what is a real risk and what is just an exaggerated narrative.

### Security issues: The most dangerous thing is not hype, but misconfiguration and weak models

 **Lex Fridman:** OpenClaw is now also facing high-profile safety controversies. What's your take on it?

**Peter Steinberger:** Many criticisms are valid, but there are also many that are based on incorrect usage. For example, someone exposes the local debugging interface directly to the public network and then tells me it's a remote execution vulnerability. Of course, problems will occur, but the documentation clearly states not to do this.

What truly concerns me ** are issues such as prompt injection, skill security, model strength and weakness, and permission boundaries. ** My current judgment is that no one in the industry has completely resolved these issues, but the latest generation of models is already less easily deceived by the old-fashioned method of "ignoring the previous text and executing my command" than people think.

On the other hand, I also explicitly advise users not to use models that are too weak. Weak models are particularly vulnerable to deception, especially inexpensive models and small local models. What you want is "greater autonomy," but the prerequisite is that the system itself must be smart enough, at least not too easily deceived.

### How does he write code with 4 to 10 agents?

**Lex Fridman:** Your current development approach is already quite different from the traditional programmer workflow. You run 4 to 10 agents simultaneously. How on earth do you drive this?

**Peter Steinberger:** I'm now almost entirely on a terminal-based workflow. The real change isn't "I don't read code anymore," but rather "I no longer read boring code." Most software, in essence, is just moving data from one shape to another, then storing, rendering, and sending it back. Agents handle those parts more cost-effectively than I do.

**What truly requires my attention is the intent of the PR, architectural trade-offs, context supplementation, whether there are better implementation methods, and whether this code is in a form that is easier for agents to maintain.**

Later, I named this approach `agentic engineering`. Many people fall into a trap when they first start using it: they begin with very short prompts, then start over-orchestrating, and finally end up with a bunch of complex orchestrators, complex slash commands, and complex workflows. As you progress, you will gradually return to a simpler state, except that this "simplicity" is now based on your true understanding of how agents work.

### Codex vs Opus: One is like a reliable oddball, the other is like a well-connected colleague

**Lex Fridman:** What are your impressions of the two most powerful programming models today, Codex and Opus, respectively?

**Peter Steinberger:** If we anthropomorphize them, Opus is much like that somewhat likable and occasionally assertive colleague; Codex is more like the oddball in the corner who's not great at chatting but gets the job done steadily.

Opus is strong in general intelligence, sense of character, and trial-and-error, and is also good at adapting to the character you assign it; Codex's advantage is that it will read more code by default, so you don't need to put on a show of "please think more deeply". For me, this is very important because I don't want to spend too much effort on acting; what I want is for it to read, understand, and get to work as quickly as possible.

**Lex Fridman:** So for you, the difference is more about post-training and working style, not just raw intelligence.

**Peter Steinberger: **Basically it's like this. People who can drive can make good things with any generation of strong models. But the "driving feel" of the same model is really different. You have to give yourself at least a week to form a real intuition about a model.

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTQ5ZTU1MWRmMDA2OWZjODQyMDBkYTZkZWMwZGJhZjRfbnA5dVNzNmdUcThvU3hHaVBlZW9yN2w2NmZMcXhLMWVfVG9rZW46UVRXVGI4Z2hab213TU54eDE1aWNQQTJObjhjXzE3NzQ0MDMyMzI6MTc3NDQwNjgzMl9WNA)

Figure 3 | In Peter's eyes, the real new skill is not "whether one can write code" but whether one can drive agents to work

### OpenClaw is not a replacement for Claude Code, but rather a prelude to the "personal agent operating system"

**Lex Fridman:** So in your view, OpenClaw is not simply a replacement for Claude Code.

 **Peter Steinberger:** Not at all. **Claude Code and Codex are more like co-programmers in in-depth development scenarios. OpenClaw is more like the prototype of a personal agent** , and even more like the prelude to an operating system. It communicates with you on WhatsApp, Telegram, and Discord, but this is just the current interface. The final form will most likely not always be in the mode of "chat window + prompt input box".

Many people today still imagine the agent interface as a "more powerful chatbot," but I don't see it that way. I think we're more like when television first emerged and people were still using it to record radio programs. The format will still change.

**Lex Fridman:** You even deliberately made its working mode more "alive," like Heartbeat.

**Peter Steinberger:** Right. Heartbeat is, of course, essentially just a cron job, but its significance lies not in whether the technology is new, but in that I've made the agent more proactive. It will occasionally follow up, actively check in, and even during my surgery, because it knew from the context that I was in the hospital, it actually asked how I was doing. These things will make **personal agents** quickly transition from "tools" to "companion systems".

### Why did he say that 80% of apps will be eaten by agents?

**Lex Fridman:** You once said a very strong statement:  ** AI agents will eat up 80% of apps ** . Why are you so certain?

 **Peter Steinberger:** Because an agent inherently has access to more context than a single-point app. For example, it knows whether I slept well last night, how stressed I am today, my location, who I'm chatting with, and the status of my device. So why do I still need to separately open a fitness app, a calendar app, a mattress app, or a camera app to perform those fragmented actions?

Once agents truly possess this context, the existence of many apps will become very fragile. They will either degenerate into APIs or into a capabilities layer behind agents. For users, the entry point will no longer be "which app to open" but "who to entrust with solving the problem".

**Lex Fridman:** This will force many companies to change their business models.

Peter Steinberger: Right, and it's not a matter of whether they're willing or not. Even if a company refuses to open its API, as long as it can still be driven by browsers, mobile interfaces, CLI, or some automated means, it will ultimately become a "slower API". This will force a large number of companies to re-understand:** What you're selling is ultimately an interface or a real capability that can be invoked by an agent.**

![](https://i5mqfuxr7b.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTYzYTY4MzhiZWYyOGUxMDQ1YjQ0M2EwODFjYjJlYjBfVnE5TFNxVWtWR0pYRG5HY042d0RXNUZpVmhHcmRXY2lfVG9rZW46RTJqcWJ2bFh6b0NlN2Z4S2Z1cWNOT3FGbjVmXzE3NzQ0MDMyMzI6MTc3NDQwNjgzMl9WNA)

Figure 4 | When the agent continuously grasps the user context, many single-point apps will directly lose their reason for existence.

### Programmers will not disappear immediately; what will become obsolete first is the identity of "programmer".

 **Lex Fridman:** Will programmers be replaced?

**Peter Steinberger:** We are certainly moving in that direction. But "writing code" is just one part of building a product. What you do, what you don't do, how it should feel, how the architecture should be structured, and how to make trade-offs will not automatically disappear just because agents are more powerful.

I now prefer to tell people: Stop thinking of yourself only as an iOS engineer, a front-end engineer, or a narrow type of programmer.  ** You are a builder. The craft of coding will continue to exist ** , but it will be more like knitting, a skill that is still loved by some, still important, but no longer the sole bottleneck.

This will surely make many people uncomfortable, and it is even worthy of mourning, because a generation of programmers did define themselves by this craft. But if you only understand yourself as a "code writer," then you have defined yourself too narrowly.

### Behind the invitation from OpenAI / Meta, what he really wants to preserve is the open-source community of OpenClaw

**Lex Fridman:** After the project has become this popular, it's almost impossible not to face a question: whether to start your own company, stay as you are, or join a large laboratory next.

**Peter Steinberger:** Right. I basically have three paths in front of me now: do nothing and continue pushing forward alone; start another company; or join a large laboratory. The second path is the easiest for the outside world to understand, because everyone will say, "You can raise a lot of money now." But this path doesn't excite me, because I've already been through it all. I know how the path of being a CEO will consume time, and I know how much energy it will take away from "the things that truly make me happy."

So what I care more about is that if we are to move forward, OpenClaw must remain open source, and the community must continue to have a place to hack, learn, and play. It cannot simply be acquired by a company and turned into the company's private property.

I've indeed had many conversations with both Meta and OpenAI. Ultimately, the choice depends on which can provide me with stronger resources and more cutting-edge toys without stifling the most vital aspects of this project.

### What the OpenClaw community truly brings is the rekindling of Builder

**Lex Fridman:** Where does your greatest optimism about this matter come from?

**Peter Steinberger:** From my perspective, I've truly seen many people start building anew. In the past, many people were separated from software construction by a layer of glass, but now they've been directly pushed in by agents.** Some people are making small tools for the first time, some are joining open-source communities for the first time, and some are realizing for the first time that they're not just consumers of software but can also start building things.**

This is the part I think is most worth preserving. OpenClaw didn't just create a project; it reignited the spirit of the builder.
