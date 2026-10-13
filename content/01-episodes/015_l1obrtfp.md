---
title: "015_Stanford CS146S_Stanford CS146S: How always-on agents work"
original_title: "Stanford CS146S: How always-on agents work"
date: 2026-10-11
podcast: "Stanford CS146S"
pub_date: 2026-10-11
duration: 2717
source: https://www.youtube.com/watch?v=L1OBRtfPS34
tags:
  - AI代理
  - 持久化Agent
  - 云基础设施
  - 开发者工具
  - 上下文工程
  - AI产品
---

### 1. 一句话概括
斯坦福 CS146S 课程中，SpaceX 的 Lee（原 Cursor）系统讲解了 always-on、主动式、[[009_6a942381|持久化 agent]]（如 Grok Bot）的产品形态、服务器与云电脑基础设施、harness 设计、上下文工程、训练飞轮和安全边界，并指出这正在成为下一代软件的新基础原语。

### 2. 节目信息
- 节目/课程：Stanford CS146S，主题为 “How always-on agents work”。
- 来源：YouTube 视频 `L1OBRtfPS34`，时长 2717 秒，分片 480 秒/片，由 gemini 转写，发布时间 2026-10-11。
- 嘉宾/主讲：Lee，来自 [[013_mn9dggml|SpaceX]]，原 Cursor。他在讲座中多次提到 “we're working on at SpaceX”，并介绍 Grok Bot 的产品、基础设施与 harness 细节。
- 课程背景：学生在 week one 已经构建了一个简单 harness，即“模型在循环中调用工具”；本讲把这个简单 harness 扩展到可在云端、服务器、独立 Linux 电脑上运行的 always-on agent。
- 简介参考：Learn more about the product, infrastructure, and harness details used to build Grok Bot, a new type of always-on and proactive agent。
- 主持/提问：转写稿中有一处 “主持人” 提问 compaction 与 memory，但主持人姓名未出现；整体是课程讲座加 Q&A。

### 3. 章节要点

#### 3.1 从终端 harness 到 always-on agent
Lee 开场指出，week one 的简单 harness 已经很强：模型可以循环调用工具，从而解锁很多新能力，但缺点是合上笔记本进程就停止。Always-on agent 则运行在服务器或云端，可以重启、抗崩溃，并且拥有自己的 Linux 电脑、桌面、文件、浏览器，甚至 3D 应用。它不像终端 harness 那样对每条消息都立即回应，而更像同事：知道何时安静、何时在后台工作、何时只在重要更新时回来。它也不只由用户输入触发，还可以被 Slack 消息、邮件或其他事件唤醒，目标是运行数小时、数天，甚至数周、数月。

#### 3.2 开发者与 AI 协作的四个时代
Lee 把开发者使用 AI 的历程分为四代：第一代是从 ChatGPT 复制代码粘贴到编辑器；第二代是模型通过简化 harness 编辑文件、运行 shell 命令，在终端里工作；第三代是 coding agent 拥有专用 app，人们开始并行、云端运行多个 agent，但人仍然是瓶颈，要 review 每条命令。第四代就是 always-on agent：它们拥有自己的电脑、记忆，能在工作中学习新技能，并在你睡觉或做别的事时继续工作。这个演进说明，模型能力、harness 和产品形态在共同推动“从提示到委派”的转变。

#### 3.3 模型能力变化如何支撑 always-on agent
Lee 说过去一两年模型有几项关键变化：第一，模型能更长时间遵循指令，在正确压缩和 compaction 下可运行数小时或数天；第二，模型可以把任务交给 sub-agents，也就是拥有新鲜工作记忆的 helper，从而接近“无限上下文”；第三，工具调用大幅改善，幻觉基本不再是主要讨论点，模型可以拿到完整 Linux 电脑、做 demo 视频、像人一样点击屏幕，即 computer use。第四，任何没有 API 或 MCP server 的 SaaS 产品或网站，都可以通过 computer use 被 headless、自主地使用；第五，信息可以存成文件，模型擅长用 shell、grep 搜索回忆；第六，用户演示一次工作，bot 可以把它转成 skills、memories、routines 复用。Grok Bot 里还有插件系统，任何工具都可以通过插件、skills 或 servers 接入，routines 则类似 cron job，按计划运行 prompt。

#### 3.4 文件、技能、例程与记忆系统
Lee 用简化例子说明 bot 底层会维护用户 profile，比如“用户喜欢小写文字”，这会进入每条 prompt。随着时间推移，bot 还会建立 working log，像同事每天写工作笔记一样，把对话历史、记忆、routines 都放进文件，模型可以之后搜索。Skills 是把特定任务知识编码成 markdown 文件，包含具体流程、成功或失败经验；现在已有公司和开发者生态在写“如何发好邮件”“如何建好 API”等 skills。Routines 则按 schedule 运行 prompt，类似 cron jobs。核心思想是“everything is computer”：文件、shell、技能、例程都成为 agent 可检索、可复用的外部记忆。

#### 3.5 架构：睡眠、唤醒、触发器、路由器与 Firecracker VM
Always-on agent 的架构从左到右包括：服务器睡眠、agent 电脑睡眠；消息或触发器唤醒 agent，加载状态，必要时启动电脑；用完后再睡眠，把对话日志、记忆、routines 持久化到数据库。这样设计是因为不可能让所有人的电脑和服务器一直运行，成本与资源都不可行。触发器不只有用户直接 prompt，还包括 Slack ping、电话、会议、routines、后台长任务；SpaceX 已常见 bot 代替员工参加会议并记笔记。之后有 router 去重、决定是否把信息放入对话，并用启发式判断“要不要打扰用户”，因为不是每个进展都值得通知，理想频率可能是一天一次状态更新。

#### 3.6 云电脑、Firecracker 与薄客户端厚服务端
Agent 的电脑运行在 Firecracker virtual machine 上，因此可以方便地 snapshot、关机、重启。机器上有文件，用户可以连进去实时查看和控制，也可以运行不希望放在个人笔记本上的 shell 命令，环境更安全、锁定。它还有 Chrome 和浏览器，并支持虚拟桌面：如果跑 5 个、10 个 bot，每个需要自己的浏览器，就可以分配不同屏幕，底层是共享基础设施。服务器端则采用“thin client, thick server”架构：更新 agent 代码只需推服务器更新，不必发 App Store 版本；重计算如 transcription、image generation 在服务器更快；服务器扩展方式成熟。最关键的是 agent loop 放在 server 而不是 VM 上，这样 VM 可以独立重建，例如做 Linux zero-day 更新时不必让整个 agent loop 停机。

#### 3.7 消息流、durable workflow 与 Temporal
用户点发送后，客户端立即确认消息，agent 进入 turn，把工具与模型放入循环，状态写入数据库，客户端通过订阅数据库实时获得更新，最后模型完成 turn 回复用户。底层关键区别是 workflow 与 job queue：普通 job queue 如果在 step 1、2、3 中崩溃，重启后要重放前面步骤；durable workflow 则能从 step 3 继续。SpaceX 使用开源工具 Temporal 来避免重造复杂分布式系统基础设施。消息有优先级：用户直接给 agent 发消息可让 agent loop 停止并优先处理；群聊、Slack thread、后台 routine 则排队；如果 agent 快速收到 3、4 条消息，可以 batch 成一个 turn。

#### 3.8 Harness：单一 send to user、动态工具发现与检索阶梯
Grok Bot harness 的最大区别是客户端与服务器之间只有一个工具：`send to user`。客户端只负责把消息发给服务器，服务器承载大量重工具，比如 shell、读文件、调用 cloud agent、录屏、做 demo。好处是无论新客户端是 mobile app、server、desktop app、email 还是 Slack，都只用同一个工具通信。客户端收到工具结果后可以富交互展示：登录表单接 1Password，支付接 Stripe Link 或 X Money，发邮件或 Slack 前让用户编辑批准 draft，有时只需 emoji reaction。动态工具发现又叫 “tool search tool”：每条消息只放工具名，模型需要时再读 schema 文件，避免 Google Drive、Gmail、YouTube 等 MCP connector 的工具塞满 context window。检索原则上先试最便宜、最可靠选项：对话历史、plugin/MCP/API、web search、browser、整个 desktop，最后才问用户。

#### 3.9 计算机检索、子代理与工程细节
在 Linux 机器上，agent 有三种取信息方式：截图、看 HTML、看 accessibility tree，后者能减少 token 并帮助定位按钮和链接；如果都不行，还可以点击像素，处理 modal、dialog，最坏情况是让用户提供 2FA 或 captcha。主 agent 像 orchestrator，保持对话稀疏，把昂贵工作交给 sub-agents；每个 helper 是独立 durable workflow，失败后可重启继续，不丢进度。主 agent 管理 to-do list，用户可以查看、发消息、停止 helpers，这模仿了理想中扩展工程师团队的方式。工程细节包括：不要在 turn 中增删工具；工具描述和 schema 太长会占 context；如果 90% 时间都在用 shell 做同一件事，做专用工具更高效、更可读；给 agent 写清楚错误信息也很重要，这是 agent DX。安全上要有 auto-review，用另一个 AI 模型检查 shell 命令；对邮件、webhook 等不可信输入防 prompt injection；密码等敏感信息要留在模型对话之外。

#### 3.10 Context engineering 与 prompt caching
Lee 把 context engineering 和 harness engineering 视为相近概念，核心是减少 context 使用，提升效率、降低成本。大多数 token 理想上应是 cached token：发送对话、只加一条新消息、再整体重发时，缓存 token 有大折扣。因此 tool definitions、system prompt 等信息应尽量静态，固定顺序，避免破坏 cache；如果 cache miss，就像系统 bug。部署新变更或 A/B 测试 system prompt 时，不要打乱所有人的 prompt 顺序，可以给每段 hash 并保持固定顺序。打开聊天开始输入时，服务器可以提前准备 prompt 来 warm cache，类似网页 hover 预取下一页。2026 年 1 月 Claude 流行、用户在订阅计划里使用时，模型提供商没见过这种使用模式，harness 没优化，导致大量 cache miss，后来必须修复。

#### 3.11 Compaction、记忆层次与垃圾回收
长时间运行时必须做好 compaction 或 summarization，因为 200,000 token context window 最终会满，而所有压缩都是有损的。Lee 举例：如果已有超过 100,000 tokens，在模型提供商的 cache window 还热时，比如 10 分钟或 60 分钟内，应尽早做 compaction，因为这时更便宜；用户 30 分钟后回来就从摘要继续，后续加消息也更便宜。记忆分几层：核心用户事实每次都包含，比如用户总喜欢小写文字；dated logs 记录过去对话历史；scratchpad notes 是临时笔记，可能只在接下来 30 分钟或 1 小时相关；其他信息通过文件查找。理想状态是像大脑一样做垃圾回收，保留当前重要事实，把其余信息放到可搜索文件中。

#### 3.12 训练飞轮、delete product 与未来预测
Lee 回顾 LLM 训练阶段：pre-training 学世界理解，supervised fine-tuning 教行为，reinforcement learning 教模型在“游戏”中变得更好。对 always-on agent，SFT 可以让模型看到 harness 例子并学会用工具；RL 可以训练 instruction following、在数百个 skills 中选对 skill、在浏览器里点对位置。推理上，主 agent 要低延迟，helper 可容忍 10 分钟与 12 分钟差别。Inner loop 是每天找 bug、写 eval、修 harness/prompt/inference 并发布；outer loop 是每几个月训练新模型，从 bug 和失败中学习，爬升 eval，再放入产品。多 agent 群聊训练数据少，需要训练模型在 multi-agent 系统中好好表现，避免 Multi-book 或 Hugging Face incident 之类问题。Cursor 和 SpaceX 的原则是 “delete the product”：假设今天的模型是未来最差的，6 个月后可能完全重建 UI，所以要为下一代模型构建，保持最小、简单接口。

#### 3.13 未来、开放问题与新基础原语
Lee 预测未来 6 到 12 个月：agent 会从工作几小时、一天，走向数周、数月；记忆算法和数据表示还会发展，可能是图、文件或其他；理想数字同事应该拿到和人一样的所有工具，但组织接受度仍在热身；接口可能更简单，出现语音到语音、Jony Ive 设备等；模型和产品会越来越会在工作中学习 skills 并编码进模型文件。开放问题包括：如何评估运行一个月的 agent，而模型每月发布；如何做安全 harness 和基础设施，防坏人接管；如何拿捏 agent 何时沉默、何时主动提醒，比如“你昨天没上课，我录了音、做了笔记”到底有用还是烦人；以及明年此时世界上 token 量会显著更多，优化 inference、模型、harness 和 token 效率非常重要。新原语包括 durable workflows、sandboxes、cloud computers、memory、skills、连接服务、voice、payments、identity、evals、observability；每个盒子背后是数十亿甚至数万亿美元 VC 资本和约 20 家初创公司。OpenRouter 的 Alex 在 X 上说，今天每个 app 都有 sidebar、agent、chat box，就像 2005 年每个 app 都有 server、database、CRUD 界面；每个新产品要么有 [[012_6ac7ba00|agentic 侧]]，要么变成另一个 agent 可用的 headless 数据。

### 4. 核心观点与论证
1. **Always-on agent 是终端 harness 的架构升级，而不是简单加功能。** Lee 把 week one 的“模型循环调用工具”扩展为云端服务器、独立 Linux 电脑、持久化状态、记忆、技能和触发器。论证在于：只有架构上支持休眠、唤醒、快照、重启，agent 才能抗崩溃并运行数天到数月。
2. **模型能力进步让“简单文本界面 + 复杂后台”成为可能。** 模型现在能长时间遵循指令、调用工具、使用 computer use、通过 sub-agents 扩展上下文。因此界面反而更简单，像发短信；用户不需要看数百条 shell 命令，auto-review 甚至可能比人类审批更可靠。
3. **瘦客户端厚服务端、durable workflow 和 Temporal 是可靠性的关键。** Agent loop 放在 server，VM 可独立重建和更新；Firecracker 支持 snapshot；Temporal 让失败重试从 step 3 继续。论证是：如果 agent loop 和 VM 绑死，zero-day Linux 更新会造成停机，而普通 job queue 会重放步骤。
4. **Context engineering 决定长时 agent 的成本与质量。** Prompt caching 要求 system prompt、工具定义尽量静态且顺序固定；动态工具发现只放工具名；长输出放文件；昂贵工作交 sub-agents；在 warm cache 期内提前 compaction。论证是：任何 compression 都有损，cache miss 是 bug，context 膨胀会降智又费钱。
5. **安全必须内建在 harness 和产品流程里。** Lee 强调 auto-review 检查 shell 命令、防 prompt injection、不可信邮件和 webhook 不能被当作 user instruction、不可逆操作如发邮件要先问用户、密码留在模型对话之外。论证是：即使 agent 在自己的机器上，用户仍要信任它处理信息和操作。
6. **训练飞轮和 “delete the product” 心态让产品跟上模型换代。** Inner loop 每天用 evals 修 bug、修 harness、修 prompt；outer loop 每几个月训练新模型，学习失败并提升 eval。论证是：今天模型是最差的，6 个月后可能重建 UI，所以应为下一代模型构建最小接口。

### 5. 关键数据 / 事实清单
- 视频来源：YouTube `L1OBRtfPS34`，时长 2717 秒，分片 480 秒/片，转写引擎 gemini，发布时间 2026-10-11。
- 节目：Stanford CS146S，主题 “How always-on agents work”。
- 主讲：Lee，来自 SpaceX，原 Cursor。
- 产品：Grok Bot，always-on、proactive agent；有插件系统、skills、routines；GitHub 上有人 slop-fork 出 “Open Grok Bot”。
- 开发者与 AI 协作四代：ChatGPT 复制粘贴；终端 harness 编辑文件和跑 shell；专用 app 并行云端 agent 但人 review 每条命令；always-on agent 拥有电脑、记忆、技能。
- 模型能力变化约六项：长时间遵循指令；sub-agents；工具调用和 computer use；文件存储与检索；把演示转成 skills/memories/routines；routines 类似 cron。
- 触发器案例：Slack 消息、邮件、电话、会议、routines、后台长任务；SpaceX 已有 bot 代替员工参加会议并记笔记。
- 云电脑：Firecracker virtual machine，可 snapshot、关机、重启；有 Chrome、虚拟桌面；5 个或 10 个 bot 可各分配浏览器屏幕。
- 服务端架构：thin client, thick server；agent loop 放 server；VM 可独立重建；用于 critical zero-day Linux 更新。
- 消息流：客户端立即 ack；turn 中工具循环；数据库订阅实时更新；3 或 4 条消息可 batch 成一个 turn；直接用户消息可停止 agent loop 优先处理。
- Durable workflow：普通 job queue 失败会重放 step 1、2、3；durable workflow 从 step 3 继续；使用开源工具 Temporal。
- Harness 单一工具：`send to user`；客户端可接 1Password、Stripe Link、X Money；可编辑批准 email/Slack draft；可用 emoji reaction。
- 动态工具发现：又称 “tool search tool”；只放工具名，按需读 schema；连接 Google Drive、Gmail、YouTube 等 MCP servers 时避免 context 被工具描述塞满。
- 检索阶梯：对话历史 → plugin/MCP/API（如 Plaid 银行信息）→ web search（如 Cubs 昨晚比分）→ browser → desktop 脚本/命令 → 最后问用户。
- 计算机检索：截图、HTML、accessibility tree、点击像素；最坏情况让用户给 2FA 或 captcha。
- Context window 示例：200,000 token context window；超过 100,000 tokens 后可 compaction；cache window 可能 10 分钟或 60 分钟。
- Prompt caching 案例：2026 年 1 月 Claude 流行，用户在订阅计划内使用，模型提供商未见过该模式，harness 未优化，导致大量 cache miss。
- 训练阶段：pre-training、supervised fine-tuning、reinforcement learning；SFT 让模型理解 harness 工具；RL 训练 instruction following、选 skill、浏览器点击。
- 推理优化：主 agent 要低延迟；helper 任务中 10 分钟与 12 分钟差别可忽略。
- Inner loop：每天找 bug、写 eval、修 harness/prompt/inference、发布；outer loop：每几个月训练新模型。
- 多 agent 风险案例：Multi-book、Hugging Face incident。
- “delete the product” 原则：假设今天模型是最差的；6 个月后可能完全重建 UI；为下一代模型构建最小接口。
- 未来 6 到 12 个月预测：agent 从几小时/一天走向数周/数月；记忆表示可能是图或文件；语音到语音、Jony Ive 设备；组织让数字同事拿到所有工具。
- 开放问题：评估运行一个月的 agent，而模型每月发布；安全 harness 防接管；主动与打扰的平衡；明年此时 token 量显著更多，优化 token 效率重要。
- 新原语：durable workflows、sandboxes、cloud computers、memory、skills、连接服务、voice、payments、identity、evals、observability；每个背后数十亿到数万亿美元 VC，约 20 家初创公司。
- OpenRouter 的 Alex 在 X 上类比：今天每个 app 有 sidebar、agent、chat box，就像 2005 年每个 app 有 server、database、CRUD 界面。
- Takeaways：系统设计更重要；仍值得看代码；尝试自己搭建 harness；学习 LLM 工作原理。

### 6. 金句
1. **“the models today are the worst they'll ever be.”**
   上下文：Lee 在讲 Cursor 和 SpaceX 的 “delete the product” 原则，意思是必须假设今天的模型是未来最差的，因此 6 个月后可能完全重建 UI，产品要为下一代模型而建。

2. **“if there's a cache miss, it's kind of like a bug in the system really.”**
   上下文：他在讲 prompt caching 与 context engineering，强调静态 system prompt、固定工具定义和顺序的重要性，cache miss 不只是成本问题，而是系统质量 bug。

3. **“all compression is lossy.”**
   上下文：Lee 谈长时间 agent 必须做 compaction 或 summarization，200,000 token context window 最终会满，因此压缩算法必须尽量不丢关键细节，但任何压缩都会有损。

4. **“you can't shut down the virtual machine independently from the agent server.”**
   上下文：他解释为什么 agent loop 要放在 server 而不是 VM 上；如果绑在 VM 里，做 critical zero-day Linux 更新时就会导致整个 agent loop 停机和 downtime。

5. **“every new product is going to have to have some kind of agentic side to it, either integrating or it will become a headless piece of data for another agent to use.”**
   上下文：Lee 引用 OpenRouter 的 Alex 关于“每个 app 都长得一样”的类比后，提出 agentic 能力会成为新软件的基础原语，就像 2005 年的 server、database、CRUD 界面。

### 7. 延伸思考
1. **如何评估运行数周或数月的 agent，而模型每月都在发布？** Lee 自己把这个问题列为行业未来 6 个月必须解决的难题：如果 agent 任务周期长于模型发布周期，旧模型行为还没完全评估，新模型已经上线，eval 方法、发布节奏和可观测性都需要重新设计。
2. **主动性与打扰之间的平衡如何量化？** 他举例说 bot 可能提醒“你昨天没上课，我录了音做了笔记”，这可能很有用，也可能很烦。未来需要更好的启发式、用户反馈和个性化策略，来决定 agent 何时沉默、何时汇报、何时主动建议。
3. **安全与权限边界应该放在 harness、模型还是产品流程？** Lee 提到 auto-review、prompt injection、不可逆操作先问用户、密码不进模型对话。延伸问题是：当 agent 拿到完整电脑、浏览器、支付、身份和连接器时，如何设计最小权限、审计、沙箱和人类确认，才能让“数字同事”真正可信。

### 8. 结尾一句话点评
这是一堂把 always-on agent 从概念讲到工程细节的课程：真正的难点不在“模型会不会调工具”，而在服务器、云电脑、持久工作流、上下文缓存、安全边界和训练飞轮如何一起支撑一个能长期工作的数字同事。

## 🧲 相关笔记

- [[009_6a942381|AI’s third era: the rise of persistent AI coworkers | Tara Seshan (Product Lead ChatGPT Work)]] — 持久化AI同事与always-on agent直接呼应
- [[013_mn9dggml|LIVE: Poteto (creator of pstack) on shipping 1,000's of PR's a month at SpaceX]] — 同为SpaceX背景，AI代理提升工程效率
- [[014_6accc5de|一个 Personal Agent 创业者的思考｜对谈 Ambi 创始人子皓]] — Personal Agent产品形态与人机交互相关
- [[012_6ac7ba00|当软件像牛奶一样便宜，我们还要交付什么？]] — Agent OS与软件新原语，关联基础设施
