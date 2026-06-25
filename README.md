# Vercel eve 框架 · 新手小白完整教程

> 这份教程带你用 **Vercel 的 eve 框架**，从一行代码都没有的状态出发，亲手做出一个**能 7×24 持久运行、可以正式上线**的 AI agent。我们尽量说人话：不堆术语、每一步都能照着敲、敲完就有结果。你不需要是后端高手，只要会用命令行、能看懂一点 JavaScript/TypeScript 就够了。
>
> ⚠️ **特别说明**：eve 目前仍处于 **beta（公测）阶段**，接口、命令和目录结构可能随版本更新而变化。本教程讲的是思路和方法，**具体写法请始终以 [Vercel 官方文档](https://vercel.com) 为准**——当教程和官方文档不一致时，听官方文档的。

## 📌 适合谁读 / 预备知识 / 学完你能做到什么

**适合谁读**
- 完全没做过 AI agent、想找一条"从零到上线"完整路径的新手
- 会用 Cursor / Claude 之类工具写过点小脚本，但没真正部署过线上服务的人
- 想搞清楚 eve 里 tools、skills、connections 这些概念到底各管什么的开发者
- 已经在用其他 agent 框架，想快速对比、上手 eve 的同学

**预备知识（有最好，没有也能跟）**
- 会打开终端、敲基本命令（`cd`、`ls`、安装个东西）
- 能看懂基础的 JavaScript / TypeScript（函数、对象、`async/await`）
- 有一个能上网的电脑，以及一个邮箱用来注册账号
- 知道"环境变量""API key"大概是什么概念（不懂也没关系，正文会讲）

**学完你能做到什么**
- 在本地 5 分钟跑起属于你自己的第一个 agent
- 看懂 `agent/` 目录里每个文件/文件夹的职责，不再"看天书"
- 给 agent 加上工具（Tools）、操作手册（Skills）、外部连接（Connections）
- 让 agent 定时自动干活（Schedules）、对外提供入口并做好鉴权（Channels）
- 给 agent 写自动化测试（Evals），并最终把它**部署上线到 Vercel**
- 能独立读懂一个真实的 eve 项目，照葫芦画瓢做出自己的

## 🗺️ 目录

1. 认识 Vercel 与 eve：这是什么、为什么学
2. 准备工作：账号、Node、CLI 与模型访问
3. 5 分钟跑起你的第一个 agent
4. 核心概念地图：agent/ 目录里的每个角色
5. Tools：给 agent 一双手
6. Skills：按需加载的"操作手册"
7. Connections：让 agent 连接外部世界（MCP）
8. Channels：对外入口与鉴权（别把门敞开）
9. Schedules：让 agent 定时自己干活
10. Sandbox：模型的代码草稿纸
11. Evals：给 agent 写自动化测试
12. 部署上线：把 agent 发布到 Vercel
13. 实战拆解：读懂一个真实的 eve 项目（muse-agent）
14. 常见问题、排错与下一步

**附录 A** · 本地与私有化（自托管）部署

---

## 1. 认识 Vercel 与 eve：这是什么、为什么学

欢迎来到这本教程的第一章。在你写下任何一行代码之前，我们先把"地图"看清楚：你要去哪里、要用什么工具、为什么是这套工具。这一章不需要你打开终端，也不假设你写过任何 AI agent 或后端服务——读完它，你会对"我们到底在学什么、学完能做出什么东西"有一个清晰、踏实的预期。

> 💡 小贴士：如果你是那种"先动手再说"的人，也别跳过这一章。eve 的很多设计（比如"用文件名来定义功能"）一旦你提前理解了背后的理念，后面学起来会快得多、少踩很多坑。

### 1.1 先认识地基：Vercel 是什么

我们要学的框架叫 **eve**，它由一家叫 **Vercel** 的公司出品，并且专门跑在 Vercel 上。所以我们得先花一分钟认识 Vercel。

**Vercel 是一个云端部署平台。** 打个比方：你在自己电脑上写好了一个网站或一段程序，但全世界的用户没法访问你的电脑——你需要把它"搬"到一台 24 小时在线、有公网地址的服务器上，让别人通过一个网址就能用到。Vercel 干的就是这件事：你把代码交给它，它负责构建、部署、分发，最后给你一个像 `https://你的应用.vercel.app` 这样的网址。

Vercel 最有名的产品是 **Next.js**（一个做网站/网页应用的框架）。但对我们更重要的是 Vercel 的另一个能力：**Serverless Functions（无服务器函数）**，在 Vercel 里也叫 **Vercel Functions**。

这个名字有点唬人，我们拆开看：

- "Serverless（无服务器）" 不是说"没有服务器"，而是说**你不用自己去管服务器**。你不需要租一台机器、装系统、配置环境、操心它会不会崩、要不要扩容。你只管写一段"接到请求就执行"的代码，Vercel 帮你把它跑起来，有人访问时自动启动，没人访问时自动休眠。
- "Function（函数）" 是说：你的后端逻辑被组织成一个个"收到请求 → 处理 → 返回结果"的小单元。

> 💡 小贴士：你可以把 Vercel Functions 想象成"按需开灯"的房间——有人进来（请求来了）灯就亮（函数启动），人走了灯就灭（函数休眠），你不用为没人时空转的电费买单。这种"用多少算多少"的模式，正是后面理解 eve 计费和"暂停时不耗算力"的基础。

这里要顺带认识一个词：**冷启动（cold start）**。当一个函数休眠了一段时间后，第一次再被访问时，需要先"醒过来"——这个唤醒过程就叫冷启动，会比"已经醒着"时慢一点点。记住这个词，它在下一节会变得很关键。

### 1.2 主角登场：eve 是什么

现在主角来了。

**eve 是 Vercel 出品的一个开源框架，用来构建、运行和扩展"持久化的后端 AI agent"。** 它的官方一句话定位是："**The framework for building agents（构建 agent 的框架）**"。

我们逐个词拆解这句定位，因为每个词都很重要：

**先说"agent（智能体）"是什么。** 一个 AI agent，简单说就是一个"会自己干活的 AI 助手"。普通的聊天 AI 你问一句它答一句；而 agent 不只是回话，它能在一次任务里**自己调用工具**——比如查天气、读数据库、调用某个 API、写文件——然后根据工具返回的结果继续思考、继续行动，直到把任务完成。它有一个"想 → 用工具 → 看结果 → 再想"的循环（这个循环业内叫 **agent loop**）。eve 的一个核心卖点就是它**替你管好了这个循环**，你不用自己手写。

**再说"后端（backend）"。** 我们要做的不是一个跑在浏览器里的网页，而是一个跑在服务器上的服务——它有自己的网址，可以被 Slack 机器人、网页聊天框、定时任务、甚至一行 `curl` 命令来触发。它在云端默默干活。

**最后说"持久化（durable）"。** 这是 eve 最有特色、也最值得你期待的一点，我们留到 1.5 节专门讲透。

eve 给自己找了一个非常好懂的类比：

> **"就像 Next.js 之于网页应用，eve 之于 agent。"**

意思是：Next.js 出现之前，每个团队做网站都要把路由、构建、部署这些"管道工程"重新搭一遍；Next.js 把这些标准化了，让你专注于"我的网站要做什么"。eve 想为 agent 做同样的事——让你专注于"我的 agent 要做什么"，而不是反复搭建"让 agent 能在生产环境跑起来"的那一堆底层管道。

> 💡 小贴士：你会在 eve 的文档和本教程里反复看到一个词——"plumbing（管道/底层管道工程）"。它泛指那些"每个 agent 都需要、但跟你的业务无关、又烦又容易出错"的基础设施：会话怎么保存、模型请求怎么路由、密钥怎么管、代码怎么隔离执行……eve 的全部价值，几乎都可以归结为"帮你把这些 plumbing 一次性解决了"。

### 1.3 eve 为什么存在：它解决了一个真实的痛

了解一个工具"为什么被造出来"，往往比记住它的 API 更有助于理解它。

eve 的来历很接地气。Vercel 内部用 AI agent 做了几百个应用之后，他们发现了一个反复出现的问题，官方原话是：

> *"每个团队在他们的 agent 能干任何事之前，都在一遍遍地搭建同样的底层管道，而且这些东西从一个使用场景到下一个使用场景完全没法复用。"*

换句话说：写"agent 的核心逻辑"其实不难，难的是把它变成一个**能在生产环境稳定运行**的东西——要能保存会话、要能安全地跑模型生成的代码、要能管好各种第三方密钥、要能在崩溃后恢复……每个团队都在重复造这些轮子。

于是 eve 的核心主张是：

> *"构建一个 agent，应该意味着定义它要做什么，而不是去组装它在生产环境运行所需要的所有零件。"*

所以 eve 是 Vercel 分析了自己内部几百个 agent 的共同模式后提炼出来的框架——也是**"Vercel 自己用来构建和运行 agent 的那个框架"**。这一点值得你放心：它不是一个实验玩具，而是从真实生产中长出来的工具。

> ⚠️ 注意：eve 不跟你比"哪个模型更聪明""哪种 agent 设计更厉害"。它竞争的是**生产/运维这一层**——也就是"怎么让 agent 稳定、可靠、可控地在线上跑"。理解这个定位，能帮你判断什么问题该靠 eve 解决、什么问题该靠你自己的提示词和工具设计解决。

### 1.4 核心理念：用文件来"声明"一个 agent

这是 eve 最独特、也是你必须从第一天就建立的心智模型。eve 把自己描述为一个 **"文件系统优先（filesystem-first）"** 的框架。这是什么意思？

**它的核心原则是一句话："文件的名字和它在目录树里的位置，就是它的定义。"** 官方还有一句更直白的说法："**The files are the interface（这些文件就是接口）**"。

我们用一个例子立刻说清。假设你想给 agent 加一个"查天气"的能力（在 eve 里这种能力叫 **tool（工具）**）。你要做的是——在 `agent/tools/` 这个目录下，新建一个文件，**给它起名叫 `get_weather.ts`**：

```
agent/
└── tools/
    └── get_weather.ts      ← 这个文件就定义了一个名叫 get_weather 的工具
```

注意这里的关键：**这个工具的名字 `get_weather`，是从文件名来的**，不是你在代码里写 `name: "get_weather"` 写出来的。事实上，你在 eve 里**永远不会写 `name` 或 `id` 字段**——身份完全来自文件的路径。

这跟你可能见过的其他做法很不一样。很多框架要求你维护一个"注册表"：每加一个功能，都要去某个中心文件里登记一笔"我有这个工具、它叫什么、在哪"。eve 完全不需要这种登记。它的口号是：

> **"不需要单独的注册表——发现是自动的。"**

那么 eve 是怎么把这一堆文件变成一个能运行的 agent 的？它走的是一个三步流程，你可以记成 **发现 → 编译 → 运行（discover → compile → runtime）**：

1. **发现（Discover）**：在构建时，eve 自动扫描你的 `agent/` 目录，按目录名识别出各种能力（`tools/` 里是工具、`connections/` 里是外部服务连接，等等）。你放对了位置，它就找得到，无需任何额外配置。
2. **编译（Compile）**：eve 把这些能力校验一遍，编译出一份"清单（manifest）"，放进一个叫 `.eve/` 的目录里。这一步它还会把每个能力注册给模型，让模型"知道"自己有哪些本事。
3. **运行（Runtime）**：编译出来的成品，就是一个能部署的应用，跑在前面讲的 **Vercel Functions** 上。运行时，agent 通过标准的 agent loop 来调用工具、加载技能。

把这三步连起来看，你就能理解 eve 的那句精炼总结了——**"eve 通过接管 agent loop，把一个文件变成一种能力。"** 你负责"放文件"，eve 负责把文件变成"活的能力"并在云端跑起来。

> 💡 小贴士：因为身份来自文件路径，所以有一个超好用的命令叫 `eve info`——它会列出 eve 当前"发现"到的所有工具、技能、连接、定时任务、渠道、子 agent 等等。以后当你"明明加了个工具，agent 却用不上"时，第一反应就是跑 `eve info` 看看 eve 到底有没有发现它（很可能是文件放错了目录，或文件名不符合规则）。

> ⚠️ 注意：正因为"文件名就是定义"，文件名和它所在的目录就变得非常讲究，不能随便起。这在后面会有不少具体规则（比如连接（connections）的文件名要用短横线写法 dash-case），现在你只要先记住"文件名是有意义的、不是随便取的"这个意识就够了。

### 1.5 "持久化 / durable" 到底意味着什么

现在回到我们之前埋的那个词：**durable（持久化）**。eve 把它放在最显眼的位置——官方标语就是 "**Durable by default（默认就是持久化的）**"。这个特性，是 eve 区别于"随手搭个 AI 脚本"的最大分水岭，值得讲透。

我们先理解 eve 里的两个基本概念：

- **会话（session）**：一段"持久的对话或任务"。当某个渠道（比如一个 Slack 消息）或一个 HTTP 请求触发了你的 agent，就开启了一个 session。
- **轮次（turn）**：在一个 session 里，用户每发一条消息、或每来一个外部事件，就产生一个 turn。在一个 turn 中，agent 可以调用工具、加载技能、读写沙箱里的文件、把活儿交给子 agent，并把过程实时流式地推送给客户端。

那"持久化"的魔法在哪？看官方这段话（很关键，建议读两遍）：

> *"eve 的会话运行在 Vercel Workflows 之上。Workflows 会把进度作为一份事件日志持久保存下来，并通过确定性地重放这份日志来重建状态——因此一个会话可以在冷启动、重新部署、以及长时间暂停（等待下一条消息或某个工具结果）之后存活下来。"*

把它翻译成大白话：**eve 不是把会话状态记在内存里（那样一断电、一重启、一冷启动就全没了），而是把"发生过的每一步"都记成一本流水账（事件日志），存到可靠的地方。** 任何时候需要恢复，它就照着这本流水账"重放"一遍，把状态精确地重建回来。

这在实践中意味着三件非常实在的好事：

1. **能扛冷启动。** 还记得 1.1 节说的冷启动吗？普通脚本一旦函数休眠、内存清空，进行中的任务就丢了。eve 不会——会话状态在流水账里躺着，函数重新醒来后照样能接着干。
2. **能扛重新部署。** 这条特别惊艳：官方明确说，**"一个正在执行任务的会话，如果你这时推送了新版本，它会在它启动时所在的那个版本上跑完。"** 也就是说，你部署新代码，不会打断正在进行中的任务——它会用旧版本善始善终。
3. **能扛长时间暂停。** agent 可以停在某一步，等一个人来审批、或等下一条消息、或等一个外部工具结果——**暂停多久都行**。而且这里有个对钱包很友好的细节：一个停在"等待审批"状态的会话，**会一直停在那里等，哪怕等到天荒地老，也不消耗任何算力。** 这正是 1.1 节"按需开灯"模式的好处在 agent 世界里的体现。

> 💡 小贴士：这种"每一步都记账、可以精确重放"的模式还有一个副产品——**回滚很安全**。借助 Vercel 原生的即时回滚能力，出了问题可以一键切回旧版本。

> ⚠️ 注意（一个真实的坑）：因为 eve 靠"重放流水账"来恢复，所以有一条铁律——**已经成功跑完的步骤不会在同一个会话里重新执行**，eve 会直接重放那一步记录下来的结果。但是！一个**执行到一半被中断**的步骤，恢复时**会重新跑一遍**。这意味着：如果你的某一步有"非幂等的副作用"——比如扣款、发邮件——它有可能被执行两次。eve 的官方建议是：把这类操作设计成幂等的，或者用"审批门（approval）"把它挡住。这个细节你现在只要有个印象，等真正写到这类工具时会再展开。

### 1.6 底层四大支柱

eve 的"持久化"和那一堆它替你解决的"管道工程"，并不是凭空变出来的——它们建立在 Vercel 的四个底层系统之上。你现在不需要会用它们，但需要知道它们各自管什么，因为这四个名字会贯穿整本教程。

我们用一个比喻把它们串起来：把你的 agent 想象成一个**新入职的、能干活的员工**——

| 支柱 | 它管什么 | 比喻 |
|---|---|---|
| **Vercel Workflows** | 持久化会话状态：把进度记成事件日志并重放，让会话能在冷启动、重新部署、长时间暂停后恢复 | 员工的**记事本+记忆**，让他随时被打断都能接着干 |
| **Vercel Sandbox** | 隔离地执行代码：把 agent 生成的代码"彻底关在你的应用运行时之外" | 给员工一间**带门的隔离实验室**，让他在里面捣鼓，不会波及主办公区 |
| **AI Gateway** | 路由模型请求 + 处理供应商故障转移（fallback） | 员工和各家"大脑供应商"之间的**总机/调度台**，一个不通自动转另一个 |
| **Vercel Connect** | 管理外部服务的 OAuth 令牌和 API 密钥；带"交互式 OAuth、同意授权、令牌自动刷新" | 帮员工保管各个系统**门禁卡和钥匙**的钥匙管理员 |

逐个稍微展开一点：

- **Vercel Workflows** 就是 1.5 节那本"流水账"的引擎，是"持久化"的真正来源。它基于 Vercel 开源的 Workflow SDK。
- **Vercel Sandbox（沙箱）** 解决一个真实的安全焦虑：AI 有时会生成代码并想运行它。你肯定不希望这些代码直接在你的应用主进程里乱跑。沙箱给它一个隔离的、一次性的执行环境。这里有一个**特别重要、初学者极容易搞混的区别**，请现在就记住：**你写的工具（tool）代码，跑在你的"应用运行时"里**（能读环境变量、能用你的共享代码）；而**沙箱是另一回事，是模型自己捣鼓代码的"草稿间"**。两者不是一个地方。这个区别后面讲工具和沙箱时会反复强调。
- **AI Gateway** 让你用一个统一的字符串（形如 `"anthropic/claude-opus-4.8"`）来指定模型，由它负责把请求路由到对应的供应商，并在某家出问题时自动重试/转移。它的另一个大好处是：在 Vercel 上，模型请求靠 OIDC 身份认证，**你甚至不用自己管供应商的 API key**。
- **Vercel Connect** 负责跟外部服务（比如 Linear、Slack、你公司的某个内部 API）打交道时的认证。它能处理"用户点击同意授权"这种交互式 OAuth 流程，把令牌加密存好，到期了还自动帮你刷新。

还有一个你会经常用到的第五个系统——**Vercel Observability（可观测性）**，它负责把 agent 的每一次运行、token 用量、性能表现都展示出来（你在 eve 项目里主要通过 **Agent Runs** 这个界面看到它）。等你跑起第一个 agent，它会成为你排查问题的好帮手。

> 💡 小贴士：你现在完全不需要去单独学这四个系统。eve 的妙处恰恰是——它把它们**打包好、连好线**摆在你面前，你大多数时候只是在"享受"它们带来的能力，而不是直接操作它们。把这张表当成一张"地图图例"，遇到对应名词时回来看一眼就行。

### 1.7 一个重要前提：eve 还处于 beta 阶段

在你投入学习之前，必须诚实地告诉你一件事：**eve 目前处于 beta（测试）阶段。**

eve 文档里有一句明确的提示（大意）：eve 当前处于 beta 阶段，受相关条款约束；**框架、API、文档和行为，在正式发布（GA, general availability）之前都可能发生变化。**

这对你这个新手意味着什么？

- **核心理念是稳的，细节可能会动。** 本教程会教你"文件系统优先""持久化会话""四大支柱"这些根本性的设计——这些短期内不太会变。但某些具体的 API 名称、参数、默认值，未来版本里有可能调整。
- **以官方文档为准。** 当你遇到本教程和你手上版本行为不一致时，**永远以你当前 eve 版本的官方文档为准**。

> ⚠️ 注意：beta 软件的另一个特点是文档可能"散落多处、且会更新"。所以养成一个好习惯：遇到拿不准的 API，先用 `eve info` 看看自己项目的实际情况，再去查官方文档，而不是凭印象写死。

> 💡 小贴士：本教程在涉及"不同来源说法不一致"或"官方未明确写死"的地方，会坦诚地标注出来并建议你以官方文档为准，绝不会为了讲得顺口而编造一个并不存在的用法。你也应该带着同样的审慎去对待任何 beta 框架的教程。

### 1.8 学完这本教程，你能做出什么

讲了这么多概念，最后给你一张"预期图"——让你知道为什么这些东西值得学，以及它们最终会拼成什么。

通读这本教程并动手实践后，你将能够：

- **从零搭起一个 eve 项目**，理解 `agent/` 目录里每一类文件分别声明了什么能力。
- **给 agent 写一段稳定的人格与指令**（instructions），让它知道自己是谁、该怎么干活、边界在哪。
- **给 agent 加上工具（tools）**——让它能真正"动手"：查数据、调 API、读写文件。
- **接入外部服务（connections）**——把 Linear、某个内部 API，甚至一个 MCP 服务器变成 agent 能调用的能力，而模型自始至终看不到背后的网址和密钥。
- **加上技能（skills）和子 agent（subagents）**，让 agent 在面对复杂任务时"按需加载知识"或"把活儿分包出去"。
- **用审批门（approval）守住危险操作**——比如让"扣款""下单"这类动作必须先经过人类点头才执行。这正是 eve "可以无限期免费暂停等待"那个特性的实战用途。
- **配置定时任务（schedules）**，让 agent 按自己的时钟干活（每日摘要、定时同步、巡检）。
- **把 agent 部署到 Vercel 生产环境**，并通过 Agent Runs 观察它每一次运行的细节。

为了让这一切不停留在抽象层面，本教程会贯穿引用一个**真实的、生产形态的示例项目 `muse-agent`**——一个跑在 eve 上的 OKX 交易 agent。它会用一个很有说服力的真实场景，把上面这些概念串成一个整体。比如，它用一条"读"连接（无需审批，因为只是查询）和一条"交易"连接（配了 `approval: always()`，意味着**这条连接上的每一次工具调用都会持久地停下来、等一个人类点头批准之后才继续**）——这正是"持久化暂停 + 审批门"在真实金钱场景下的、性命攸关的安全用法。

> 💡 小贴士：现在你完全不用看懂 `approval: always()` 这行代码。把它当成一个"诱饵"就好——它在向你预告：学到后面，你会亲手用 eve 这套机制，给一个能动真金白银的 agent 装上可靠的"安全刹车"。这就是把 eve 学透之后，你能掌握的那种实打实的能力。

下一章，我们就把手弄脏——从安装环境、跑通 `eve init` 脚手架，到看着你的第一个 agent 在终端里活过来。

---

## 2. 准备工作：账号、Node、CLI 与模型访问

在写第一行 agent 代码之前，我们要先把"地基"打好。这一章的目标很简单：让你的电脑具备**跑起来一个 eve agent** 所需要的一切。

别担心，需要装的东西不多，一共就四样：

1. 一个 **Vercel 账号**（你的 agent 最终跑在 Vercel 上）
2. **Node.js**（eve 是用 JavaScript/TypeScript 写的，要靠 Node 来运行）
3. 一个能访问的 **模型**（通过 Vercel 的 **AI Gateway**，agent 才能"思考"）
4. 一些命令行工具（**Vercel CLI**，以及 eve 自带的 `eve` 命令）

我们一步步来。每一步我都会先告诉你"这是干嘛的"，再给你可以直接复制粘贴的命令。

> 💡 小贴士：本章所有终端命令都可以原样复制到你的终端（macOS 的"终端"App、Linux 的 shell，或 Windows 的 PowerShell / WSL）里执行。看到 `$` 或 `#` 提示符不用管，那只是示意，复制命令本身即可。

---

### 2.1 注册 Vercel 账号

**它是什么 / 为什么需要它？**
Vercel 是一个云平台。你可以把它理解成"放你应用的房子"——eve 框架做出来的 agent，最终就是部署到 Vercel 上运行的。eve 底层依赖的一整套能力（持久化会话、沙箱、模型路由等）也都是 Vercel 提供的服务。所以第一步，先有一个 Vercel 账号。

操作很简单：

1. 打开浏览器，访问 [https://vercel.com](https://vercel.com)
2. 点击 **Sign Up**（注册）
3. 推荐用 **GitHub** 账号登录注册——后面用 Git 部署会更顺手（GitHub 是一个存放代码的平台，如果你还没有 GitHub 账号，可以先去 [https://github.com](https://github.com) 免费注册一个）
4. 按提示完成邮箱验证等步骤

注册完成后，你会进入 Vercel 的控制台（Dashboard）。这就够了，暂时不用在网页上做别的操作。

> 💡 小贴士：免费的 Hobby 计划足够你跟着本教程做完所有练习。eve 的费用是按底层用到的 Vercel 资源和模型用量计的——本地学习阶段几乎不产生费用。具体计费以 Vercel 官方文档为准。

> ⚠️ 注意：eve 目前处于 **beta（测试阶段）**。框架、API、文档和行为在正式发布前都可能变化。所以如果你看到本教程的某条命令或字段和官方最新文档对不上，请以**官方文档**为准。

---

### 2.2 安装 Node.js（必须 >= 24）

**它是什么 / 为什么需要它？**
Node.js 是让 JavaScript/TypeScript 代码能在你电脑上（而不是浏览器里）运行的"引擎"。eve 框架本身、它的命令行工具、你写的 agent 代码，全都跑在 Node 上。没有 Node，什么都启动不了。

> ⚠️ 注意（这是本章最关键的一条）：**eve 要求 Node.js 版本 >= 24**。这不是"建议"，是硬性要求——`eve` 命令在更低版本上会直接拒绝运行。很多新手卡在第一步，就是因为电脑上装的是旧版 Node（比如 18 或 20）。请务必确认是 24 或更高。

#### 先检查你现在的版本

打开终端，运行：

```bash
node -v
```

如果它输出类似 `v24.3.0`（开头的数字 >= 24），恭喜，你可以直接跳到下一节。如果输出的是 `v18.x`、`v20.x`，或者提示 `command not found`（没装），那就继续往下看。

#### 用 nvm 安装/切换 Node 版本（推荐）

直接从官网装 Node 当然也行，但更省心的方式是用 **nvm**（Node Version Manager，Node 版本管理器）。它能让你在同一台电脑上装多个 Node 版本、随时切换——以后哪个项目要别的版本，也不用重装。

> 💡 小贴士：下面是 macOS / Linux 的做法。Windows 用户请使用对应的 **nvm-windows**（项目地址 `github.com/coreybutler/nvm-windows`），安装后用法类似。

第一步，安装 nvm：

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

安装脚本跑完后，**关掉终端再重新打开**（让 nvm 生效），然后验证：

```bash
command -v nvm
```

如果输出了 `nvm`，说明装好了。接着安装并启用 Node 24：

```bash
nvm install 24
nvm use 24
nvm alias default 24
```

这三条命令分别是：安装 Node 24、在当前终端切换到 24、把 24 设为以后默认版本。

最后再确认一次：

```bash
node -v
npm -v
```

`node -v` 应该显示 `v24.x.x` 或更高；`npm -v` 会显示一个版本号——**npm**（Node 的包管理器）是随 Node 一起装好的，不用单独安装，后面用它来下载各种工具包。

> 💡 小贴士：你可能听说过 `pnpm`、`yarn` 这些其他包管理器。eve 的脚手架会沿用你启动时所用的包管理器，但作为新手，**用自带的 `npm` 完全足够**，本教程的命令也都以 `npm` / `npx` 为主。`npx` 是 npm 自带的一个命令，作用是"临时下载并运行某个工具包"，不用先全局安装——后面会经常用到它。

---

### 2.3 安装 Vercel CLI 并登录

**它是什么 / 为什么需要它？**
CLI 是 "Command Line Interface"（命令行界面）的缩写。**Vercel CLI** 就是 Vercel 提供的一个命令行工具，让你在终端里直接和 Vercel 打交道——把项目和云端关联、拉取环境变量、部署上线等。eve 的部署和模型访问都会用到它。

#### 全局安装 Vercel CLI

运行：

```bash
npm i -g vercel
```

`-g` 表示"全局安装"，装完之后你在任何目录都能直接用 `vercel` 命令。验证一下：

```bash
vercel --version
```

能打印出一个版本号就说明装好了。

> 💡 小贴士：如果你不想全局安装，也可以在项目里用 `npx vercel ...` 的方式临时调用，效果一样。新手建议先全局装一个，用起来更直接。

#### 登录

安装完成后，把这个 CLI 和你刚注册的 Vercel 账号关联起来：

```bash
vercel login
```

它会让你选择登录方式（比如用 GitHub、邮箱等），然后通常会自动打开浏览器让你确认。确认后回到终端，看到登录成功的提示就可以了。

> 💡 小贴士：`vercel login` 只需要做一次。之后这台电脑就"记住"你的身份了，部署、拉取配置等操作都不会再让你重复登录。

#### 关于 eve 自己的命令行工具

除了 Vercel CLI，eve 框架**还自带一个 `eve` 命令**（比如 `eve dev` 启动本地开发服务器、`eve deploy` 部署）。这个命令**不需要你现在单独安装**——等下一章用 `npx eve@latest init` 创建项目时，它会作为项目依赖自动装进来，到时候用 `npx eve ...` 或项目脚本（如 `npm run dev`）调用即可。这里你先有个印象就好。

---

### 2.4 配置模型访问（AI Gateway）

这是新手最容易困惑、但其实并不难的一步。我们先讲清楚原理。

**它是什么 / 为什么需要它？**
你的 agent 要"会思考"，背后得有一个大语言模型（比如 Claude、GPT 这类）。但 agent 代码里**并不直接写某个模型厂商的密钥**，而是写一个"模型名字"，让 Vercel 的 **AI Gateway** 帮你把请求路由到对应的模型厂商去。

你可以把 AI Gateway 想象成一个"前台总机"：你只要说"我要找 anthropic 家的 claude-opus-4.8"，总机就帮你接通，还顺带处理好厂商之间的故障切换。这样做的好处是：你的代码里只写一个干净的字符串，密钥和路由这些脏活累活交给 Gateway。

在 agent 代码里，模型就是这么一个字符串（这是下一章才会写的内容，这里先感受一下形态）：

```ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-opus-4.8",
});
```

这个 `"anthropic/claude-opus-4.8"` 就是一个 **AI Gateway 模型 id**，格式是 `<厂商>/<模型>`。它会经由 AI Gateway 被路由到真正的模型。

> 💡 小贴士：你在不同资料里可能见到不同的示例模型，比如 `anthropic/claude-sonnet-4.6`、`openai/gpt-5.4-mini` 等，它们都是合法的 Gateway 模型字符串。本教程统一用 `anthropic/claude-opus-4.8` 作为例子，和真实示例项目保持一致。具体有哪些可用模型，以 Vercel AI Gateway 官方文档（`/ai-gateway/models`）为准。

#### 部署到 Vercel 上时：用 OIDC，自动搞定

好消息：当你的 agent **部署在 Vercel 上**运行时，模型访问的凭证是**自动**处理的。Vercel 通过一种叫 **OIDC** 的机制自动给你的请求做认证，你**不需要手动配置任何模型厂商的 API key**。这也是为什么 eve 强调"在 Vercel 上免密钥"。

具体怎么开启？关键是把你的本地项目和一个 Vercel 项目**关联（link）**起来。下一章创建项目后，你会用到这条命令：

```bash
eve link
```

它会把当前目录关联到一个 Vercel 项目，并**取回 AI Gateway 凭证**。关联之后，部署上去就能直接调用模型了。

> 💡 小贴士：`eve link` 是 eve 自带的命令（项目里通过 `npx eve link` 调用）。它做了两件事：把目录链到 Vercel 项目 + 取回 AI Gateway 凭证。等你有了项目再运行它即可，现在不用着急。

#### 本地开发时：也要能访问模型

这一点很多新手会忽略：**当你在自己电脑上用 `eve dev` 跑 agent 时，它同样需要能连上模型**——否则 agent 一句话也回不了。本地环境拿不到 Vercel 云端那套自动 OIDC，所以你需要让本地具备访问 AI Gateway 的能力。两种常见方式：

**方式一：关联 Vercel 项目后获得 `VERCEL_OIDC_TOKEN`。**
把项目关联到 Vercel（用 `eve link`，它内部会完成 Vercel 项目链接）之后，本地会获得一个 `VERCEL_OIDC_TOKEN`，eve 用它就能在本地访问 AI Gateway。对大多数人来说这是最省事的路径。

**方式二：设置一个 `AI_GATEWAY_API_KEY` 环境变量。**
你也可以在 Vercel 控制台创建一个 AI Gateway 的 API key，然后在本地把它放进环境变量。最简单的做法是在项目根目录建一个 `.env.local` 文件，写上：

```bash
AI_GATEWAY_API_KEY=你的key值
```

eve 的命令在启动时会**先读取 `.env` / `.env.local`**，所以放在这里就能被自动识别。

> ⚠️ 注意：`.env.local` 里放的是**机密信息**，千万不要把它提交到 Git 仓库或分享给别人。eve 脚手架创建项目时一般会帮你把它加进 `.gitignore`；如果你手动建，请自己确认它没有被 Git 跟踪。所有密钥都应放在环境变量里，**绝不要写死在源代码中**。

> 💡 小贴士：概念上你只需要记住一句话——**"部署在 Vercel 上靠 OIDC 自动认证；本地开发则需要 `VERCEL_OIDC_TOKEN`（来自 `eve link`）或一个 `AI_GATEWAY_API_KEY`"**。两者有其一即可。

---

### 2.5（可选）安装 Vercel Plugin，给 Claude Code / Cursor 加部署技能

这一节是**可选的、锦上添花**的内容。如果你正在用 **Claude Code** 或 **Cursor** 这类 AI 编程助手写代码，可以给它们加上一套"会部署"的技能。

**它是什么 / 为什么需要它？**
Vercel 提供了一个 plugin（插件），装上之后，你的 AI 编程助手就多了一批和 Vercel 部署相关的能力和斜杠命令（slash command），比如可以直接让它执行 `/vercel-plugin:deploy prod` 来部署到生产环境。本质上是让 AI 助手更"懂"怎么帮你跟 Vercel 打交道。

安装命令：

```bash
npx plugins add vercel/vercel-plugin
```

装完之后，你的编程助手里就会出现像 `/vercel-plugin:deploy prod` 这样的命令。

> 💡 小贴士：如果你现在还不用 AI 编程助手，或者只想先把基础流程跑通，**完全可以跳过这一节**，对后面的学习没有任何影响。

---

### 2.6 环境自检清单

把上面几步做完后，花一分钟对照下面的清单做个"体检"。每一条都能在终端里验证，确保你真的准备好了再进入下一章。

**1. Node 版本 >= 24**

```bash
node -v
```

> 期望：输出 `v24.x.x` 或更高。若不是，回到 2.2 用 `nvm use 24` 切换。

**2. npm 可用**

```bash
npm -v
```

> 期望：输出一个版本号（随 Node 一起安装，无需单独装）。

**3. Vercel CLI 已安装**

```bash
vercel --version
```

> 期望：输出一个版本号。若提示 `command not found`，回到 2.3 执行 `npm i -g vercel`。

**4. 已登录 Vercel**

```bash
vercel whoami
```

> 期望：输出你的 Vercel 用户名。若提示未登录，回到 2.3 执行 `vercel login`。

**5. 确认 npx 通畅（网络可访问 npm）**

```bash
npm exec --version
```

> 期望：输出一个版本号。这只是确认 `npx`（即 `npm exec`）这套机制可用、网络通畅。下一章我们会用 `npx eve@latest init` 创建项目——那条命令会自动下载 eve 脚手架，所以这里只要 `npx` 本身正常就行。

**6. 模型访问凭证（心中有数即可）**

> 这一条暂时不用在终端验证。你只需要确认自己理解了 2.4 的结论：部署到 Vercel 靠 OIDC 自动认证；本地开发要么先 `eve link` 拿到 `VERCEL_OIDC_TOKEN`，要么准备一个 `AI_GATEWAY_API_KEY`。等下一章真正创建并运行项目时，我们会把它落到实处。

逐项核对清单，分别准备的几样东西：

| 你准备的东西 | 它是干嘛的 | 自检命令 |
|---|---|---|
| Vercel 账号 | 你的 agent 最终运行的云平台 | `vercel whoami` |
| Node.js >= 24 | 运行 eve 和你的 agent 代码 | `node -v` |
| npm / npx | 下载和运行各种工具包 | `npm -v` |
| Vercel CLI | 在终端里和 Vercel 交互、部署 | `vercel --version` |
| AI Gateway 凭证 | 让 agent 能访问模型 | （见 2.4，下一章落实） |
| Vercel Plugin（可选） | 给 AI 编程助手加部署技能 | —— |

全部打勾之后，你的开发环境就准备好了。下一章，我们就用 `npx eve@latest init` 创建出你的第一个 eve agent，并在本地把它真正跑起来。

---

## 3. 5 分钟跑起你的第一个 agent

前两章我们聊了 eve 是什么、它为什么存在。话说得再多，不如真的跑一个出来。这一章我们就从零开始，把第一个 agent 在你自己的电脑上跑起来，并且真的跟它对上话。

承诺一下：只要你的网络没问题、Node 版本对，这一章走完大概就是 5 分钟。

### 3.0 先确认一件事：Node 版本

eve 的命令行工具要求 **Node.js >= 24**（npm 会随它一起装好）。先在终端里确认一下你的版本：

```bash
node -v
```

如果打印出来的版本号是 `v24.` 开头或更高，就可以继续。如果低于 24，请先去升级 Node，否则后面的命令会报错。

> 💡 小贴士：不确定怎么升级 Node？最省心的办法是用 [nvm](https://github.com/nvm-sh/nvm)（Node 版本管理器），一行 `nvm install 24` 就能装好。具体安装步骤以 nvm 官方文档为准。

### 3.1 一条命令，生成一个 agent

eve 提供了一个脚手架命令（"脚手架" = 帮你把项目骨架自动搭好的工具，省得你一个文件一个文件手敲）。打开终端，运行：

```bash
npx eve@latest init my-agent
```

这里逐个拆解一下：

- `npx`：Node 自带的工具，意思是"临时下载并运行某个 npm 包"，你不用提前全局安装 eve。
- `eve@latest`：用最新版本的 eve 包。
- `init my-agent`：执行 `init`（初始化）命令，并把新项目放进一个叫 `my-agent` 的新文件夹里。这个名字你可以随便改成自己喜欢的。

这条命令会帮你做好这几件事：

1. 新建一个 `my-agent` 目录；
2. 安装依赖（`eve`、`ai`、`zod` 这三个包）；
3. 初始化 Git（方便你之后做版本管理）；
4. 然后提示你：要不要打开一个支持的"编程助手"对话窗口，或者直接启动开发服务器。

> 💡 小贴士：如果你已经有一个项目，想把 eve **加进去**而不是新建，就把名字换成 `.`（一个点，代表"当前目录"）：
> ```bash
> npx eve@latest init .
> ```
> 这种"加到现有项目"的模式有一点不同：它不接受 `--channel-web-nextjs` 这个选项（该选项用来顺便生成一个 Next.js 网页聊天应用）。如果之后需要网页端，再单独运行 `eve channels add web` 即可。

### 3.2 进目录、装依赖

如果上一步的脚手架已经替你装好了依赖，这一步可以略过。但为了让你心里有数（也为了"加到现有项目"那种情况），完整流程是这样的：

```bash
cd my-agent
npm install
```

- `cd my-agent`：进入刚生成的项目目录。
- `npm install`：把 `package.json` 里声明的依赖装到本地。

> ⚠️ 注意：如果你是用纯手动方式把 eve 集成进一个已有项目（不走脚手架），那么依赖需要你自己装。手动安装命令是：
> ```bash
> npm install eve@latest ai zod
> ```

### 3.3 两个文件，就是一个 agent

这是 eve 最让人上头的地方，先记住这句话：

> **文件的名字和它在目录里的位置，就是它的定义。**

换句话说，eve 是"文件优先"的框架。你不需要写一大堆注册代码告诉框架"我这里有个 agent、它叫什么、配置在哪"。你只要把文件放在对的位置，eve 在构建时会自己去**发现（discover）**、**校验（validate）**、**编译（compile）**成一个能跑的应用。

那么，最小的一个 agent 需要几个文件？答案是 **两个**：

```text
my-agent/
├── package.json
├── tsconfig.json
└── agent/
    ├── agent.ts          # 运行配置：用哪个模型
    └── instructions.md   # 系统提示词：agent 的"人设"和规则
```

`agent/` 目录就是 eve 的"地盘"。它会扫描这个目录，按目录名自动识别各种能力（工具、技能、连接……我们后面章节会逐个讲）。现在你只需要认识里面这两个核心文件。

> 💡 小贴士：脚手架实际生成的文件可能比这两个多（比如还会带上 `agent/channels/eve.ts` 这种入口文件）。但**就 agent 本身而言**，`agent.ts` + `instructions.md` 这两个就足够构成一个能对话的 agent 了。其余的我们留到后面章节。

### 3.4 逐行看 `agent.ts`

打开 `agent/agent.ts`，最小的内容长这样：

```ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-opus-4.8",
});
```

一行一行看：

- `import { defineAgent } from "eve";`：从 `eve` 包里引入 `defineAgent` 这个函数。它的作用是"定义这个 agent 的运行配置"。
- `export default defineAgent({ ... })`：把配置作为这个文件的**默认导出**。eve 就是靠这个默认导出来识别你的 agent 配置的。
- `model: "anthropic/claude-opus-4.8"`：指定这个 agent 用哪个大模型。这里的字符串是 `"<提供商>/<模型名>"` 的格式，会通过 Vercel 的 **AI Gateway**（模型网关）来路由请求。

关于 `model` 字段，有几点值得新手特别留意：

- **`agent.ts` 这个文件本身是可选的**，但**只要你写了它，`model` 字段就是必填的**。
- 这里的模型字符串只是一个示例。脚手架默认用的可能是另一个模型（例如 `anthropic/claude-sonnet-4.6`），不同来源给的示例也不太一样。它们都是合法的 AI Gateway 模型字符串，你照着脚手架生成的那个用就好，不必纠结。（顺带一提：`anthropic/claude-sonnet-4.6` 这个默认值只在你**完全不写** `agent.ts` 时才会自动生效。）
- AI Gateway 支持哪些具体模型 id，并没有一份写死的清单，请以官方文档 / AI Gateway 模型页为准。

> ⚠️ 注意：`defineAgent` 的能力来自"文件放在哪"，而不是塞进配置对象。所以你**不会**在这里写 `tools`、`temperature`、`maxSteps` 之类的字段——那些要么是放文件（比如工具放在 `agent/tools/`），要么走别的机制。现在不用记，知道"配置很薄"这件事就行。

如果你的项目还没链接到 Vercel，想在**本地**让模型跑起来，需要给它一把"钥匙"——也就是 AI Gateway 的 API key。脚手架生成的项目一般会有一个 `.env`（或 `.env.local`）文件来放这类密钥。本地最常见的是设置 `AI_GATEWAY_API_KEY`。具体怎么获取这把 key，以及它和 Vercel OIDC 的关系，我们留到后面的"部署"章节细讲；如果脚手架在初始化时引导你登录 / 链接了 Vercel，这一步可能已经替你处理好了。

> 💡 小贴士：eve 命令运行时会**先**自动读取项目根目录下的 `.env` / `.env.local`。所以你把密钥放进这两个文件之一，命令就能读到，不用手动 export。

### 3.5 逐行看 `instructions.md`

第二个核心文件是 `agent/instructions.md`。它是这个 agent 的**系统提示词（system prompt）**——你可以把它理解成 agent 的"永久人设和行为准则"。

它和普通"对话内容"最大的区别是：**它在每一次模型调用前都会被自动加到最前面**。也就是说，无论用户问什么，这段话始终生效。所以它适合写"我是谁、我说话什么风格、我有哪些边界"这种**稳定、长期**的规则。

最小的 `instructions.md` 可以短到只有一句话：

```md
You are a concise assistant. Use tools when they are available.
```

翻译过来就是："你是一个简洁的助手。有工具可用时就用工具。"

关于 `instructions.md`，新手记住这几条就够了：

- 它在**根目录是必需的**（一个 agent 必须有系统提示词，否则不知道自己该是谁）。它可以是 `agent/instructions.md`，也可以是 TypeScript 形式的 `agent/instructions.ts`（用于动态生成提示词），但**两者不能同时在根目录存在**——同时存在会在构建时直接报错。
- **指令本身永远不执行代码**。它只是"说明书"。需要真正去做事（查 API、写文件、下单……）得靠后面章节讲的"工具（tool）"。
- 保持它**短而稳定**。把那些又长又分场景的流程，留给后面要讲的"技能（skill）"，按需加载，不要让每一轮对话都背着一大段不相干的提示词。

> 💡 小贴士：到这里你应该体会到"文件即配置"的爽点了：一个模型字符串 + 一段 markdown 提示词，就是一个能跑的 agent。没有注册表、没有样板初始化代码，改行为就是改文件。

### 3.6 启动本地开发服务器，跟 agent 对话

文件就绪，现在让它跑起来。在项目根目录运行：

```bash
npm run dev
```

脚手架在 `package.json` 里把 `dev` 这个脚本映射到了 eve 的 `dev` 命令，所以上面这条等价于直接运行：

```bash
npx eve dev
```

> 💡 小贴士：`eve` 命令在不带任何子命令时，**默认就是 `eve dev`**。所以以下三种写法都能启动开发服务器，挑顺手的用：
> ```bash
> npm run dev
> npx eve dev
> npx eve
> ```
> （eve 的入门文档建议"照着生成的 README 来"，README 里的项目脚本是权威的；`eve dev` 和 `npm run dev` 两种写法都能用。）

`eve dev` 会启动一个本地开发服务器，并带上一个**交互式终端界面（terminal UI）**。这个界面里有一个可以直接打字的输入框——也就是一个跟你的 agent 对话的"聊天窗口"，就在你的终端里。你输入一句话回车，agent 就会回你。它还支持热更新（HMR）：你改了 `instructions.md` 或 `agent.ts`，不用重启就能看到效果。

下面是一段"我说了什么、它怎么回"的示例对话，帮你建立直觉（具体措辞每次都会不一样，这里只是示意）：

```text
你 ▸ 你好，你是谁？

agent ▸ 你好。我是一个简洁的助手，可以回答问题、在有工具时帮你
        调用工具来完成任务。需要我做点什么吗？

你 ▸ 用一句话解释一下什么是 eve。

agent ▸ eve 是一个文件优先的框架，用来构建跑在 Vercel 上的持久化
        后端 AI agent——你用文件定义 agent 能做什么，剩下的生产
        环境管线交给框架。
```

这段对话之所以"知道自己是个简洁的助手"，正是因为我们在 `instructions.md` 里写了那句系统提示词。改一改那句话再回车提问，你会立刻看到它的人设变了——这就是"文件即配置"的现场效果。

> ⚠️ 注意：如果一启动就报模型相关的错误（比如认证失败），多半是本地还没配好 AI Gateway 的凭证。回到 3.4 那一段，确认 `.env` / `.env.local` 里设置了 `AI_GATEWAY_API_KEY`（或者你已经通过 Vercel 链接拿到了 `VERCEL_OIDC_TOKEN`）。

### 3.7 用 `eve info` 看看 eve 到底"发现"了什么

跑通对话之后，还有一个特别值得养成习惯的命令——`eve info`。它会把 eve **解析、发现并编译**出来的东西原原本本列给你看，是验证"我的文件有没有放对位置"的最佳工具。

新开一个终端标签页（让 `eve dev` 继续开着），在项目根目录运行：

```bash
npx eve info
```

它会打印出这个 agent 已被解析出的细节，比如：当前用的模型、加载的指令、有哪些工具（tools）、技能（skills）、子 agent（subagents）、定时任务（schedules）、通道（channels），以及编译产物和诊断信息。对我们这个最小项目来说，工具 / 技能那些大概率还是空的——这完全正常，因为我们还没加。

如果你想要机器可读的格式（比如脚本里解析），加上 `--json`：

```bash
npx eve info --json
```

> 💡 小贴士：以后你新加了一个工具或连接，发现 agent "看不见"它，第一反应就应该是跑一下 `eve info`，看看 eve 到底有没有发现这个文件。十有八九是文件名或目录放错了——别忘了第 2 章提到的坑：连接（connection）文件名要用 **dash-case**（短横线命名，如 `okx-read.ts`）。

### 3.8 本章小结

我们用一条命令搭好了项目，认识了构成一个 agent 的两个核心文件，并真的跟它聊上了天：

- `npx eve@latest init my-agent` 生成项目（往已有项目里加用 `init .`）；进目录、`npm install`。
- 一个最小 agent = `agent/agent.ts`（`defineAgent({ model })`，写了它 `model` 就必填）+ `agent/instructions.md`（系统提示词，根目录必需，且不执行代码）。
- `npm run dev` / `eve dev` / `eve`（无子命令默认即 `dev`）启动本地开发服务器和交互式终端 UI，直接在终端里跟 agent 对话，支持热更新。
- `eve info`（加 `--json` 可机器读）查看 eve 发现并编译了什么——排查"文件没放对"的首选工具。

最关键的体会是那句贯穿全书的话：**文件的名字和位置就是它的定义。** 没有注册表、没有样板代码，改行为就是改文件。下一章，我们就在这个最小骨架上，给 agent 加上它的第一个"工具"，让它真的能动手做事。

---

## 4. 核心概念地图：agent/ 目录里的每个角色

在前面几章里，你已经亲手跑起来了第一个 agent。但你可能心里还有点发虚：那个 `agent/` 文件夹里到底装了些什么？每个文件是干嘛的？什么时候该新建一个？为什么有的项目只有两个文件，有的却塞了十几个文件夹？

这一章不教你写新代码。它的唯一目标，是帮你在脑子里画出一张**地图**——把 `agent/` 目录里的每个"角色"摆到正确的位置上，让你以后看到任何一个 eve 项目，都能一眼认出"哦，这个文件夹是负责 X 的"。

> 💡 小贴士：建议你把这一章当成"字典+导航"。现在通读一遍建立印象就够了，不必背下每个 API 细节。后面每一章会专门深入讲其中一个角色，到时候再回头看这张地图，会越来越清晰。

### 4.1 先理解一句话：eve 是"文件优先"的框架

在写任何代码之前，你必须先吃透 eve 最核心的设计哲学，否则后面所有东西都会显得很奇怪。

eve 官方的说法是：**"文件的名字和它在目录树里的位置，就是它的定义。"**（"A file's name and place in the tree are its definition."）

这句话翻译成人话就是：**你不需要写一个"注册表"或"配置中心"去告诉 eve "我有哪些工具、哪些技能"。你只要把文件放到对的文件夹、起对的名字，eve 自己就会发现它们。**

举个最直观的例子：

- 你在 `agent/tools/` 文件夹里放一个叫 `get_weather.ts` 的文件 → eve 就认为你有了一个名叫 `get_weather` 的工具。
- 你在 `agent/connections/` 里放一个 `linear.ts` → 你就有了一个名叫 `linear` 的外部连接。

> ⚠️ 注意：正因为"身份来自路径"，所以在这些文件里你**永远不会写 `name` 或 `id` 字段**。名字就是文件名。这是新手最容易踩的坑——别去 `defineTool` 里找 `name` 参数，它不存在。

官方还有一个特别好懂的类比：**"就像 Next.js 之于网页应用，eve 之于 agent。"** 如果你写过 Next.js，会知道在 `pages/` 或 `app/` 里放个文件就自动有了一个路由——eve 对 agent 用的是同一套思路。

### 4.2 关键流程：发现 → 编译 → 运行

既然"放文件就生效"，那 eve 在背后到底做了什么？整个过程分三步，记住这三个词，你就理解了 eve 的工作方式：

1. **发现（Discover）**：当你启动 eve 时，它会在构建阶段**扫描** `agent/` 目录，按文件夹名字自动识别每个角色。官方原话：*"发现是自动的，不需要单独的注册表。"*
2. **编译（Compile）**：eve 把扫描到的所有能力（工具、技能、连接……）整理好，**编译成一份清单（manifest）**，存放在一个叫 `.eve/` 的文件夹里。这一步相当于把"你写的文件"翻译成"模型能理解的能力列表"。
3. **运行（Runtime）**：编译好的 agent 作为一个可部署的应用跑在 **Vercel Functions** 上。当有消息进来时，agent 通过标准的"agent 循环"去调用工具、加载技能。

用一句话串起来就是：

> **你放文件 → eve 发现 → 编译成 manifest → 运行。**

> 💡 小贴士：`.eve/` 是 eve 自动生成的产物目录（编译结果），你不需要手动改它，通常也会把它加进 `.gitignore`。如果你想确认"eve 到底发现了我的哪些文件"，可以运行 `eve info`（加 `--json` 可输出 JSON）。这是排查"我新建的工具怎么没生效"的第一招。

```bash
eve info
```

### 4.3 总览表：agent/ 里的每个角色

下面这张表是本章的核心。先扫一眼，建立整体印象；表格下面会逐个展开讲。

| 角色（文件/目录） | 一句话职责 | 何时才需要它 |
|---|---|---|
| `agent/agent.ts` | **运行配置**：指定用哪个模型、以及一些运行参数 | 起步几乎必备（当它存在时，`model` 字段必填）。省略它时 eve 用默认模型。 |
| `agent/instructions.md` | **系统提示词**：agent 的永久性格、身份与规则 | **根目录必备**。这是 agent 的"灵魂"。 |
| `agent/tools/` | **工具**：模型能调用的、有类型的函数（查 API、跑查询、写文件） | 需要 agent "动手做事"时。每个文件一个工具。 |
| `agent/skills/` | **技能**：按需加载的流程或参考资料（只加指令，不加工具） | 有些冗长/分场景的操作手册，不想塞进每一轮提示词时。 |
| `agent/connections/` | **连接**：接入外部世界（MCP 服务器 或 OpenAPI 接口） | 需要让 agent 用上某个外部服务（如 Linear、某交易所）时。 |
| `agent/channels/` | **渠道**：对外的入口与鉴权（HTTP、Slack 等） | 默认已有 HTTP 入口；只有要改鉴权或接新平台时才建文件。**仅限根目录。** |
| `agent/schedules/` | **定时任务**：让 agent 按 cron 自己定时跑起来 | 需要"每天/每小时"自动执行（日报、同步、清理）时。**仅限根目录。** |
| `agent/sandbox.ts`（或 `agent/sandbox/`） | **沙箱**：模型的隔离代码/文件草稿空间 | 想定制沙箱后端或网络策略时；不写也有默认沙箱。 |
| `evals/`（与 agent/ 平级） | **自动化测试**：给 agent 打分的检查 | 想为 agent 行为加测试、防止退步时。 |
| `agent/instrumentation.ts` | **可观测性**：接 OpenTelemetry 导出链路追踪 | 想把追踪数据导到 Braintrust/Datadog 等外部系统时。**仅限根目录。** |

还有几个本章不展开、但你迟早会遇到的角色，先混个脸熟：

| 角色 | 一句话职责 |
|---|---|
| `agent/subagents/<id>/` | **子 agent**：拥有独立对话历史与状态的子级 agent，父 agent 可委派任务给它。 |
| `agent/lib/` | **共享代码**：纯供 import 的辅助代码，不会被当成工具/技能发现。 |
| `agent/hooks/` | **钩子**：订阅生命周期与流式事件的模块。 |

> ⚠️ 注意：表里有几个角色标了"**仅限根目录**"（`channels/`、`schedules/`、`instrumentation.ts`）。这是因为 eve 允许嵌套子 agent，但这几个角色只有最外层的主 agent 能拥有——子 agent 不能自己开渠道或定时任务。现在记住这条规则即可，子 agent 的细节后面会讲。

### 4.4 新手最重要的一句话：起步只需两个文件

看完上面那张表，你可能会被吓到——"这么多文件夹，我都得写吗？"

**完全不用。** 这是本章你最该记住的一点：

> 上面表格里，**绝大多数角色都是可选的**。一个能跑起来的最小 eve agent，只需要两个东西：`agent/agent.ts`（选哪个模型）+ `agent/instructions.md`（agent 的身份与规则）。

其余的工具、技能、连接、定时任务、沙箱、测试、可观测性……都是**等你真的需要那个能力时，再添一个文件**。eve 的设计就是这样：你按需"长出"能力，而不是一开始就配置一大堆。

下面我们就按"从必备到可选"的顺序，把每个角色讲清楚——它是什么、为什么需要、何时才加。

### 4.5 必备的两个角色

#### agent.ts —— 模型与运行配置

`agent.ts` 决定**这个 agent 用哪个大模型，以及怎么运行**。它用 `defineAgent` 来定义：

```ts
// agent/agent.ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-opus-4.8",
});
```

要点：
- 它的导入来自 `eve` 包：`import { defineAgent } from "eve";`。
- `agent.ts` 这个文件本身是**可选的**——但**一旦你写了它，`model` 字段就必填**。
- 如果你完全不写 `agent.ts`，eve 会用一个默认模型（脚手架默认是 `anthropic/claude-sonnet-4.6`）。
- 除了 `model`，`defineAgent` 还能配一些进阶选项（比如 `compaction` 上下文压缩、`outputSchema` 结构化输出等），这些后面专门讲，起步不用管。

> ⚠️ 注意：有个反直觉的点——`defineAgent` **不接受** `tools`、`temperature`、`subagents`、`sandbox` 这类字段。在 eve 里，"有哪些工具/子 agent/沙箱"是由你**往哪个文件夹放文件**决定的，不是在 `agent.ts` 里列出来的。再次印证"文件即接口"。

> 💡 小贴士：`model` 写成 `"<提供商>/<模型名>"` 这种字符串时，会走 Vercel 的 AI Gateway 路由。部署到 Vercel 上用 OIDC 自动鉴权（不用填 provider key）；本地开发则需要 `AI_GATEWAY_API_KEY`。具体哪些模型名可用、以官方文档为准。

#### instructions.md —— 系统提示词（agent 的灵魂）

如果说 `agent.ts` 决定了 agent 用的是哪颗"大脑"，那 `instructions.md` 就决定了它的**性格、身份和规矩**。

它是 agent 的**常驻系统提示词**——会被加到**每一次**模型调用的最前面。官方把它定义为"agent 的永久身份，而不是一段流程"。

最小的 `agent/instructions.md` 可以简单到只有一句话：

```md
You are a concise assistant. Use tools when they are available.
```

要点：
- 它在**根目录是必备的**（子 agent 上可选）。
- eve 会读取 `agent/instructions.md`，把它编进编译后的 manifest。
- 一条关键规则：**"指令永远不执行代码。"**（"Instructions never run code."）想让 agent "做事"，那是工具（tools）的活儿，不是指令的活儿。指令只负责"告诉它该怎么做人、守什么规矩"。

> 💡 小贴士：把指令写得**短而稳定**——只放语气、人设、每一轮都适用的硬性约束。那些又长又分场景的操作手册，应该放进**技能（skills）**里按需加载，而不是塞进每一轮的提示词，否则既浪费 token 又干扰模型。这正好引出下一组角色。

### 4.6 让 agent "能做事"的角色

#### tools/ —— 模型能调用的有类型函数

**工具是 agent 能在对话中调用的、一个个有类型的动作**：调一个 API、跑一条数据库查询、写一个文件。这是让 agent 从"只会聊天"变成"能干活"的关键。

每个工具是 `agent/tools/` 下的**一个文件**，文件名（去掉扩展名、转成 snake_case）就是模型看到的工具名。比如 `agent/tools/get_weather.ts` → 工具名 `get_weather`。

```ts
// agent/tools/get_weather.ts
import { defineTool } from "eve/tools";
import { z } from "zod";

export default defineTool({
  description: "Get the current weather for a city.",
  inputSchema: z.object({ city: z.string().min(1) }),
  async execute({ city }, ctx) {
    return { city, condition: "Sunny", temperatureF: 72 };
  },
});
```

三个必填部分：`description`（写给模型看的说明）、`inputSchema`（输入的类型约束，常用 zod）、`execute`（真正干活的函数）。

> ⚠️ 注意（一个重要的安全区分）：**工具代码跑在你的"应用运行时"里，而不是沙箱里。** 这意味着工具能访问 `process.env` 环境变量、能 import 你 `lib/` 里的共享代码。这跟下面要讲的"沙箱"是两个完全不同的执行环境，新手务必分清。

> 💡 小贴士：工具可以加"人工审批"门槛。比如一个退款工具，你可以用 `needsApproval: always()`（来自 `eve/tools/approval`）要求**每次调用都必须人工批准**才执行。这在涉及真金白银或不可逆操作时极其重要——后面讲工具的章节会专门展开。

#### skills/ —— 按需加载的流程

**技能是更大的流程或参考资料，由模型"按需"加载。** 跟"每一轮都在线"的指令不同，技能平时不占用提示词；只有当模型判断当前任务和某个技能匹配时，才会调用内置的 `load_skill` 工具，把那份技能的内容加进当前这一轮。

记住这条关键区别：**技能只添加"指令"，不添加"工具"。** 官方原话：*"技能用于可复用的工作流、多步骤说明、或不该常驻在提示词里的领域知识。"*

最简单的技能就是一个 markdown 文件（比如 `agent/skills/plan_a_trip.md`），文件第一行非空内容就是"什么时候该用它"的路由提示：

```md
Use when the user wants to plan a trip or build an itinerary.
Gather the destination, dates, and budget first, then propose a day-by-day plan.
```

所以**指令 vs 技能**的分工是这样的：

| 维度 | 指令（instructions） | 技能（skills） |
|---|---|---|
| 加载方式 | 永远在线，每一轮都加 | 按需，通过 `load_skill` 加载 |
| 用途 | 永久身份 / 标准规矩 | 可选流程 / 参考资料 |
| 能否跑代码 | 永不跑代码 | 也不跑代码（要执行用工具） |

> 💡 小贴士：技能还能通过命令行从社区安装（用一个独立的 `skills` 命令行工具，注意它和 `eve` 是两个不同的命令）。起步阶段你不用管这个，知道有这回事即可。

### 4.7 连接外部世界与对外入口

#### connections/ —— 接入外部服务（MCP / OpenAPI）

**连接（connection）让你的 agent 用上外部服务。** 它是一个有类型的集成，对接对象只有两种：一个 **MCP 服务器**，或者**任何带有兼容 OpenAPI 3.x 文档的 API**。

eve 会自动发现远端的工具，把它们交给模型，并在背后帮你处理鉴权。一个重要的安全特性：**模型永远看不到连接的 URL 和凭证**——这些细节被挡在模型之外。

连接只有两个定义函数，都来自 `eve/connections`：`defineMcpClientConnection`（接 MCP）和 `defineOpenAPIConnection`（接 OpenAPI）。文件放在 `agent/connections/` 下，文件名就是连接名。

```ts
// agent/connections/linear.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: "https://mcp.linear.app/sse",
  description: "Linear workspace: issues, projects, cycles, and comments.",
  auth: {
    getToken: async () => ({ token: process.env.LINEAR_API_TOKEN! }),
  },
});
```

模型通过内置的 `connection_search` 工具来发现这些远端工具（这个工具**只有在你声明了连接时才会出现**），然后用 `<连接名>__<工具名>` 这种限定名来调用，比如 `linear__list_issues`。

> ⚠️ 注意（来自真实项目的坑）：连接文件名要用**短横线命名（dash-case）**，比如 `okx-read.ts`；并且 eve 在编译/发现阶段就会**严格校验 URL**——一个写错或为空的连接 URL 会让构建直接失败。所以确保 URL 在编译时能解析出来。

> 💡 小贴士：连接也能像工具一样加审批门槛。真实的交易 agent 里，"只读"连接不加审批（每个工具都是查询，安全），而"交易"连接加 `approval: always()`（同样来自 `eve/tools/approval`），让每一笔操作都暂停等待人工批准——这是处理真金白银的核心安全控制。

#### channels/ —— 对外入口与鉴权

**渠道（channel）是平台和你的 agent 之间的"边缘适配器"。** 它负责三件事：把平台传来的输入转成一条用户消息、持有"续聊句柄"（resume 用）、决定怎么把回复发出去。

这里有个对新手特别友好的设计：**每个 eve 应用默认就自带一个 HTTP 渠道**，哪怕你根本没建 `agent/channels/eve.ts` 文件。也就是说，你不用配渠道，agent 就已经能通过 HTTP 接收消息了。

那什么时候才需要建渠道文件？**最常见的就是要改鉴权（谁能访问你的 agent）。** 来看一个真实项目的例子：

```ts
// agent/channels/eve.ts
import { eveChannel } from "eve/channels/eve";
import { localDev, vercelOidc, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc(), httpBasic({ username, password })],
});
```

这里的 `auth` 是一个数组，eve 会**按顺序逐个尝试，第一个匹配上的胜出**；如果全都没匹配上，就返回 `401` 拒绝。这套机制有个重要的安全默认行为——**"默认失败即关闭"（fails closed）**：生产环境的浏览器流量，除非你明确配了接受它的鉴权器，否则一律拒绝。

> 💡 小贴士：除了 HTTP，eve 还内置了 Slack、Discord、Teams、Telegram、Twilio、GitHub 等平台的渠道工厂，也支持自定义渠道。要快速加一个，可以用脚手架命令 `eve channels add`（命令行里目前列出的可选类型是 `slack` 和 `web`）。这些都是后面的进阶内容，起步不用碰。

> ⚠️ 注意：渠道的"鉴权（谁能进来）"和上面连接的"鉴权（agent 怎么登录外部服务）"是**两套完全独立的系统**。前者管"入站"，后者管"出站"。新手很容易把这两个 auth 搞混，记住它们方向相反就好。

### 4.8 自动化与运维相关的角色

这一组角色不是"让 agent 多一项能力"，而是关于"让 agent 自动跑起来、跑得安全、看得见"。起步阶段基本都用不上，但你应该知道它们的存在和职责。

#### schedules/ —— 定时任务

**定时任务让 agent 按它自己的时钟启动**：每日摘要、定时同步、清理扫描、心跳检测。每个定时任务是 `agent/schedules/` 下的一个文件，携带一个 cron 表达式。

```ts
// agent/schedules/heartbeat.ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "*/5 * * * *",
  markdown: "Pull open Linear issues and POST a summary to the metrics endpoint.",
});
```

cron 是标准的 5 字段表达式（分 时 日 月 周）。部署到 Vercel 上时，每个定时任务会变成一个 **Vercel Cron Job，按 UTC 时区评估**。

> ⚠️ 注意：`eve dev`（本地开发服务器）**不会**按 cron 节奏触发定时任务——这是新手常见的困惑。本地想测试，得通过专门的开发派发路由手动触发一次。

> ⚠️ 注意：用 `markdown` 写的定时任务跑在"任务模式（task mode）"下——eve 运行 agent 后会**丢弃输出**（但任务模式里 agent 仍然可以调用工具、写后端、记日志）。任务模式有一条关键限制：它**不能暂停去等人工审批或 OAuth 登录**。所以一个需要审批才能执行的工具（比如要 `approval: always()` 的交易工具，它会"暂停等待"）在任务模式里就用不了——这恰好成了一道天然的安全边界。

#### sandbox.ts —— 模型的代码沙箱

**每个 eve agent 都恰好有一个沙箱**：一个隔离的、bash 风格的计算环境，有自己的文件系统（根目录是 `/workspace`）。它是**模型的草稿空间**，用来跑那些不可信的、模型自己生成的命令。

你**不需要写任何代码**就有一个默认沙箱；只有想定制沙箱后端或网络策略时，才加 `agent/sandbox.ts`（或 `agent/sandbox/` 目录）。

```ts
// agent/sandbox/sandbox.ts
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    vercel: { networkPolicy: "deny-all" },
    docker: { networkPolicy: "deny-all" },
  }),
});
```

> ⚠️ 注意（这是 eve 里最重要的安全区分，必须记住）：**工具跑在你的"应用运行时"里（能读 `process.env`、能 import `lib/`），而沙箱是另一个隔离的世界。** 沙箱是给模型生成的、不可信代码用的隔离草稿区。别把这两个混为一谈——把敏感凭证放进沙箱就违背了它存在的意义。上面例子里的 `networkPolicy: "deny-all"` 就是把沙箱的对外网络访问彻底锁死（在支持该策略的后端上生效）。

#### evals/ —— 自动化测试

**eval 是给 agent 打分的自动化检查。** 它让你的 agent 跑真实的会话，然后给结果评分——本质上就是 agent 版的"单元测试/集成测试"，用来防止你改了东西之后行为悄悄退步。

注意它的位置：`evals/` 目录是和 `agent/` **平级**的（在项目根目录下，不在 `agent/` 里面）。测试文件以 `.eval.ts` 结尾。

```ts
// evals/smoke.eval.ts
import { defineEval } from "eve/evals";
import { includes } from "eve/evals/expect";

export default defineEval({
  description: "Basic smoke test.",
  async test(t) {
    await t.send("What is the weather in Brooklyn?");
    t.completed();
    t.calledTool("get_weather");
    t.check(t.reply, includes("Sunny"));
  },
});
```

用 `eve eval` 命令运行；退出码为 `0` 表示所有"硬性检查"都通过了，方便接入 CI。`evals/` 目录下需要恰好有一个 `evals.config.ts` 配置文件（最简形式就是 `defineEvalConfig({})`）。

> 💡 小贴士：纯确定性的检查（比如"是否调用了某工具""回复里是否包含某字符串"）不需要任何"裁判模型"。只有当你用到"让另一个大模型来评判回复质量"这类断言（`t.judge.*`）时，才需要配一个 judge 模型。起步阶段，确定性检查就够用了。

#### instrumentation.ts —— 可观测性

**`instrumentation.ts` 负责接 OpenTelemetry，把 agent 运行的链路追踪导出到外部系统**（如 Braintrust、Datadog、Honeycomb 等）。

```ts
// agent/instrumentation.ts
import { defineInstrumentation } from "eve/instrumentation";
```

这个文件有个很特别的设计：**只要存在一个 `defineInstrumentation` 的默认导出，就自动启用了遥测导出**——没有什么 `isEnabled` 开关。eve 会自动发现 `agent/instrumentation.ts` 并在服务启动时、任何 agent 代码之前运行它。

> 💡 小贴士：**就算你完全不写这个文件，也照样能观测 agent。** eve 会自动给每次运行打上 `$eve.*` 属性，驱动 Vercel 上的 **Agent Runs** 视图——在那里你能看到每次运行的触发来源、token 用量、每一步的耗时、工具调用的入参和结果等等。`instrumentation.ts` 是给"想把数据导到 Vercel 之外"的进阶需求准备的，新手起步完全不用碰。

### 4.9 把地图收进脑子里

这一章信息量不小，但你真正要带走的，其实就三句话：

1. **文件即接口**：放对文件夹、起对名字，eve 自动发现——别找 `name`/`id` 字段，它们不存在。
2. **发现 → 编译 → 运行**：你放文件 → eve 扫描发现 → 编译成 `.eve/` 里的 manifest → 跑在 Vercel Functions 上。想确认发现了什么，用 `eve info`。
3. **起步只需两个文件**：`agent.ts`（选模型）+ `instructions.md`（定身份）。其余所有角色——工具、技能、连接、渠道、定时任务、沙箱、测试、可观测性——都是**等你真需要那个能力时再加一个文件**。

从下一章开始，我们就会沿着这张地图，一个角色一个角色地深入。现在你已经有了"全局视野"，再钻进细节就不会迷路了。

---

## 5. Tools：给 agent 一双手

到目前为止，你的 agent 已经有了"身份"（`instructions.md` 里的系统提示）和"大脑"（`agent.ts` 里选定的模型）。但它还只能说话，不能做事——你问它"布鲁克林现在天气怎么样？"，它要么瞎编，要么老实承认自己不知道。

这一章，我们给它装上"手"。在 eve 里，这双手就叫 **tool（工具）**。

### 5.1 什么是 tool？为什么需要它

> 💡 小贴士：一句话理解——**tool 是模型在对话过程中可以主动调用的一个"带类型的动作"**。它可以是"查一次天气 API"、"跑一条数据库查询"、"往某个后端写一条记录"。模型自己决定什么时候调、传什么参数；真正执行这个动作的，是你写的 TypeScript 代码。

打个比方：模型是一个很聪明但被关在房间里的人，它能思考、能说话，但碰不到外面的世界。tool 就是房间墙上的一排按钮，每个按钮旁边贴着一张说明（"按我可以查天气，需要告诉我城市名"）。模型读这些说明，决定该按哪个、怎么按；按下去之后，外面真正发生的事由你来实现，结果再递回房间给它看。

在 eve 里，每个 tool 就是 `agent/tools/` 目录下的**一个文件**。这呼应了 eve 最核心的设计原则：

> **"文件的名字和它在目录树里的位置，就是它的定义。"**

也就是说，你**永远不用**在代码里写 `name: "get_weather"` 这种字段——文件叫 `get_weather.ts`，这个 tool 在运行时的名字就是 `get_weather`。身份来自路径。

> ⚠️ 注意：tool 的代码运行在你的**应用运行时（app runtime）**里，可以读 `process.env`、可以 import 你在 `lib/` 里写的共享代码。它**不是**在沙箱（sandbox）里跑的。沙箱是后面单独一章会讲的、给模型用的"草稿环境"，和 tool 是两回事。这个区别很重要，先记住：**tool 代码 ≠ 沙箱代码**。

### 5.2 用 `defineTool` 写第一个 tool

eve 提供了一个工厂函数 `defineTool`，从 `eve/tools` 导入。你用它来描述一个 tool 的三件事：

- **`description`**：写给模型看的一句话说明，告诉它"这个 tool 是干什么的"。
- **`inputSchema`**：这个 tool 需要什么入参，用 zod 来定义和校验。
- **`execute`**：真正干活的函数，拿到入参、做事、返回结果。

我们先把完整的例子放出来，然后逐行拆解。新建文件 `agent/tools/get_weather.ts`：

```ts
// agent/tools/get_weather.ts
import { defineTool } from "eve/tools";
import { z } from "zod";

export default defineTool({
  description: "Get the current weather for a city.",
  inputSchema: z.object({ city: z.string().min(1) }),
  async execute({ city }, ctx) {
    return { city, condition: "Sunny", temperatureF: 72 };
  },
});
```

逐行解释：

```ts
import { defineTool } from "eve/tools";
```
从 `eve/tools` 导入工厂函数 `defineTool`。所有 tool 都要用它来定义。

> ⚠️ 注意：`defineTool` 不是可有可无的"装饰"。eve 在内部会给它打上一个标记（brand），用来确认"这确实是一个合法的 tool"。如果你直接 `export default { description: ..., ... }` 一个裸对象，eve 会拒绝它、报错。**一定要用 `defineTool` 包起来。**

```ts
import { z } from "zod";
```
导入 zod。zod 是一个用来"描述数据长什么样"的库——你用它写一份"入参规格说明书"，它就能在运行时帮你检查实际传进来的数据是否符合。eve 在 `init` 时已经帮你装好了 zod（连同 `eve`、`ai` 一起），直接用即可。

```ts
export default defineTool({
```
**用 `export default` 把这个 tool 作为文件的默认导出。** eve 在编译时会扫描 `agent/tools/` 下的每个文件，读取它的默认导出。

```ts
  description: "Get the current weather for a city.",
```
`description`（必填）。这句话**是给模型读的**，不是给人读的注释。模型靠它来判断"用户现在的需求，是不是该调用这个 tool"。所以要写得清楚、具体。这里写的是"获取某个城市的当前天气"。

```ts
  inputSchema: z.object({ city: z.string().min(1) }),
```
`inputSchema`（必填），用 zod 定义入参。`z.object({ ... })` 表示"入参是一个对象"，里面：
- `city: z.string().min(1)` —— 有一个叫 `city` 的字段，类型是字符串，且长度至少为 1（即不能是空字符串）。

eve 会把这份 schema 翻译成模型能理解的格式，告诉模型"调用我的时候，请给我一个 `city` 字符串"。模型生成入参后，eve 会**先用这份 schema 校验一遍**，校验通过才会真正执行你的代码。

> 💡 小贴士：`inputSchema` 除了 zod，也接受任何 Standard Schema，或者一个普通的 JSON Schema 对象；用 zod 的好处是 `execute` 里的入参类型能被 TypeScript 自动推断出来。`z.object({ ... })` 是最常用的写法，因为 tool 的入参几乎总是"一个带若干字段的对象"。你可以往里加更多字段，例如同时要城市和单位：
> ```ts
> inputSchema: z.object({
>   city: z.string().min(1),
>   unit: z.enum(["celsius", "fahrenheit"]),
> }),
> ```
> 如果这个 tool **完全不需要入参**，也不能省略 `inputSchema`——请传一个空对象 `z.object({})`。

```ts
  async execute({ city }, ctx) {
    return { city, condition: "Sunny", temperatureF: 72 };
  },
```
`execute`（必填），真正干活的函数。它有两个参数：

- 第一个是 **`input`**——就是校验过的入参。这里我们用解构 `{ city }` 直接把 `city` 取出来。因为入参是用 zod 定义的，TypeScript 会自动推断出 `city` 是 `string` 类型，写代码时有完整的类型提示。
- 第二个是 **`ctx`**（类型 `ToolContext`）——一个"上下文"对象，里面装着这次调用的会话信息（`ctx.session`）、拿沙箱句柄的方法（`ctx.getSandbox()`）、拿外部服务令牌的方法（`ctx.getToken(...)`）等等。这个例子里我们用不到，但它就在那儿，需要时随时取用。

`execute` 可以是同步的，也可以是 `async`（异步）的——真实场景里你多半要 `await` 一个网络请求，所以通常写成 `async`。这里我们为了演示直接返回了一个写死的结果。

返回值就是这个 tool 的产出。**默认情况下，你 return 的完整对象会原样递回给模型**，模型据此继续往下说话。

> ⚠️ 注意：返回值**必须是可 JSON 序列化的**——也就是普通的对象、数组、字符串、数字、布尔、`null`。`Date`、`Map`、`Set`、`NaN`、有循环引用的对象都不行，返回前请先转换成普通值（比如把 `Date` 转成字符串）。
>
> 还有一条安全铁律：**不要从 tool 里返回密钥、凭证、不必要的个人信息或大段敏感内容。** 返回前先过滤、最小化、脱敏。tool 的输出会进入模型的上下文，一旦返回就等于"给模型看了"。

### 5.3 tool 是怎么被命名和发现的

这一点前面提过，这里正式讲清楚，因为它是 eve 的核心机制：

- **发现（discovery）是自动的。** 你不需要在任何地方"注册"你的 tool，也不需要在 `agent.ts` 里列一个 `tools: [...]` 数组（事实上 `defineAgent` **根本不支持** `tools` 字段）。eve 在 `build` / `dev` 时会自动扫描 `agent/tools/` 目录，把每个文件都当作一个 tool 收进来。
- **名字来自文件名。** 文件名去掉扩展名后的那一段，就是模型看到的 tool 名字，规则是 **snake_case（下划线连接的小写 ASCII）**。所以：
  - `agent/tools/get_weather.ts` → tool 名 `get_weather`
  - `agent/tools/refund_charge.ts` → tool 名 `refund_charge`

> 💡 小贴士：想确认 eve 到底发现了哪些 tool、名字对不对？在项目根目录跑：
> ```bash
> npx eve info
> ```
> 它会列出当前 app 解析出来的全部 tool、skill、subagent、channel 等。这是检查"我的文件放对地方了吗、被发现了吗"的最快方法。

### 5.4 在对话里，这一切是怎么连起来的

光看代码可能还是抽象。我们把"一次真实对话"从头到尾走一遍，看 `get_weather` 是怎么被用上的。

假设你已经写好了上面的 `get_weather.ts`，然后启动本地开发服务器：

```bash
npx eve dev
```

现在你在对话里输入：

> What is the weather in Brooklyn?

接下来 eve 和模型之间发生了这样一串事：

1. **模型看到"工具清单"。** 在这次回答开始时，eve 已经把 `get_weather` 的 `description`（"Get the current weather for a city."）和它的入参 schema（需要一个 `city` 字符串）一起喂给了模型。注意：此时你的 `execute` 代码**一行都没跑**——模型只看到"说明书"，看不到实现。

2. **模型自己决定要调用它。** 模型读懂了你的问题，发现这正好匹配 `get_weather` 的描述，于是决定调用，并根据你的话生成入参：`{ "city": "Brooklyn" }`。**是模型主动决定的，你没有写任何"如果用户问天气就调 get_weather"的 if 判断。**

3. **eve 校验入参。** 在真正执行前，eve 拿你的 `inputSchema`（`z.object({ city: z.string().min(1) })`）去校验模型生成的 `{ city: "Brooklyn" }`。`"Brooklyn"` 是非空字符串，校验通过。

   > ⚠️ 注意：如果模型生成的入参不合法（比如漏了 `city`，或给成了数字），校验就会失败。这道关卡的存在，意味着你的 `execute` 函数里拿到的入参**一定是符合 schema 的**，你不用在函数开头再手写一堆"city 存在吗、是字符串吗"的防御性检查。（校验失败时 eve 抛给模型的具体错误形态，文档未明确写出，以官方文档为准。）

4. **eve 执行你的 `execute`。** 校验通过后，eve 才真正调用 `execute({ city: "Brooklyn" }, ctx)`。你的代码运行在应用运行时里，返回：
   ```ts
   { city: "Brooklyn", condition: "Sunny", temperatureF: 72 }
   ```

5. **返回值回到模型。** 这个对象被递回给模型，模型据此组织自然语言，回复你类似："布鲁克林现在天气晴朗，约 72°F。"

整个循环——**看说明书 → 决定调用 → 生成入参 → 校验 → 执行 → 拿结果继续说话**——就是 eve 替你管好的"agent loop"。你只需要写好那一个文件。

> 💡 小贴士：想亲眼看到这串过程？本地 `eve dev` 的终端 UI 会把每一次工具调用展示出来。部署到 Vercel 后，还可以在 **Agent Runs** 视图里看到每一步的时间线、模型决定调用了哪个 tool、传了什么入参、`execute` 返回了什么。

### 5.5 进阶一点点：只让模型看到"精简版"返回值（可选）

有时候你的 `execute` 想返回一个很丰富的对象（比如一段完整的接口响应），但你**不希望把这么多东西全塞进模型的上下文**——既费 token，又可能夹带模型不需要看的字段。

这时可以加一个可选字段 `toModelOutput`，专门用来"塑形"模型看到的那一份：

```ts
toModelOutput(output) {
  return { type: "text", value: `Report for ${output.domain}: score ${output.score}.` };
},
```

它接收你 `execute` 的**完整返回值**，但它的结果**只影响模型看到的内容**。返回 `{ type: "text", value: string }`（纯文本）或 `{ type: "json", value: unknown }`（结构化 JSON）二选一即可（这个函数也可以是 `async` 的）。

> 💡 小贴士：`toModelOutput` 只裁剪"给模型的那一份"。channel 的事件处理器、hooks 仍然能通过 `action.result` 拿到你 `execute` 的**完整**返回值。所以你可以让模型只读到一句话摘要，同时让 Slack 渲染出一张模型从没见过的精美卡片。这部分细节后面 channel 章节再展开，这里知道有这么个东西即可。

### 5.6 引出"审批（approval）"：有些动作不能让模型擅自做

查天气是只读的、没有副作用，模型想调就调、调多少次都无所谓。但有些动作**有真实副作用**——发一封邮件、扣一笔钱、下一单交易。这类动作你大概率**不想让模型完全自己说了算**。

eve 为此提供了"人工审批（approval）"机制：某个 tool 可以被设成"必须等人点头批准，才真正执行"。

最简单的用法是给 tool 加一个 `needsApproval` 字段。审批的几个预设来自 `eve/tools/approval`：

```ts
import { always, once, never } from "eve/tools/approval";

needsApproval: always()  // 每次调用都要人工批准
needsApproval: once()    // 仅本会话第一次调用要批准，之后放行
needsApproval: never()   // 默认值：不需要批准
```

举个有副作用的例子——退款：

```ts
// agent/tools/refund_charge.ts
import { defineTool } from "eve/tools";
import { always } from "eve/tools/approval";
import { z } from "zod";

export default defineTool({
  description: "Refund a charge.",
  inputSchema: z.object({ chargeId: z.string(), amount: z.number() }),
  needsApproval: always(),
  async execute(input) {
    return refund(input);
  },
});
```

加了 `needsApproval: always()` 之后，模型每次想调用 `refund_charge`，都不会直接执行，而是**停下来等一个人批准**。

> 💡 小贴士：审批的等待是"免费"且"持久"的。被审批拦住的会话会**停在那里安静等待，需要的话可以无限期地等下去，期间不消耗任何计算资源**；一旦有人批准，会话会从原地继续往下跑。这背后是 eve 的持久会话能力在支撑（前面概念章节提过的 Vercel Workflows，它用一份事件日志把会话停在原地）。

> ⚠️ 注意：`needsApproval` 还可以传一个函数，按入参动态决定要不要审批，比如"金额超过 1000 才需要批准"：
> ```ts
> needsApproval: ({ toolInput }) => (toolInput?.amount ?? 0) > 1000,
> ```

审批不只能"按工具"配，**也能"按连接（connection）"配**——也就是说，你可以让某个外部服务的**所有**操作都走审批。这正是后面会讲的连接章节里一个很重要的安全模式。比如那个真实的交易 agent 例子，就在它的交易连接上设了 `approval: always()`，让**每一笔交易都必须人工批准后才会执行**——这是它防止"真金白银被模型乱动"的核心安全控制。

> 这里先把"审批"这个概念铺垫到位：**它就是一道人工闸门，挡在有副作用的动作前面。** 具体到连接上怎么配、批准/拒绝时会发生什么、它和登录授权的先后顺序如何，我们留到"连接"那一章详细展开。

### 5.7 小结

这一章你学会了给 agent 装上手：

- 一个 tool 就是 `agent/tools/` 下的一个文件，用 `defineTool`（来自 `eve/tools`）定义。
- 三个核心字段：`description`（写给模型看的说明）、`inputSchema`（用 zod 定义并自动校验入参，最常用 `z.object({ ... })`，无入参就用 `z.object({})`）、`execute`（真正干活、返回可 JSON 序列化的结果）。
- tool 的名字来自**文件名**（snake_case），发现是**自动**的，不用注册；用 `npx eve info` 可以核对。
- 在一次对话里，模型**自己决定**调用哪个 tool、生成入参；eve 先**校验**入参，再**执行**你的代码，最后把返回值**递回模型**继续对话。
- tool 代码跑在**应用运行时**，不是沙箱；返回值默认全给模型看，可用 `toModelOutput` 裁剪。
- 对有副作用的动作，可以用 `needsApproval`（`always()` / `once()` / `never()` 或一个判断函数）要求**人工审批**后才执行——审批也能按连接整体配置，细节留到连接章节。

下一步，我们会看看比单个 tool 更"重"的能力组织方式（skill），以及如何把 agent 接到外部世界的真实服务上（connection）。

---

## 6. Skills：按需加载的"操作手册"

到目前为止，你已经学会了用 instructions.md 给 agent 一个"永久人设"，用 tools 给它一双"能干活的手"。但很快你会遇到一个新麻烦：有些知识太长了。

设想你的 agent 需要"按一套 12 步的退款流程办事"，或者"写周报时严格遵守公司的格式规范和措辞禁忌"。这些东西如果全塞进 instructions.md，会有两个问题：

1. instructions.md 是**每一轮对话都会被完整读一遍**的（它是"常驻系统提示词"）。把退款流程、周报规范、领域百科全都堆进去，等于每次哪怕用户只是说句"你好"，模型也得先把这几千字读一遍。
2. 上下文窗口（context window，模型一次能"看到"的文字总量）是有限的。塞太多无关内容，既费钱（按 token 计费），又容易"撑爆上下文"，把真正重要的信息挤出去。

skills 就是为解决这个问题而生的。

### 6.1 skill 是什么：一句话和一个类比

> 💡 小贴士：一句话理解 skill —— **它是写成 markdown 文件的"操作手册"，平时收在抽屉里，只有当前任务真正需要时才被拿出来读。**

打个比方：instructions.md 像是贴在你办公桌上、每天一抬头就看到的"工作守则"（语气、底线、身份）；而 skill 像是书架上一本本厚厚的《退款操作手册》《周报写作规范》—— 你不会把整个书架都背在脑子里，只有客户来退款时，你才会抽出那一本翻开照着做。

eve 的做法正是如此：

- 你把每个流程写成一个 markdown 文件，放进 `agent/skills/` 目录。
- eve 在**构建时（build time）**扫描这个目录，把每个 skill 的"简介"（description）连同一个内置的 `load_skill` 工具一起，告诉模型："我手上有这么几本手册，分别讲什么"。
- 当模型判断当前任务和某本手册相关时，它自己调用 `load_skill`，eve 就把那本手册的完整 markdown 内容**加载进当前这一轮对话**。

换句话说：**平时模型只看到一行简介；真要用了，才加载全文。** 这就避免了"把所有知识都常驻在 instructions 里撑爆上下文"。

> 💡 小贴士：知识库里有一句很精炼的官方表述 ——「Skills are larger procedures or reference material that the model loads on demand. Use skills for repeatable workflows, multi-step instructions, or domain knowledge that should not be part of the always-on prompt.」即：skills 是**按需加载**的较长流程或参考资料，适合放可复用的工作流、多步骤说明、不该常驻提示词的领域知识。

### 6.2 skill 和 tool 到底有什么不同？

新手最容易混淆的就是 skill 和 tool。它们都是写在 `agent/` 下的文件、都能扩展 agent 的能力，但本质完全不同。记住下面这张对照表，胜过读十遍文档：

| 维度 | Tool（工具） | Skill（技能） |
|---|---|---|
| 本质 | 一个**可执行的、有类型的函数** | 一段**给模型读的过程性知识/步骤** |
| 模型拿它做什么 | **调用**它，传入参数，拿回返回值 | **阅读**它，照着上面写的步骤去思考和行动 |
| 会不会跑代码 | 会，`execute()` 真的会执行 | 不会，它只是文字 |
| 典型用途 | 查天气、发起退款、写数据库 | "退款要先核对这 5 项，再分 3 步操作" |
| 加载时机 | 模型每轮都能看到工具清单 | 简介常驻，**全文按需加载** |

一句话总结这个区别：

> **tool 是"手"（能动手干活的函数），skill 是"说明书"（教模型该怎么干的文字）。**

它们经常是搭档关系。比如一个退款 skill 里可能会写：「第 3 步，调用 `refund_charge` 工具，金额参数填用户确认过的数字」—— 这里 skill 负责讲清楚**流程和判断**，真正动手退钱的是 `refund_charge` 这个 tool。

> ⚠️ 注意：知识库里有一句关键提醒 ——「**Skills add instructions, not tools.**」skill 加载进来的只是**指令文字**，它**不会**给 agent 新增任何可调用的工具。如果你需要一个新的可执行动作，那永远是写 tool（见第 5 章），而不是写 skill。

> 💡 小贴士：还有第三个概念叫 **subagent（子 agent）**，新手阶段先有个印象即可。知识库的官方说法是：「Unlike a skill, which adds instructions to the running agent, a subagent runs as a separate agent with fresh conversation history and state.」即 skill 是给**当前**这个 agent 加指令，而 subagent 是另起一个"对话历史和状态全新"的独立 agent。本章只讲 skill。

### 6.3 文件放哪、怎么写：最小可运行示例

eve 的核心理念是"文件的名字和位置就是它的定义"。skill 也不例外：**把 markdown 文件放进 `agent/skills/` 目录就行了，不需要在任何地方"注册"它。**

#### 写法一：一个单独的 markdown 文件（最简单）

最朴素的写法是直接放一个 `.md` 文件，**第一行非空文字就是给模型看的"路由提示"**（也就是模型用来判断"这本手册该不该翻开"的依据）。注意：这种无 frontmatter 的写法里，文件**不要**写 Markdown 标题（如 `# ...`），第一行非空文字会被直接当作路由提示。

```md
Use when the user wants to plan a trip or build an itinerary.
Gather the destination, dates, and budget first, then propose a day-by-day plan.
```

把上面的内容存为 `agent/skills/plan_a_trip.md` 即可。文件名 `plan_a_trip.md` 决定了这个 skill 的名字（`plan_a_trip`），第一行那句 `Use when the user wants to plan a trip…` 就是模型判断要不要加载它的线索。

#### 写法二：带 frontmatter 的 markdown（更可靠的路由，推荐）

当你希望模型更准确地判断"什么时候该用这个 skill"时，建议用 frontmatter（文件顶部用 `---` 包起来的元信息）显式写一个 `description` 字段。习惯上把这个文件命名为 `SKILL.md`，单独放在一个以 skill 命名的子目录里。下面这段内容存为 `agent/skills/research/SKILL.md`：

```md
---
description: Research unfamiliar topics before answering with confidence.
---

When the task is novel or ambiguous, gather evidence first...
（下面继续写你的详细步骤）
```

> ⚠️ 注意：知识库中明确记录的 frontmatter 字段**只有 `description`**（字符串）一个。`name`、`version`、`tags` 之类的字段在知识库里**没有出现**，请不要凭印象写上去 —— 一切以官方文档为准。另外，"`description` 到底是必填还是只是强烈推荐"这一点尚不完全确定，所以稳妥起见，凡是想让路由可靠的 skill，都老老实实写上 `description`。

#### 写法三：多文件的 skill（一个目录）

如果一本"手册"还需要带附属资料（参考清单、模板、脚本等），可以用目录形式组织，里面**必须有一个带 `description` frontmatter 的 `SKILL.md`** 作为入口：

```
agent/skills/research/
├── SKILL.md          # 入口，必须含带 description 的 frontmatter
├── references/       # 参考资料
├── assets/           # 素材
└── scripts/          # 脚本
```

> 💡 小贴士：skill 在运行时会被放进沙箱（sandbox）的 `/workspace/skills/...` 目录里。沙箱是后面章节的内容，这里你只需知道：skill 的附属文件最终能在 agent 的"草稿空间"里被取用。

#### 关于 `defineSkill`（TypeScript 写法）

除了写 markdown，eve 也提供了一个 `defineSkill` 函数（从 `eve/skills` 导入），让你用 TypeScript 来定义 skill —— 这在需要"根据用户动态生成手册内容"时才用得上，属于进阶玩法。它的基本样子是这样的：

```ts
import { defineSkill } from "eve/skills";

export default defineSkill({
  description: "Research unfamiliar topics before answering with confidence.",
  markdown: "When the task is novel or ambiguous...",
  files: {
    "references/checklist.md": "# Checklist\n\n- Find primary sources.\n",
  },
});
```

知识库中记录的 `defineSkill` 字段就这三个：`description`（字符串）、`markdown`（字符串）、`files`（"相对路径 → 文件内容"的映射）。除此之外的选项知识库没有写，请不要自行发挥 —— 进阶用法以官方文档为准。

> 💡 小贴士：新手阶段，你 99% 的需求用"写法一/写法二"的纯 markdown 就能搞定。`defineSkill` 先知道有这么个东西即可，等真的需要"每个用户加载不同手册"时再回头查文档。

#### 还能用 CLI 安装现成的 skill

你也不必什么都自己写。社区有一个 skills 目录站（`https://skills.sh`），可以用一条命令把别人写好的 skill 装进你的 `agent/skills/`：

```bash
npx skills add vercel-labs/agent-skills
```

> ⚠️ 注意：这里的 `skills` 是一个**独立的命令**（来自单独的 `skills` 包），和 `eve` 这个命令不是一回事。在 eve 项目里运行它时，它会检测到你在用 eve 并询问：

```text
Detected an eve project. Install skills for eve?
● Yes / ○ No
```

选 **Yes**，它就会装进 `agent/skills/` 目录；选 **No**，则会装给你本地的 AI 编码助手（而不是你的 eve agent）。

### 6.4 怎么确认 skill 真的被加载了？

写完 skill 后，你肯定想验证它有没有生效。方法很直接：

1. 跑起本地开发服务器：

```bash
npx eve dev
```

2. 给 agent 发一句"应该会触发这个 skill"的话（比如对上面那个 `plan_a_trip` skill，就说"帮我规划一趟旅行"）。
3. 打开 **Agent Runs**（运行记录）视图，确认其中出现了模型调用 `load_skill` 的记录 —— 这就证明 skill 被成功匹配并加载了。

> 💡 小贴士：你也可以用 `npx eve info` 命令，它会列出 eve 在你项目里发现的所有 tools、skills、subagents、schedules、channels 等。如果你新写的 skill 没出现在列表里，多半是文件位置或文件名出了问题。

### 6.5 判断标准：什么时候用 skill，什么时候塞进 instructions？

这是本章最该带走的实战判断力。给你一组可以直接套用的标准。

**把内容放进 instructions.md，当它满足：**

- 是 agent 的**永久身份/人设**：语气、它是谁、它代表谁。
- 是**每一轮都适用**的标准约束：始终保持礼貌、绝不泄露密钥、永远用中文回复……
- **短而稳定**。知识库的建议是：instructions 要"short and stable"。

**把内容写成 skill，当它满足下面任意一条：**

- 是一段**较长的流程或参考资料**（多步骤说明、领域百科、规范手册）。
- 是**可复用的工作流**（repeatable workflow）—— 同一套步骤会在不同任务里反复用到。
- 它**只在特定情况下才相关**：大部分对话根本用不上它，没必要让它每轮都占着上下文。
- 一旦你发现 instructions.md 越写越长、开始往里堆"操作步骤"，那就是该把它们搬进 skill 的信号了。

用一张表收尾，把这两者的分工钉死：

| 维度 | Instructions | Skills |
|---|---|---|
| 加载方式 | 常驻，每一轮都加载 | 按需，经 `load_skill` 加载 |
| 用途 | 永久身份 / 标准规则 | 可选流程 / 参考资料 |
| 能否跑代码 | 不跑代码（要执行就用 tool） | 不跑代码（要执行就用 tool） |

> 💡 小贴士：知识库甚至把"把可复用的指令搬进 skill、只在相关时加载"列为一条**省钱建议** —— 因为常驻在 instructions 里的每个字，都会在每一轮对话里被重新计费一次。skill 按需加载，自然就省下了大量无谓的 token 开销。一举两得：上下文更干净，账单更便宜。

---

到这里，你已经掌握了 eve 的三件"能力积木"中的两件半：instructions（常驻人设）、tools（能动手的函数），以及本章的 skills（按需翻开的操作手册）。记住那个核心判断：**要"执行"就写 tool，要"教模型怎么做"且内容长又不常用，就写 skill，剩下短而永恒的才留给 instructions。** 下一章我们将进入更激动人心的部分 —— 让 agent 连接外部世界。

---

## 7. Connections：让 agent 连接外部世界（MCP）

到这一章为止，我们的 agent 已经会"说话"（instructions）、会"动手"（tools）、能在需要时翻阅"操作手册"（skills）。但它能做的事，全都局限在你亲手写进 `agent/tools/` 里的那几个函数。

可现实世界里，有太多能力是别人早就做好了的：Linear 上的 issue、GitHub 上的 PR、一个交易所的行情接口……你总不能为每一个外部服务，自己从零手写一遍 API 调用、塞进 tool 里吧？

这就是 **connections（连接）** 要解决的问题。

### 7.1 connection 是什么：一根"即插即用"的能力插头

> 💡 小贴士：打个比方——tool 是你自己焊死在主板上的一个芯片；connection 则是一个 **USB 接口**，你把一台外部设备（比如一个 MCP 服务器）插上去，它自带的一堆功能就自动出现在 agent 面前，随取随用。

更准确地说，**connection 是 agent 与某个外部服务之间一条带类型、带鉴权的"通道"**。你在 `agent/connections/` 下放一个文件声明这条通道，eve 就会：

1. 去远端把对方提供的工具列表"问"出来；
2. 把这些工具交给模型，让模型知道"我现在还能调用这些东西"；
3. 帮你打理鉴权（token），并且——**模型永远看不到这条连接的 url 和凭证**。

这最后一点很关键：URL 和 token 这类敏感配置，被关在连接定义里，既不进模型的 prompt，也不进工具实现细节。模型只知道"有这么个工具能用"，不知道它背后连去哪、用什么钥匙。

eve 里只有**两种** connection，都从 `eve/connections` 导入：

- `defineMcpClientConnection` —— 连接一个 **MCP 服务器**（本章主角）；
- `defineOpenAPIConnection` —— 连接任何带有兼容 OpenAPI 3.x 文档的 API。

> ⚠️ 注意：**`defineConnection` 这个 API 不存在**。只有上面这两个。别凭直觉写一个通用的 `defineConnection`，编译不会认。

本章我们专讲第一种，也就是最常见的 **MCP 连接**。

#### 什么是 MCP？

MCP（Model Context Protocol，模型上下文协议）你可以先简单理解为：**一种让"工具提供方"和"AI 应用"对话的标准插头规格**。只要一个服务按 MCP 规范开了个"MCP 服务器"，任何支持 MCP 的 AI 框架（eve 就是其中之一）都能把它的工具接进来用。这就是为什么市面上越来越多服务（Linear、GitHub 等）直接提供官方 MCP server——一次实现，处处可接。

### 7.2 文件即定义：connection 的名字从哪来

和前面 tools、skills 一样，eve 是**文件系统优先**的：connection 的名字，来自**文件名**，你不写 `name` 字段。

```text
agent/connections/linear.ts   →  连接名 "linear"
```

> ⚠️ 注意（本机实测坑）：**连接文件名要用 dash-case（短横线小写）**。真实项目 muse-agent 里的文件就叫 `okx-read.ts`、`okx-trade.ts`。养成这个习惯，能避开一类莫名其妙的命名问题。

### 7.3 第一个 MCP 连接：最小可运行示例

我们先看一个最小、能跑的写法。把下面这段放进 `agent/connections/linear.ts`：

```ts
// agent/connections/linear.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: "https://mcp.linear.app/sse",
  description: "Linear workspace: issues, projects, cycles, and comments.",
  auth: {
    getToken: async () => ({ token: process.env.LINEAR_API_TOKEN! }),
  },
});
```

逐个字段拆解：

- **`url`（必填）** —— 对方 MCP 服务器的地址。**这个地址必须说 Streamable HTTP 或 SSE 协议**（这点是个大坑，7.7 节专门讲）。
- **`description`（必填）** —— 一句给**模型看**的话，描述这条连接里大概有些什么能力。模型靠它来判断"我该不该来这里找工具"，所以要写得清楚、面向模型。
- **`auth.getToken`** —— 一个返回 token 的异步函数。eve 会用它拿到的 token，以 `Authorization: Bearer <token>` 的形式发给远端。

如果你连的是一个**本地、不需要鉴权**的 MCP 服务器，可以直接把 `auth` 整个去掉：

```ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: "http://localhost:3001/mcp",
  description: "Local dev server.",
});
```

> 💡 小贴士：关于 `auth.getToken` 的返回值——它返回一个 `TokenResult`，形如 `{ token, expiresAt? }`。`expiresAt` 是可选的过期时间（以毫秒时间戳计）。eve 对 token 的缓存是**按步（per-step）**进行的，凭证从不会到达模型那一侧。

> ⚠️ 注意：**绝不要给"敏感的第三方服务"用无鉴权连接。** 去掉 `auth` 只适合三种场景：故意公开的服务、纯本地服务、或本身已被别的机制保护起来的服务。

### 7.4 模型怎么"发现"并调用这些工具

声明了连接，但模型怎么知道连接里有哪些工具、又怎么调用它们？这里有两个关键机制。

#### connection_search：模型的"工具搜索引擎"

只要你的 agent 声明了**至少一个** connection，eve 就会自动给模型多塞一个内置工具：**`connection_search`**。

它的作用，官方描述是："在已声明的各个连接里发现工具；匹配到的工具随即变为可直接调用。"换句话说，模型不会一上来就背下所有外部工具（那会撑爆上下文），而是**先用 `connection_search` 去"搜"它当下需要的工具**，搜到了，那些工具才"浮现"出来变得可调用。

> 💡 小贴士：这也是为什么真实项目 muse-agent 的 `instructions.md` 里会专门提一句 `connection_search`——是在提醒模型："想用外部能力？先去 search 一下。"你在写自己的 instructions 时，也可以加这么一句引导。

#### 调用名长这样：`<connection>__<tool>`

被发现的工具，调用时用的是**限定名（qualified name）**，格式是：

```text
<连接名>__<工具名>
```

中间是**两个下划线**。举例：

- linear 连接里的 `list_issues` 工具 → 调用名 `linear__list_issues`
- muse-agent 的 okx-read 连接里的工具 → 调用名 `okx-read__<tool>`

这个命名规则的好处是：来自不同连接的工具天然带了"命名空间"，绝不会撞名。

### 7.5 真实对照：只读连接 vs 写操作连接

光看 Linear 的例子还不够直观。我们来看真实生产项目 **muse-agent**（一个跑在 eve 上的 OKX 交易 agent）里的两条连接。它俩几乎一模一样，**唯一的关键差别，就是一个安全闸**——这正是本章最重要的部分。

#### okx-read.ts：只读、无 approval

```ts
// agent/connections/okx-read.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: process.env.OKX_READ_MCP_URL ?? "http://127.0.0.1:8000/mcp",
  description: "...",
  auth: {
    getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }),
  },
});
```

这条连接接的是"读"——查行情、查账户余额这类操作。**它没有 `approval` 字段**，所以连接里的**每一个工具都是一次普通查询**，模型想调就调，不需要任何人点头。读数据是安全的，这样最顺手。

它的工具会以 `okx-read__<tool>` 的形式被调用。

#### okx-trade.ts：写操作、`approval: always()`

```ts
// agent/connections/okx-trade.ts
import { defineMcpClientConnection } from "eve/connections";
import { always } from "eve/tools/approval";

export default defineMcpClientConnection({
  url: process.env.OKX_TRADE_MCP_URL ?? "http://127.0.0.1:8000/mcp",
  description: "...",
  auth: {
    getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }),
  },
  approval: always(),
});
```

这条连接接的是"写"——**真金白银的下单交易**。它和上面那条几乎一样，但多了两处：

1. 顶部多了一行 `import { always } from "eve/tools/approval";`
2. 配置里多了一行 `approval: always()`

就这两行，构成了整个 agent 的**"人在回路"（human-in-the-loop）安全闸**。

### 7.6 `approval: always()` 到底做了什么

我们把这件事讲透，因为它是 eve 最有价值的设计之一。

`approval` 字段控制"这条连接上的工具被调用前，要不要先经过人工批准"。它接收三种取值，都来自 `eve/tools/approval`：

```ts
import { never, once, always } from "eve/tools/approval";

approval: never()   // 不需要批准（这也是省略 approval 时的默认行为）
approval: once()    // 只在本 session 第一次调用时要批准，之后放行
approval: always()  // 每一次调用都要批准
```

当 okx-trade 用了 `approval: always()`，会发生这样一串事：

1. 模型决定调用 okx-trade 上的某个下单工具。
2. 在工具**真正执行之前**，这次调用被拦下来。整个 session **durably（持久地）停在 `session.waiting` 状态**，等一个人来批准。
3. 这个"等待"是**不烧钱的**——它靠底层 Vercel Workflows 的事件日志把现场持久化下来，**可以一直等下去，哪怕等到天荒地老，也不消耗任何计算资源**。
4. 人批准了 → session 从原地**恢复**，工具继续执行，下单成功。人拒绝了 → 工具不执行。

> 💡 小贴士：为什么这能"等很久还不烧钱"？回忆第 1 章讲的 **durable session（持久会话）**：eve 的 session 跑在 Vercel Workflows 上，每一步都被记进事件日志、可以被精确重放。所以一个停在审批点的 session，本质上只是"日志里记了一笔，等下一个事件"，自然不占用任何运行中的计算。

对一个会动用真钱的 agent 来说，这意味着：**模型可以自由地思考、决策、甚至"想"下单，但它永远无法在没有人点头的情况下，真的把单子打出去。** 这就是"人在回路"安全闸的全部意义——把"AI 自动化"和"人类最终拍板"干净利落地焊在了一起。

> ⚠️ 注意：`approval` 是 connection 这一层的闸，针对**整条连接上的所有工具**。它和单个 tool 上的 `needsApproval` 是同一套 `always / once / never` 取值（都来自 `eve/tools/approval`），只是作用对象不同——一个管整条连接，一个管单个工具。

#### 对照表：muse-agent 的两条连接

| | `okx-read.ts` | `okx-trade.ts` |
|---|---|---|
| 用途 | 读：查行情、查余额 | 写：真实下单交易 |
| `approval` 字段 | 无（即 `never()`） | `always()` |
| 调用时 | 模型直接调，立即执行 | 每次都先停在 `session.waiting` 等人批准 |
| 调用名 | `okx-read__<tool>` | `okx-trade__<tool>` |
| 安全性 | 查询无副作用，安全 | 真金白银，必须人工把关 |

> 💡 小贴士：这是一个非常值得照搬的模式——**把"读"和"写"拆成两条独立的连接，只给"写"那条挂 `approval`**。粒度清晰，安全边界一眼可见。

### 7.7 头号大坑：stdio-only 的 MCP server 连不上

这是新手最容易栽的地方，必须单独拎出来讲。

#### 协议要求：必须是 Streamable HTTP 或 SSE

还记得 7.3 节那句"`url` 必须说 Streamable HTTP 或 SSE"吗？eve 的连接**只支持这两种传输方式（transport）**：

- **Streamable HTTP**
- **SSE**（Server-Sent Events）

但现实中，**很多 MCP 服务器只支持 `stdio`**（标准输入/输出）这种传输方式——它们被设计成在本地、由父进程直接拉起来、通过管道通信，根本不开 HTTP 端口。

> ⚠️ 注意：**stdio-only 的 MCP server，不能直接连到 eve。** 你在 `url` 里也填不出一个合法地址来——它压根没有 HTTP/SSE 端点。

#### 解法：架一个 HTTP/SSE bridge（桥接）

办法是：在 stdio MCP server **前面**架一个 **bridge（桥接/代理）**，由它把 stdio 转换成 HTTP/SSE，然后让 eve 去连这个 bridge 暴露出的 HTTP 地址。

真实项目 muse-agent 就是这么做的：它要用的 `okx-trade-mcp` 是个 stdio-only 的服务器，于是用 **supergateway** 这个工具把 `stdio → HTTP` 转了一道，eve 连的是 supergateway 暴露出来的 HTTP 端点。

整条链路长这样：

```text
eve connection  ──HTTP──▶  supergateway（bridge）  ──stdio──▶  okx-trade-mcp（stdio-only）
```

> 💡 小贴士：能做这件事的桥接工具不止一个，社区里还有 `mcp-proxy`、`mcp-bridge`、`mcp-remote` 等。不过这些都**不是 eve 官方捆绑或文档化的工具**，eve 文档既不内置也不指定某个 bridge。具体选哪个、怎么配，请以各工具自己的文档为准。这里你只需要记住那条铁律：**stdio MCP server 不能直连 eve，中间需要一个 HTTP/SSE bridge。**

### 7.8 第二个大坑：url 会在编译期被"急切校验"

最后一个本机实测坑，也会让你 fail fast（快速失败）。

eve 在**发现/编译阶段就会急切地（eagerly）校验你的连接 url**。也就是说，如果你把 url 写错了、写空了、或者写成一个解析不出来的字符串，**编译会当场失败**，而不是等到运行时模型真去调用才报错。

> ⚠️ 注意：这意味着，连接的 url **在编译那一刻就得能解析成一个合法地址**。如果你像 muse-agent 那样用环境变量兜底：
>
> ```ts
> url: process.env.OKX_READ_MCP_URL ?? "http://127.0.0.1:8000/mcp",
> ```
>
> 那个 `??` 后面的兜底地址就很重要——它保证了即使环境变量没设，url 也始终是一个合法的字符串，编译期校验就能过。

这其实是好事：eve 宁可在你按下编译键的那一秒就告诉你"url 错了"，也不愿意让一个坏连接悄悄溜到生产环境、等真有人用的时候才崩。这叫 **fail fast**，是一种对你友好的"早失败"设计。

### 7.9 收尾：可选的工具过滤

最后补一个实用的小功能。一条 MCP 连接可能暴露出几十个工具，但你未必想全都给模型用——尤其当对方服务器里混着一些危险的"写"工具时。你可以用 `tools` 字段做过滤，**二选一**（`allow` 和 `block` 只能用其中一个）：

```ts
// 白名单：只允许这两个工具，其余一律不给模型
tools: { allow: ["search_issues", "get_issue"] }
```

```ts
// 黑名单：屏蔽掉这个危险工具，其余照常
tools: { block: ["delete_issue"] }
```

- 官方建议**优先用 `allow`**，尤其是面对那种暴露了写操作的 MCP 服务器时——白名单能给出"最小安全暴露面"。
- 被过滤掉的工具**不会出现在 `connection_search` 里**，模型连看都看不到，自然也无从调用。

> 💡 小贴士：`allow`/`block` 列表里到底是精确匹配工具名，还是支持正则之类的模式，eve 文档目前没有明确说明。保险起见，就按"精确工具名"来写；如有更复杂的需求，以官方文档为准。

---

**本章小结**

- connection 是 agent 接入外部能力的"插头"，最常见的是 MCP 连接，用 `defineMcpClientConnection`（来自 `eve/connections`）声明。
- 核心字段：`url`（必填，须为 Streamable HTTP/SSE）、`description`（必填，写给模型看）、`auth.getToken`（返回 token）、`approval`（人工批准闸）。
- 模型用内置的 `connection_search` 发现工具，调用名形如 `<connection>__<tool>`。
- **`approval: always()` 是核心安全设计**：让每次调用都 durably 停在 `session.waiting` 等人批准、零计算消耗地等待、批准后原地恢复——这就是"人在回路"安全闸，muse-agent 用它守住真金白银的下单。
- 三个必记的坑：连接文件名用 **dash-case**；url 在**编译期被急切校验**（写错即 fail fast）；**stdio-only 的 MCP server 连不上，需要 HTTP/SSE bridge**（如 supergateway）。

---

## 8. Channels：对外入口与鉴权（别把门敞开）

前面几章里，我们一直在 `eve dev` 这个温暖的小窝里跟 agent 聊天。但你心里肯定憋着一个问题：**部署上线之后，外面的世界到底怎么"敲门"找到我的 agent？谁又能拦住不该进来的人？**

这一章就专门讲这两件事：

- **Channels（频道）** —— agent 对外开的"门"。
- **Auth（鉴权）** —— 守在门口的"保安"。

记住一句话，这是本章的灵魂：**eve 默认是"fail closed"（失败即关闭）的——门默认是锁死的，你不主动配一个能放行的保安，谁都进不来（返回 401）。** 这是好事，别嫌它麻烦。下面我们一点点把它讲透。

### 8.1 Channel 是什么：agent 对外的入口

> 💡 小贴士：可以把 channel 理解成"翻译 + 门卫"。外面的世界有各种形态——浏览器、命令行 `curl`、Slack 消息、Discord 指令……它们说的"话"格式都不一样。Channel 的职责就是把这些五花八门的输入，统一翻译成 agent 能听懂的一句"用户消息"，再把 agent 的回答送回去。

在 eve 里，channel 是**文件优先**的（跟前面学的 tool、connection 一样）：

- 所有 channel 文件放在 `agent/channels/` 目录下。
- **文件名（去掉扩展名）就是 channel 的 id**，导出方式是 `export default`。
- channels 是**根级专属**的——子 agent（subagent）不能声明自己的 channel。

这里有一个对新手特别友好、但也特别容易踩坑的事实：

> ⚠️ 注意：**每个 eve 应用默认就已经暴露了一个 HTTP channel，哪怕你的 `agent/channels/` 里一个文件都没有。** 也就是说，你什么都不写，agent 也能通过 HTTP 被访问到。你创建 `agent/channels/eve.ts` 这个文件，通常**不是为了"开启"它，而是为了"覆盖默认配置"——最常见的目的就是配置鉴权（auth）。**

eve 还内置了一批"平台 channel 工厂函数"，各对应一个外部平台：

| 工厂函数 | 来自 | 对应平台 |
|---|---|---|
| `eveChannel` | `eve/channels/eve` | 默认 HTTP API（浏览器 / curl / SDK） |
| `slackChannel` | `eve/channels/slack` | Slack |
| `discordChannel` | `eve/channels/discord` | Discord |
| `teamsChannel` / `telegramChannel` / `twilioChannel` / `githubChannel` | `eve/channels/<平台>` | 对应平台 |

除了这些，还能用 `defineChannel`（来自 `eve/channels`）手写完全自定义的 channel。本章我们把火力集中在最基础、也最重要的那个：**默认的 `eveChannel`**。

> 💡 小贴士：怎么判断该用哪个 channel？简单记：浏览器/网页用 eve channel（配合前端 `useEveAgent`）；本地脚本、SDK、`curl` 也走 eve channel（就是默认那个）；Slack/Discord 等聊天平台用它们各自的 channel；其它特殊场景才动手写 `defineChannel`。

### 8.2 默认的 eveChannel 暴露了哪些 HTTP API

`eveChannel()` 会把一组**标准的会话路由**挂载到 `/eve/v1/session*` 路径下。这组路由默认就有，即使你没写 `agent/channels/eve.ts`。

先看一个最小的覆盖写法，把骨架立起来：

```ts
// agent/channels/eve.ts
import { eveChannel } from "eve/channels/eve";
import { localDev, vercelOidc } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc()],
});
```

这个 channel 暴露的核心 HTTP 路由如下：

| 方法 + 路径 | 作用 |
|---|---|
| `GET /eve/v1/health` | 健康检查。**永远公开**，跳过鉴权（给负载均衡 / 监控用）。 |
| `POST /eve/v1/session` | **开一个新会话**，发出第一条消息。 |
| `POST /eve/v1/session/:sessionId` | 给已有会话**发后续消息**（follow-up）。 |
| `GET /eve/v1/session/:sessionId/stream` | **流式拉取**这个会话产生的事件（NDJSON 格式）。 |
| `GET /eve/v1/info` | 返回一份 JSON 快照，用于查看模型、工具、技能、channels 等配置。 |

我们逐个来理解这条"一来一回"的链路。

**第一步：开会话。** 你向 `POST /eve/v1/session` 发一条消息，eve 帮你创建一个**持久会话（durable session）**，并在返回里给你两个关键句柄：

- `sessionId` —— 这个会话的唯一 id（响应头里也有一个 `x-eve-session-id`）。
- `continuationToken` —— "续接令牌"，是这个对话表面（surface）的**恢复句柄**，发后续消息时要带上它。

> 💡 小贴士：你可以把 `sessionId` 想成"这通对话的档案编号"，把 `continuationToken` 想成"接着往下说时要出示的接力棒"。一个会话同一时刻只有一根有效的接力棒，过期的旧令牌会被拒绝。

**第二步：流式接收回答（NDJSON）。** agent 干活是个**长时间、逐步推进**的过程：它可能调工具、加载技能、写沙箱文件……所以它不是"一次性返回一大坨"，而是**一行一个事件地流式吐出来**。这种格式叫 **NDJSON**（Newline-Delimited JSON，即"每行一个 JSON 对象"），响应的 content-type 是 `application/x-ndjson; charset=utf-8`。

```bash
curl http://127.0.0.1:3000/eve/v1/session/<sessionId>/stream
```

你会看到类似 `session.started`、`turn.started`、`message.appended`、`message.completed`、`session.completed` 这样一行行的事件。如果连接断了想续上，可以按事件序号"快进"：

```bash
curl "http://127.0.0.1:3000/eve/v1/session/<sessionId>/stream?startIndex=<已收到的事件数>"
```

**第三步：发后续消息。** 当 agent 这一轮干完、进入等待状态（你会在流里看到 `session.waiting`），你就可以接着这个会话往下说——注意要带上 `continuationToken`：

```bash
curl -X POST http://127.0.0.1:3000/eve/v1/session/<sessionId> \
  -H 'content-type: application/json' \
  -d '{"continuationToken":"<token>","message":"那再帮我查一下皇后区。"}'
```

> ⚠️ 注意：上面响应体里出现的 `continuationToken`、`sessionId` 的具体字段名和取值格式（比如令牌带 `eve:` 前缀、id 形如 `ses_01h...`）在不同文档页里略有出入，属于"示意性"的样子。真正可靠的拿法是：**从你这次开会话的响应里读出实际值，再原样用回去**，不要把示例里的格式当成铁律写死。

### 8.3 鉴权的核心：auth 是一个"按顺序匹配"的数组

现在到了本章最关键的部分。门开好了，得有保安。

在 `eveChannel({ auth: [...] })` 里，`auth` 既可以是单个鉴权器，也可以是一个**数组**。当它是数组时，里面的每一项都是一个"鉴权器（authenticator）"。eve 会**按顺序**一个一个地走这个数组，规则非常简单清晰：

- 某一项返回了一个**有效的会话鉴权（`SessionAuthContext`）** → **接受，立刻停止往下走**（"第一个匹配上的胜出"）。
- 某一项返回 `null` / `undefined`（表示"这不归我管"）→ **跳过，试下一项**。
- 某一项**主动抛出错误** → 以一个明确的状态码**拒绝**（比如 401 / 403）。

最关键的一条收尾规则：

> ⚠️ 注意：**如果数组里每一项都跳过了（没人认领），最终结果就是 `401`。** 这就是 "fail closed / 默认拒绝"。极端例子：`auth: []`（空数组）会**拒绝一切请求**——因为没有任何一项能放行。

官方对此有一句几乎是逐字的描述：**eve 默认 fail closed——生产环境的浏览器流量会被直接拒绝，除非你配置了一个明确接受它的鉴权器；而要允许匿名访问，必须显式地放一个 `none()`。**

这套"按顺序走、第一个匹配胜出、全不匹配就 401"的逻辑，守护的是这三条会真正干活的路由：

- `POST /eve/v1/session`
- `POST /eve/v1/session/:sessionId`
- `GET /eve/v1/session/:sessionId/stream`

而 `GET /eve/v1/health` 永远是公开的，不走这套鉴权。

> 💡 小贴士：理解了"数组按顺序匹配"，你就能理解一个重要的排列原则——**把你自己的、更具体的鉴权器放在前面，把兜底的 catch-all helper 放在后面。** 顺序就是优先级。

### 8.4 逐个认识 eve/channels/auth 里的鉴权器

这些鉴权器都从 `eve/channels/auth` 导入。下面挑本章重点要求覆盖的几个讲透，其余的列个表知道有就行。

#### localDev()：只放行"本地 / Vercel dev"

```ts
import { localDev } from "eve/channels/auth";
```

它的作用是：**只有当请求是发给本机回环地址（loopback）时，才放行**，并认证为一个合成的 `local-dev` 身份。所谓回环地址，就是 `localhost`、`*.localhost`、`127.0.0.0/8`、`::1` 这些"指向本机"的主机名。

它判断的依据是**请求 URL 里的主机名（hostname）**，而不是看 `process.env.VERCEL` 这种环境变量。另外有一个进程级的例外：当你跑 `vercel dev`（此时 `VERCEL=1` 且 `VERCEL_ENV=development`）时，即使主机名不是回环地址，它也会放行本地开发服务器。

> ⚠️ 注意（这是个真坑）：`localDev()` 信任的是请求里"自报"的主机名。如果你的部署前面没有一个会规范化请求的代理，攻击者就有可能通过伪造 `Host` 头来冒充本地请求、骗过 `localDev()`。所以：**永远不要只靠 `localDev()` 一个人守门**，一定要在它上面叠一个真正的鉴权器（比如下面的 `httpBasic` 或 `vercelOidc`）。

#### vercelOidc()：放行 Vercel 签发的部署令牌

```ts
import { vercelOidc } from "eve/channels/auth";
```

这是部署到 Vercel 时**最常用**的那条路径。它会校验请求里携带的一个 **Vercel OIDC** 身份令牌（一个 bearer JWT）。

通俗讲：当你的 agent 跑在 Vercel 上，平台内部那些合法的调用方（比如你自己项目的运行时、子 agent 之间的调用）会自带一个由 Vercel 签发的令牌，`vercelOidc()` 能认出并放行它们——零配置就能用。**凡是为当前 `VERCEL_PROJECT_ID`（也就是你自己这个项目）签发的令牌，它总是接受。**

它同样是 fail closed 的：当相关的 Vercel 环境变量（如 `VERCEL_PROJECT_ID`）没设置时，它会返回"不匹配"，而不是稀里糊涂放行。

> 💡 小贴士：如果你**不是**部署在 Vercel 上，那就没必要带上 `vercelOidc()`——除非你确实想接受 Vercel 签发的令牌。它还能通过 `subjects: [...]` 配合 `vercelSubject(...)` 来接受**其它**项目签发的令牌（用于跨项目调用），不过这是进阶用法，新手阶段先放一放，以官方文档为准。

#### httpBasic({ username, password })：给"运维者/服务"用的凭证

```ts
import { httpBasic } from "eve/channels/auth";
```

这是最朴素、最好懂的一种：**HTTP Basic 认证**，就是一对"用户名 + 密码"。调用方在请求头里带上这对凭证，对得上就放行。它适合给**运维人员或后端服务**用——你想用 `curl` 从命令行手动戳一下生产环境时，靠的就是它。

> ⚠️ 注意：密码这种敏感值**绝对不要硬编码进源码**。eve 的约定是把路由鉴权用的密码放在环境变量里（文档里提到的变量名是 `ROUTE_AUTH_BASIC_PASSWORD`），它只在运行时从 channel 定义里重新读取，**不会被打包进编译产物**。`httpBasic()` 构造函数确切接受哪些参数形态，知识库里没有完整记录，以官方文档为准——你只要知道它需要拿到一组 username/password 即可。

#### placeholderAuth()：仅供开发期"开门"，生产务必删掉

```ts
import { placeholderAuth } from "eve/channels/auth";
```

当你用 `eve init` 脚手架生成一个新项目时，它会自动帮你写好一个 `agent/channels/eve.ts`，长这样：

```ts
import { eveChannel } from "eve/channels/eve";
import { localDev, placeholderAuth, vercelOidc } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc(), placeholderAuth()],
});
```

`placeholderAuth()` 是一个**脚手架占位用的护栏**。它的行为是：在生产环境里返回一个"结构化的 401"，附带一条"鉴权还没配好"的提示——好让脚手架自动生成的那个网页聊天应用能友好地告诉你"嘿，你还没配鉴权呢"。

> ⚠️ 注意：**`placeholderAuth()` 绝不是真正的鉴权，上生产前一定要换掉或删掉它。** 官方部署指南里有一句几乎逐字的提醒："上生产前替换掉 `placeholderAuth()`"。
>
> 顺带说一个好消息：就算你直接把 `agent/channels/eve.ts` 整个删掉，eve 也会回退到默认的 `[localDev(), vercelOidc()]`——而这个组合**依然不会**放行生产环境的浏览器用户。所以你不会因为删文件就把门敞开。

#### 其余鉴权器（知道有就行）

| 鉴权器 | 什么时候用 |
|---|---|
| `none()` | 显式接受匿名流量（务必作为数组的**最后一项**）。 |
| `jwtHmac(...)` | 你用一个共享密钥来签发 JWT。 |
| `jwtEcdsa(...)` | 你要校验由别的系统用非对称密钥签发的 JWT。 |
| `oidc(...)` | 校验来自任意 issuer 的 OIDC 令牌。 |

> 💡 小贴士：这些进阶鉴权器的确切构造参数，知识库里没有完整记录，以官方文档为准。新手阶段，把 `localDev` / `vercelOidc` / `httpBasic` / `placeholderAuth` 这四个吃透，就足够覆盖绝大多数场景了。

### 8.5 真实示例：muse-agent 的 eve.ts

光看 helper 列表还是抽象，我们直接看一个**真实生产项目** `muse-agent`（一个跑在 eve 上的 OKX 交易 agent）是怎么配的：

```ts
// agent/channels/eve.ts
import { eveChannel } from "eve/channels/eve";
import { localDev, vercelOidc, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc(), httpBasic({ username, password })],
});
```

我们用前面学的规则把它从头读一遍：

1. **`localDev()`** —— 本地开发时（请求发给回环地址），放行。你在自己机器上 `eve dev` 调试时走的就是它。
2. **`vercelOidc()`** —— 部署到 Vercel 后，平台内部 / 自己项目签发的部署令牌，放行。
3. **`httpBasic({ username, password })`** —— 运维者凭证。当你想从外部用 `curl` 戳一下这个生产 agent 时，靠这对用户名密码进来。

走的就是"**按顺序匹配，第一个返回会话鉴权的胜出，全不匹配就 401**"。

注意两个值得学习的取舍：

- 自定义的 `httpBasic` 排在两个内置 helper **后面**——这里它本身就是兜底的运维入口，所以放最后是合理的。（一般原则仍是"自己更具体的鉴权器放前面"，具体顺序按你的语义来。）
- 这个项目**故意不用 `placeholderAuth()`**——因为它是个跑真金白银交易的生产 agent，绝不能留一个开发期占位的"软门"。它从一开始就配了真正能拦人的鉴权。

> 💡 小贴士：把这三个鉴权器和它守护的东西联系起来想——这是一个能下真实交易单的 agent，它的交易 connection 还配了 `approval: always()`（每笔交易都要人审批）。门口的 auth（谁能进来）和里面的 approval（进来后能不能动手）是**两套独立的防线**，缺一不可。

### 8.6 用 curl 调用 /eve/v1/session（带 Basic 认证）

理论讲完，来一发能直接复制粘贴的实战。假设你已经把 agent 部署好了（或者本地用 `httpBasic` 配好了门），下面用 HTTP Basic 认证开一个新会话。

最直白的写法，用 `curl` 的 `-u` 把用户名密码带上：

```bash
curl -X POST https://<你的应用域名>/eve/v1/session \
  -u 'myuser:mypassword' \
  -H 'content-type: application/json' \
  -d '{"message":"帮我看看当前账户余额。"}'
```

`-u 'myuser:mypassword'` 会被 `curl` 自动编码成标准的 `Authorization: Basic ...` 请求头——这正是 `httpBasic()` 期待的格式。

成功的话，响应体里会带回 `sessionId` 和 `continuationToken`，响应头里也有 `x-eve-session-id`。拿到 `sessionId` 后，就能接着去拉流式事件（流接口同样受鉴权保护，记得带上同一组凭证）：

```bash
curl -u 'myuser:mypassword' \
  https://<你的应用域名>/eve/v1/session/<sessionId>/stream
```

> 💡 小贴士：在本地 `eve dev` 里，因为 `localDev()` 已经放行了回环地址，你**不需要**带 `-u` 也能调通本地接口。Basic 认证主要是给**部署后**从外部访问用的。把基址换成 `http://127.0.0.1:3000` 就是纯本地调用：
> ```bash
> curl -X POST http://127.0.0.1:3000/eve/v1/session \
>   -H 'content-type: application/json' \
>   -d '{"message":"你好，在吗？"}'
> ```

### 8.7 进阶一瞥：自定义钩子与抛错拒绝

`eveChannel` 还能做得更细。两个你以后可能会用到、现在先眼熟的能力：

**自定义钩子（`onMessage` + `events`）。** 你可以在消息进来时插一脚（`onMessage`），或者订阅会话生命周期事件（`events`）：

```ts
import { eveChannel, defaultEveAuth } from "eve/channels/eve";
import { localDev, vercelOidc } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc()],
  onMessage(ctx, message) {
    const callerId = ctx.eve.caller?.principalId ?? "anonymous";
    return { auth: defaultEveAuth(ctx), context: [`HTTP caller ${callerId} sent: ${message}`] };
  },
  events: {
    "message.completed"(eventData, channel, ctx) {
      console.log("eve 回复完成", { continuationToken: channel.continuationToken, sessionId: ctx.session.id });
    },
  },
});
```

- `onMessage(ctx, message)` 返回 `{ auth, context }`，其中 `context` 是一个字符串数组（往本轮注入一些额外上下文）。
- `events` 里每个处理器的签名是 `(eventData, channel, ctx)`。

**主动抛错，用精确的状态码拒绝。** 如果你想在自己的鉴权逻辑里"明确地踹人出门"，可以抛出这两个错误：

```ts
import { ForbiddenError, UnauthenticatedError } from "eve/channels/auth";

throw new UnauthenticatedError({ code: "authentication_required", message: "请先登录。" }); // 401
throw new ForbiddenError({ message: "你没有这个工作区的权限。" });                              // 403
```

> ⚠️ 注意：鉴权只解决"谁能敲门进来"，它**不替你做"会话归属/越权"的检查**。也就是说，A 用户拿着合法凭证进来后，能不能去读 B 用户的会话——这层"按用户/租户/会话"的授权判断，需要你自己在业务里实现。`ctx.session.auth` 里提供了 `auth.current`（当前这轮的调用者）和 `auth.initiator`（最初开会话的人）供你判断。

### 8.8 Slack / Discord 等其它 channel（简述）

如果你想让 agent 接入聊天平台，eve 也提供了对应的 channel，用 `eve channels add` 脚手架或手写文件即可。这里只给你一个最小印象，**细节以官方文档为准**：

**Slack** —— 凭证走 **Vercel Connect** 托管，你**不用**自己管理 `SLACK_BOT_TOKEN` 之类的密钥：

```ts
// agent/channels/slack.ts
import { connectSlackCredentials } from "@vercel/connect/eve";
import { slackChannel } from "eve/channels/slack";

export default slackChannel({ credentials: connectSlackCredentials("slack/my-agent") });
```

它的路由是 `/eve/v1/slack`。Slack 的一个现实约束是：**它没法在本地测**（消息流向是 Slack → Vercel Connect → 你部署好的项目），所以要对着 preview/生产环境测。

**Discord** —— 由于 Discord 有"3 秒内必须 ACK"的硬性要求，这个 channel 会先快速签收、再在后台跑：

```ts
// agent/channels/discord.ts
import { discordChannel } from "eve/channels/discord";

export default discordChannel();
```

它需要 `DISCORD_PUBLIC_KEY`、`DISCORD_APPLICATION_ID`、`DISCORD_BOT_TOKEN` 等环境变量，路由是 `POST /eve/v1/discord`。

> 💡 小贴士：Slack、Discord、Teams、Telegram、Twilio、GitHub、Linear 这些 channel 各有不少平台专属的配置（事件订阅、按钮交互、HITL 渲染方式等）。本教程不展开，等你真的要接某个平台时，再去翻它对应的官方 channel 文档即可。

### 8.9 本章小结

- **Channel 是 agent 对外的门**，文件放在 `agent/channels/`，文件名即 channel id，根级专属。
- **默认就有一个 HTTP channel**；写 `agent/channels/eve.ts` 主要是为了覆盖配置（最常见就是配 auth）。
- 默认的 `eveChannel` 暴露了 `/eve/v1/session*` 这组路由：`POST /eve/v1/session` 开会话、`POST /eve/v1/session/:sessionId` 发后续、`GET /eve/v1/session/:sessionId/stream` 拉 NDJSON 流式回答；`GET /eve/v1/health` 永远公开。
- **`auth` 是一个按顺序匹配的数组**：第一个返回会话鉴权的胜出；全不匹配就 **401**。这就是 **fail closed / 默认拒绝**，`auth: []` 会拒绝一切。
- 四个新手必会的鉴权器：`localDev()`（只放行本地/Vercel dev，别单独用）、`vercelOidc()`（放行 Vercel 部署令牌）、`httpBasic({username,password})`（运维者凭证）、`placeholderAuth()`（仅开发期占位，**生产务必移除**）。
- 真实项目 `muse-agent` 用的就是 `[localDev(), vercelOidc(), httpBasic({ username, password })]`，并刻意不用 `placeholderAuth()`。
- 别忘了：**鉴权只管"谁能进门"，不管"进门后能不能动手"**——后者是工具/连接层的 approval 的事（见前面讲 connection 的章节）。

下一章，我们会走出"对外"这一面，去看 agent 自己怎么"对内"按计划主动干活——也就是 Schedules（定时任务）。

---

## 9. Schedules：让 agent 定时自己干活

到目前为止，我们的 agent 都是"被动"的：有人发消息（HTTP 请求、Slack at 一下），它才醒来干活。但很多真实场景里，你希望 agent 自己掐着点上班——每天早上整理一份摘要、每小时扫一遍市场、每隔几分钟清理一次过期数据。这就是 **Schedule（定时任务）** 要解决的问题。

> 💡 小贴士：可以把 schedule 想象成给 agent 装了一个闹钟。闹钟一响，agent 就自动开始干你预先写好的那件事，干完继续睡，等下一次响铃。

这一章我们会讲清楚：怎么用 `defineSchedule` 定义一个定时任务、"task 模式"到底是什么意思、为什么定时任务"天生就没有写/审批权限"（这是一个很重要的安全设计）、部署到 Vercel 后它会变成什么、以及在本地开发时怎么手动触发来调试。

### 9.1 一个 schedule 长什么样

在 eve 里，**每个定时任务就是 `agent/schedules/` 目录下的一个文件**。和你前面学过的 tool、connection 一样，eve 遵循"文件即定义"的原则——文件的名字和位置就是它的身份，你不需要在代码里写 `name` 字段。

我们直接看 muse-agent 这个真实项目里的例子。它有一个每小时扫一次市场行情的任务，文件叫 `agent/schedules/market_scan.ts`：

```ts
// agent/schedules/market_scan.ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 * * * *",
  markdown: "...", // 这里写一段提示词，告诉 agent 这次要做什么
});
```

就这么短。两个关键东西：

- **`cron`**：一个 cron 表达式，决定"什么时候触发"。这里的 `"0 * * * *"` 意思是"每小时的第 0 分钟"，也就是每小时整点跑一次。
- **`markdown`**：一段 markdown 提示词，决定"触发后让 agent 做什么"。它的内容就像你在聊天框里给 agent 发的一条消息——比如"扫一遍当前主流币种的行情，记录有没有异常波动"。

> 💡 小贴士：`defineSchedule` 来自 `eve/schedules` 这个导入路径，别和 tool（`eve/tools`）、connection（`eve/connections`）搞混了。

`defineSchedule` 接受的字段大致是这样的：

```ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 9 * * 1-5",                 // 必填：cron 表达式
  markdown: "整理昨天的销售数据并发到指标接口。", // task 模式：一段提示词
  // 或者用 run，但 markdown 和 run 只能二选一（见 9.5）
});
```

> ⚠️ 注意：`cron` 是**必填**的；`markdown` 和 `run` 这两个字段**必须且只能提供其中一个**，不能同时写，也不能都不写。如果你两个都写了（或都不写），eve 在编译时（类型层面）就会报错。本章先把重点放在 `markdown` 这种最常见、最简单的写法上，`run` 那种更高级的写法在 9.5 简单介绍。

关于文件名和任务名：任务的名字就是它在 `schedules/` 目录下的路径去掉扩展名。`agent/schedules/market_scan.ts` 的任务名就是 `market_scan`。eve 也支持嵌套目录，比如 `agent/schedules/billing/sweep.ts` 的任务名就是 `billing/sweep`。

> ⚠️ 注意：`schedules/` 是 **root-only** 的——只有顶层 agent 能有定时任务，子 agent（subagent）不能声明自己的 `schedules/`。

### 9.2 cron 表达式速查

如果你以前没接触过 cron，别紧张，它其实很简单。cron 表达式是一串用空格隔开的 **5 个字段**，从左到右分别代表：

```
┌───────────── 分钟 (minute,        0-59)
│ ┌───────────── 小时 (hour,          0-23)
│ │ ┌───────────── 日   (day-of-month, 1-31)
│ │ │ ┌───────────── 月   (month,        1-12)
│ │ │ │ ┌───────────── 星期 (day-of-week,  0-6，0=周日)
│ │ │ │ │
* * * * *
```

每个字段可以是：具体数字（`9`）、星号 `*`（表示"每个"）、范围（`1-5`）、间隔（`*/5` 表示"每 5 个单位"）、列表（`1,15`）。eve 的 cron 精度是**到分钟**。

下面这张速查表覆盖了最常用的写法：

| cron 表达式 | 含义 |
|---|---|
| `* * * * *` | 每分钟（调试常用，生产慎用） |
| `*/5 * * * *` | 每 5 分钟 |
| `0 * * * *` | 每小时整点（muse-agent 的 market_scan 用的就是这个） |
| `0 0 * * *` | 每天 00:00 |
| `0 9 * * *` | 每天 09:00 |
| `0 9 * * 1-5` | 周一到周五，每天 09:00 |
| `30 8 * * 1` | 每周一 08:30 |
| `0 0 1 * *` | 每月 1 号 00:00 |
| `0 0 * * 0` | 每周日 00:00 |

> ⚠️ 注意：部署到 Vercel 后，所有 cron 都是**按 UTC（协调世界时）评估**的，不是你本地时区！比如你想让任务在北京时间（UTC+8）早上 9 点跑，UTC 时间是前一天的凌晨 1 点，应该写 `0 1 * * *`，而不是 `0 9 * * *`。算错时区是新手最常踩的坑。eve 文档没有提到可以配置非 UTC 时区，所以请始终按 UTC 来折算。

### 9.3 "task 模式"到底是什么意思

这是本章最需要理解透的一个概念。

当你给 schedule 写了 `markdown` 提示词，闹钟一响时，eve 会做这样一件事：**用这段提示词运行一次 agent，然后把 agent 产生的输出直接丢掉。** 这种"跑一次、不要返回值"的运行方式，eve 把它叫做 **task 模式**。

为什么要把输出丢掉？因为定时任务没有"对话对象"——没有人在聊天框那头等着看 agent 说了什么。它就是一个后台干活的工人，干完就完，没人需要它的"回话"。

> 💡 小贴士：可以这样对比理解。平时你在聊天里问 agent 问题，是"对话模式"，agent 的回答会流式返回给你看。而 schedule 的 task 模式是"派活模式"——你只是派它去干一件事，不关心它嘴上说了啥，只关心它实际做了什么。

那"丢掉输出"是不是意味着这次运行白跑了？**完全不是。** task 模式下，agent 在运行过程中**照样可以调用工具（tool）、调用连接（connection）、往后端写数据、记日志**。被丢掉的只是它最后那段面向人类的"回话文本"，它干的实事都是真实生效的。

举个完整的 task 模式例子：

```ts
// agent/schedules/heartbeat.ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "*/5 * * * *",
  markdown: "拉取所有未关闭的 Linear issue，把摘要 POST 到指标接口。",
});
```

每 5 分钟，eve 就会用这段话驱动 agent 跑一次。agent 会去调用相关工具拉取 issue、整理摘要、调用接口提交——这些动作都真实发生，只是它最后那句"我已经提交了 12 条摘要"会被丢掉，因为没人在听。

如果你更喜欢用 markdown 文件而不是 TypeScript，也可以这样写（用 frontmatter 放 cron，正文就是提示词）：

```md
---
cron: "0 0 * * 0"
---

清理过期的工作流状态。
```

> ⚠️ 注意：`.md` 形式的 schedule，frontmatter 里只放 `cron`（不接受其他字段），正文（frontmatter 下面的部分）就是提示词，效果等同于 TypeScript 形式里的 `markdown` 字段。

### 9.4 关键安全点：定时任务为什么"天生没有写/审批权限"

这是 schedule 设计里最重要的一条安全规则，也是 muse-agent 这个真实交易项目能放心跑定时任务的原因。请务必读懂这一节。

先回忆一下你在前面"审批"和"连接"章节学到的：eve 有一套**人工审批（approval）机制**。当一个工具或连接配了 `approval: always()`，agent 想调用它时会**停下来、durably（持久地）等一个真人点"同意"**，在批准之前这次会话会一直挂起（park）在那里等着，而且不消耗算力。muse-agent 的下单连接 `okx-trade.ts` 就配了 `approval: always()`，确保每一笔真金白银的交易都要人点头。

现在关键来了。eve 官方文档对 task 模式有一句明确的描述：

> "A task-mode session runs to completion or fails, and cannot park to wait for a person or an OAuth sign-in."
> （task 模式的会话要么跑到结束、要么失败，它**不能停下来等一个人、也不能停下来等 OAuth 登录授权**。）

把这两件事拼起来，你就明白了那条安全链条：

1. task 模式**不能 park 等人**（这是它的硬性约束）。
2. 任何需要人工审批（`approval: always()` / `once()`）的工具或连接，调用时都必须 park 等人点同意。
3. 所以，**定时任务在 task 模式下根本无法调用那些需要审批的工具**——一调用就会卡在"等人审批"这一步，而 task 模式偏偏不允许卡在这一步。

落到 muse-agent 上就是：它的下单连接 `okx-trade` 配了 `approval: always()`，所以 `market_scan` 这个定时任务**从结构上就不可能下单**——它就算想下单，也会卡在审批 park 这一步，而 task 模式不允许 park。于是这个每小时的定时任务**天生就只有"只读扫描"的能力，没有任何交易/写入授权**。这不是靠"我们小心点别写代码让它下单"来保证的，而是框架机制从根上堵死的。

> ⚠️ 注意（一个常被说错的细节）：严格按官方文档，task 模式本身**并不是"完全不能写"**——它能调用工具、能往后端写数据。它真正不能做的，是**停下来等人工审批或等 OAuth 登录**。所以准确的说法是：*task 模式会丢弃输出，且不能 park 等待人工审批 / OAuth；它并非笼统地被禁止一切写操作。* muse-agent 的定时任务之所以没有交易权限，是因为它的交易连接恰好要求审批（会 park），而不是因为"schedule 一律不能写"。理解这个因果关系，你才能正确地设计自己的定时任务。

这条规则给你的实践指导是：

- 想让定时任务安全地干"只读 / 低风险写入"的活（扫描、汇总、同步、心跳）——放心用 task 模式。
- 凡是危险动作（下单、转账、删除、发外部邮件），就给对应的工具/连接挂上 `approval: always()`。这样它在普通对话里仍然会请人审批，而在定时任务里则**根本无法被触发**，相当于双重保险。

### 9.5 进阶：`run` 处理器（需要把结果送到某个频道时）

如果你的定时任务不是"干完就算"，而是想把结果**主动推送到某个频道**（比如每分钟检查告警、有告警就发到 Slack），那 `markdown` 就不够用了——因为 task 模式会把输出丢掉，而且它不能 park 等 Slack 那边的回复。这时候用另一个字段：`run`。

```ts
// agent/schedules/critical-alerts.ts
import { defineSchedule } from "eve/schedules";
import slack from "../channels/slack.js";

export default defineSchedule({
  cron: "* * * * *",
  async run({ receive, waitUntil, appAuth }) {
    waitUntil(
      receive(slack, {
        message: "检查有没有新的严重告警。只在确实有告警时才汇报。",
        target: { channelId: "C0123ABC" },
        auth: appAuth,
      }),
    );
  },
});
```

简单解释一下处理器拿到的三个东西（这里只做入门了解，细节以官方文档为准）：

- **`receive(channel, {...})`**：把这次的活"派发"到另一个频道上去跑（这里是 Slack），相当于在那个频道上开了一次新会话。
- **`waitUntil(promise)`**：延长这次定时任务的存活时间，确保你 `receive` 出去的会话和正在飞的请求都能跑完，不会因为函数提前返回就被掐断。所以要把 `receive` 包在 `waitUntil` 里。
- **`appAuth`**：一个预先准备好的"应用身份"，用来给这次派发的会话做身份认证。

> 💡 小贴士：和 task 模式不同，`run` 处理器派发出去的会话跑在持久运行引擎上，**是可以 park 的**（比如等 Slack 那边的人回复）。"不能 park"这条限制只针对 `markdown` 的 task 模式。入门阶段你大概率用 `markdown` 就够了，`run` 留个印象即可。

### 9.6 部署到 Vercel 后：每个 schedule 变成一个 Vercel Cron Job

当你把 eve 应用部署到 Vercel，框架会在构建时**把每一个 `defineSchedule(...)` 转换成一个 Vercel Cron Job**（具体来说，每个 `cron` 会被写进 `.vercel/output/config.json`）。从此以后，是 **Vercel 平台**按你写的 cron 表达式、**按 UTC 时间**来定时触发你的 agent。

部署后你可以在 Vercel 控制台里看到和管理它们：

- **Settings → Cron Jobs**：确认你的定时任务都被正确注册了。
- **Observability → Cron Jobs**：查看每个任务的执行历史。
- **Observability → Logs**：看每一次运行的具体日志。

> 💡 小贴士：这意味着你不用自己搭一台服务器开着 crontab，也不用操心"机器会不会半夜挂了导致任务不跑"。定时触发这件事完全交给 Vercel 平台托管。

> ⚠️ 注意：如果你不部署在 Vercel，而是自己用 `eve build && eve start` 来跑（`eve start` 会启动 Nitro 的定时任务运行器，能正常触发 schedule），那要小心一个坑：如果你把构建产物塞进某些只服务 HTTP、不启动 Nitro 定时任务运行器的进程管理器/容器/预设里，**schedule 会被编译出来但永远不会触发**。这种情况下要么用 `eve start`、要么换一个支持 Nitro 定时任务的宿主、要么干脆用你自己的调度器去打一个受保护的路由。

### 9.7 本地调试：`eve dev` 不会按点触发，要手动 dispatch

这是开发阶段一定会遇到的问题，也是新手最容易困惑的地方。

> ⚠️ 注意：**本地 `eve dev` 永远不会按 cron 节奏自动触发你的 schedule。** 你不可能为了测一个"每小时跑一次"的任务，真的傻等一小时。

那本地怎么调试定时任务？eve 提供了一个**仅限开发环境**的"手动触发"路由，让你随时按一下就把某个 schedule 立刻跑一次。先把 dev server 跑起来：

```bash
npx eve dev
```

然后用 `curl` POST 到 dispatch 路由，路径里带上你的任务名（就是文件路径去扩展名得到的那个名字）：

```bash
curl -X POST http://localhost:3000/eve/v1/dev/schedules/market_scan
```

成功后会返回类似这样的 JSON，告诉你这次触发起了哪些会话：

```json
{ "scheduleId": "market_scan", "sessionIds": ["..."] }
```

拿到 `sessionIds` 里的 id，你就可以去订阅它的事件流，实时看 agent 这次跑了些什么：

```bash
curl http://localhost:3000/eve/v1/session/<sessionId>/stream
```

关于这个 dispatch 路由的几个要点：

- 它走的是**和生产环境 cron 完全相同的派发路径**，所以本地手动触发一次，行为就跟线上真实触发一次一样可靠。每次触发都会把这个 schedule 跑一次。
- 路径里的 `:scheduleId` 是文件路径去扩展名得到的名字。如果是嵌套任务（如 `billing/sweep`），名字里的 `/` 需要做 URL 编码。
- 触发一个不存在的任务名会返回 `404`，并附带一个 `availableScheduleIds` 列表告诉你有哪些任务可用；漏写任务名（路径里 id 为空）会返回 `400`，提示 `Missing schedule id.`。
- 这个路由**只在 `eve dev` 下存在**，生产环境根本不会挂载它。也因为 dev server 是本地的，所以它**不需要任何认证**就能调用。

> 💡 小贴士：如果你在写 eval（评测），还有一个对应的辅助方法 `t.target.dispatchSchedule("market_scan")` 可以在测试里触发 schedule，效果和上面的 dispatch 路由一致。

### 9.8 完整真实示例：muse-agent 的 market_scan

把这一章串起来，我们完整地看一遍 muse-agent 里那个每小时只读市场扫描任务，它麻雀虽小但五脏俱全：

```ts
// agent/schedules/market_scan.ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 * * * *", // 每小时整点（UTC）触发一次
  markdown: "...",   // task 模式提示词：扫描当前市场行情
});
```

这个小文件背后，是本章所有概念的合力：

1. **`cron: "0 * * * *"`** → 每小时整点跑。部署到 Vercel 后变成一个按 UTC 评估的 Vercel Cron Job。
2. **用了 `markdown` → task 模式**：eve 用这段提示词驱动 agent 跑一次，丢弃输出，但 agent 可以调用只读连接 `okx-read` 去查行情、可以记日志。
3. **天生没有交易权限**：它想下单就得调用 `okx-trade` 连接，而该连接是 `approval: always()`，调用会 park 等人审批——但 task 模式不能 park。于是这个定时任务从结构上就只能"看"不能"动手"，安全地每小时自动巡检一遍市场。
4. **本地调试**靠 `curl -X POST http://localhost:3000/eve/v1/dev/schedules/market_scan` 手动触发，因为 `eve dev` 不会自己按点跑。

到这里，你已经掌握了让 agent 自己掐着点干活的全部基础：用 `defineSchedule` 写 cron + 提示词，理解 task 模式"丢输出但能干实事、但不能 park 等审批"的语义，知道它部署后变成按 UTC 跑的 Vercel Cron Job，以及本地怎么手动触发来调试。下一章我们会继续往下走。

---

## 10. Sandbox：模型的代码草稿纸

到目前为止，我们已经学会了让 agent 调用 tools（执行你写的动作）、读 skills（按需加载知识）、连 connections（接外部系统）。这一章要讲的 sandbox 又是另一种东西——它是**模型自己的一块"草稿纸"**：一个隔离的、bash 风格的小计算环境，专门给模型用来跑命令、写临时文件、处理数据。

它听起来很强大，但用途其实很窄。本章的核心任务，就是帮你**分清"什么该进 sandbox、什么不该"**，并教你在涉及敏感环境时如何用一行配置把它锁死。

### 10.1 sandbox 到底是什么

先打个比方：你请了一个很聪明的助手帮你算账。你会给他一张草稿纸让他随便涂写演算，但你**不会**把公司金库的钥匙交到他手里。

sandbox 就是那张草稿纸——它是模型的临时工作区，而不是通往你业务系统的通道。

具体来说：

- **每个 eve agent 都恰好有一个 sandbox**。它是一个隔离的 bash 风格计算环境，自带文件系统，根目录在 `/workspace`。
- 框架内置的工具 `bash`、`read_file`、`write_file`（还有 `glob`、`grep`）默认就操作这块空间。模型想跑个 `python script.py`、解压个文件、用 `jq` 处理一段 JSON，都在这里发生。
- 它不需要你写任何代码就能用——`agent/sandbox/` 这个配置文件是**可选的**，不写也有一个默认 sandbox 在工作。

> 💡 小贴士：sandbox 的定位是"跑不可信 / 模型自己生成的代码"。模型在对话里临时拼出来的一段脚本，就让它在 sandbox 里跑，跑坏了也波及不到你的真实环境。

### 10.2 最关键的一条：sandbox 不是用来访问你的业务系统的

这是整个 eve 框架里**最容易踩、也最值得反复强调的一条安全边界**，请务必记牢：

> ⚠️ **注意：tools 跑在你的"应用运行时"（app runtime），不在 sandbox 里。**
>
> - **tools**（你在 `agent/tools/` 里写的那些）运行在你的应用运行时中，可以读 `process.env`、import 共享的 `lib/` 代码、参与 eve 的持久暂停/恢复机制。
> - **sandbox** 是模型的草稿空间，跑的是不可信 / 模型生成的命令，**默认拿不到你的环境变量和业务代码**。

换句话说，这两件事走两条完全不同的路：

| 你想做的事 | 该用什么 | 在哪里运行 |
|---|---|---|
| 让模型跑一段它临时写的脚本、处理文件 | sandbox（`bash` 等内置工具） | 隔离的 `/workspace` 草稿空间 |
| 让 agent 查数据库、调你公司内部 API | **tool**（`defineTool`） | 你的 app runtime（能读 env / lib） |
| 让 agent 接外部服务（MCP / OpenAPI） | **connection**（见第 7 章） | app runtime，凭据由框架代管 |

所以，**当你想让 agent "访问外部系统 / 业务系统"时，答案永远是 connections（或 tools），绝不是把 URL 和密钥塞进 sandbox 里。** connections 的设计目标之一就是"把服务地址和凭据挡在模型的提示词之外"——模型连那个 URL 和 token 长什么样都看不到。把这些东西放进 sandbox，等于把金库钥匙递给了草稿纸。

> 💡 官方对此有一句很贴切的话：默认 sandbox **不能**替代你为应用所需配置的网络策略、凭据管理、数据保留与删除等控制。它只是个隔离的执行环境，不是安全策略本身。

### 10.3 用 `defineSandbox` 配置 sandbox

如果你确实需要自定义 sandbox（比如要锁死网络、要预装某个命令行工具），就写一个 sandbox 定义文件。

入口函数是 `defineSandbox`，从 `eve/sandbox` 导入。文件放哪儿有两种写法，二者都受支持：

- **简写**：`agent/sandbox.ts`（只放定义）。
- **文件夹**：`agent/sandbox/sandbox.ts`（定义 + 一个 `workspace/` 目录用来预置文件）。

> 💡 小贴士：如果你想给 sandbox 预先放几个种子文件（比如一份配置模板），就用文件夹形式，把文件放进 `agent/sandbox/workspace/**`，它们会被铺到运行时的 `/workspace/...` 下。注意 `lib/` 是"只能 import"的共享代码，永远不会进 sandbox；另外 `agent/sandbox/workspace/skills/` 这种路径是不允许的，skills 要放在 `agent/skills/` 里（它们会被自动铺到 `/workspace/skills/...`）。

最小示例（指定后端）：

```ts
// agent/sandbox.ts
import { defineSandbox } from "eve/sandbox";
import { vercel } from "eve/sandbox/vercel";

export default defineSandbox({
  backend: vercel(),
});
```

这里出现了一个新概念：`backend`（后端）。它决定"这块草稿纸实际由谁来提供算力"。

### 10.4 backends：草稿纸跑在哪台机器上

同样一段 bash，可以在云上的虚拟机里跑，也可以在你本机的 Docker 容器里跑，还可以用一个纯 JS 模拟器对付。backend 就是用来选这个"实际执行环境"的。eve 提供了几个 backend，分别从各自的子路径导入：

| backend | 跑在哪 | 导入子路径 |
|---|---|---|
| `vercel(...)` | Vercel Sandbox（云上、临时的 Firecracker 微虚拟机） | `eve/sandbox/vercel` |
| `docker(...)` | 本机的 Docker 容器 | `eve/sandbox/docker` |
| `microsandbox(...)` | 本机的轻量级虚拟机 | `eve/sandbox/microsandbox` |
| just-bash | 本机纯 JS 的 bash 模拟器，**无守护进程、无真实二进制、无网络隔离** | `eve/sandbox/just-bash` |
| `defaultBackend(...)` | 自动按优先级挑一个 | `eve/sandbox` |

> ⚠️ **注意：just-bash 是本地开发的"兜底"方案。** 它不是真正的隔离环境——没有真实的二进制命令（你的 `jq`、`python` 在这里可能根本不存在），也没有网络隔离。它的价值是让你在没装 Docker、也不在 Vercel 上时，`eve dev` 仍然能跑起来不报错。**绝不要把 just-bash 当成生产环境的安全边界。**

对新手来说，最省心的做法是用 `defaultBackend()`：它会按优先级自动挑——大致顺序是 **Vercel（在 Vercel 上部署时）→ Docker（本机有可用的 Docker）→ microsandbox（本机支持时）→ just-bash（兜底）**。这样同一份代码，本地开发和云上部署都能跑，无需改代码。

`defaultBackend()` 还允许你给每个候选后端单独配参数：

```ts
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    vercel: { networkPolicy: "deny-all" },
    docker: { networkPolicy: "deny-all" },
  }),
});
```

> 💡 小贴士：`vercel`、`docker`、`microsandbox` 这几个工厂函数的名字是确定的；just-bash 的工厂函数确切叫什么（`justbash` 还是 `justBash`）目前不完全确定，以官方文档为准。各 backend 接受的具体配置项也以官方文档为准——上面用到的 `networkPolicy` 是确定的。

### 10.5 networkPolicy：给草稿纸装一道防火墙

`networkPolicy` 控制 sandbox 里的代码**能不能、能往哪联网**。这是 sandbox 配置里你最该关心的一项。它有三种模式：

- **`"allow-all"`** —— **默认值**。不限制出网，模型在 sandbox 里可以装包、下载依赖、访问任意公网地址。
- **`"deny-all"`** —— **拒绝一切出站网络访问，连 DNS 都不放行**。这能显著降低"不可信代码 + 私密数据"被偷偷外传（数据外泄）的风险。
- **用户自定义** —— 默认拒绝，再用白名单逐项放行（允许某些域名、某些地址段等）。属于进阶用法，需要时查官方文档。

eve 这一层的写法很直观：

```ts
networkPolicy: "allow-all"   // 默认：随便联网
networkPolicy: "deny-all"    // 锁死：一点都不让出网
```

> ⚠️ **注意：网络策略是否真正"强制执行"，取决于 backend。** 在支持它的后端上（如 vercel、microsandbox），`"deny-all"` 是真的会拦截出网；而 just-bash 这种没有隔离能力的后端根本谈不上"强制执行"。所以**安全不能只靠在代码里写 `deny-all`，还要确保你跑在一个真正会执行这条策略的后端上**。

### 10.6 实战：muse-agent 为什么默认拒绝出网

我们来看一个真实的生产级项目 `muse-agent`（一个跑在 eve 上的 OKX 交易 agent）是怎么配 sandbox 的。它的 `agent/sandbox/sandbox.ts` 只有几行：

```ts
// agent/sandbox/sandbox.ts
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    vercel: { networkPolicy: "deny-all" },
    docker: { networkPolicy: "deny-all" },
  }),
});
```

读懂这几行，你就理解了"为什么默认拒绝出网更安全"这件事：

1. **这个 sandbox 纯粹是模型的草稿空间。** 真正的交易、查询全都走 connections（`okx-read`、`okx-trade`），凭据由框架代管，模型看不到。sandbox 里压根不需要联网。
2. **既然不需要联网，那就一点都不给。** 对 vercel 和 docker 两个后端都明确写上 `networkPolicy: "deny-all"`（其中 vercel 会真正执行这条策略）。
3. **这是一道纵深防御。** 这是个碰真钱的交易 agent，处理的是敏感数据和私密上下文。万一模型在 sandbox 里跑了某段不该跑的代码，`deny-all` 保证它在支持该策略的后端上**没有任何出网通道**可以把数据偷偷发出去。把出网默认关掉，比"先开着、出事再补"安全得多。

换句话说，muse-agent 的思路是：**sandbox 默认就该是个"断网的草稿纸"**，需要联网是例外、要显式开；而不是默认全开、出了事再去堵。这就是"默认拒绝（deny by default）"这种安全设计的精髓。

### 10.7 新手一句话决策：到底要不要配 sandbox？

讲了这么多，给你一条可以直接照搬的判断标准：

> **不确定要不要配 sandbox？起步阶段可以完全不配**——eve 自带的默认 sandbox 已经够你跑通整个流程了。
>
> **一旦你开始让模型执行代码、并且涉及敏感环境**（碰真钱、处理用户私密数据、连着内部系统），**就用 `networkPolicy: "deny-all"` 把它收紧**，并确认你跑在一个会真正执行这条策略的 backend（如 vercel / microsandbox）上。

最后，把本章最重要的三句话钉在脑子里：

1. **sandbox 是模型的草稿纸，不是你业务系统的入口。** 要访问外部 / 业务系统，走 connections 或 tools，绝不把 URL 和密钥塞进 sandbox。
2. **tools 跑在 app runtime（能读 env / lib），sandbox 跑不可信代码（默认拿不到这些）**——这两条路千万别混。
3. **涉及敏感环境时，默认 `deny-all`，需要联网再显式放行。** 默认拒绝，永远比默认放行安全。

---

## 11. Evals：给 agent 写自动化测试

到这一章为止，你已经能写出一个有工具、有连接、有指令的 agent 了。现在有个绕不开的问题：你怎么知道它"还能正常工作"？

普通后端代码你改一行、跑一遍单元测试，红了就知道哪里坏了。但 agent 不一样——它的"行为"是由一个大模型在每一次对话里**临场决定**的。同样一句话，今天它老老实实调了 `get_weather` 工具，明天换了个模型版本、或者你把 instructions 改了两个字，它可能就自作主张直接编了个温度回给你。这种"代码没改、行为却变了"的现象，业内叫 **行为漂移（behavior drift）**。

> 💡 小贴士：你可以把 agent 想象成一个新来的实习生。你给他写了岗位说明书（instructions）、发了工具箱（tools），但他每天的具体表现还是会浮动。Evals 就是你给这个实习生定期出的"随堂小测验"——固定题目、固定评分标准，分数掉了你立刻能发现。

这一章我们就来学 eve 给 agent 写自动化测试的机制：**evals**。

### 11.1 evals 是什么，和单元测试有什么不同

**eval（evaluation 的简称）= 一个"打分的检查项"：它真的把你的 agent 跑起来、发一条消息进去、然后对返回的结果做断言。**

它和你熟悉的单元测试，关键区别在于"跑得有多真"：

- 单元测试通常只测一个函数，把外部依赖都 mock 掉。
- eval 走的是**和真实用户一模一样的 HTTP 接口**：runner（运行器）会启动或连上一个真正的 agent server，像真用户那样发起会话、驱动它跑完，再对返回内容打分。

换句话说，eval 测的不是"某个工具函数对不对"，而是"整个 agent 在收到这句话时，行为对不对"——它有没有跑完一轮、有没有调用预期的工具、回复里有没有该出现的内容。这正好对准了我们最担心的"行为漂移"。

> 💡 小贴士：eval 不会替你测试工具内部的业务逻辑（那是普通单元测试该干的事）。eval 关心的是 agent 这个"决策者"的外在行为。两者互补，不是二选一。

### 11.2 evals/ 目录：它长什么样

eval 的文件**不放在 `agent/` 里面，而是放在和 `agent/` 平级的 `evals/` 目录**（也就是项目根目录下）。这是 eve "文件即接口"原则的又一次体现——你不用注册任何东西，eve 会自动扫描 `evals/` 目录来发现你的用例。

一个典型的 evals 目录是这样的：

```text
my-agent/
├── agent/
├── evals/
│   ├── evals.config.ts          # 全局配置，必须恰好有一个
│   ├── smoke.eval.ts            # 一个用例
│   └── weather/
│       ├── brooklyn-forecast.eval.ts
│       └── no-tools-for-greetings.eval.ts
└── package.json
```

几条规则先记住：

- 每个 **`.eval.ts`** 文件默认就是**一个 eval 用例**。
- **用例的身份来自它的文件路径**，你不用、也不能自己写 `id` 或 `name`。比如 `evals/weather/brooklyn-forecast.eval.ts` 的 id 就是 `weather/brooklyn-forecast`。
- `evals/` 下面还可以放普通的 `.ts` 文件（不带 `.eval.ts` 后缀）作为共享的辅助代码，它们不会被当成用例。

> ⚠️ 注意：`evals/` 和 `agent/` 是**兄弟目录**，不是父子。新手很容易手滑把 eval 文件塞进 `agent/evals/`，那样 eve 是发现不了它的。

### 11.3 evals.config.ts：全局配置（必须恰好有一个）

`evals/` 目录下**必须恰好有一个 `evals.config.ts`**，它是整套 evals 的全局配置入口。这个文件用 `defineEvalConfig`（从 `eve/evals` 导入）定义。

最省心的写法，就是真实项目 muse-agent 用的那一行——传一个空对象：

```ts
// evals/evals.config.ts
import { defineEvalConfig } from "eve/evals";

export default defineEvalConfig({});
```

是的，**空配置就完全够用了**。`defineEvalConfig` 的所有字段都是可选的，常见的有：

- `judge`：给"评审模型"指定一个默认模型（下一节解释什么是评审模型）。如果你的用例全是确定性断言，这一项可以完全不写。
- `reporters`：报告器，把每次 eval 的结果上报到外部系统（比如 Braintrust）。不写就只在控制台打印汇总。
- `maxConcurrency`：并发上限（正整数，默认 8）。
- `timeoutMs`：每个用例的默认超时时间。

如果你确实想配一个默认评审模型 + 一个报告器，长这样：

```ts
// evals/evals.config.ts
import { defineEvalConfig } from "eve/evals";
import { Braintrust } from "eve/evals/reporters";

export default defineEvalConfig({
  judge: { model: "openai/gpt-5.4-mini" },
  reporters: [Braintrust({ projectName: "my-agent" })],
});
```

> 💡 小贴士：刚起步时别想太多，直接 `defineEvalConfig({})`。等真的需要评审模型或报告器了再回来加，反正它们都是可选的。

### 11.4 .eval.ts：用 defineEval 写一个用例

每个 `.eval.ts` 文件用 **`defineEval`**（同样来自 `eve/evals`）定义一个用例。它的结构非常简单：唯一必填的字段是 `test(t)`——一个函数，eve 在里面给你一个 `t` 对象，你用它来"驱动 agent + 写断言"。

先看真实项目 muse-agent 的冒烟用例骨架：

```ts
// evals/smoke.eval.ts
import { defineEval } from "eve/evals";

export default defineEval({
  description: "开机冒烟：agent 能接收请求并完成一轮。",
  async test(t) {
    await t.send("...");
    t.completed();
  },
});
```

这短短几行其实做了三件事：

1. `description`（可选）：给这个用例写一句人话说明，方便日后看报告时认得出来。
2. `await t.send("...")`：**向 agent 发一条用户消息**，并等它把这一轮（turn）跑完。这就是在模拟一个真实用户的输入。
3. `t.completed()`：**断言这一轮成功完成了**（没有失败）。

`defineEval` 还有几个可选字段，起步阶段用不到，先有个印象就行：`judge`（给这个用例单独指定评审模型）、`timeoutMs`（单独的超时）、`tags`（打标签，方便用 `--tag` 筛选运行）、`metadata`、`reporters`。

> 💡 小贴士：`test(t)` 里你想发几条消息、写几条断言都可以。`t.send(...)` 是 `async` 的，记得 `await`，否则断言可能在 agent 还没跑完时就执行了。

### 11.5 t 对象：怎么驱动 agent、怎么断言

`t` 既是"遥控器"（驱动 agent），也是"裁判"（写断言）。这一节挑新手最常用的几个介绍。

**驱动 agent（发消息、读回复）：**

- `t.send(input)`：发一条用户消息，等这一轮跑完。
- `t.reply`：拿到最后一条助手回复的文本（没有则为 `null`）。
- `t.sessionId`：当前会话 id。

**确定性断言（run-level 方法）：**

这一类断言**不需要任何模型来判断对错**——它们看的是"客观发生了什么"，比如有没有完成、调没调某个工具。它们默认就是**硬性门禁（gate）**：一旦失败，整个 eval 判为失败，`eve eval` 退出码非 0。常用的有：

```ts
t.completed();                    // 这一轮成功完成（最常用的冒烟断言）
t.calledTool("get_weather");      // 断言调用过名为 get_weather 的工具
t.notCalledTool("refund_charge"); // 断言没有调用过某个工具
t.usedNoTools();                  // 断言这一轮一个工具都没调
t.messageIncludes("Sunny");       // 回复里包含某段文字（可传字符串或正则）
```

> ⚠️ 注意：`t.calledTool(...)` 和 `t.usedNoTools()` 互斥，别在同一个用例里同时写这两条——一个断言"调过工具"、一个断言"没调过工具"，逻辑上必然打架。

**对具体值做断言（`t.check`）：**

如果你想对"某个具体的值"做判断，用 `t.check(value, assertion)`，断言构造器从 `eve/evals/expect` 导入：

```ts
import { includes, equals, matches } from "eve/evals/expect";

t.check(t.reply, includes("sunny"));            // 回复包含子串（门禁）
t.check(parsed, equals({ city: "Brooklyn" }));  // 深度相等（门禁）
t.check(parsed, matches(WeatherSchema));        // 匹配某个 schema（门禁）
```

把上面这些拼起来，一个稍微完整点的、**完全确定性**的天气用例长这样（来自知识库示例）：

```ts
// evals/weather/brooklyn-forecast.eval.ts
import { defineEval } from "eve/evals";
import { includes } from "eve/evals/expect";

export default defineEval({
  description: "天气 agent 的基本消息与工具调用覆盖。",
  async test(t) {
    await t.send("What is the weather in Brooklyn?");
    t.completed();
    t.calledTool("get_weather");
    t.check(t.reply, includes("Sunny"));
  },
});
```

注意：上面这一整个用例，**没有用到任何模型来评判**——它问的全是"跑完了没""调了哪个工具""回复里有没有 Sunny"这类有客观答案的问题。这就是所谓 **deterministic（确定性）用例**，它**不需要评审模型**，跑起来快、稳、不花钱。

### 11.6 什么时候才需要"评审模型"

那什么时候才需要一个额外的模型来当裁判？

当你想判断的东西**没有客观对错、需要"理解语义"**时。比如"这条回复有没有正确地回答问题""有没有引用来源""总结得准不准"——这些光靠字符串匹配判断不了。这时就要请出 **LLM-as-judge（用大模型当裁判）**，对应 `t` 上的 `t.judge.*` 这一族断言：

```ts
t.judge.autoevals.closedQA("回复里引用了一个来源");
t.judge.autoevals.factuality(reference);
```

关于评审模型，有三件事新手必须分清：

1. **只有 `t.judge.*` 这类断言才需要评审模型。** 前面 11.5 里那些 `t.completed()` / `t.calledTool()` / `t.check(...)` 统统不需要——它们是确定性的。
2. **`t.judge.*` 默认是"软断言（soft）"**，意思是分数不达标不会直接判失败，只会被记录下来（除非你加 `--strict`，或手动 `.gate(...)` 把它变成硬门禁）。这和确定性断言"默认硬门禁"正好相反。
3. **评审模型从哪来？** 按"就近优先"解析：单次调用指定的 > 用例上 `defineEval({ judge })` 指定的 > 全局 `defineEvalConfig({ judge })` 指定的。如果一个都没配，却调用了 `t.judge.*`，会被记成一次**失败的门禁**——所以用 judge 之前一定要配好模型。

> ⚠️ 注意：评审模型本身也是要调模型、要花钱、要凭证的（字符串模型 id 走 AI Gateway，需要 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN`）。这也是为什么我们强烈建议**起步只写确定性用例**——又快又免费，先把"行为漂移"的警报线拉起来再说。

### 11.7 eve eval：针对本地 dev server 跑起来

写好用例，怎么跑？一条命令：

```bash
eve eval
```

它做的事是：在本地**启动一个 dev server**（也就是把你的 agent 真正跑起来），然后把 `evals/` 下所有用例依次发给它、收集结果、打印汇总。

> 💡 小贴士：muse-agent 项目的 `package.json` 把 `eval` 脚本映射到了 `eve eval`，所以你也可以用 `npm run eval` / `pnpm eval`，效果一样。

几个起步阶段够用的玩法：

```bash
eve eval                 # 跑 evals/ 下全部用例（针对本地 dev server）
eve eval smoke           # 只跑 id 为 smoke 的用例
eve eval weather         # 跑 evals/weather/ 目录下的所有用例（按目录前缀匹配）
eve eval --list          # 只列出发现了哪些用例，不真的跑（确认 eve 认得你的文件）
eve eval --verbose       # 把每个用例的 t.log 输出实时打到屏幕上
```

> 💡 小贴士：命令后面跟的位置参数，要么**精确匹配某个用例 id**（比如 `smoke`），要么**按目录前缀匹配**（比如 `weather` 命中 `evals/weather/` 下的全部用例）。

关于**退出码**，记住最关键的几条：**`0` 表示所有门禁都通过了**；`1` 表示有用例失败了（门禁挂了、执行报错，或加了 `--strict` 后软断言没达标）；`2` 表示配置出错。这正是 evals 能接进 CI 的原因——CI 看退出码就知道这次改动有没有让 agent 行为退化。

如果你想测一个**已经部署上去**的 agent（而不是本地），加 `--url`：

```bash
eve eval --url https://你的-app.vercel.app
```

同一套 `.eval.ts` 文件，本地和远程都能跑，不用改一行。

> ⚠️ 注意：`eve eval`（本地，针对 dev server）发请求时**不带任何鉴权**——dev server 本来就是给本地用的。但 `--url` 打到真实部署时鉴权规则要复杂得多（涉及 Vercel 项目匹配、OIDC token 等）。新手起步先专注本地 `eve eval` 就好，远程鉴权细节以官方文档为准。

### 11.8 起步建议：先写一个 smoke 冒烟测试

讲了这么多，新手到底第一步该做什么？答案非常明确：**先写一个 smoke（冒烟）测试。**

"冒烟测试"这个词来自硬件：新做的板子第一次通电，先看会不会冒烟。放到 agent 上，它的含义就是最低限度的一句话——**"我的 agent 能不能接收一条请求，并且顺利完成一轮？"** 不测它聪不聪明，只测它"还活着、还能跑通"。

这恰恰就是 muse-agent 真实在用的那个用例。完整步骤如下。

第一步，确保 `evals/evals.config.ts` 存在（前面说过，必须恰好有一个）：

```ts
// evals/evals.config.ts
import { defineEvalConfig } from "eve/evals";

export default defineEvalConfig({});
```

第二步，新建 `evals/smoke.eval.ts`：

```ts
// evals/smoke.eval.ts
import { defineEval } from "eve/evals";

export default defineEval({
  description: "开机冒烟：agent 能接收请求并完成一轮。",
  async test(t) {
    await t.send("Hello! Can you introduce yourself in one sentence?");
    t.completed();
  },
});
```

第三步，跑起来：

```bash
eve eval smoke
```

看到退出码为 0、汇总里这个用例是 `passed`，你就有了第一道"行为漂移"的警报线。

这个起步选择有三个好处，值得你品一下：

- **它是确定性的**，不需要评审模型，不花一分模型钱，也不依赖任何凭证。
- **它几乎不会误报**：只要 agent 没彻底坏掉，它就能过；一旦哪天它真过不了，说明你的 agent 连"接收消息并跑完一轮"这种最基本的能力都丢了——这绝对值得你立刻停下来查。
- **它给你搭好了脚手架**。有了这个能跑通的 smoke，往后你每加一个能力，就顺手在 `evals/` 里加一个对应的确定性用例（"问天气要调 get_weather""打招呼时不该乱调工具"……），你的"测验题库"就这样一点点长起来了。

> 💡 小贴士：养成习惯——每修一个 agent 的 bug，就补一个 eval 把它"钉死"。下次行为再漂回去，这个 eval 会第一时间替你喊出来。这就是 evals 存在的全部意义：让 agent 的行为变得**可回归、可验证**，而不是每次上线都靠人肉聊两句碰运气。

至此，你已经掌握了给 eve agent 写自动化测试的完整起步路径：`evals/` 目录 + 一个 `evals.config.ts` + 一个 smoke 冒烟用例 + `eve eval` 跑起来。接下来随着 agent 能力变多，你只需要不断往这个框架里加用例就好。

---

## 12. 部署上线：把 agent 发布到 Vercel

到这一章为止，你已经把一个 agent 的"零件"都装齐了：`agent.ts` 里选好了模型，`instructions.md` 写好了人格，`tools/` 里有工具，`connections/` 接了外部服务，`channels/` 决定谁能进门，可能还有 `schedules/` 定时任务。但这些到目前为止全都只在你自己电脑的 `eve dev` 里转。

本章要做的，就是把它**搬到云上**，让它有一个真正的公网地址、24 小时在线、能被 Slack 或别人调用。我们用的平台是 Vercel——eve 本来就是为它而生的，所以这一步出乎意料地顺。

> 💡 小贴士：可以这样理解"部署"——你现在的 agent 像一道在自家厨房做好的菜，部署就是把这道菜的"做法 + 食材清单"打包送到一家中央厨房（Vercel），由它批量、稳定地做出来端给所有客人。你的源代码不变，变的只是运行它的地方。

### 12.1 先装 Vercel CLI 并登录

Vercel CLI 是一个命令行工具，让你不用打开网页就能从终端把项目推上去。先全局安装它：

```bash
npm i -g vercel
```

然后登录。这一步会打开浏览器让你用 Vercel 账号授权一次，之后这台电脑就记住你了：

```bash
vercel login
```

> 💡 小贴士：如果你还没有 Vercel 账号，去 `https://vercel.com` 免费注册一个即可。`vercel login` 会引导你完成全部步骤。

### 12.2 两种部署：预览（preview）和生产（production）

Vercel 有一个很贴心的设计：**每次部署默认是"预览部署"，只有你明确指定才会推到"生产"**。

- **预览部署**：给你一个临时的、独立的 URL，用来自己先点一点、测一测，不影响线上正在跑的版本。相当于"内部试吃"。
- **生产部署**：推到你的正式域名，所有真实用户访问的就是这个版本。相当于"正式上菜"。

在项目根目录（也就是有 `agent/` 文件夹的那一层）运行下面这条，做一次预览部署：

```bash
vercel
```

第一次运行时它会问你几个问题：要不要把当前目录关联（link）到一个 Vercel 项目、项目叫什么名字等等。一路按提示回答即可。跑完后，终端会打印出一个预览 URL。

确认预览没问题后，再推生产：

```bash
vercel --prod
```

> 💡 小贴士：eve 还提供了一条更省事的封装命令 `eve deploy`，它本质上就是帮你跑 `vercel deploy --prod`，并且会**先帮你装好依赖、再拉取环境变量**。如果你的项目已经 link 过 Vercel 项目，直接 `eve deploy` 即可一步到生产；如果还没 link 且终端是交互式的，它会先带你完成 link。两种方式都行，新手建议先用 `vercel` / `vercel --prod` 把流程走通，理解每一步在干什么。

无论用哪种方式，一个 eve agent 部署后就是一个**普通的 Vercel 项目**——你之后也可以把代码推到 Git 仓库，让 Vercel 在每次 push 时自动部署。

### 12.3 部署后怎么验证它真的活着

部署完别急着关终端。eve 的每个 app 都默认暴露一个**健康检查路由** `/eve/v1/health`，它永远是公开的（不需要任何认证），专门给负载均衡器和你这种"想确认它没死"的人用。

把下面的 `<your-app>` 换成你拿到的真实域名：

```bash
curl https://<your-app>/eve/v1/health
```

能正常返回，说明服务起来了。接着可以发一条真实消息试试——注意这条 `POST /eve/v1/session` 路由是**需要认证**的，下一节会讲怎么配，这里先看形状：

```bash
curl -X POST https://<your-app>/eve/v1/session \
  -H 'content-type: application/json' \
  -d '{"message":"Hello from production"}'
```

你也可以用本地的 `eve dev` 去"遥控"一个已部署的 app，得到和本地一样的交互式终端界面：

```bash
eve dev https://<your-app>
```

### 12.4 环境变量与密钥：绝不要写死在代码里

这是上线最容易踩坑、也最该认真对待的一节。

你的 agent 里几乎一定用到了一些**密钥**：连接外部服务的 token、运维者登录用的用户名密码、模型的 API key……回想前面几章里那些写法：

```ts
auth: { getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }) }
```

注意它们读的是 `process.env.XXX`，**而不是把真实的 token 直接敲进代码**。这是一条铁律：

> ⚠️ 注意：永远不要把密钥、token、密码硬编码（hardcode）进源代码。一旦写进代码，它就会进 Git 历史、进部署产物，任何能看到代码的人都能拿到。正确做法是让代码读 `process.env.<名字>`，把真实值放在**环境变量**里。

#### 本地：用 `.env` / `.env.local`

eve 的命令行在启动时会**先加载 `.env` 和 `.env.local`**。所以本地开发时，你只要在项目根目录建一个 `.env.local` 文件，把密钥写进去：

```bash
OKX_BRIDGE_TOKEN=你的真实token
ROUTE_AUTH_BASIC_PASSWORD=你的运维密码
```

> ⚠️ 注意：务必把 `.env.local`（以及任何含密钥的 `.env*`）加进 `.gitignore`，不要提交到 Git。`eve init` 脚手架通常已经帮你配好了，但请自己确认一遍。

#### 线上：放进 Vercel 项目的环境变量

本地的 `.env.local` **不会**被部署上去（这是好事）。所以线上需要的每一个密钥，都要单独配置到 Vercel 项目的环境变量里。两种配法：

1. 在 Vercel 网页后台：进入你的项目 → Settings → Environment Variables，逐条添加。
2. 用 CLI 在部署时注入，例如 `vercel deploy --prod -e OKX_BRIDGE_TOKEN=xxx`（`-e KEY=value` 设置运行时环境变量；一般还是推荐用后台，便于管理和复用）。

哪些东西要进 Vercel 环境变量？把你代码里所有 `process.env.*` 读到的、且属于密钥/凭证性质的都过一遍，典型的有：

- **连接（connection）用到的 token**，例如 `OKX_BRIDGE_TOKEN`、`LINEAR_API_TOKEN`、各种 `*_API_KEY`。
- **运维者 / 操作者凭证**，例如 channel 路由认证用的 `ROUTE_AUTH_BASIC_PASSWORD`，以及你自己实现的 JWT/OIDC 签名密钥。
- **模型凭证**（下一节细说）。
- 若你用了 Vercel 的预览保护，本地跑 `eve dev` 前需要设 `VERCEL_AUTOMATION_BYPASS_SECRET`。

> 💡 小贴士：配好线上环境变量后，可以用 `vercel env pull` 把它们拉回本地、生成一个 `.env.local`，省得本地线上两边手动同步。

#### 一个常被忽略的好消息：路由认证密钥不会进编译产物

你在 channel 里配的路由认证（比如 `httpBasic` 用到的 `ROUTE_AUTH_BASIC_PASSWORD`、各种签名密钥）是从**环境变量**里读的，eve 在启动时才从 channel 定义里重新取它们，**不会把它们烤进编译后的静态产物**。所以只要你老老实实用 `process.env`，密钥就不会泄漏到 `.eve/` 这类构建文件里。

### 12.5 模型经 AI Gateway 访问

你的 agent 要调用大模型（比如 `anthropic/claude-opus-4.8`）。eve 默认让这些请求走 **Vercel AI Gateway**——你可以把它理解成一个"模型总机"：你只管报一个 `<提供商>/<模型>` 的字符串，它负责路由到对应的模型提供商、处理失败重试。好处是你不必为每个提供商单独管一把 key。

每次走 AI Gateway 的请求都需要认证，有两种方式：

- **在 Vercel 上（推荐，零配置）**：当你的项目已经 link 到 Vercel 后，平台会自动提供一个 `VERCEL_OIDC_TOKEN`，模型请求通过 **OIDC** 自动认证，**你不需要管任何模型 API key**，也没有 key 需要轮换。这是上线后最省心的路径。
- **在 Vercel 之外，或本地开发**：设置环境变量 `AI_GATEWAY_API_KEY`，AI SDK 会自动用它。

```bash
export AI_GATEWAY_API_KEY="你的AI Gateway密钥"
```

> 💡 小贴士：本地要让脚手架默认模型跑起来，你需要 `AI_GATEWAY_API_KEY`，**或者**一个 `VERCEL_OIDC_TOKEN`（运行 `vercel link` 关联项目后即可获得）。所以一个常见的最小本地配置就是：先 `vercel link`，本地就能借到 OIDC 凭证。

如果你不想走 Gateway、而是直连某个提供商（比如直接用 `@ai-sdk/anthropic`），那就需要装对应的 provider 包并提供该提供商自己的 key（如 `ANTHROPIC_API_KEY`）。这属于进阶用法，新手用 Gateway 默认路径即可。

> 💡 小贴士：还有一条更省事的命令 `eve link`——它会把目录关联到 Vercel 项目**并顺手取回 AI Gateway 凭证**，对刚上手的人很友好。

### 12.6 可观测性：上线后怎么"看见"agent 在干什么

agent 部署上去之后，你最关心的是：它到底跑了哪些会话？花了多少 token？某次出错卡在哪一步？eve 在这方面几乎是**开箱即用**的。

#### Agent Runs 视图（默认就有，无需任何配置）

eve 会给**每一次 workflow run（每一次会话/任务的执行）自动打上一组 `$eve.*` 属性**。正是这些属性，驱动了 Vercel 后台里的 **Agent Runs** 视图。你什么都不用配（不需要 `instrumentation.ts`），它就在那儿。

在 Vercel 项目里找到 **Agent Runs**（路径形如 `项目 → Observability → Agent Runs`），你能看到：

- **总览**：按触发来源（比如 Slack、HTTP）统计的运行次数随时间变化；token 用量（输入 / 输出 / 缓存分开算）；一张运行列表表格（触发它的那条消息、触发类型、输入输出 token、轮次数、耗时、时间）。
- **单次运行详情**：用了哪个模型、什么触发、哪个部署；逐 turn 拆解，包括每一步的耗时（含 skill 加载和每一次工具调用）、输入、输出、推理过程、工具调用（参数 + 结果），以及输入 / 缓存 / 输出各类 token 计数。

对新手来说，调试线上问题第一站就该是这里：哪次跑挂了、模型说了什么、工具传了什么参数、卡在哪个审批上，一目了然。

> ⚠️ 注意（隐私）：Agent Runs 会记录运行过程中的输入输出。作为部署者，当你的 agent 处理个人、敏感或受监管的数据时，你可能需要依照适用法律、在你的隐私声明中披露这种记录行为。这是你的责任，eve 不会替你自动加这类披露。

#### 可选进阶：用 `instrumentation.ts` 接 OpenTelemetry

如果默认的 Agent Runs 还不够，你想把追踪数据导出到自己的可观测性平台（Braintrust、Honeycomb、Datadog、Jaeger 等任何兼容 OpenTelemetry 的后端），就可以加一个 `agent/instrumentation.ts` 文件。

它的工作方式很有意思，记住几个要点就不会出错：

- 这个文件是**根级（root-only）**的，eve 会**自动发现**它，并在服务启动时、任何 agent 代码运行之前先执行它。
- **只要这个文件存在并有默认导出，就隐式开启了 OTel 导出——没有任何额外的开关，也没有 `isEnabled` 字段。**
- 用 `eve/instrumentation` 里的 `defineInstrumentation` 来定义。**它没有顶层的 `exporter` 字段**——exporter（导出器）是在 `setup` 回调里创建的。

一个典型例子（把追踪导到 Braintrust）：

```ts
// agent/instrumentation.ts
import { BraintrustExporter } from '@braintrust/otel';
import { defineInstrumentation } from 'eve/instrumentation';
import { registerOTel } from '@vercel/otel';

export default defineInstrumentation({
  setup: ({ agentName }) =>
    registerOTel({
      serviceName: agentName,
      traceExporter: new BraintrustExporter({
        parent: `project_name:${agentName}`,
        filterAISpans: true,
      }),
    }),
});
```

`defineInstrumentation` 可配的字段：

| 字段 | 作用 | 默认值 |
|---|---|---|
| `setup` | 注册 OTel provider，回调收到 `{ agentName }`；exporter 在这里创建 | — |
| `recordInputs` | 是否把每一步完整的消息历史记到 span 上 | `true` |
| `recordOutputs` | 是否把模型输出记到 span 上 | `true` |
| `functionId` | 覆盖 span 上的函数名 | 默认为 agent 名 |
| `events` | 事件回调（如 `step.started`，可返回挂到模型调用 span 上的 `runtimeContext`） | — |

> ⚠️ 注意（敏感场景）：如果你的 agent 会处理敏感的输入或输出，把 `recordInputs` 和 `recordOutputs` 设为 `false`。这样追踪里就不会带上完整的消息内容和模型输出，既保护隐私也减小数据体积。

> 💡 小贴士：连最简单的 `export default defineInstrumentation({})` 都会隐式开启 OTel。所以如果你**还不想**接外部追踪，干脆别建这个文件——光靠默认的 Agent Runs 就够你用很久了。

### 12.7 schedules 会自动变成 Vercel Cron

如果你写了 `agent/schedules/` 下的定时任务，这里有一个让人安心的事实：**部署到 Vercel 后，每一个 `defineSchedule(...)` 都会被自动转换成一个 Vercel Cron Job**，按 UTC 时区评估它的 cron 表达式（比如 `"0 9 * * 1-5"` 表示工作日 UTC 09:00 触发）。你不用单独去配什么定时器。

部署后可以在 Vercel 后台 **Settings → Cron Jobs** 里确认它们已注册，在 **Observability → Cron Jobs** 看执行历史，在 **Observability → Logs** 看每次运行的日志。

> ⚠️ 注意：本地的 `eve dev` **永远不会**按 cron 的节奏自动触发定时任务——这是设计如此，免得你本地一开机就被定时任务轰炸。本地想测，用前面章节讲过的开发派发路由手动触发一次即可。

> 📌 一个容易踩的坑：Vercel 的 Cron Job 由 Vercel 平台自身的调度器触发。如果你不是部署在 Vercel、而是把构建产物搬到了别的容器或进程管理器上，定时任务能编译出来、却不一定会自动触发——这时要用 `eve start`（它会启动调度运行器），或换一个支持该调度的宿主，或干脆用你自己的外部调度器去打一个受认证保护的路由。本教程的标准路径（部署到 Vercel）不受此影响。

### 12.8 一句话提一下：Vercel Passport

如果你读过 Vercel 的发布博客，可能会看到 **Passport** 这个名字。简单说一句即可：**Passport 不是 eve 的一部分**，它是 Vercel 同期发布的一个企业级**身份治理**产品——把你公司内部的 app 和 agent 默认放到组织的身份提供商（IdP，基于 OIDC）后面，做集中的访问治理与审计。eve 负责"造和跑这个 agent"，Passport 负责"把它放到公司身份体系后面管起来"，两者分工不同。对新手教程而言，知道有这么个东西、不要和 eve 的功能混为一谈，就足够了，具体配置以官方文档为准。

### 12.9 上线前检查清单

把 agent 推到生产之前，对着下面这张清单逐项过一遍：

- [ ] **本地 `eve dev` 跑通**：基本对话、关键工具、连接都能正常工作。
- [ ] **没有任何硬编码密钥**：全文搜一遍，确认所有 token / 密码 / API key 都走 `process.env`，没有一个是写死的字面量。
- [ ] **`.env.local` 已在 `.gitignore` 里**：确认含密钥的文件不会被提交。
- [ ] **线上密钥已进 Vercel 环境变量**：把代码里每一个 `process.env.*`（连接 token、运维凭证、签名密钥等）都在 Vercel 项目里配好。
- [ ] **模型凭证就绪**：在 Vercel 上用 OIDC（项目已 link）即可；本地确认有 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN`。
- [ ] **路由认证不是占位符**：确认你的 `agent/channels/eve.ts` 用的是真正的认证器（如 `vercelOidc()` / `httpBasic()` / 自定义 `AuthFn`），**而不是脚手架默认的 `placeholderAuth()`**。eve 默认 **fails closed（默认拒绝）**——没配好认证，生产环境的浏览器流量会被直接拒绝，这是保护你，但你必须主动替换占位符。
- [ ] **高风险工具有审批门 / 认证门**：涉及花钱、改数据等不可逆操作的工具或连接，确认挂了 `needsApproval`（如 `always()`）或连接级 `approval`，别让模型一句话就把事办了。
- [ ] **敏感场景关掉记录**：如果接了 `instrumentation.ts` 且处理敏感数据，确认 `recordInputs` / `recordOutputs` 已按需设为 `false`，并补齐隐私披露。
- [ ] **定时任务确认无误**：检查 cron 表达式（记住是 **UTC**），部署后到 Vercel 后台确认 Cron Job 已注册。
- [ ] **先预览再生产**：先 `vercel` 做预览部署、用 `/eve/v1/health` 和一条真实消息验证，再 `vercel --prod`。

走完这张清单，你的 agent 就正式上线了——一个有公网地址、durable（可持久恢复）、可观测、带审批护栏的生产级后端 AI agent。恭喜你，从零跑完了 eve 的完整旅程。

> 📦 **想完全脱离 Vercel？** 如果你需要把 agent 私有化／自托管在自己的机器、内网或服务器上，跳到文末 **附录 A · 本地与私有化（自托管）部署**。

---

## 13. 实战拆解：读懂一个真实的 eve 项目（muse-agent）

前面 12 章，我们一个概念一个概念地拆零件：`defineAgent`、`defineTool`、`defineMcpClientConnection`、channels、sandbox、schedules、evals……每一块都单独讲过。但新手最常见的卡点不是"看不懂某一块"，而是"看得懂零件，却拼不出整机"——一个真正能跑在生产环境、还碰真金白银的 agent，到底长什么样？这些零件是怎么组合到一起的？

这一章我们换一种方式：不再讲新 API，而是带你**逐文件读懂一个真实项目**。

> 💡 小贴士：读别人的代码是成长最快的方式。这一章相当于一次"代码导览"，请你打开心态——每个文件我们都会先问"它解决什么问题"，再看"它怎么写的"。

### 13.1 先认识这个项目：muse-agent 是什么

`muse-agent` 是一个真实的、生产形态的 **OKX 交易 agent**（OKX 是一家加密货币交易所）。它跑在 **eve@0.13.5** 上。简单说，它能做两类事：

- **读**行情、查账户、扫市场（只读、无风险）；
- **下单交易**（写操作、碰真钱、有风险）。

正因为它要碰真钱，这个项目把"安全"这件事做到了极致。它最值得新手学习的，不是"怎么调 OKX 接口"，而是**它如何用纯代码、分层地把"能力"和"安全"组合在一起**。读完这一章，你会建立一个非常重要的直觉：

> 一个生产级 agent，安全不是某一处的"开关"，而是**多层防线叠加**——每一层都独立失效仍能兜底。

我们后面会重点讲它的"分层代码级安全模型"，先把每个文件过一遍。

### 13.2 第一个文件：package.json —— 项目的"身份证"

任何 Node 项目，第一个该看的都是 `package.json`，它告诉你这个项目用什么版本、能跑哪些命令。

```json
{
  "type": "module",
  "engines": { "node": ">=24" },
  "scripts": {
    "dev": "eve dev",
    "build": "eve build",
    "start": "eve start",
    "info": "eve info",
    "eval": "eve eval"
  },
  "dependencies": {
    "eve": "0.13.5",
    "ai": "^7.0.0-beta",
    "zod": "^4"
  }
}
```

逐行读这里的关键信息：

- **`"engines": { "node": ">=24" }`**：eve 要求 **Node.js >= 24**。这不是建议，是硬性要求——Node 版本不够，连 `eve dev` 都起不来。

> ⚠️ 注意/坑：很多新手第一次跑 eve 项目失败，原因就是 Node 版本太老。先用 `node -v` 确认你在 24 或以上。

- **`scripts`**：这五个脚本，本质上就是把 eve 的 CLI 命令映射成了 `npm run`。这意味着下面两种写法完全等价：

```bash
npm run dev
# 等价于
npx eve dev
```

  这五个命令我们前面都讲过，这里快速对照一下它们在这个项目里的角色：

  | 脚本 | 实际命令 | 干什么 |
  |---|---|---|
  | `dev` | `eve dev` | 本地开发服务器 + 终端交互 UI，边写边试 |
  | `build` | `eve build` | 把 agent 编译进 `.eve/`，产出可部署产物 |
  | `start` | `eve start` | 跑已编译产物（也是生产里真正触发定时任务的命令） |
  | `info` | `eve info` | 打印 eve 发现到的所有东西：工具、连接、通道、定时任务…… |
  | `eval` | `eve eval` | 跑 eval 测试套件 |

> 💡 小贴士：当你 clone 一个陌生的 eve 项目、想快速知道"它到底有哪些能力"时，第一件事就是 `npm run info`（即 `eve info`）。它会把 eve 自动发现的工具、连接、通道、定时任务等全列出来——这正是"文件即接口"理念的好处：你不用翻代码，CLI 直接告诉你。

- **`dependencies`**：只有三个直接依赖——`eve`、`ai`、`zod`。这正是 `npx eve@latest init` 脚手架装的那三件套。`ai` 是 AI SDK，`zod` 用来写工具的 `inputSchema`。

### 13.3 agent.ts —— agent 的大脑（用哪个模型）

```ts
// agent/agent.ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-opus-4.8",
});
```

还记得第 3 章吗？`agent.ts` 是 agent 的运行时配置文件。这个文件本身是**可选的**，但**一旦存在，`model` 就是必填的**。

这里只配了一个字段 `model`，值是 `"anthropic/claude-opus-4.8"`。这是一个 **AI Gateway 模型字符串**，格式是 `"<provider>/<model>"`。

为什么这里能只写一个字符串、不写任何 API key？回忆第 3 章和第 12 章的关键点：

- 部署在 Vercel 上时，模型请求经过 **AI Gateway**，通过 **OIDC** 自动鉴权，**你不需要管理任何 provider 密钥**；
- 本地开发时，需要一个 `AI_GATEWAY_API_KEY`（或 `VERCEL_OIDC_TOKEN`）。

> 💡 小贴士：你可能注意到不同文档里出现过不同的模型字符串（`anthropic/claude-sonnet-4.6`、`openai/gpt-5.4-mini` 等）。它们都是合法的 Gateway 字符串。muse-agent 选了 `claude-opus-4.8`——一个能力更强的模型，因为交易决策需要更强的推理。具体哪些模型字符串可用，以官方 AI Gateway 文档为准。

注意这里**没有** `tools`、`temperature`、`subagents` 这些字段。这是 eve 的核心理念：**能力来自文件系统的位置，而不是写在配置里**。工具放进 `agent/tools/` 就自动被发现，你永远不需要在 `defineAgent` 里手动登记它们。（采样参数如 temperature 不写在这里，需要时通过 `modelOptions` 转发给模型调用。）

### 13.4 instructions.md —— agent 的"永久人格"

```md
You are a concise assistant. ...
（这里是描述能力、操作规程、边界的系统提示词）
```

`instructions.md` 是 agent 的**常驻系统提示词**——它会被拼进每一次模型调用，是 agent 的"永久身份"，而不是某个一次性的步骤。第 4 章讲过，它**在根目录是必填的**。

muse-agent 的 `instructions.md` 是一份纯 markdown 系统提示词，按三块组织：

1. **能力（capabilities）**：告诉模型"你能做什么"——能读行情、能下单等；
2. **操作规程（how-to-operate）**：告诉模型"该怎么做事"——比如下单前先确认、遵守什么流程；
3. **边界（boundaries）**：告诉模型"什么不能做"——风险红线、禁止行为。

一个值得学习的细节：这份 instructions **提到了 `connection_search`**。

为什么要在提示词里提它？回忆第 7 章：当 agent 声明了连接（connections）后，eve 会自动注入一个内置工具 **`connection_search`**，模型用它来"发现"各个连接下有哪些可调用的工具。在 instructions 里点名 `connection_search`，等于在告诉模型："你想用 OKX 的某个能力时，先用 `connection_search` 去查有哪些工具可用，再按 `<连接名>__<工具名>` 的格式调用。"

> 💡 小贴士：这是一个很实用的提示词技巧——**主动告诉模型框架给了它哪些内置工具、该怎么用**。模型不会凭空知道你的连接里有什么，`connection_search` 就是它的"目录检索"入口。

> ⚠️ 注意/坑：instructions **永远不执行代码**。它只是文字。任何需要"真正做事"的行为（调接口、下单），都必须通过工具或连接来完成，不能指望写在 instructions 里就生效。

### 13.5 两个连接：okx-read.ts 与 okx-trade.ts —— 能力分层的关键一刀

这是整个项目最精彩的部分。muse-agent 把"读"和"写"拆成了**两个独立的连接文件**，而不是一个。

先看只读连接：

```ts
// agent/connections/okx-read.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: process.env.OKX_READ_MCP_URL ?? "http://127.0.0.1:8000/mcp",
  description: "OKX 只读行情/账户查询……",
  auth: {
    getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }),
  },
});
```

再看交易连接：

```ts
// agent/connections/okx-trade.ts
import { defineMcpClientConnection } from "eve/connections";
import { always } from "eve/tools/approval";

export default defineMcpClientConnection({
  url: process.env.OKX_TRADE_MCP_URL ?? "http://127.0.0.1:8001/mcp",
  description: "OKX 下单/交易……",
  auth: {
    getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }),
  },
  approval: always(),   // ← 关键就这一行
});
```

两个文件几乎一模一样，**唯一的实质区别是 `okx-trade.ts` 多了 `approval: always()`**。我们来逐点拆。

#### 连接是什么、为什么用两个

回忆第 7 章：**连接（connection）是 agent 与外部服务之间的一座桥**，可以是一个 MCP 服务器，也可以是一个带 OpenAPI 文档的接口。这里两个都是 MCP 连接（`defineMcpClientConnection`）。

> 💡 小贴士：连接的运行时名字来自**文件名**——`okx-read.ts` → 连接名 `okx-read`。所以模型调用只读工具时，看到的工具名是 `okx-read__<工具>`；调用交易工具时是 `okx-trade__<工具>`。这种 `<连接名>__<工具名>` 的限定命名，正是第 7 章讲过的调用约定。模型永远看不到连接里的 `url` 和 `token`——凭据被框架挡在提示词之外。

为什么要拆成两个连接，而不是一个连接里既有读又有写？因为**拆开之后，"读"和"写"就可以有不同的安全策略**。这就引出了关键的一行。

#### `approval: always()` —— 每一笔写操作都停下等人批

`okx-read.ts` **没有** `approval` 字段。第 7 章讲过，连接的 `approval` 默认是 `never()`——也就是不需要审批。所以这个连接下的**每一个工具都是一次纯查询，畅通无阻**。读行情、查余额，没风险，不该打断 agent。

`okx-trade.ts` 写了 `approval: always()`。`always()` 来自 `eve/tools/approval`，语义是"**每一次调用都需要人工审批**"。它的效果是：这个连接下的**每一个工具**，在真正执行之前，都会让会话**持久挂起在 `session.waiting` 状态，一直等到有人点了"批准"，才会继续执行**。

这就是这个项目的"**真金白银安全控制**"。任何一笔下单，agent 都不能自己拍板——它必须停下来，等一个人类按下批准键。

> 💡 小贴士：这里有一个 eve 非常优雅的设计——**等待审批是"免费"的**。一个挂起等审批的会话靠 Vercel Workflows 的事件日志持久化，**可以无限期地等下去，而完全不消耗计算资源**。它不是在那儿空转烧钱，而是真正"暂停"了。人什么时候来批都行，哪怕过了几个小时。

> ⚠️ 注意/坑：`approval` 加在**连接**上，作用于这个连接下的**所有**工具。所以你不用给每个交易工具单独加审批——一行 `approval: always()`，整条"写"通道就全锁上了。这正是"按读/写拆成两个连接"带来的好处：策略可以整条连接统一施加。

#### 一个真实的"坑"：连接文件名与 url 校验

muse-agent 在实际使用中验证过两个容易踩的坑，新手务必记住：

> ⚠️ 注意/坑（已验证）：
> 1. **连接文件名要用 dash-case（短横线命名）**，比如 `okx-read.ts`、`okx-trade.ts`。
> 2. **eve 在发现/编译阶段会"提前"校验连接的 url**——一个写错的、或空的 url 会**立刻失败**（fail fast），而不是等到运行时才报错。所以要确保 url 在编译时就能解析出来（这也是为什么上面用了 `process.env.OKX_READ_MCP_URL ?? "http://127.0.0.1:8000/mcp"`——给一个兜底默认值，保证编译期 url 一定非空）。

### 13.6 为什么需要一个"桥"：stdio MCP → HTTP/SSE

这里要插入一个对新手很关键、但容易被跳过的架构知识点。

第 7 章讲过：eve 的 MCP 连接，**`url` 必须说 Streamable HTTP 或 SSE 这两种"语言"之一**。换句话说，eve 只会通过 HTTP/SSE 去连一个 MCP 服务器，**不直接支持 stdio**。

但现实里，**很多 MCP 服务器只会说 stdio**（标准输入输出，本质上是"启动一个本地子进程，通过管道收发消息"）。muse-agent 用的 `okx-trade-mcp` 就是一个**只支持 stdio 的 MCP 服务器**。

这就矛盾了：eve 要 HTTP/SSE，服务器只会 stdio，两边对不上。

解决办法是**加一座桥（bridge）**：一个中间程序，一头用 stdio 启动并连上那个 MCP 服务器，另一头对外暴露成 HTTP/SSE，给 eve 来连。muse-agent 用的桥是 **supergateway**（一个把 stdio MCP 转成 HTTP 的工具）。

用一张图理解这个数据流：

```
eve 连接 (okx-trade.ts)
   │  HTTP/SSE
   ▼
supergateway（桥：HTTP/SSE  <—>  stdio）
   │  stdio（启动子进程，管道收发）
   ▼
okx-trade-mcp（只会 stdio 的 MCP 服务器）
   │
   ▼
OKX 交易 API
```

把这条经验**一般化**，记成一句话就够了：

> ⚠️ 注意/坑：**stdio 的 MCP 服务器不能直接连进 eve，你需要一座 HTTP/SSE 桥。** eve 本身不打包、也不指定用哪个桥；社区有一些工具可以做这件事，具体选型以官方文档与各工具自身文档为准。

> 💡 小贴士：连接配置里的 `OKX_READ_MCP_URL`、`OKX_BRIDGE_TOKEN` 这些环境变量，指向的其实就是这座桥的地址和访问它所需的令牌——eve 连的是桥，桥再去连真正的 MCP 服务器。

### 13.7 channels/eve.ts —— 谁能敲门（有序鉴权、fail closed）

连接管的是"agent 往外怎么登录别人的服务"（出站）；**通道（channel）管的是"谁能从外面进来、触达这个 agent"（入站）**。这是两套完全独立的鉴权系统，第 8 章讲过，别混淆。

```ts
// agent/channels/eve.ts
import { eveChannel } from "eve/channels/eve";
import { localDev, vercelOidc, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [
    localDev(),
    vercelOidc(),
    httpBasic({ username, password }),
  ],
});
```

#### 通道是什么、为什么要写这个文件

第 8 章讲过一个重要前提：**每个 eve 应用默认就暴露一个 HTTP 通道**，哪怕你**根本不写** `agent/channels/eve.ts`。那为什么 muse-agent 还要专门写这个文件？

答案是：**为了覆盖默认行为，主要是配置路由鉴权（route auth）**。默认的鉴权对一个碰真钱的生产 agent 来说不够用，所以它显式接管。

#### 有序的鉴权走查（auth walk）

`auth` 接收一个数组，eve 会**按顺序逐个走查（walk）**这个数组。第 8 章详细讲过这套语义，这里在真实项目上再过一遍：

- 某一项返回了一个有效的鉴权上下文 → **接受，停止走查（第一个匹配的赢）**；
- 某一项返回 `null`/`undefined` → **跳过，试下一项**；
- 某一项**抛出异常** → **以特定状态码拒绝**；
- **所有项都跳过了 → 返回 `401`**。

muse-agent 这三项，正好覆盖三类合法来访者：

| 顺序 | 鉴权器 | 接受谁 |
|---|---|---|
| 1 | `localDev()` | 本地开发：发往 loopback（`localhost`/`127.x`/`::1`）主机名的请求，以及 `vercel dev` |
| 2 | `vercelOidc()` | 部署在 Vercel 上时，携带有效 Vercel OIDC 令牌的调用方（含 eve 自己的内部运行时/子 agent 调用） |
| 3 | `httpBasic({ username, password })` | **运维者**：拿着共享用户名/密码的人工/服务访问 |

第三项 `httpBasic` 是这个项目特意加的——它给了一个**运维者人工入口**：一个真人运维，可以拿账号密码直接打这个 agent 的 HTTP 接口（比如手动发一条指令、或者去批准一笔挂起的交易）。

> ⚠️ 注意/坑：`localDev()` 是按**请求 URL 的主机名**判断 loopback 的，并不读 `process.env.VERCEL`。它信任对外宣称的主机名——如果没有规范化的代理挡在前面，能注入 `Host` 头的攻击者可以伪造它。所以**永远不要只靠 `localDev()` 跑生产**，一定要在它之上再叠一个真实鉴权器（muse-agent 正是这么做的：后面还有 `vercelOidc()` 和 `httpBasic()`）。

#### fail closed（默认关门）：为什么走到尽头是 401

这套设计最关键的安全性质叫 **fail closed**——**默认是"关门"的**。eve 官方对路由鉴权的原话是：默认 fail closed，**生产环境的浏览器流量会被拒绝，除非你显式配置了一个接受它的鉴权器；而匿名访问必须用一个显式的 `none()` 才放行**。

落到 muse-agent 上：如果一个请求**三项都不匹配**（不是本地、没有有效 Vercel 令牌、也没带对账号密码），走到数组尽头，eve 就回 **`401`**，把它挡在门外。**它不会"不确定就放行"，而是"不确定就拒绝"。**

> 💡 小贴士：注意 muse-agent **故意没有用 `placeholderAuth()`**。`placeholderAuth()` 是脚手架默认塞进去的占位鉴权器，作用是在你还没配好真鉴权时，于生产环境返回一个"鉴权还没配好"的 `401`。muse-agent 是个真项目，鉴权早就配好了，所以它直接换成了三段真鉴权——这正是从"脚手架占位"走向"生产配置"该做的事。

> ⚠️ 注意/坑：路由鉴权只管"谁能进门"，**它不负责"会话归属（session ownership）"**——即它不会自动保证"A 用户只能看 A 自己的会话"。如果你需要按用户/租户隔离，得自己在业务层做。对 muse-agent 这种运维者驱动的小范围 agent 来说够用，但你做多用户产品时要记住这一点。

> 💡 小贴士：还有一个特例值得知道——`GET /eve/v1/health` 这个健康检查路由**永远是公开的**，会跳过整套鉴权走查，方便负载均衡器/探活监控来 ping。它不在 fail-closed 的管辖范围内。

### 13.8 sandbox/sandbox.ts —— 模型的"草稿纸"，全锁死（deny-all）

```ts
// agent/sandbox/sandbox.ts
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    vercel: { networkPolicy: "deny-all" },
    docker: { networkPolicy: "deny-all" },
  }),
});
```

#### sandbox 是什么、它不是什么

第 10 章讲过一个极其重要、也极容易搞混的安全区分：

- **工具（tools）跑在你的应用运行时（app runtime）**——能读 `process.env`、能 import 共享的 `lib/` 代码；
- **sandbox 不是这个**。sandbox 是一个隔离的、bash 风格的计算环境，文件系统根在 `/workspace`，是**模型自己的"草稿纸"**——用来跑模型生成的、不可信的命令和文件操作。

所以对 muse-agent 来说，**真正的交易能力根本不在 sandbox 里**——交易走的是连接（okx-trade），跑在受控的应用运行时。sandbox 仅仅是模型偶尔需要算点东西、写个临时文件时的隔离草稿空间。

> ⚠️ 注意/坑：千万别把"工具能碰真钱"和"sandbox"画等号。它们是两个世界：工具/连接在应用运行时（有凭据、有真实能力），sandbox 是模型的隔离玩具沙坑（不可信、被关起来）。理解这条区分，你才真正懂了 eve 的安全模型。

#### `networkPolicy: "deny-all"`：连 DNS 都禁

`defaultBackend(...)` 会按优先级自动选后端：部署在 Vercel 上用 Vercel Sandbox，本地有 Docker 用 Docker，都没有就退回到 just-bash（纯 JS 的本地兜底，没有真实二进制、也没有网络隔离）。

这里给 `vercel` 和 `docker` 两个后端都配了 **`networkPolicy: "deny-all"`**。第 10 章讲过，`deny-all` 的语义是——**禁止一切出站网络访问，连 DNS 都禁**。这极大降低了"不可信代码把私有数据外传出去（数据外泄）"的风险。

换句话说：就算模型在草稿纸上写了一段想偷偷往外发数据的代码，在支持该策略的后端上，它**根本连不上外网**。

> 💡 小贴士：为什么交易 agent 要把沙箱网络锁死？因为这台机器上流过的可能是账户、持仓、密钥相关的敏感信息。把模型的草稿空间断网，等于给"万一模型被诱导生成恶意代码"上了一道物理保险。这是纵深防御（defense in depth）的又一层。

> ⚠️ 注意/坑：`deny-all` 的效果**取决于后端是否真的强制执行它**。Vercel Sandbox、microsandbox、Docker 这类有真实隔离的后端能落实；而本地兜底的 just-bash **没有网络隔离**，这条策略对它就不是真正的物理隔离。所以本地用 just-bash 跑只是为了开发方便，真正的安全保证在部署后的 Vercel Sandbox 上。

### 13.9 schedules/market_scan.ts —— 自动巡检，但天生没有交易权

```ts
// agent/schedules/market_scan.ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 * * * *",
  markdown: "……（每小时扫一遍市场、记录观察的提示词）",
});
```

#### schedule 是什么

第 9 章讲过：**定时任务（schedule）让 agent 按自己的时钟启动**——不用人来发消息，到点自动跑。这里 `cron: "0 * * * *"` 是标准的 5 段 cron 表达式，意思是**每小时的第 0 分钟跑一次**（在 Vercel 上按 **UTC** 计算）。

#### "task mode"（任务模式）与它天生的安全性质

这个 schedule 用的是 **`markdown` 字段**（而不是 `run` 处理器）。第 9 章讲过，`markdown` 形式是**任务模式（task mode）**：eve 拿这段提示词把 agent 跑一遍，然后**把输出丢掉**。agent 在跑的过程中**仍然可以调用工具、写后端、打日志**。

但任务模式有一个**关键的、天生的限制**：

> **任务模式的会话要么跑完、要么失败，它无法"挂起"去等一个人、也无法去等一次 OAuth 登录。**

把这条限制和前面 13.5 的 `approval: always()` 拼在一起，一个漂亮的结论就出来了：

- 交易连接 `okx-trade` 的每一个工具都是 `approval: always()`——一调用就要**挂起等人批准**；
- 但任务模式**根本不能挂起去等人**；
- 所以这个每小时跑的 `market_scan` schedule，**在结构上就不可能下任何一笔单**——它一碰交易工具就需要挂起（park），而 task mode 不允许挂起，于是这条路天生走不通。

换句话说：**这个定时巡检任务，是"只读"的，而且这个"只读"不是靠提示词约束出来的，是靠机制保证的**。它每小时扫一遍市场、记录观察，但**没有交易权**——这是设计层面的硬保证。

> ⚠️ 注意/坑：要把这个性质说精确。task mode 不是"什么都不能写"——按 eve 文档，它**能**调工具、能写后端、能打日志。它**唯一不能做**的是"挂起去等人工审批 / 等 OAuth 登录"。market_scan 之所以没有交易权，恰恰是因为交易工具被 `approval: always()` 强制要求挂起，而 task mode 不能挂起——两个机制叠加，才得出"巡检无交易权"。别简化成"定时任务一概不能写"。

> 💡 小贴士：开发时怎么测这个定时任务？第 9 章讲过，**`eve dev` 永远不会按 cron 节奏自动触发 schedule**。本地想立刻跑一次，用开发派发路由手动触发即可：
> ```bash
> curl -X POST http://localhost:3000/eve/v1/dev/schedules/market_scan
> ```
> 部署到 Vercel 后，每个 schedule 会变成一个按 UTC 计算的 Vercel Cron Job。

### 13.10 evals —— 一次"冒烟测试"

```ts
// evals/evals.config.ts
import { defineEvalConfig } from "eve/evals";
export default defineEvalConfig({});
```

```ts
// evals/smoke.eval.ts
import { defineEval } from "eve/evals";

export default defineEval({
  description: "……",
  async test(t) {
    await t.send("……");
    t.completed();
  },
});
```

第 11 章讲过，**eval 是一个会打分的检查**：把 agent 跑真实会话，再给结果判分。它跑的是和真实用户一样的 HTTP 接口。注意 `evals/` 目录是 `agent/` 的**同级兄弟目录**，不在 `agent/` 里面。

muse-agent 的 eval 配置非常朴素：

- **`evals/evals.config.ts` 是 `defineEvalConfig({})`**——空配置。第 11 章讲过，`evals/` 根目录下**必须恰好有一个** `evals.config.ts`。这里什么都没配，因为这个项目的测试是**完全确定性的（deterministic）**，不需要 LLM 当裁判（judge）——而**确定性的 eval 树不需要配 judge 模型**。

- **`evals/smoke.eval.ts` 是一个"冒烟测试"**：发一句话（`t.send(...)`），然后断言会话正常跑完（`t.completed()`）。

什么叫"冒烟测试"？就是最基础的"它还活着吗"检查——不深究业务对错，只确认"发一条消息，agent 能正常完成一轮、不崩"。对交易 agent 来说，这是 CI 里的第一道防线：每次改动后跑一下 `eve eval`，确认最基本的链路没被改坏。

```bash
npm run eval
# 等价于
eve eval
```

它会对着本地开发服务器把这个冒烟测试跑一遍。

> 💡 小贴士：`t.completed()` 这类**运行级断言默认是"硬门禁（gate）"**——一旦没通过，这条 eval 就标记为 `failed`，`eve eval` 的退出码非 0，CI 就会红。这正是你想要的：链路一断，CI 立刻拦下。
>
> 反过来，像 `t.judge.*` 这种用 LLM 判分的断言**默认是"软"的**，而且**需要配置 judge 模型**；muse-agent 的确定性树用不到它们，所以 `defineEvalConfig({})` 空着也没问题。

### 13.11 把它串起来：分层的代码级安全模型

现在最重要的一步来了。把前面那些文件**抽象成一张"防线图"**，你就看懂了一个生产级交易 agent 到底是怎么把"能力"和"安全"分层组合的。

muse-agent 的安全，不靠任何单点开关，而是**多层独立的、代码级的防线叠加**。我们聚焦三道在代码里看得见、可验证的核心防线：

| 层 | 在哪个文件 | 机制 | 它挡住了什么 |
|---|---|---|---|
| ① 读/写连接分离 | `okx-read.ts` 与 `okx-trade.ts` | 把只读和交易拆成两个独立连接，从而能各自施加不同策略 | "读"畅通无阻不打断，"写"被单独圈出来上锁——两类能力不再混在一处 |
| ② 连接层审批 | `okx-trade.ts` | `approval: always()`，每一笔写操作都**持久挂起等人批准** | 任何一笔下单都不能由 agent 自己拍板，必须真人点批准 |
| ③ 通道层鉴权 | `channels/eve.ts` | 运维者多段有序鉴权，走到尽头 **fail closed（401）** | 没有合法身份的请求根本进不了门 |

逐层再说一遍它们各自独立兜底的逻辑：

**① 读/写连接分离，让"写"能被单独上锁。** 还记得 13.5 吗？muse-agent 不把读和写塞进同一个连接，而是拆成 `okx-read` 和 `okx-trade` 两个文件。这一步本身不直接"挡住"什么，但它是后面一切的前提：只有把"写"圈成一个独立连接，你才能对整条写通道统一施加一个策略（下一层的 `approval`），而不影响只读查询的顺畅。能力按风险被切开，是分层安全的第一刀。

**② 连接层 `okx-trade` 用 `approval: always()` 给每笔写操作装"急停按钮"。** 被圈出来的交易连接，被 `approval: always()` 整条罩住——每一次调用都停下、持久挂起、等一个人类批准才继续。这是"人在回路（human-in-the-loop）"的硬控制，也是整个项目最核心的真金白银安全阀。

**③ 通道层运维者鉴权 fail closed。** 在更外层，连"谁能让这个 agent 开始干活"都被锁住了：多段有序鉴权，匹配不上就 401。攻击者连门都进不来，自然谈不上触发后面两层。

这几层的精妙之处在于**它们彼此独立**：

- 即使有人绕过了通道鉴权（第③层失效），他想下单也得过审批（第②层）；
- 即使审批被误点放行了某一笔（第②层失效），也只波及被明确圈进交易连接的那些工具，只读查询通道完全不受影响（第①层把能力切开了，爆炸半径被限制住）；
- 它们不共享同一个"开关"，所以不会被一处疏漏一锅端。

这就是**纵深防御（defense in depth）**——也是这一章最想让你带走的东西。

> 💡 小贴士：除了这三层，前面 13.8 的 sandbox `deny-all` 其实是又一道独立防线——它把模型的"草稿纸"断网，堵住数据外泄。把它算上，muse-agent 至少有四道彼此独立的代码级防线。重点不是"几道"，而是这个习惯：**每加一项能力，都顺手想清楚它对应的那道防线该加在哪一层。**

> 💡 小贴士：回头看，你会发现这些防线**全部是用前面 12 章学过的、eve 原生的零件拼出来的**：连接（第 7 章）、连接上的 `approval`（第 7 章）、通道与路由鉴权（第 8 章）、sandbox 网络策略（第 10 章）。eve 没有发明一个叫"安全模块"的东西——**安全就是把这些零件按职责分层组合的结果**。这正是 eve "文件即接口"理念的威力：每个文件只管一件事，而整机的安全性，从这些各司其职的文件里自然浮现出来。

### 13.12 本章小结

我们逐文件走完了一个真实的生产 agent：

- **`package.json`**：Node>=24，五个脚本映射到 `eve dev/build/start/info/eval`，三件套依赖。
- **`agent.ts`**：只配 `model: "anthropic/claude-opus-4.8"`；能力不写在这里，来自文件位置。
- **`instructions.md`**：常驻人格，分能力/操作规程/边界三块，主动提到 `connection_search`。
- **`connections/okx-read.ts` 与 `okx-trade.ts`**：按读/写拆成两个连接，区别只在交易连接多了 `approval: always()`。
- **为什么要桥**：eve 只连 HTTP/SSE，stdio 的 MCP 服务器必须经一座桥（如 supergateway）转成 HTTP。
- **`channels/eve.ts`**：多段有序鉴权，fail closed 到 401，故意不用 `placeholderAuth()`。
- **`sandbox/sandbox.ts`**：模型的隔离草稿纸，`deny-all` 锁死出站网络——它不是交易能力所在地。
- **`schedules/market_scan.ts`**：每小时 task mode 巡检，因"不能挂起等审批"而天生没有交易权。
- **`evals`**：一条确定性冒烟测试，空 `defineEvalConfig({})`，不需要 judge。

而把这些串起来的那条主线是：**一个生产级 agent，是把"能力"和"安全"分层组合的产物。** muse-agent 的分层代码级安全模型——①把读/写拆成两个连接、②交易连接每笔 `approval: always()` 停下等人批、③通道层运维者鉴权 fail closed（再加上 sandbox 断网这道额外防线）——就是这种分层思维最好的范本。

读懂了它，你就不只是会写一个能跑的 agent，而是开始具备**把一个 agent 安全地放进生产**的判断力了。

---

## 14. 常见问题、排错与下一步

写到这里，你已经从零搭起了一个 eve agent：定义了 tool、写了 instructions、加了 connection、配了 channel auth、跑了 eval，甚至可能已经部署上了 Vercel。但真实世界里，第一次跑总会卡在某个地方——可能是 `eve dev` 起不来，可能是浏览器一直被拒之门外（401），也可能是你写好的文件 eve "看不见"。

这一章不是新功能，而是一份**急救手册**。我把新手最容易踩的坑、对应的症状、以及"怎么查"按主题列出来。最后给一条循序渐进的学习路径，告诉你"接下来该往哪走"。

> 💡 小贴士：排错的第一原则是**先看清楚 eve 到底认为发生了什么**，而不是凭感觉猜。本章会反复用到两个"显微镜"：终端里 `eve dev` 的 Agent Runs 视图，和 `eve info`（看 eve 发现/编译了什么）。遇到任何"我以为应该……但它没……"，先把这两个打开。

---

### 14.1 环境问题：Node 版本不够（必须 >= 24）

eve 对 Node 版本有硬性要求：**Node.js >= 24**（npm 随之自带）。这是新手最常见的第一个坑——你装了 eve，敲 `npx eve@latest init`，结果一堆莫名其妙的报错，或者 `eve dev` 直接崩。很多时候根因就是 Node 太旧。

先确认你当前的 Node 版本：

```bash
node -v
```

如果输出的是 `v18.x`、`v20.x`、`v22.x`，那就是不够。eve 要求 `v24` 或更高。

> ⚠️ 注意：不要把 eve 的 Node 要求和"普通 Vercel 部署"的要求搞混。普通 Vercel 项目部署只需要 Node 18+，但 **eve 框架本身要求 Node 24+**。这两个数字不一样，以 eve 的 24+ 为准。

如果版本不够，推荐用版本管理器（比如 nvm）切换：

```bash
nvm install 24
nvm use 24
```

切完再 `node -v` 确认一下，然后重新跑 `npx eve@latest init`。

> 💡 小贴士：真实示例项目 `muse-agent` 的 `package.json` 里写了 `"engines": { "node": ">=24" }`。你自己的项目里保留这一行是个好习惯——它能在 CI 和团队协作时提早暴露"有人用了旧 Node"的问题。

---

### 14.2 连接文件名要 dash-case，url 在编译期被急切校验

连接（connection）是新手第二个高频踩坑点，而且这两个坑都是**在编译期（发现/编译阶段）就失败（fail fast）**，不会等到运行时才暴露——这其实是好事，因为它逼你早点改对。要注意：这类错误用 `tsc --noEmit` 是查不出来的（类型上完全合法），得靠 `eve info` 的发现阶段才会暴露。

#### 坑一：文件名必须是 dash-case

连接文件放在 `agent/connections/` 下，**文件名就是连接的运行时名字**（没有 `name` 字段）。在实战中验证过的一个硬规则是：**连接文件名要用 dash-case（短横线连接）**——只允许小写 ASCII 字母、数字和短横线，且以字母开头。例如：

```text
agent/connections/okx-read.ts      ✅
agent/connections/okx-trade.ts     ✅
agent/connections/okx_read.ts      ❌（下划线会被拒绝）
```

如果你的连接一直没被正确识别，先检查文件名是不是用了下划线或驼峰。（注意：tool 和 schedule 的文件名是允许下划线的，这条 dash-case 规则**只针对连接**。）

#### 坑二：url 在编译期被急切（eager）校验

eve 在**发现 / 编译阶段就会急切校验连接的 url**——如果 url 写错了、是空的、或者编译时无法解析，它会**当场失败（fail fast）**，而不是等你真正去调用连接时才报错。

```ts
// agent/connections/okx-read.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  url: process.env.OKX_READ_MCP_URL ?? "http://127.0.0.1:8000/mcp",
  description: "OKX 行情只读接口：查询价格、深度、持仓等。",
  auth: { getToken: async () => ({ token: process.env.OKX_BRIDGE_TOKEN }) },
});
```

注意上面这个真实写法的精髓：它用了 `?? "http://127.0.0.1:8000/mcp"` 给了一个**兜底默认值**。这样即使 `process.env.OKX_READ_MCP_URL` 在编译时还没设上，url 也不会是 `undefined`/空字符串，急切校验就能过。而 `auth.getToken` 回调是**惰性**的（运行时才执行），所以那里只读环境变量是没问题的。

> ⚠️ 注意：如果你的 url 完全依赖一个编译期还不存在的环境变量（比如 `url: process.env.SOME_URL!`，而 `.env` 里压根没有它），编译会直接失败。排查方法：
> 1. 确认连接文件名是 dash-case；
> 2. 确认 url 在**编译时**就能解析成一个合法的字符串——要么写死，要么给 `?? "默认值"` 兜底，要么确保对应的环境变量已在 `.env` / `.env.local` 里（`eve` 启动时会先加载这两个文件）；
> 3. 记住 MCP 连接的 url **必须是 Streamable HTTP 或 SSE 地址**（见 14.5）。

---

### 14.3 channel 一直返回 401：因为 eve "fails closed"

你部署上去之后，从浏览器或脚本访问，**一直收到 401**——这几乎总是 eve 的设计本意，而不是 bug。

eve 的路由认证（route auth）原则是 **fails closed（默认拒绝）**。官方原文：*"eve fails closed by default: production browser traffic is rejected unless you configure an authenticator that accepts it, and anonymous access requires an explicit `none()`."* 翻译过来就是：**生产环境的浏览器流量默认被拒，除非你显式配了一个接受它的认证器；想允许匿名访问必须显式写 `none()`。**

要理解 401，先理解 `auth` 数组是怎么走的。`eveChannel({ auth: [...] })` 接收一个**有序数组**，eve 按顺序逐个走（walk）：

- 某个认证器返回了一个有效身份 → **接受并停止**（第一个匹配的赢）；
- 返回 `null` / `undefined` → **跳到下一个**；
- **抛异常** → 用具体状态码**拒绝**。

如果**所有认证器都跳过了 → 最终返回 401**。而且 **`auth: []`（空数组）会拒绝一切**——这就是 fails closed。

下面是真实示例项目 `muse-agent` 的 channel，可以照着对照检查：

```ts
// agent/channels/eve.ts
import { eveChannel } from "eve/channels/eve";
import { localDev, vercelOidc, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [localDev(), vercelOidc(), httpBasic({ username, password })],
});
```

这三个认证器的含义：
- `localDev()`：只接受发往 **loopback 主机名**（`localhost`、`*.localhost`、`127.0.0.0/8`、`::1`）的请求——这就是你本地 `eve dev` 能直接用、但线上浏览器不行的原因。
- `vercelOidc()`：接受 Vercel 签发的部署 OIDC token（内部 / runtime 调用方零配置可用）。
- `httpBasic({ username, password })`：运营 / 服务方用的共享用户名密码。

**排查 401 的清单：**

1. **你到底是谁？** 如果你是从公网浏览器手动访问一个部署，上面这套 `auth` 里**没有任何一个认证器接受普通匿名浏览器用户**——所以 401 是对的。你需要加一个能接受你这类调用方的认证器（比如 `httpBasic(...)`，并带上正确的凭证），或者在确实想公开时显式加 `none()` 作为**最后一项**。
2. **凭证对不对？** 如果你用的是 `httpBasic`，检查请求里的用户名密码、以及部署环境里的 `ROUTE_AUTH_BASIC_PASSWORD` 等环境变量是否配好。路由认证用到的密钥都在**环境变量**里，不会进编译产物。
3. **顺序对不对？** 把你自己的认证器放在 `localDev()` / `vercelOidc()` 这类 catch-all 之前。
4. **是不是 `placeholderAuth()` 在拦你？** `eve init` 脚手架默认生成的 `agent/channels/eve.ts` 里有 `placeholderAuth()`，它在生产环境会返回一个结构化的 401，提示"认证还没配好"。这是占位用的，**上生产前必须替换掉**（官方原话：*"Replace `placeholderAuth()` before production."*）。顺带一提，如果你直接**删掉**这个 channel 文件，eve 会回退到默认的 `[localDev(), vercelOidc()]`——这同样不会放行生产环境的浏览器用户。
5. **健康检查例外。** `GET /eve/v1/health` 永远是公开的，会跳过整个认证流程——所以如果你只是想验证"服务活着没有"，打这个端点，它不该返 401。

> 💡 小贴士：401 表示"我不知道你是谁（没通过认证）"，403 表示"我知道你是谁但你没权限"。在自定义认证器里，`return null` 是"跳过、交给下一个"，而 `throw new UnauthenticatedError(...)`（401）/ `throw new ForbiddenError(...)`（403）是"当场拒绝、不再往下走"。别把"跳过"和"拒绝"搞混。

---

### 14.4 schedules 本地不会自动触发：用 dev dispatch 路由手动触发

你写了一个 `agent/schedules/market_scan.ts`，cron 设的是 `0 * * * *`（每小时），结果在本地 `eve dev` 里**死等也不触发**——这不是 bug。

**`eve dev` 永远不会按 cron 节奏触发 schedules。** 这是设计：本地开发时你不希望一个每小时的 cron 在你调试时自己跑起来。

那本地怎么测？用 **dev dispatch 路由**手动触发一次：

```bash
curl -X POST http://localhost:3000/eve/v1/dev/schedules/market_scan
# -> { "scheduleId": "market_scan", "sessionIds": ["..."] }
```

- 路径里的 `:scheduleId` 就是**文件路径推导出来的名字**（去掉扩展名）。如果是嵌套的（比如 `agent/schedules/billing/sweep.ts`，名字是 `billing/sweep`），要把 `/` 做 URL 编码。
- 它会用和生产 cron 完全相同的派发路径，**带外（out of band）触发恰好一次**，并返回启动的 session id。
- 拿到 `sessionId` 后可以订阅它的事件流：`GET /eve/v1/session/:sessionId/stream`。

几个会帮你排错的返回码：

- 名字写错（未知 id）→ `404`，并附带 `availableScheduleIds`（直接告诉你有哪些可用的名字）。
- 缺 id / id 为空 → `400 { "error": "Missing schedule id." }`。

> ⚠️ 注意：dev dispatch 路由是 **`eve dev` 专用**的，生产里根本不挂载（也因此它本地不需要 auth）。
>
> 另一个相关的生产坑：部署到 Vercel 时，每个 `defineSchedule(...)` 会变成一个 **Vercel Cron Job，按 UTC 时间求值**（`"0 9 * * 1-5"` 是 UTC 09:00 触发）。如果你不是部署在 Vercel，而是用 `eve build && eve start` 自托管，`eve start` 会启动 Nitro 的定时任务 runner、schedules 能正常跑；但**如果你把产物塞进某个只服务 HTTP、不启动 Nitro 定时 runner 的进程管理器 / 容器**，schedules 能编译却不会触发——这时要么用 `eve start`，要么用一个支持 Nitro 定时任务的宿主，要么用你自己的调度器去打一个受认证保护的路由。

> 💡 小贴士：还记得 `market_scan` 用的是 `markdown`（task 模式）吗？task 模式会运行 agent 然后**丢弃输出**，agent 仍可调用工具、写后端、记日志，但**不能为了等人工审批或 OAuth 登录而暂停（park）**。所以一个 task 模式的定时任务，没法去用那些需要审批才能跑的工具/连接——这正是 `muse-agent` 的定时扫描"天生没有下单权限"的原因（下单连接是 `approval: always()`，调用时会 park 等审批，而 task 模式无法 park）。

---

### 14.5 stdio MCP server 连不上：要加一个 HTTP/SSE bridge

你在本地有一个能跑的 MCP server，但它是 **stdio（标准输入输出）** 类型的，配进 eve 连接后死活连不上——这是因为 **eve 的 MCP 连接只支持 Streamable HTTP 或 SSE 这两种传输方式，不直接支持 stdio。**

MCP 连接的 `url` 字段"必须能说 Streamable HTTP 或 SSE"。一个纯 stdio 的 MCP server 没有 url 可言，自然接不进来。

解决办法：**在 stdio server 前面架一个 HTTP/SSE 桥（bridge / 代理），把 stdio 转成 HTTP**，然后让 eve 连这个桥的 HTTP 地址。

真实示例 `muse-agent` 就是这么干的：它的 `okx-trade-mcp` 是个纯 stdio 的 MCP server，于是用 **supergateway** 把 stdio 转成 HTTP，再让 eve 连接桥出来的 `http://...` 地址。

可以这样概括这条规则：**stdio MCP server 不能直接连 eve，你需要一个 HTTP/SSE 桥。** 桥起好之后，连接文件就是普通的 HTTP url：

```ts
// agent/connections/some-stdio-server.ts
import { defineMcpClientConnection } from "eve/connections";

export default defineMcpClientConnection({
  // 指向桥暴露出来的 HTTP/SSE 地址，而不是 stdio 进程本身
  url: "http://127.0.0.1:8000/mcp",
  description: "通过 HTTP/SSE 桥接入的某 MCP server。",
});
```

> ⚠️ 注意：eve 文档**并没有自带或指定**某个具体的桥工具。`muse-agent` 用的是 supergateway；社区里还有别的桥（如 `mcp-proxy`、`mcp-bridge`、`mcp-remote` 等），但这些**不是 eve 官方文档里的东西**，选型与配置以各自项目文档为准。关键点只有一个：eve 那一端拿到的必须是 HTTP/SSE url。

---

### 14.6 模型调用失败：先查 AI Gateway 配置 / 凭证

agent 启动了，但一发消息就在调模型这一步失败——绝大多数情况是**模型凭证 / AI Gateway 没配好**。

理解一下模型字符串是怎么被解析的。当你在 `agent.ts` 里写：

```ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-opus-4.8",
});
```

这个 `"<provider>/<model>"` 形式的字符串会**经过 AI Gateway 路由**。它怎么认证，取决于你在哪运行：

- **在 Vercel 上**：通过 **OIDC** 解析，**不需要任何 provider 密钥**——前提是项目已经 link 过（`eve link` 会顺便取回 AI Gateway 凭证）。
- **在本地 / 非 Vercel 环境**：需要设 **`AI_GATEWAY_API_KEY`**（gateway 密钥），或者一个 `VERCEL_OIDC_TOKEN`（`vercel link` 之后会有）。

**排查模型调用失败的清单：**

1. **本地是不是缺 gateway 密钥？** 在本地跑时，确认环境里有 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN`。eve 启动时会先加载 `.env` / `.env.local`，把密钥放进去：
   ```bash
   export AI_GATEWAY_API_KEY="你的_gateway_key"
   ```
2. **项目 link 过吗？** 跑一下 `eve link`，它会把目录关联到 Vercel 项目并取回 AI Gateway 凭证。
3. **模型字符串拼对了吗？** 形式必须是 `<provider>/<model>`，比如 `anthropic/claude-opus-4.8`、`openai/gpt-5.4-mini`。有效的 gateway 模型 id 列表并没有官方逐一枚举，**以官方文档 / AI Gateway 的模型页为准**——别凭记忆硬写一个不存在的型号。
4. **想绕过 gateway？** 如果你改用了直连 provider 的 `LanguageModel` 形式（比如 `anthropic("claude-opus-4.8")`），那就不走 gateway 了，这时需要的是 **provider 自己的密钥**（如 `ANTHROPIC_API_KEY`）和对应的 provider 包，而不是 `AI_GATEWAY_API_KEY`。

> 💡 小贴士：在 Vercel 上，`VERCEL_OIDC_TOKEN` 是自动可用的（"没有密钥要轮换"）。一个常见的兜底写法是 `process.env.AI_GATEWAY_API_KEY || process.env.VERCEL_OIDC_TOKEN`——本地用 key，线上用 OIDC。
>
> 另外注意：eval 里如果用到了 `t.judge.*`（LLM 评审），它**也要一个 judge 模型 + 模型凭证**。配了模型但没凭证时，judge 类的 eval 会**显式跳过**（不会让整套 eval 失败）；而纯确定性的 eval 树根本不需要 judge 模型。

---

### 14.7 "为什么我的文件没被发现？"——用 `eve info` 排查

eve 是**文件优先**的框架：**文件的名字和它在目录树里的位置，就是它的定义。** 你从不在 `define*` 调用里写 `name` 或 `id` 字段——身份完全由路径推导。这套机制很优雅，但也意味着**只要文件放错位置或起错名字，eve 就会当它不存在**，而且通常不报错——它只是"没看见"。

当你发现"我明明写了一个 tool / connection / skill / schedule，但 agent 用不了 / 列表里没有"，第一件事就是跑：

```bash
eve info
```

或者要机器可读的 JSON：

```bash
eve info --json
```

`eve info` 会列出 eve **实际解析到的**东西：生效的路由、tools、skills、subagents、schedules、channels、编译产物，以及诊断信息。把你期望看到的名字和它实际列出的对照——**没出现就是没被发现**。

发现不了的常见原因，对照路径推导规则查：

- **位置不对。** tool 必须在 `agent/tools/`，connection 在 `agent/connections/`，skill 在 `agent/skills/`，schedule 在 `agent/schedules/`，channel 在 `agent/channels/`。放错目录就不会被当成对应能力。
- **名字推导出来不是你以为的那个。** `agent/tools/get_weather.ts` → tool 名 `get_weather`；`agent/connections/linear.ts` → 连接名 `linear`；`agent/skills/summarize.md` → skill 名 `summarize`；`agent/subagents/researcher/agent.ts` → subagent 名 `researcher`。
- **命名约定踩坑。** tool 的文件名 slug 会被规范成 **snake_case**；而**连接文件名要 dash-case**（见 14.2）。两类不一样，别混用。
- **某些目录是 root-only。** `channels/`、`schedules/`、`instrumentation.ts` 只能在根 agent 下，subagent 里放这些不会生效。
- **互斥/重复导致编译失败。** 比如根目录同时存在 `instructions.md` 和 `instructions.ts` 是构建错误（不会静默二选一）。这类问题 `eve info` 的诊断信息或编译报错会提示。

> 💡 小贴士：`.eve/` 目录放的是编译产物（manifest 等）。当你怀疑"是不是缓存了旧状态"，可以重启 `eve dev` 让它重新 discover → compile。但绝大多数"文件没被发现"的根因还是**命名 / 位置**，`eve info` 几乎总能一眼定位。

> 💡 小贴士：eve 在 discovery 阶段**绝不会真的执行**你写的 tool——它先只看描述符（descriptor）。所以 `eve info` 里列出某个 tool，只代表"它被发现并注册了"，不代表它的 `execute` 跑过。要验证真正的执行，得在 `eve dev` 里触发它、再去 Agent Runs 里看调用详情。

---

### 14.8 一张速查表

把上面的坑浓缩成一张"症状 → 大概率原因 → 怎么查"的表，遇事先扫一眼：

| 症状 | 大概率原因 | 怎么查 / 怎么修 |
|---|---|---|
| `init` / `dev` 直接崩、奇怪报错 | Node 版本 < 24 | `node -v`；用 `nvm install 24 && nvm use 24` |
| 连接编译期就 fail fast | 文件名不是 dash-case，或 url 编译时无法解析 | 文件名改 dash-case；url 写死或 `?? "默认值"` 兜底 |
| 浏览器 / 脚本一直 401 | route auth fails closed，没有认证器接受你 | 检查 `auth` 数组与顺序、凭证、是否还留着 `placeholderAuth()` |
| schedule 本地不触发 | `eve dev` 不按 cron 跑 | `curl -X POST .../eve/v1/dev/schedules/<name>` 手动触发 |
| stdio MCP server 连不上 | eve 只支持 HTTP/SSE，不支持 stdio | 加一个 HTTP/SSE 桥（如 supergateway），连桥的 url |
| 一调模型就失败 | 缺 AI Gateway 凭证 / 没 link 项目 | 设 `AI_GATEWAY_API_KEY` 或 `eve link`；核对模型字符串 |
| 写的文件 agent 用不了 | 命名 / 位置不符合路径推导规则 | `eve info`（或 `--json`）对照实际发现了什么 |

---

### 14.9 下一步：从"跑通"到"做深"

恭喜——能读到这里，你已经具备了独立搭一个 eve agent 并把它排错、部署上线的能力。但教程只能带你到"跑通"，真正的成长来自动手把它做深。下面是一条建议的进阶路径，从易到难：

1. **把 `get_weather` 换成一个真实 API。** 教程里的 `get_weather` 返回的是写死的"Sunny, 72F"。把它的 `execute` 改成真的去调一个天气 API（记住：tool 跑在你的**应用 runtime** 里，能读 `process.env`、能 import 共享的 `lib/` 代码，而不是跑在 sandbox 里）。这是从玩具到真实的第一步。返回前别忘了**过滤、最小化、脱敏**——不要把密钥或多余的敏感数据返回给模型。

2. **加一个 skill。** 把一段"可复用的多步流程 / 领域知识"从 instructions 里挪出来，做成 `agent/skills/` 下的一个 skill（按需通过 `load_skill` 加载，不占用每一轮的 prompt）。也可以用 skills CLI 直接装现成的：
   ```bash
   npx skills add vercel-labs/agent-skills
   ```
   （在 eve 项目里它会问你是否装进 `agent/skills/`，选 Yes。）

3. **写更多 eval。** 给你的 agent 多写几个 `*.eval.ts`，覆盖关键行为（用了正确的 tool、回复包含关键信息、没乱调工具）。把它接进 CI：
   ```bash
   eve eval --strict --junit .eve/junit.xml
   ```

4. **部署到 Vercel，去看 Agent Runs。** 部署后，进项目的 **Agent Runs** 视图（无需任何配置就自动出现）。看每次 run 的触发来源、token 用量、逐轮的 Timings / Tool Calls / Reasoning——这是你理解 agent 真实行为、做性能与成本优化的主战场。
   ```bash
   eve deploy
   ```

5. **读官方文档与源码。** 本教程基于 eve 当前的能力写成，但 eve 的很多深层 API 细节最权威的来源是官方文档和 GitHub 仓库（仓库里 `docs/` 下有大量参考页）。遇到本教程没覆盖、或你拿不准的 API，**以官方文档为准**。

#### 资源链接

- eve 文档（Vercel）：https://vercel.com/docs/eve
- eve 官网：https://eve.dev/
- eve GitHub 仓库：https://github.com/vercel/eve
- Vercel 入门：https://vercel.com/docs/getting-started-with-vercel

> ⚠️ 注意：**eve 目前处于 beta 阶段**。官方明确说明：框架、API、文档和行为**在正式发布（GA）前都可能变化**。本教程里的命令、字段名、模型字符串等都可能随版本调整——当你发现某处和实际行为对不上时，**永远以官方文档和你本地实际安装的 eve 版本为准**。把本章当成"排错的思路和地图"，而不是一成不变的标准答案。

---

到这里，《Vercel eve 框架·新手小白完整教程》就告一段落了。你已经走完了从"完全不懂 AI agent"到"能独立搭建、排错、部署一个 durable agent"的全程。剩下的，就交给你自己的项目去验证了——去把那个 `get_weather` 换成你真正想做的事吧。

---

## 附录 A · 本地与私有化（自托管）部署

正文前面的第 12 章只讲了"零配置发布到 Vercel"这一条最顺滑的路。但很多同学关心的是另一个问题：**能不能完全不依赖 Vercel，把这个 agent 跑在自己的机器、内网或服务器上？** 答案是肯定的——eve 的持久化内核（Workflow SDK）是开源的，而 Vercel 提供的那一整套（Functions、托管 Workflows、Sandbox、AI Gateway、Agent Runs 面板等）是"开箱即用的便利"：你可以选择性地用，而不是非用不可。这一节就把"脱离 Vercel 私有化运行"讲清楚。

> 💡 小贴士：这是一节进阶内容。如果你还在入门阶段，先按前面的章节把 agent 在本地 `eve dev` 跑通、再用 Vercel 体验一遍完整链路，回头再看这一节会轻松很多。

> ⚠️ 注意：eve 目前处于 beta（截至 2026-06）。本节涉及的命令、配置字段、HTTP 路由都可能随版本变化，凡是落地前请**以官方文档为准**。另外，官方明确说过："At launch eve deploys to Vercel, with support for other platforms on the way."——也就是说"一键部署"目前**原生面向 Vercel**，面向其它平台的适配器仍在开发中。今天的自托管能跑通，但还做不到"一条命令"那种顺滑。

### A.1 先分清：什么是「eve 内核」，什么是「Vercel 便利」

理解自托管，先要把两件事分开：

- **eve 内核**：发现文件、校验、编译成清单、把运行时作为一个"可部署的 app"提供出来，并在底层用开源的 Workflow SDK 让会话变得持久、可恢复、抗崩溃。官方原文是：eve "discovers those files, validates them, compiles a manifest, and serves the runtime as a deployable app."——这些是开源框架本身的能力。
- **Vercel 平台便利**：Functions（托管计算）、Workflows（托管持久化会话）、Sandbox（托管沙箱）、AI Gateway（模型路由）、Agent Runs（观测面板）——这些是 Vercel 提供的托管服务，让你"零配置"享受到生产级能力。

官方定价页说得很直白（注意用词是 **can use** 和 **or model providers**）：

> "An eve deployment **can use** Vercel Functions for compute, Vercel Workflows for durable sessions, Vercel Sandbox for isolated command execution, and AI Gateway **or model providers** for model calls."

也就是说：这些都是"可以用"，不是"必须用"。文档在描述运行位置时也总是加"On Vercel"前缀（如 "On Vercel, the compiled agent runs from Vercel Functions."）——说明 Vercel 只是运行目标之一。

下面这张表是你做私有化时的"替换清单"（细节随版本变化，以官方文档为准）：

| 能力 | Vercel 托管 | 自托管 / 私有化替代 |
|---|---|---|
| 运行/部署 | Vercel Functions | `eve build` + `eve start`，跑你自己的 Node 服务 |
| 模型访问 | AI Gateway（模型字符串 + OIDC） | 直连厂商 SDK（`@ai-sdk/*` + 厂商 API key） |
| 持久化 | 托管版 Workflows | 开源 Workflow SDK（需自己提供存储后端，详见 A.5） |
| 沙箱 | Vercel Sandbox | `docker` / `microsandbox` / `just-bash` 后端 |
| 鉴权 | Vercel OIDC | `httpBasic()` / 自定义 + 反向代理 |
| 可观测性 | Agent Runs 面板 | `instrumentation.ts` 接你自己的 OpenTelemetry |

> 💡 小贴士：表里的"凭证托管 / Connect"等更上层的便利在所查文档中未确证细节，这里不展开；如果你的项目用到，请以官方文档为准。

### A.2 私有化的三个层次

"私有化"不是一个开关，而是一个谱系，从轻到重：

1. **纯本地开发跑**：其实你在前面的章节已经做过了——`eve dev` 把 agent 跑在本地（官方示例端口为 `3000`，即 `http://127.0.0.1:3000`），并带一个交互式终端 UI。这本身就是 100% 本地、不碰 Vercel 的运行方式。
2. **本地/内网长期运行**：用 `eve build` 把 agent "compile the agent into .eve/ and build the host output"（编译进 `.eve/` 并生成 host 产物），再用 `eve start`（"serve the built output"）把这份产物作为服务跑起来。它就是个标准的 Node 服务，可以常驻在你自己的机器或内网里。
3. **自有服务器/容器部署**：把同一份 host 产物部署到你的 VPS、Docker 或内网集群。官方说 eve "designed from the ground up with adapters in mind"（从设计之初就考虑了适配器），并且 "the same directory runs in production exactly as it ran on your laptop"（同一个目录在生产环境的运行方式，和它在你笔记本上运行时完全一样）——所以从开发到自托管，你的项目目录基本不用改。

> 💡 小贴士：除了 `eve dev` / `eve build` / `eve start`，还有一个 `eve info` 命令可以查看当前项目信息。另外 `npx eve dev https://your-app.vercel.app` 可以把本地的交互式开发界面指向一个**已部署的实例**，方便调试。

### A.3 模型：从 AI Gateway 切换到直连厂商

在 Vercel 上，你在 `agent.ts` 里写的是模型字符串（如 `"anthropic/claude-opus-4.8"`、`"openai/gpt-5.4-mini"`），由 **AI Gateway** 负责路由；部署到 Vercel 后还能借助 Vercel OIDC，免去自己管理 key。

自托管时你有两条路：

- **继续用 AI Gateway**（即使不在 Vercel 上跑）：在环境里提供 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN` 即可。
- **直连模型厂商**：安装对应的 AI SDK provider 包（例如 Anthropic 用 `@ai-sdk/anthropic`），并在环境里提供该厂商自己的 key（例如 `ANTHROPIC_API_KEY`）。

> ⚠️ 注意：直连 provider 时，模型在 `agent.ts` 里到底怎么写（继续用模型字符串，还是改用 provider 实例），本节不给出确切代码——这部分在所查文档中未确证，且 beta 阶段最容易随版本变动。请**以官方 agent 配置文档为准**。

### A.4 沙箱：换成本地后端

前面讲过，沙箱是"模型自己的 bash／文件草稿空间"。它通过 `eve/sandbox` 的 `defineSandbox` + `defaultBackend` 来配置，支持多种后端：`vercel`、`docker`、`microsandbox`、`just-bash`；`defaultBackend` 的选用优先级是 **Vercel → Docker → microsandbox → just-bash**（按可用性自动挑选）。

自托管时把 `vercel` 这一档拿掉即可：

- **docker**：有真正的隔离，是自托管下的推荐选择（前提是宿主机装了 Docker）。
- **microsandbox**：另一种本地隔离方案。
- **just-bash**：本地兜底，**没有网络隔离、也没有真实二进制**，只适合最简单的本地调试。

别忘了安全建议：用 `networkPolicy: "deny-all"` 把沙箱出网锁死（在支持的后端上会被强制执行）。

```ts
// agent/sandbox/sandbox.ts —— 自托管下优先用 docker，并锁死出网
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    docker: { networkPolicy: "deny-all" },
  }),
});
```

> ⚠️ 注意：上面这段是示意性写法，用于展示 `defineSandbox` / `defaultBackend` / `networkPolicy` 这几个要素如何组合。`defaultBackend` 各后端的具体参数形态在 beta 期可能调整，落地前请**对照官方 sandbox 文档**核对字段。

### A.5 持久化：开源 Workflow SDK（这块最需要你认真对待）

eve 最有价值的"持久化"——会话能在冷启动、重新部署、长时间暂停后恢复——靠的是把进度作为一份**事件日志**持久化，并通过确定性重放来重建状态。

好消息是：官方明确说了，这套机制的内核是**开源的**：

> "Under the hood, eve uses the **open-source Workflow SDK** to make sessions durable, resumable, and crash-safe."

所以"持久化"本身并不绑定 Vercel。但有一个你必须想清楚的点：

> ⚠️ 重要：事件日志总得**存在某个地方**。在 Vercel 上，这由托管版 Workflows 替你搞定；自托管时，则由这个开源 Workflow SDK 负责，而它**需要一个存储后端**来落盘这份事件日志。但具体支持哪些存储、怎么配置，本节**不提供任何示例代码**——这一点在所查资料中没有确证。请**以官方 Workflow SDK 文档为准**，这是自托管里最该亲自验证、也最不该照搬第三方博客的一环。

### A.6 鉴权与对外暴露：别把门敞开

channel 鉴权在自托管下同样重要，但有一点不同。eve 的 `eve/channels/auth` 提供了几种现成方式：

- `localDev()`：放行本机／Vercel dev，适合本地调试。
- `vercelOidc()`：放行 Vercel 签发的部署令牌——**自托管下不适用**。
- `httpBasic({ username, password })`：HTTP Basic 认证，适合对外提供服务时用。
- `placeholderAuth()`：仅开发期开放，**别用它上生产**。

关键机制要记牢：`auth` 数组**按顺序匹配、首个命中者胜出；如果全部不匹配，则返回 401（fail closed，默认拒绝）**。所以自托管对外服务时，本机调试可以用 `localDev()`，对外则用 `httpBasic()` 或你自己的鉴权实现。

生产暴露时，把 eve 服务放在**反向代理（如 Nginx / Caddy）之后**，由代理负责 TLS（https）和一层额外的访问控制。

### A.7 可观测性：替代 Agent Runs 面板

在 Vercel 上你能直接得到 **Agent Runs** 面板（在 Vercel dashboard 里查看会话、轮次、工具调用等，且**无需** `instrumentation.ts`）。自托管没有这个面板，但你可以用 `agent/instrumentation.ts`（`defineInstrumentation`）把链路数据导出到**你自己的 OpenTelemetry 后端**（如 Jaeger、Grafana Tempo 等），自己搭一套观测。

`defineInstrumentation` 支持的字段包括 `setup`、`recordInputs`、`recordOutputs`、`functionId`、`events`。

> 💡 小贴士：只要 `agent/instrumentation.ts` 里存在这个默认导出，就会**隐式启用 OTel**，无需额外开关。具体字段含义与导出格式以官方文档为准。

### A.8 动手：把 agent 跑成一个常驻服务

最小的自托管启动流程（先确保 **Node.js >= 24**）：

```bash
npm install
npm run build      # 通常对应 eve build：编译进 .eve/ 并生成 host 产物
npm run start      # 通常对应 eve start：把产物作为服务跑起来
```

> 💡 小贴士：`npm run build` / `npm run start` 具体映射到哪条命令，取决于你项目 `package.json` 里的 `scripts` 配置；脚手架生成的项目通常已经把它们接到了 `eve build` / `eve start`。你也可以直接用 `eve build` 和 `eve start`。官方示例里服务监听在 `http://127.0.0.1:3000`。

跑起来后，用官方的 HTTP 路由做一次冒烟测试：

```bash
# 开一个会话（注意路由前缀是 /eve/v1/session）
curl -i -X POST http://127.0.0.1:3000/eve/v1/session \
  -H 'content-type: application/json' \
  -d '{"message":"你好，简单介绍一下你能做什么"}'
```

上面这个 POST 的**响应头里会带一个 `x-eve-session-id`**。拿到它，就可以用下面的"重连流"接口重新连上这个会话的事件流（把 `<sessionId>` 换成实际值）：

```bash
# 重连并继续接收该会话的事件流
curl http://127.0.0.1:3000/eve/v1/session/<sessionId>/stream
```

要常驻运行，把 `eve start`（或 `npm run start`）交给 `pm2`、`systemd` 或写进 Docker 镜像；再把它放到反向代理后面对外提供服务即可。

### A.9 现实提醒：今天自托管该知道的事

- eve 仍是 **beta（2026-06）**。官方的"一键部署"目前**原生面向 Vercel**，其它平台的适配器**还在路上**（"support for other platforms on the way"）。
- 所以今天的自托管，约等于：你自己用 `eve build` / `eve start` 把 host 产物部署好，自己接模型 key、自己选沙箱后端、自己给 Workflow SDK 配存储后端、自己做反向代理和鉴权。**能跑通，但不是"一条命令"那种顺滑**。
- **什么时候选 Vercel**：想最快上线，想要现成的持久化 + 沙箱 + 观测面板。
- **什么时候选自托管**：数据／合规要求必须留在自己手里，或你想完全掌控整套技术栈。
- 因为 eve 还在快速迭代，本节涉及的命令、字段、路由都可能变化，**一切以官方文档为准**：[概念](https://vercel.com/docs/eve/concepts)、[定价与所用资源](https://vercel.com/docs/eve/pricing)、[官方文档 eve.dev](https://eve.dev/docs)。
