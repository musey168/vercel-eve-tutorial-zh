# Vercel eve · 从零手把手构建一个完整 Agent

跟着这份教程，你会**从零开始、一行行自己写**，做出一个叫 `devhelper` 的完整 AI agent，并覆盖 eve **官方全部模块**。全程不抄现成项目，每个文件都由你亲手创建。

> eve 还处于 beta，API 可能随版本变化；拿不准时以 [官方文档](https://vercel.com/docs/eve) 为准。

**预备**：会用终端、看得懂一点 TypeScript、Node.js ≥ 24、一个 Vercel 账号（用于模型访问与部署）。

**你会一章加一块，建成 `devhelper`（覆盖官方全部模块）：**

| 章 | 你新建的文件 | 官方模块 |
|---|---|---|
| 2 | `agent/agent.ts`、`agent/instructions.md` | agent 配置 + 系统提示 |
| 3 | `agent/tools/get_weather.ts` | Tools |
| 4 | `agent/skills/weekly-report.md` | Skills |
| 5 | `agent/subagents/<id>/` | Subagents |
| 6 | `agent/connections/my-service.ts` | Connections（MCP / OpenAPI） |
| 7 | `agent/channels/web.ts` | Channels + 鉴权 |
| 8 | `agent/schedules/daily-brief.ts` | Schedules |
| 9 | `agent/hooks/` | Hooks |
| 10 | `agent/sandbox/sandbox.ts`、`evals/` | Sandbox + Evals |
| 11 | `agent/instrumentation.ts` | Instrumentation |

**目录**：1. 环境准备 · 2. 跑起第一个 agent · 3. Tools · 4. Skills · 5. Subagents · 6. Connections · 7. Channels · 8. Schedules · 9. Hooks · 10. Sandbox 与 Evals · 11. Instrumentation · 12. 部署到 Vercel · 13. 排错与下一步

---

## 1. 环境准备

```bash
node -v                 # 必须 ≥ 24；不够就：nvm install 24 && nvm use 24
npm i -g vercel         # Vercel CLI，用于模型访问凭证与部署
vercel login
```

eve 通过 Vercel **AI Gateway** 调模型，本地开发也要能连上。gateway 模型本地开发需要具备 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN`（任选其一）。拉取凭证有两种方式：eve 原生的 `eve link`（链接 Vercel 项目并拉取 AI Gateway 凭证），或 Vercel CLI 的 `vercel link`，二者都在 `init` 之后做即可。

就绪检查：`node -v`(≥24)、`vercel whoami` 能输出你的账号。


---

## 2. 跑起第一个 agent

```bash
npx eve@latest init devhelper   # 脚手架：装依赖、初始化 Git、启动 dev
cd devhelper
```

一个 agent 最少只要两个文件。

**`agent/agent.ts`** —— 模型与运行配置：

```ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-sonnet-4.6", // 经 AI Gateway 路由，可换成别的模型
});
```

**`agent/instructions.md`** —— 系统提示词（根 agent 必需，决定 agent 的身份、语气与规则）：

```md
你是 devhelper，一个简洁、务实的开发者助手。
- 回答直接，给可执行的步骤。
- 不确定就说不确定，绝不编造。
```

启动并对话：

```bash
npm run dev    # = eve dev，本地交互式终端 UI；默认监听 127.0.0.1:3000（端口取 $PORT，缺省 3000）
```

> 启动脚本可能随包管理器而异（如 `pnpm dev`），实际监听地址以 `eve dev` 输出的 URL 为准。

对它说「你好，你能帮我做什么？」，能正常回话就成功了。

> 关键命令 `eve info`：打印解析后的应用，列出 eve 发现的 tools、skills、subagents、schedules、channels、路由、产物路径以及发现诊断。以后「加了文件却不生效」时，第一时间跑它——多半是文件放错目录或命名不对。


---

## 3. Tools：给 agent 一双手

工具 = agent 能调用的**有类型函数**。eve 的特色是：**文件名就是工具名，不需要任何注册表**。

新建 **`agent/tools/get_weather.ts`**：

```ts
import { defineTool } from "eve/tools";
import { z } from "zod";

export default defineTool({
  description: "查询某个城市的当前天气",
  inputSchema: z.object({ city: z.string().min(1) }), // 用 zod 定义并校验入参
  async execute({ city }) {
    // 先返回假数据跑通流程；真实项目里换成调用天气 API
    return { city, condition: "晴", temperatureC: 24 };
  },
});
```

保存后回到对话问「北京天气怎么样」。模型会自动选用 `get_weather`、入参被 zod 校验、`execute` 的返回值回到模型再组织成回答。

> 工具跑在你的**应用 runtime**（能读 `process.env`、import 共享代码），不是在沙箱里。返回前只给模型需要的字段，别把密钥或多余数据回传给模型。
>
> 有副作用/高风险的操作可以用工具自身的 `needsApproval` 字段要求**人工审批**后才执行：`import { always } from "eve/tools/approval"; needsApproval: always(),`（也可用 `once()` / `never()` / 谓词；省略时默认等同 `never()`）。注意 eve 会重放已完成步骤的记录结果而不重跑，但中途被打断的步骤会重新执行，所以扣款、发邮件这类非幂等副作用要么做成幂等，要么用审批门控。（连接层另有一个独立的 `approval` 字段，见第 5 章，那是给连接工具用的，与这里的 `needsApproval` 不是同一个东西。）


---

## 4. Skills：把可复用的流程沉淀下来

当某段「多步流程/规程」很长，别全塞进 `instructions.md`（那是始终在线的系统提示，每一轮、每次模型调用都会被前置，白白占上下文）。把它做成 **skill**，模型只在相关时按需加载。

新建 **`agent/skills/weekly-report.md`**（文件名即 skill 名 `weekly-report`）：

```md
# 生成周报

当用户要「周报」时，按下面做：
1. 收集本周完成的要点（让用户给出，或从对话里提取）。
2. 整理成三段：进展 / 风险 / 下周计划。
3. 用要点列表，控制在 200 字以内。
```

> 这是「扁平 markdown」写法。没写 frontmatter `description` 时，eve 会取正文第一行（剥掉开头的 `#`、`>`、`*`、`-` 标记），本例即「生成周报」，作为给模型的**路由提示**——它决定模型何时加载这个 skill，所以这一行要写得能被准确匹配。

之后用户说「帮我写周报」：eve 把每个 skill 的描述连同一个框架自带的 **`load_skill`** 工具一起暴露给模型；模型判断相关时调用 `load_skill`，eve 才把这段 markdown 追加进当前回合的上下文，再照步骤执行。

> 一句话区分：**Tools 是能执行的函数；Skills 是给模型读的步骤性知识——加载一个 skill 只是往上下文里加指令，永远不会新增一个可执行面。**


---

## 5. Subagents：把子任务交给专项子 agent

子 agent（subagent）是模型可以委派一个聚焦子任务的"另一个 agent"。它独立运行——有自己全新的对话历史和状态，不与父 agent 共享上下文。三个典型用途：并行处理工作、给孩子收窄一套更小的工具、或者给某个专家一个独立身份。

> 💡 它和 skill 有本质区别：skill 是给**当前** agent 临时追加一段指令，不开新身份；subagent 是**另起一个 agent**，有独立的对话历史和状态。需要"换个脑子重新干"时用 subagent，需要"给当前脑子加条规矩"时用 skill。

eve 提供两种子 agent。

### 一、内置 `agent` 委派工具（委派给当前 agent 的副本）

模型可以直接调用内置的 `agent` 工具，把任务交给**当前 agent 的一个副本**。这个副本：

- **共享**父 agent 的 sandbox 和工具——孩子写的文件，父亲立刻能看到；
- **继承**父 agent 的 auth、connections、instructions、hooks；
- 但**对话历史和 state 是全新的**——孩子看不到父亲的历史。

调用时只需要给两个字段：

```ts
{
  message: string;        // 孩子需要的一切都写在这里，它看不到父亲的历史
  outputSchema?: object;  // 设了它，孩子就以 task 模式运行，返回结构化输出
}
```

不设 `outputSchema` 时，孩子用普通散文回话；设了它，孩子进入 task 模式，把结构化结果作为工具结果返回。这种方式零配置，适合"临时分身去并行干一段活"。

### 二、声明式 subagent（放在 `agent/subagents/<id>/`）

当你想要一个**有独立配置、独立工具、独立身份**的专家时，就声明一个子 agent。它住在 `agent/subagents/<id>/` 下，本质是一个独立的 agent root——**不从父 agent 继承任何东西**，缺失的槽位用框架默认值补齐。

> ⚠️ 与副本相反：内置 `agent` 副本继承一切、共享 sandbox；声明式 subagent 继承为零、各管各的。共享的过程要么写进它自己的 `skills/`，要么放进 `lib/` 当 helper。

下面从零建一个叫 `researcher` 的子 agent。最少只要一个 `agent.ts`：

```ts
// agent/subagents/researcher/agent.ts
import { defineAgent } from "eve";

export default defineAgent({
  description: "Investigate ambiguous questions before the parent agent responds.",
  model: "anthropic/claude-opus-4.8",
});
```

两条硬规则：

- `agent.ts` 是**必需**的，而且**必须写 `description`**——父 agent 靠它决定要不要委派；漏写 `description`，编译器直接拒绝。
- 从 `"eve"` 导入 `defineAgent`（不是 `"eve/agent"`），和根 agent 用的是同一个。

子 agent 的名字来自目录名，**没有前缀**：`agent/subagents/researcher/` 就暴露成名为 `researcher` 的委派工具。

需要的话，这个目录可以像一个完整 agent 一样长出更多槽位：

```text
agent/subagents/researcher/
├── agent.ts            # 必需，必须声明 description
├── instructions.md     # 可选（或 instructions.ts）
├── connections/
├── hooks/
├── skills/
├── lib/
├── sandbox/
├── tools/
└── subagents/          # 子 agent 可以再嵌套子 agent
```

> ⚠️ 子 agent **不能**用 `channels/` 和 `schedules/`——这两个是 root-only 的，只能放在根 agent 下。声明式 subagent 里建这两个目录不会被识别。

### 怎么看效果

跑 `eve info`，输出里会列出被发现的 subagents（以及 tools、schedules、channels 等），确认 `researcher` 出现在列表里、`description` 被正确读到，就说明声明成功了。之后让父 agent 处理一个适合委派的问题，它就会调用 `researcher` 这个工具把活派下去。

> 💡 别把委派当成审批边界。子 agent 本身不构成安全护栏——敏感工具该加 `needsApproval` 的还得加（具体写法以官方文档为准）。

> ⚠️ 一个命名陷阱：subagent 和 tool 共享同一套命名空间。如果你既有名为 `researcher` 的 subagent，又有名为 `researcher` 的 tool，eve 会直接拒绝构建，而不是替你选一个。


---

## 6. Connections：连接外部世界（MCP / OpenAPI）

要让 agent 用上外部系统（数据库、第三方 API、内部服务），用 **connection**——最常见的是 **MCP** 连接。

新建 **`agent/connections/my-service.ts`**（连接的运行时名直接来自文件名，例如 `agent/connections/linear.ts` 注册为 `"linear"`）：

```ts
import { defineMcpClientConnection } from "eve/connections";
import { always } from "eve/tools/approval";

export default defineMcpClientConnection({
  url: process.env.MY_MCP_URL!,   // 一个 Streamable HTTP / SSE 的 MCP 端点
  description: "我的外部服务：……（写清楚有哪些能力，模型靠它来检索）",
  auth: { getToken: async () => ({ token: process.env.MY_MCP_TOKEN! }) },
  approval: always(),             // 可选：该连接每次调用都要人工批准（人在回路）
});
```

- 模型用 `connection_search` 发现这些工具，调用名形如 `my-service__<tool>`（双下划线）。
- `approval: always()`：每次调用都会**停下来等人批准**再继续——做有副作用的写操作时很关键。
- ⚠️ 坑：eve 连接的 `url` 必须讲 **Streamable HTTP 或 SSE**。官方文档未出现「stdio」字样，因此 **stdio-only 的 MCP server 大概率不能直接连**，通常需要在它前面加一个 stdio→HTTP 的 bridge（这一点是由「只支持 Streamable HTTP / SSE」推导出来的，以官方文档为准）。

- 除了 MCP，连接也支持 **OpenAPI** 服务：用 `defineOpenAPIConnection`（同样来自 `eve/connections`）指向一个 OpenAPI 规范即可，用法类似（具体字段以官方文档为准）。

> 现在没有现成的 MCP 服务也没关系——本章理解概念即可，后面的章节不依赖它。


---

## 7. Channels：对外接口与鉴权

agent 默认就有 HTTP 接口；**channel** 让你给它加鉴权、或接入 Slack 等平台。注意：channel 只在**根 agent** 的 `agent/channels/` 下生效，子 agent（declared subagent）不支持声明 channel。

新建 **`agent/channels/web.ts`**：

```ts
import { eveChannel } from "eve/channels/eve";
import { localDev, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [
    localDev(),  // 放行 loopback 本机请求（仅 dev，见下方告警）
    httpBasic({ username: process.env.OP_USER!, password: process.env.OP_PASS! }),
  ],
});
```

- `auth` 数组**按顺序匹配，首个命中者胜出；全部不中就返回 401**（默认拒绝，fail closed；匿名访问需显式 `none()`）。
- `localDev()` 只信任请求声明的 hostname，可被伪造的 Host 头绕过，**绝不能单独使用**，必须叠加真实鉴权器——本例的 `httpBasic` 即起此作用。
- `placeholderAuth()` 只是 `eve init` 脚手架写入的占位符，在生产环境会直接返回结构化 401，必须替换为真实鉴权器（删掉该文件则回退到 `[localDev(), vercelOidc()]`）。
- 调用接口开一个会话：

```bash
curl -u "$OP_USER:$OP_PASS" -X POST http://127.0.0.1:3000/eve/v1/session \
  -H 'content-type: application/json' \
  -d '{"message":"你好"}'
```

响应头里会带 `x-eve-session-id`，响应体里还会返回 `continuationToken`（如 `eve:7f3c...`）。后续对话轮要把 `continuationToken` 放进请求体（`{"continuationToken":"...","message":"..."}`）；事件流则用 `sessionId` 通过 `GET /eve/v1/session/:sessionId/stream` 重连（流为 NDJSON，每行一个事件对象）。


---

## 8. Schedules：让 agent 定时自动干活

新建 **`agent/schedules/daily-brief.ts`**：

```ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 9 * * *", // 每天 09:00（部署到 Vercel 后按 UTC 评估）
  markdown:
    "用只读方式汇总今天的天气和待办要点，写进运行日志。不要做任何有副作用的操作。",
});
```

- 每个 schedule 提供一个 `cron`（标准 5 字段 `minute hour day-of-month month day-of-week`，分钟粒度），并在 **`markdown`** 和 **`run`** 之间**二选一**（互斥）：
  - `markdown` = fire-and-forget 的任务模式提示词（上面这种）。
  - `run` = handler，参数含 `receive` / `waitUntil` / `appAuth`，可用 `receive(channel, { message, target, auth })` 在别的 channel 起一个会话。
  - `.md` frontmatter 形式只接受 `cron`，正文当作 markdown 提示词。
- 定时任务是 **task 模式**：它**不能停下来等人审批或等 OAuth 登录**，因此天生没有「写 / 审批权限」，适合监控、汇总这类只读工作。
- 部署后每个 schedule = 一个 Vercel Cron Job，Vercel 按 **UTC** 评估表达式（如 `"0 9 * * 1-5"` 在工作日 UTC 09:00 触发）。
- **Schedules 是 root-only**：名字来自 `schedules/` 下的路径（如 `agent/schedules/billing/sweep.ts` → `"billing/sweep"`，支持嵌套目录），声明式 subagent **不能**有 `schedules/` 目录。

### 本地调试

`eve dev` 本地**不会**按 cron 节奏自动触发。调试时用 dev 触发路由手动跑：

```sh
curl -X POST http://localhost:3000/eve/v1/dev/schedules/daily-brief
```

- 路由：`POST /eve/v1/dev/schedules/<scheduleId>`，`scheduleId` 就是路径名（这里是 `daily-brief`）。
- 成功返回 `{ "scheduleId": "daily-brief", "sessionIds": ["..."] }`；未知 id 返回 `404`（含 `availableScheduleIds` 字段）。

而用 `eve build && eve start` 跑构建后的应用会真正按 cron 触发生产计划任务。


---

## 9. Hooks：挂钩生命周期与流事件

hook 是 eve 给你在「运行时事件流」上挂逻辑的地方。每个 session 在运行时会durably（持久化）记录一连串事件（如 `session.started`、`message.completed`、`action.result`），hook 订阅这些事件，在每个事件被记录之后跑一段副作用代码——典型用途是审计日志、上报指标、告警。

> ⚠️ 注意：hook 是 observe-only（只观察）的。它**不能**往模型上下文里注入内容，也不能改写事件流。要给模型动态加内容，那是 `defineDynamic` + `defineInstructions` 的活，不是 hook。

### 放在哪里

hook 放在 `agent/hooks/` 下，一个 `.ts` 文件就是一个 hook，名字由路径推导（不写 `name`/`id`）：

| 文件 | hook 名 |
|---|---|
| `agent/hooks/audit.ts` | `audit` |
| `agent/hooks/auth/load-profile.ts` | `auth/load-profile` |

子目录会被递归发现。subagent 也可以有自己的 `hooks/`（它的 hook 只在该 subagent 作用域内触发，不会因为父 agent 的 turn 而触发）。

### 最小示例

从 `eve/hooks` 引入 `defineHook`，在 `events` 里按事件名挂处理函数。处理函数签名是 `(event, ctx) => void | Promise<void>`，可以是 async：

```ts
// agent/hooks/audit.ts
import { defineHook } from "eve/hooks";

export default defineHook({
  events: {
    async "session.started"(_event, ctx) {
      console.info("session started", { sessionId: ctx.session.id });
    },
    async "message.completed"(event) {
      console.info("model finished", { length: event.data.message?.length ?? 0 });
    },
  },
});
```

`ctx` 上能拿到 `ctx.session.id`，以及 `ctx.agent`（`name`、可选 `nodeId`）、`ctx.channel`（可选的 `kind`、`continuationToken`）。`defineHook` 的定义对象**只接受 `events` 一个键**——写别的键会编译报错。

> 💡 小贴士：你也可以挂一个 `"*"` 通配处理函数，匹配所有事件。触发顺序是：先跑具名处理函数，再跑 `"*"`。

### 怎么看效果

运行 `eve dev`，发一条消息触发一个 turn，hook 里的 `console.info` 会打到 dev 的日志里（`eve dev` 默认 `--logs stderr`）。

> ⚠️ 注意：hook **抛错就是真失败**。处理函数里抛出的异常会上浮成 `turn.failed`（若订阅的是失败级联事件则升级为 `session.failed`），不会被默默吞掉。给副作用代码包一层 `try`/`catch` 更稳妥。

### 关于可订阅的事件与签名

hooks 指南里点名的事件只有 `session.started`、`turn.completed`、`message.completed`、`action.result`，外加 `"*"` 通配；完整的事件词表归在「Sessions, runs and streaming」那一页。类型系统上，`events` 的键可以是事件联合类型里的任意成员，处理函数的 `event` 会被收窄到对应类型。

> ⚠️ 注意：完整的可订阅事件清单、各事件 `event.data.*` 载荷的字段名，**以官方文档（eve.dev hooks 指南，`/docs/guides/hooks`）以及 sessions/streaming 页为准**，本章不逐一罗列以免过时。

> 💡 小贴士：处理 `action.result` 事件、想按工具结果做判断时，可以用 `eve/tools` 里的 `toolResultFrom` 对 `event.data.result` 做类型收窄（匹配不上或 `isError` 为真时返回 `undefined`），省去手写守卫。


---

## 10. Sandbox 与 Evals

**Sandbox（可选）** —— 模型自己的 bash/文件草稿空间。一个能用的沙箱默认就存在，无需编写。想收紧就锁死出网，新建 **`agent/sandbox/sandbox.ts`**：

```ts
import { defaultBackend, defineSandbox } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend(),
  async onSession({ use }) {
    await use({ networkPolicy: "deny-all" });
  },
});
```

> `defaultBackend()` 只是按 Vercel → Docker → microsandbox → just-bash 的优先级挑后端；网络收紧走 `onSession` 里的 `use({ networkPolicy: "deny-all" })`（阻断所有出网 + DNS）。
> 沙箱只是模型跑代码的草稿空间；访问你的业务系统要走 connections，不是走沙箱。起步可以先不配（框架自带默认沙箱），等让模型执行代码时再收紧。

**Evals（建议有）** —— 给 agent 写自动化测试。两个文件：

**`evals/evals.config.ts`**（`evals/` 目录的根下必须恰好一个）：

```ts
import { defineEvalConfig } from "eve/evals";
export default defineEvalConfig({});
```

**`evals/smoke.eval.ts`**（开机冒烟测试）：

```ts
import { defineEval } from "eve/evals";

export default defineEval({
  description: "能正常接收请求并完成一轮",
  async test(t) {
    await t.send("一句话介绍你能做什么，不要执行任何操作。");
    t.completed();
  },
});
```

运行：`npm run eval`（即 `eve eval`），默认针对本地 app 运行；加 `--url <url>` 可针对已部署的远程实例运行。


---

## 11. Instrumentation：可观测性（OpenTelemetry）

部署上线后，你总想知道 Agent 到底在干什么：哪些会话跑过、每轮调了哪些工具、烧了多少 token、各步耗时多久。eve 把可观测性拆成两层：一层**默认就有、零配置**，另一层**按需开启、导出到你自己的后端**。

### 默认就有：Agent Runs 面板

把 Agent 部署到 Vercel 后，**不需要写任何 instrumentation 文件**，控制台里就会出现 Agent Runs 面板。

- 路由：`/[team]/[project]/observability/agent-runs`，eve 项目自动出现。
- 概览：按触发方式分组的 Runs、token 用量（输入 / 输出 / 缓存）、以及一张表（触发消息、触发类型、进出 token、轮次数、时长、时间）。
- Run 详情：模型 / 触发方式 / 部署，逐轮的 Timings（含 skill 加载和每一次工具调用）、Input、Output、Reasoning、Tool Calls（参数 + 结果）、每轮 token 数。

这背后的原理是：即使没有 `instrumentation.ts`，eve 也会给每个 workflow run 打上 `$eve.*` 标签（如 `$eve.input_tokens`、`$eve.tool_count`），Agent Runs 面板正是靠这些标签驱动的。

> 💡 小贴士：大多数项目到这一步就够了。先部署、先看 Agent Runs，确认有需求再考虑下面的 OpenTelemetry 导出。

> ⚠️ 注意：面板会捕获输入、输出、工具参数与结果。当你的 Agent 处理个人 / 敏感 / 受监管数据时，按适用法律，你（部署者）可能需要在隐私材料中披露这种捕获。

### 进阶：导出 OpenTelemetry span

如果你想把 AI SDK 产生的 span 接到自己的 OTel 后端（Braintrust、Raindrop、Arize、Honeycomb、Datadog、Jaeger 等），就在**根 Agent**下加一个 `agent/instrumentation.ts`。

> ⚠️ 注意：`instrumentation.ts` 是 **root-only** 的——只在根 Agent 生效，subagent 里不会被发现。

它的启用方式很特别：**只要这个文件有默认导出，OTel 就隐式开启，没有单独的开关**。eve 会自动发现 `agent/instrumentation.ts`，并在任何 Agent 代码之前、服务启动时运行它。

```ts
// agent/instrumentation.ts
import { BraintrustExporter } from '@braintrust/otel';
import { defineInstrumentation } from 'eve/instrumentation';
import { registerOTel } from '@vercel/otel';

export default defineInstrumentation({
  // setup 在服务启动时跑，负责注册 OTel provider；
  // 注意：没有顶层 exporter 字段，exporter 在 setup 内部创建。
  setup: ({ agentName }) =>
    registerOTel({
      serviceName: agentName,
      traceExporter: new BraintrustExporter({
        parent: `project_name:${agentName}`,
        filterAISpans: true,
      }),
    }),

  // 下面几个字段都可选：
  // recordInputs: true,   // 是否把完整消息历史记到 step span（默认 true）
  // recordOutputs: true,  // 是否把模型输出记到 span（默认 true）
  // functionId: 'my-agent', // 覆盖 span 上的函数名（默认 = agent name）
});
```

`defineInstrumentation` 的几个常用字段：

| 字段 | 默认 | 用途 |
|---|---|---|
| `setup` | — | 回调，参数 `{ agentName }`，在 setup 内创建并注册 OTel provider |
| `recordInputs` | `true` | 是否把完整消息历史记到 step span |
| `recordOutputs` | `true` | 是否把模型输出记到 span |
| `functionId` | agent name | 覆盖 span 上的函数名 |
| `events` | — | 按事件名挂回调；已知键为 `"step.started"` |

> 💡 小贴士：`setup` 的入参 `{ agentName }` 在编译期就已解析；`parent` 和 `filterAISpans` 是 Braintrust exporter 的选项，不是 eve 的字段。

> ⚠️ 注意：处理敏感 / 受监管 / 生产数据时，除非 exporter 与数据留存路径已经过审查，否则把 `recordInputs` 和 `recordOutputs` 都设为 `false`，避免把完整消息历史和模型输出写进外部后端。

### 怎么看效果

加好文件后，span 会按这样的层级导出到你的后端：

```text
ai.eve.turn  {eve.session.id}
  +-- ai.streamText                           step 1
  |     +-- ai.streamText.doStream            模型调用
  |     +-- ai.toolCall  {toolName: search}   工具执行
  +-- ai.streamText                           step 2
  ...
```

每个 span 上还会带这些 eve 注入的属性：`eve.version`、`eve.session.id`、`eve.environment`、`eve.turn.id`、`eve.turn.sequence`、`eve.step.index`、`eve.channel.kind`。在后端按 `eve.session.id` 一筛，就能把一整次会话的所有 turn 和工具调用串起来看。

本地排查发现问题时，用 `eve info` 确认 `instrumentation.ts` 已被发现；编译产物在 `.eve/` 目录下可供查阅。

> 💡 小贴士：`events["step.started"]` 还能在每次模型调用前注入自定义的 `runtimeContext`（配合 `eve/instrumentation` 导出的 `isChannel` 按 channel 区分），属于进阶用法，具体写法以官方文档为准。


---

## 12. 部署到 Vercel

```bash
vercel          # 预览部署（首次会引导你 link 项目）
vercel --prod   # 正式上线
eve deploy      # eve 原生命令：直接部署到 Vercel 生产环境（与上面等价的另一条路径）
```

- 部署前，用 `eve link`（或 `vercel link`）把目录关联到 Vercel 项目并拉取 **AI Gateway 凭据**。
- 把代码里用到的环境变量（`OP_USER`、`OP_PASS`、`MY_MCP_*` 等）配到 **Vercel 项目的环境变量**里，别硬编码。
- 模型经 AI Gateway 访问（本地走 gateway 模型时需 `AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN` 之一；在 Vercel 上可用 OIDC 代替 provider key）。
- 上线后进入项目 **Observability（可观测性）下的 Agent Runs** 视图（路由 `/[team]/[project]/observability/agent-runs`），能看到每次运行的触发来源、工具调用、token 用量和耗时（无需任何配置自动出现）。


---

## 13. 排错与下一步

| 症状 | 多半原因 / 处理 |
|---|---|
| 命令报错或行为怪 | `node -v` 是否 ≥ 24 |
| 加了文件不生效 | `eve info` 看是否被发现；检查目录与命名（连接名来自文件名，如 `agent/connections/linear.ts` → `"linear"`）。注：短横线文件名 / eager URL 校验属个人笔记而非官方文档，以官方文档为准 |
| channel 一直 401 | 这是 fail closed，检查 `auth` 顺序（数组按顺序走，走完无人接受即 401）与凭证 |
| schedule 本地不触发 | 正常，`eve dev` 从不按 cron 触发；手动用 dev 路由跑：`curl -X POST http://localhost:3000/eve/v1/dev/schedules/<scheduleId>`（成功返回 `sessionIds`，未知 id 返回 404）。生产用 `eve start` 才会真正按 cron 触发 |
| stdio MCP 连不上 | `url` 只支持 Streamable HTTP 或 SSE，给 stdio server 加一个 HTTP/SSE bridge |
| 模型调用失败 | 查 AI Gateway 凭证（`AI_GATEWAY_API_KEY` 或 `VERCEL_OIDC_TOKEN`）/ 项目是否已 `eve link` |

**下一步**：把 `get_weather` 换成真实 API；多写几个 `*.eval.ts` 接进 CI；接一个真实的 MCP 服务。深入就读 [eve 官方文档](https://vercel.com/docs/eve)、[eve.dev 文档](https://eve.dev/docs) 与 [GitHub 仓库](https://github.com/vercel/eve)（另见 [eve.dev/docs](https://eve.dev/docs)）。eve 是 beta，一切以官方文档为准。


---

> 本教程使用 [MIT 许可证](LICENSE)，欢迎自由使用与修改。

---

> 本教程使用 [MIT 许可证](LICENSE)，欢迎自由使用与修改。
