
### 静态tool retrieval
**ToolRetriever / ToolBench 系列**  
ToolBench/ToolLLM 把真实 API 转成大规模 tool-use 训练与评测资源，覆盖 RapidAPI 上大量真实 API，并提出 neural API retriever + ToolLLaMA + ToolEval 这样的 pipeline。它是很多后续 tool retrieval / tool learning 论文的基础。

**ToolRet**  
ToolRet 是专门面向 tool retrieval 的 benchmark，包含约 **7.6k retrieval tasks**、**43k tools**，并构造了超过 **200k training instances**。它的一个重要结论是：通用 IR 模型并不天然 “tool-savvy”，因为 tool retrieval 需要理解 tool 的功能、参数和使用场景。

**Tool-DE / Tool-REX**  
这类工作关注 “tool document 太短、太乱、不完整” 的问题。Tool-DE 从多个 tool-use 数据源构建 benchmark，并通过 document expansion、Tool-Embed、Tool-Rank 等方式增强检索和 rerank。这个方向对 continual learning 很有价值，因为新工具刚加入时往往文档质量很差。

**COLT**  
COLT 认为 tool retrieval 不能只看 query-tool semantic matching，还应该利用 query、scenario、tool 之间的 collaborative signal，通过图结构学习工具之间的共用关系。这个思路可以自然扩展到 continual setting：新工具加入时，可以利用相似任务、相似场景、co-use graph 做迁移。

**PLUTo / EasyTool / ToolRerank / MassTool / MFTR**  
这些方法分别从 query planning、tool document 标准化、hierarchical reranking、多字段建模、多任务 retrieval 等角度增强 tool retrieval。你不一定都要复现，但它们可以作为 “tool-specific strong baselines”。


|Benchmark|适合研究什么|怎么用于 continual tool retrieval|
|---|---|---|
|**ToolRet**|纯 tool retrieval，大规模工具库|最适合改造成 CL benchmark：按 domain/category/time split 成任务流|
|**TR-bench**|tool retrieval，包含 in-domain / out-of-domain / updated tools|很适合做 “新工具/更新工具” 的 evaluation；它基于 ToolBench、T-Eval、UltraTools 构建。([arXiv](https://arxiv.org/html/2406.17465v2 "Enhancing Tool Retrieval with Iterative Feedback from Large Language Models"))|
|**MTRB**|massive tool retrieval，低资源、大规模候选工具|可用于测试 few-shot continual adaptation；它设置了大规模工具环境、低资源训练，并引入 Sufficiency@k。([arXiv](https://arxiv.org/abs/2410.03212 "[2410.03212] Data-Efficient Massive Tool Retrieval: A Reinforcement Learning Approach for Query-Tool Alignment with Language Models"))|
|**Tool-DE / Tool-REX**|tool 文档增强、retriever/reranker|适合研究 “新工具文档不完整” 时如何 continual update。([arXiv](https://arxiv.org/html/2510.22670v1 "Tools are under-documented: Simple Document Expansion Boosts Tool Retrieval"))|
|**API-Bank**|API planning、retrieval、calling|适合做端到端验证：retriever 变好是否真的提升 tool-use 成功率。API-Bank 有 runnable API 评测、73 个 API tools、314 个 tool-use dialogues / 753 calls，并提供较大训练集。([ACL Anthology](https://aclanthology.org/2023.emnlp-main.187/ "API-Bank: A Comprehensive Benchmark for Tool-Augmented LLMs - ACL Anthology"))|
|**ToolBench / StableToolBench**|大规模真实 API tool-use|StableToolBench 修复了 ToolBench 里真实 API 不稳定的问题，用 virtual API server、cache 和 API simulator 提供更稳定评测。([ACL Anthology](https://aclanthology.org/2024.findings-acl.664/ "StableToolBench: Towards Stable Large-Scale Benchmarking on Tool Learning of Large Language Models - ACL Anthology"))|
|**BFCL**|function calling / tool calling|更偏 end-to-end function calling leaderboard，不是纯 retrieval，但可作为下游验证。BFCL V4 覆盖 AST、multi-turn、agentic 等版本演进。([Gorilla](https://gorilla.cs.berkeley.edu/leaderboard.html "Berkeley Function Calling Leaderboard (BFCL) V4"))|
|**ToolSandbox**|stateful tool execution|适合测试状态依赖、多轮交互、隐式 state change；不适合只看 retrieval，但适合最终 agent-level 评测。([ACL Anthology](https://aclanthology.org/2025.findings-naacl.65/ "ToolSandbox: A Stateful, Conversational, Interactive Evaluation Benchmark for LLM Tool Use Capabilities - ACL Anthology"))|
|**τ-bench**|dynamic conversation + domain API + policy|适合测真实 agent 稳定性；它通过最终 database state 和 pass^k 评估 agent 是否稳定完成任务。([arXiv](https://arxiv.org/abs/2406.12045 "[2406.12045] $τ$-bench: A Benchmark for Tool-Agent-User Interaction in Real-World Domains"))|
## Baseline 设计建议

你们的 baseline 最好分 4 层，不然 reviewer 会觉得比较不完整。

### 4.1 通用 retrieval baseline

这些是必须有的：

|类型|Baseline|
|---|---|
|lexical|BM25 / TF-IDF|
|dense bi-encoder|Contriever、E5、BGE、GTE、Sentence-BERT|
|hybrid|BM25 + dense score fusion|
|late interaction|ColBERT-style retriever|
|reranker|cross-encoder reranker / LLM reranker|
|oracle|train-on-all、gold tool set、oracle top-k|

### 4.2 Tool-specific baseline

推荐优先级：

1. **ToolRetriever / ToolBench retriever**
2. **ToolRet-style supervised retriever**
3. **Tool-DE: Tool-Embed / Tool-Rank**
4. **COLT-style graph/collaborative retrieval**
5. **IterFeedback**：用 LLM feedback 改写 query 再检索；这个方法明确针对 tool retrieval 中 query 和 tool description 难匹配的问题。
6. **PLUTo / EasyTool / ToolRerank**，根据你们算力和复现成本选择。

### 4.3 Continual learning baseline

这里是你们论文最重要的比较组：

|Baseline|含义|作用|
|---|---|---|
|**Frozen retriever + append index**|encoder 不更新，只加入新工具 embedding|最简单、稳定，但新域适应差|
|**Sequential fine-tuning**|每个 task 继续训练|必须有，通常遗忘严重|
|**Full replay**|保留所有旧数据训练|upper-ish baseline，但存储成本高|
|**Memory replay**|每个旧 task 采样少量 query-tool pair|最实用的 CL baseline|
|**Distillation**|新模型保持旧模型 top-k 分布或 embedding|减少旧工具遗忘|
|**EWC / SI / L2 regularization**|限制重要参数变化|标准 CL baseline|
|**Adapter / LoRA per stage**|主模型冻结，每阶段学 adapter|适合测试参数隔离|
|**QDC-style query projection**|不重建旧 index，让新 query 兼容旧 tool embedding|特别适合 tool registry 持续扩张|
|**Online feedback update**|用调用成功/失败在线更新|更贴近真实 agent 部署|
|**Train-on-all**|每一步用所有历史数据训练|upper bound|
|**Oracle tool retrieval**|直接给 gold tools|下游 tool-use 上限|

### 4.4 Generative / agentic baseline

作为补充：

| Baseline                               | 用途                             |
| -------------------------------------- | ------------------------------ |
| ToolkenGPT-style tool token            | 测试生成式 tool selection 是否更适合新增工具 |
| ToolGen-style virtual tool tokens      | 测试无显式 retrieval 的替代路线          |
| ReAct + top-k tools                    | 测 retrieval 对 agent 推理链的影响     |
| LLM-only selection from candidate list | 测 reranker / selector 能力       |
| LLM query rewrite + retrieval          | 测 query rewrite 对新工具适应的帮助      |