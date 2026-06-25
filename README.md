# Vercel eve · 从零手把手构建一个完整 Agent

跟着这份教程，你会**从零开始、一行行自己写**，最终做出一个叫 `devhelper` 的完整 AI agent：它有自己的工具、技能、外部连接、对外接口、定时任务和测试，最后部署上线。**全程不抄现成项目**，每个文件都由你亲手创建。

> eve 还处于 beta，API 可能随版本变化；拿不准时以 [官方文档](https://vercel.com/docs/eve) 为准。

**预备**：会用终端、看得懂一点 TypeScript、Node.js ≥ 24、一个 Vercel 账号（用于模型访问与部署）。

**你会一章加一块，建成 `devhelper`：**

| 章 | 你新建的文件 | 学到的概念 |
|---|---|---|
| 2 | `agent/agent.ts`、`agent/instructions.md` | agent 最小骨架 |
| 3 | `agent/tools/get_weather.ts` | Tools |
| 4 | `agent/skills/weekly-report.md` | Skills |
| 5 | `agent/connections/my-service.ts` | Connections（MCP） |
| 6 | `agent/channels/web.ts` | Channels + 鉴权 |
| 7 | `agent/schedules/daily-brief.ts` | Schedules |
| 8 | `agent/sandbox/sandbox.ts`、`evals/*` | Sandbox + Evals |
| 9 | — | 部署到 Vercel |

**目录**：1. 环境准备 · 2. 跑起第一个 agent · 3. Tools · 4. Skills · 5. Connections · 6. Channels · 7. Schedules · 8. Sandbox 与 Evals · 9. 部署 · 10. 排错与下一步

---

## 1. 环境准备

```bash
node -v                 # 必须 ≥ 24；不够就：nvm install 24 && nvm use 24
npm i -g vercel         # Vercel CLI，用于模型访问凭证与部署
vercel login
```

eve 通过 Vercel **AI Gateway** 调模型，本地开发也要能连上。最简单的方式是把项目 link 到 Vercel 拿到 gateway 凭证（下一步 `init` 后做即可）。就绪检查：`node -v`(≥24)、`vercel whoami` 能输出你的账号。

---

## 2. 跑起第一个 agent

```bash
npx eve@latest init devhelper
cd devhelper
npm install
```

一个 agent 最少只要两个文件。

**`agent/agent.ts`** —— 模型与运行配置：

```ts
import { defineAgent } from "eve";

export default defineAgent({
  model: "anthropic/claude-sonnet-4.6", // 经 AI Gateway 路由，可换成别的模型
});
```

**`agent/instructions.md`** —— 系统提示词（必需，决定 agent 的性格与规则）：

```md
你是 devhelper，一个简洁、务实的开发者助手。
- 回答直接，给可执行的步骤。
- 不确定就说不确定，绝不编造。
```

启动并对话：

```bash
npm run dev    # = eve dev，本地交互式终端 UI，默认 http://127.0.0.1:3000
```

对它说「你好，你能帮我做什么？」，能正常回话就成功了。

> 关键命令 `eve info`：列出 eve 发现并编译了哪些能力。以后「加了文件却不生效」时，第一时间跑它——多半是文件放错目录或命名不对。

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
> 有副作用/高风险的操作可以要求**人工审批**后才执行（见第 5 章连接层的 `approval`）。

---

## 4. Skills：把可复用的流程沉淀下来

当某段「多步流程/规程」很长，别全塞进 `instructions.md`（那样每一轮都占上下文）。把它做成 **skill**，模型只在相关时按需加载。

新建 **`agent/skills/weekly-report.md`**：

```md
# 生成周报

当用户要「周报」时，按下面做：
1. 收集本周完成的要点（让用户给出，或从对话里提取）。
2. 整理成三段：进展 / 风险 / 下周计划。
3. 用要点列表，控制在 200 字以内。
```

之后用户说「帮我写周报」，模型会加载这个 skill，再照步骤执行。

> 一句话区分：**Tools 是能执行的函数；Skills 是给模型读的步骤性知识。**

---

## 5. Connections：连接外部世界（MCP）

要让 agent 用上外部系统（数据库、第三方 API、内部服务），用 **connection**——最常见的是 **MCP** 连接。

新建 **`agent/connections/my-service.ts`**（⚠️ 连接文件名用**短横线**；`url` 在编译期就会被校验）：

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

- 模型用 `connection_search` 发现这些工具，调用名形如 `my-service__<tool>`。
- `approval: always()`：每次调用都会**停下来等人批准**再继续——做有副作用的写操作时很关键。
- ⚠️ 坑：eve 连接要求 **HTTP/SSE**。**stdio-only 的 MCP server 不能直接连**，需要在它前面加一个 stdio→HTTP 的 bridge。

> 现在没有现成的 MCP 服务也没关系——本章理解概念即可，后面的章节不依赖它。

---

## 6. Channels：对外接口与鉴权

agent 默认就有 HTTP 接口；**channel** 让你给它加鉴权、或接入 Slack 等平台。

新建 **`agent/channels/web.ts`**：

```ts
import { eveChannel } from "eve/channels/eve";
import { localDev, httpBasic } from "eve/channels/auth";

export default eveChannel({
  auth: [
    localDev(),  // 放行本机 / dev
    httpBasic({ username: process.env.OP_USER!, password: process.env.OP_PASS! }),
  ],
});
```

- `auth` 数组**按顺序匹配，首个命中者胜出；全部不中就返回 401**（默认拒绝，fail closed）。生产环境别用 `placeholderAuth()`。
- 调用接口开一个会话：

```bash
curl -u "$OP_USER:$OP_PASS" -X POST http://127.0.0.1:3000/eve/v1/session \
  -H 'content-type: application/json' \
  -d '{"message":"你好"}'
```

响应头里会带 `x-eve-session-id`，用它可以重连到该会话的事件流。

---

## 7. Schedules：让 agent 定时自动干活

新建 **`agent/schedules/daily-brief.ts`**：

```ts
import { defineSchedule } from "eve/schedules";

export default defineSchedule({
  cron: "0 9 * * *", // 每天 09:00（部署到 Vercel 后按 UTC 评估）
  markdown:
    "用只读方式汇总今天的天气和待办要点，写进运行日志。不要做任何有副作用的操作。",
});
```

- 定时任务是 **task 模式**：它**不能停下来等人工审批**，因此天生没有「写 / 审批权限」，适合监控、汇总这类只读工作。
- 部署后每个 schedule = 一个 Vercel Cron Job。`eve dev` 本地**不会**按点自动触发，调试时用 dev 的触发路由手动跑。

---

## 8. Sandbox 与 Evals

**Sandbox（可选）** —— 模型自己的 bash/文件草稿空间。想收紧就锁死出网，新建 **`agent/sandbox/sandbox.ts`**：

```ts
import { defineSandbox, defaultBackend } from "eve/sandbox";

export default defineSandbox({
  backend: defaultBackend({
    vercel: { networkPolicy: "deny-all" },
    docker: { networkPolicy: "deny-all" },
  }),
});
```

> 沙箱只是模型跑代码的草稿空间；访问你的业务系统要走 connections，不是走沙箱。起步可以先不配，等让模型执行代码时再收紧。

**Evals（建议有）** —— 给 agent 写自动化测试。两个文件：

**`evals/evals.config.ts`**（根目录必须恰好一个）：

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

运行：`npm run eval`（= `eve eval`，针对本地 dev server 跑）。

---

## 9. 部署到 Vercel

```bash
vercel          # 预览部署（首次会引导你 link 项目）
vercel --prod   # 正式上线
```

- 把代码里用到的环境变量（`OP_USER`、`OP_PASS`、`MY_MCP_*` 等）配到 **Vercel 项目的环境变量**里，别硬编码。
- 模型经 AI Gateway 访问；上线后进项目的 **Agent Runs** 视图，能看到每次运行的触发来源、工具调用、token 用量和耗时（无需任何配置自动出现）。

---

## 10. 排错与下一步

| 症状 | 多半原因 / 处理 |
|---|---|
| 命令报错或行为怪 | `node -v` 是否 ≥ 24 |
| 加了文件不生效 | `eve info` 看是否被发现；检查目录与命名（连接文件名要短横线） |
| channel 一直 401 | 这是 fail closed，检查 `auth` 顺序与凭证 |
| schedule 本地不触发 | 正常，用 dev 触发路由手动跑 |
| stdio MCP 连不上 | 加一个 HTTP/SSE bridge |
| 模型调用失败 | 查 AI Gateway 凭证 / 项目是否已 link |

**下一步**：把 `get_weather` 换成真实 API；多写几个 `*.eval.ts` 接进 CI；接一个真实的 MCP 服务。深入就读 [eve 官方文档](https://vercel.com/docs/eve) 与 [GitHub 仓库](https://github.com/vercel/eve)。eve 是 beta，一切以官方文档为准。

---

> 本教程使用 [MIT 许可证](LICENSE)，欢迎自由使用与修改。
