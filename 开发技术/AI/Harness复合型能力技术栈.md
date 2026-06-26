你可以把现在的 AI Agent 人才分成三层。

第一层是**模型调用者**：会调用 LLM API、会写 prompt、会接 LangChain/Dify/Cursor。这类人很多，不稀缺。

第二层是**Agent 工程师**：会设计 Agent Loop、Tool Use、MCP、Memory、规划、反思、沙盒执行、权限控制、失败重试、日志追踪、评测体系，能把一个 Agent 从 demo 做成稳定系统。这是 DeepSeek 现在很缺的一类。

第三层是**Harness 研究/工程复合型人才**：不只是实现功能，而是能回答“为什么这个 Agent 会失败”“上下文该怎么压缩”“工具该怎么设计才容易被模型调用”“长期记忆什么时候有用、什么时候反而污染决策”“如何构建 benchmark 证明方法有效”。报道中提到的 Harness 研究员要求偏科研，包括科研经验、论文、独立提出 idea、快速转成原型、熟练使用 AI Agent 工具开发等。

# 技术栈
## 基础工程：Python + Web + 系统能力
- Python：异步、类型标注、Pydantic、FastAPI、pytest、日志、异常处理。
- Linux / Shell：文件系统、进程、权限、环境变量、Docker、常见命令。
- Git / GitHub：分支、PR、issue、代码 review、CI。
- 后端 API：REST、WebSocket、SSE streaming、鉴权、限流、幂等、重试。
- 数据存储：PostgreSQL、Redis、SQLite、对象存储、向量数据库。
- 前端基本能力：React/Next.js 足够，不一定要很深，但要能做 Agent 产品界面、任务追踪、trace 展示。


## LLM API 与推理工程

你必须非常熟悉 LLM API 的底层交互，而不是只会 `client.chat.completions.create()`。

要掌握：

- OpenAI-compatible API / Anthropic-compatible API。
- streaming 输出。
- JSON mode / structured output。
- function calling / tool calling。
- token 计算、上下文窗口、输入输出成本。
- retry、timeout、rate limit、fallback model。
- KV Cache / prefix cache / context caching。
- thinking mode / reasoning effort / non-thinking mode 的取舍。

DeepSeek 官方 API 文档明确写到，它的 API 兼容 OpenAI/Anthropic 格式，并且可接入 Claude Code、GitHub Copilot、OpenCode 等 Agent 工具。 DeepSeek 也有 Context Caching：相同前缀请求可以触发缓存命中，从而减少重复计算；这就是你看到 JD 里提到 KV Cache 的现实工程背景。


## Agent Harness
**Harness 可以理解为：模型之外，让 Agent 真正能工作的整套工程外壳。**
包括：
- Agent Loop：观察 → 规划 → 调工具 → 读结果 → 修正 → 继续执行 → 终止。
- Tool Use：工具定义、参数 schema、错误处理、工具选择、工具结果压缩。
- Planning：任务拆解、计划更新、子任务管理。
- Memory：短期记忆、长期记忆、用户画像、任务状态、经验总结。
- Context Engineering：给模型塞什么、不塞什么、如何压缩、如何排序、如何避免污染。
- Subagent：把复杂任务拆给不同专家 agent。
- Multi-Agent：多个 agent 协作、仲裁、投票、冲突解决。
- Guardrails：权限、人工确认、危险操作拦截。
- Observability：trace、日志、tool call 轨迹、失败原因分析。
- Evaluation：构造任务集、自动打分、人工评估、A/B 测试。

OpenAI 对 Agent 的定义也强调：Agent 是会规划、调用工具、在专家之间协作，并保持足够状态来完成多步任务的应用；当应用自己拥有编排、工具执行、审批、状态管理时，就进入 Agents SDK 这类更复杂模式。 Anthropic 的工程文章也强调，成功的 Agent 实现通常不是最复杂框架，而是简单、可组合的模式，并区分了固定代码路径的 workflows 和由 LLM 动态决定流程的 agents。


| 关键词                 | 你要掌握到什么程度                                              |
| ------------------- | ------------------------------------------------------ |
| LLM API             | 会封装多模型调用、streaming、重试、超时、成本统计、结构化输出                    |
| KV Cache            | 理解推理阶段为什么需要 KV cache，prefix caching 如何影响长上下文成本         |
| Agent Loop          | 能手写一个 ReAct/Plan-Act-Reflect 循环，而不是只用框架                |
| Tool Use            | 会设计 JSON Schema 工具、执行工具、返回结果、处理失败                      |
| Reasoning           | 理解 CoT、test-time compute、self-consistency、verification |
| Planning            | 会做任务拆解、计划更新、计划失败回滚                                     |
| Skills              | 理解 skill 是可加载的能力包：说明文档、代码、资源、工具集合                      |
| MCP                 | 会写 MCP server/client，把外部系统暴露给 Agent                    |
| Memory              | 能实现写入、检索、更新、遗忘、冲突处理、记忆评测                               |
| Subagent            | 会把任务分派给代码 agent、搜索 agent、审查 agent 等                    |
| Multi-Agent         | 理解协作成本、上下文污染、角色冲突、通信协议                                 |
| Prompt Engineering  | 会写指令，但不迷信 prompt                                       |
| Context Engineering | 会控制上下文结构、压缩、排序、引用、状态管理                                 |
| Harness Engineering | 能把以上所有东西工程化、可观测、可评测、可迭代                                |


# 信息源
### 第一层：官方源

- DeepSeek 官网、API Docs、GitHub、技术报告。DeepSeek 官网本身就把 R1、V3、Coder、Math、VL 等研究入口放在 Research 下，并有 Join Us 入口。
- DeepSeek API Docs：重点看 Tool Calls、Context Caching、JSON Output、Thinking Mode、Agent Integrations。
- OpenAI Developers / Agents SDK / Evals。
- Anthropic Engineering：尤其是 Agent、Tool Use、MCP、Context、Claude Code。
- Model Context Protocol 官方文档。MCP 官方定义是连接 AI 应用和外部数据、工具、工作流的开放标准。

### 第二层：论文与 benchmark

- arXiv cs.AI / cs.CL / cs.SE。
- Papers with Code。
- SWE-bench leaderboard。
- BFCL leaderboard。
- Terminal-Bench。
- OSWorld。
- GAIA。
- AgentBench、ToolBench、AppWorld、WebArena、τ-bench、LiveCodeBench。

你读论文不要平均用力，优先看四类：

1. Agent architecture。
2. Tool use / function calling。
3. Memory / long-context / context compression。
4. Agent evaluation / benchmark。

DeepSeek-V3 技术报告值得读，因为里面有 MoE、MLA、FP8、训练和推理工程等底层内容。 DeepSeek-R1 报告值得读，因为它解释了如何用强化学习激发推理能力，以及 R1-Zero、冷启动数据、多阶段训练、蒸馏等逻辑。

### 第三层：工程社区

关注这些：

- GitHub Trending：agent、mcp、llm、rag、eval、swe-agent。
- LangChain / LangGraph / LangSmith。
- LlamaIndex。
- OpenAI Agents SDK。
- PydanticAI。
- AutoGen。
- CrewAI。
- Smolagents。
- vLLM / SGLang / LiteLLM。
- MCP server registry。
- Cursor / Claude Code / OpenCode / Aider / Cline / Codex CLI 的更新。

框架是拿来拆的，不是拿来崇拜的。你要学的是它们背后的设计：状态管理、工具协议、上下文组织、trace、eval。

### 第四层：X / Hacker News / Reddit

重点关注：

- DeepSeek 核心成员。
- Anthropic / OpenAI / LangChain / Cursor / Cognition / Factory / Aider / OpenCode 的工程师。
- Simon Willison。
- swyx。
- Harrison Chase。
- Shishir Patil / Gorilla BFCL 相关团队。
- SWE-bench / OSWorld / Terminal-Bench 相关作者。

这类信息源适合捕捉“还没写成论文但工业界已经在做”的东西。