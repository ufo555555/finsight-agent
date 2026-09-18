# Agent 项目选型与项目式学习计划

> 调研日期：2026-09-18
>
> 适用对象：已经接触过 Agent 概念，但更适合通过项目边做边学的初学者
>
> 默认投入：10 周，每周 8–12 小时（选定项目后可按你的时间重排）

## 1. 先说结论

你的学习思路没有问题，而且通常比“先把所有理论学完再动手”更适合 Agent 开发。不过要做一个关键调整：**不能只把开源项目运行起来或改 Prompt**。那样既学不深，也不足以成为有说服力的简历项目。

一个合格的简历级二次开发至少要同时具备：

1. **明确的真实场景**：为某一类用户解决具体问题，而不是“万能聊天机器人”。
2. **可解释的 Agent 架构**：工具调用、状态、规划、记忆、工作流或多 Agent 协作中至少覆盖四项。
3. **工程完整度**：前后端/API、配置管理、异常处理、日志、测试、部署说明。
4. **效果证据**：自建评测集、成功率/正确率/成本/延迟对比，而不是只展示几张运行截图。
5. **你的原创改动**：能够清楚回答“原项目有什么问题、你改了什么、为什么这样设计、效果提升多少”。

建议采用 **20% 定向理论 + 60% 编码与调试 + 20% 复盘和评测**，不要重新进行一轮漫长的系统理论学习。

## 2. 候选项目总览

评分是针对“初学者做成简历项目”的适配度，不代表项目本身的绝对优劣。

| 选项 | 项目与建议改造方向 | 入门难度 | 可定制性 | Agent 知识覆盖 | 简历潜力 | 综合建议 |
|---|---|---:|---:|---:|---:|---:|
| **A（首选）** | Deep Agents from Scratch → 垂直领域任务/研究 Agent | 3/5 | 5/5 | 5/5 | 5/5 | **9.3/10** |
| **B** | OpenAI Agents SDK 客服示例 → 业务流程多 Agent 系统 | 2.5/5 | 5/5 | 4.5/5 | 4.5/5 | **8.9/10** |
| **C** | Local Deep Researcher → 本地隐私知识研究助手 | 3/5 | 4.5/5 | 4/5 | 4/5 | **8.4/10** |
| **D（挑战）** | GPT Researcher → 行业级全栈研究平台 | 4.5/5 | 5/5 | 5/5 | 5/5 | **8.1/10** |

## 3. 候选项目详解

### A. Deep Agents from Scratch（推荐）

- GitHub：[langchain-ai/deep-agents-from-scratch](https://github.com/langchain-ai/deep-agents-from-scratch)
- 可搭配：[langchain-ai/deepagents](https://github.com/langchain-ai/deepagents) 和 [Deep Agents UI](https://github.com/langchain-ai/deep-agents-ui)
- 技术栈：Python、LangGraph、Jupyter、LLM API、搜索工具；扩展阶段可加入 FastAPI/前端。
- 开源许可：MIT。

**为什么最适合你**

它不是一上来就丢给你一个庞大的成品，而是用 5 个递进式 Notebook 展示完整 Agent 的形成过程：ReAct 循环、TODO 规划、文件系统上下文卸载、子 Agent 隔离与委派，最后组合为完整研究 Agent。你可以先看懂每个增量，再把 Notebook 中的能力重构成自己的工程。

**推荐个性化方向（选一个即可）**

- 求职情报 Agent：解析 JD、调研公司、匹配个人经历、生成面试研究包。
- 技术选型 Agent：并行调研框架，按约束生成带证据的 ADR/选型报告。
- 论文复现 Agent：检索论文、抽取实验配置、形成复现 TODO、跟踪结果。
- 课程学习 Agent：读取课程资料，规划任务，调用代码执行/检索工具，维护错题与进度。

**要补的原创工程层**

持久化任务、人工审批、失败重试、结构化输出、评测集、成本/延迟记录、Web UI、Docker 部署。完成这些后，它会从“课程作业”变成真正的作品。

**主要风险**

示例默认依赖外部模型和搜索 API；需控制调用费用。不要直接把 Notebook 当最终项目提交。

### B. OpenAI Agents SDK 客服示例

- GitHub：[openai/openai-agents-python](https://github.com/openai/openai-agents-python)
- 起始示例：[examples/customer_service](https://github.com/openai/openai-agents-python/tree/main/examples/customer_service)
- 技术栈：Python、OpenAI Agents SDK、Pydantic；扩展阶段可加入 FastAPI、SQLite/PostgreSQL、React/Next.js。
- 开源许可：MIT。

**为什么值得选**

SDK 的核心概念相对少，示例直接覆盖工具、上下文、路由/交接与追踪，初学者更容易看懂一次 Agent 运行到底发生了什么。相比研究 Agent，它更适合展示业务工作流、权限边界和确定性流程。

**推荐个性化方向**

- 电商售后 Agent：订单查询、退换货规则、风险操作人工审批、工单升级。
- 校园事务 Agent：课程、教务、场地、失物招领等多部门分流。
- SaaS 技术支持 Agent：知识库检索、故障诊断、自动收集环境信息、升级人工支持。

**必须做的升级**

不要停在命令行 Demo。把示例抽成独立仓库，加入真实或模拟数据库、RAG、输入/输出 Guardrail、会话持久化、人工审批、回放评测和可视化界面。

**主要风险**

默认路径与 OpenAI API 结合紧密；需要 API 预算。若只改 Agent 名称和 Prompt，简历价值很低。

### C. Local Deep Researcher

- GitHub：[langchain-ai/local-deep-researcher](https://github.com/langchain-ai/local-deep-researcher)
- 技术栈：Python、LangGraph、Ollama/LM Studio、DuckDuckGo/SearXNG/Tavily。
- 开源许可：MIT。

**为什么值得选**

项目结构比完整研究平台小，工作流也很清楚：生成查询 → 搜索 → 总结 → 反思知识缺口 → 继续搜索 → 输出带引用报告。它支持本地模型，并可用无需 API Key 的搜索入口，适合预算有限、重视隐私、希望先完整跑通闭环的人。

**推荐个性化方向**

- 企业内部资料 + Web 混合研究助手。
- 本地隐私论文/法规研究助手。
- 中英文双语信源交叉验证助手。

**必须做的升级**

增加本地文档 RAG、信源可信度评分、引用核验、任务中断恢复、不同模型/搜索策略对照实验、独立 UI 和部署方案。

**主要风险**

原始循环相对单一，若不增加评测和工程能力，容易显得像普通 RAG Demo。本地模型效果还受设备内存和模型工具调用能力影响。

### D. GPT Researcher（挑战路线）

- GitHub：[assafelovic/gpt-researcher](https://github.com/assafelovic/gpt-researcher)
- 技术栈：Python、FastAPI、LangGraph/LangChain、多 Agent、Next.js、TypeScript、Tailwind、MCP、Docker。
- 开源许可：Apache-2.0。

**为什么简历潜力高**

它已经是一个较完整的全栈研究系统，包含规划、并行研究、信源汇总、报告生成、前端、文档处理、多种导出格式、测试和部署相关代码。适合希望体现“读懂大型开源项目并做系统级改造”的人。

**推荐个性化方向**

- 竞品与市场情报平台。
- 投研资料梳理平台（只能做信息研究，不包装成投资建议）。
- 招投标/政策变化研究平台。

**主要风险**

仓库规模大、依赖多，初学者很容易把时间耗在环境、前端和历史兼容代码上，而没有真正理解 Agent。只有当你已经能独立完成普通 Python Web 项目，或愿意投入 12–16 周时才建议选。

## 4. 我的推荐顺序

1. **优先选 A**：学习路径最顺，既能理解底层模式，又留出了足够的原创空间。最适合“之前系统学过但学不进去”的情况。
2. **偏业务应用选 B**：如果你希望项目更像真实公司的客服/运营系统，并想较快形成可演示产品。
3. **预算有限或重视本地隐私选 C**：可先用本地模型跑通，但要确认电脑配置。
4. **已有 Python Web + 前端基础再选 D**：作品上限最高，但第一次 Agent 项目失败风险也最高。

## 5. 为什么没有推荐另外几个热门仓库

- [langchain-ai/open_deep_research](https://github.com/langchain-ai/open_deep_research)：架构和测试仍很有学习价值，但仓库已于 **2026-08-21** 归档，只建议作为参考材料，不建议作为新项目主线。
- [langchain-ai/open-agent-platform](https://github.com/langchain-ai/open-agent-platform)：官方已标注 deprecated/archived，不宜作为当前技术栈的主项目。
- [OpenHands](https://github.com/OpenHands/OpenHands)：项目很优秀，但对第一次 Agent 项目来说范围过大，容易变成阅读大型系统，而不是完成自己的作品。
- [microsoft/ai-agents-for-beginners](https://github.com/microsoft/ai-agents-for-beginners)：非常适合按需补课，涵盖工具、Agentic RAG、规划、多 Agent、协议、记忆、部署和安全；但它本质是课程仓库，应该作为伴学资料，而不是最终简历项目。

## 6. 10 周通用学习计划

> 原则：每周都必须有一个可运行、可解释、可提交的成果。理论只在本周功能需要时学习。

| 周次 | 本周目标 | 动手任务 | 只补这些知识 | 验收产物 |
|---|---|---|---|---|
| 第 0 周（2–3 小时） | 完成选型与环境检查 | 选 A/B/C/D；确认 Python、Git、API/本地模型；Fork 后建立自己的项目说明 | Git 分支、虚拟环境、环境变量安全 | 选型说明、成功启动截图、首次提交 |
| 第 1 周 | 跑通并画出原项目 | 完成一条端到端任务；记录每次模型调用、工具调用和状态变化 | Agent loop、ReAct、tool calling、structured output | 架构图、运行时序、源码导读笔记 |
| 第 2 周 | 不看答案重写最小 Agent | 自己实现一个单 Agent + 2 个工具；为工具输入输出加类型 | Prompt 边界、Pydantic/Schema、异常处理 | 可测试的最小 Agent、单元测试 |
| 第 3 周 | 加入状态与记忆 | 区分短期会话状态、任务状态、长期记忆；实现一种持久化 | Context、checkpoint、memory、token budget | 可恢复的多轮任务、状态说明 |
| 第 4 周 | 加入规划与可靠性 | 任务拆分、反思/重试、超时、最大步数、幂等处理 | Plan-and-execute、循环停止条件、fallback | 故障注入记录、失败后可恢复演示 |
| 第 5 周 | 加入多 Agent 或确定性工作流 | 只为真正需要隔离的任务增加 2–3 个角色；比较单/多 Agent | Handoff、agent-as-tool、context isolation、并发 | 架构决策记录和对比实验 |
| 第 6 周 | 完成垂直场景 MVP | 接入领域数据和 3–5 个真实工具；设计人工审批点 | RAG、MCP（需要时）、HITL、权限边界 | 10 条真实任务均可执行的 MVP |
| 第 7 周 | 工程化 | API、持久化、日志、配置、基础 UI、Docker；敏感操作默认拒绝 | FastAPI、数据库、日志、容器基础 | 一条命令启动、README 安装指南 |
| 第 8 周 | 建立评测系统 | 自建 30–50 条测试集；记录完成率、正确率、引用率、成本、延迟 | 离线评测、LLM-as-judge 局限、回归测试 | 基线与改进版对比表、失败案例集 |
| 第 9 周 | 部署与可观测性 | 部署 Demo；增加 trace、费用统计、健康检查；处理并发和限流 | Observability、缓存、rate limit、安全 | 可访问 Demo、监控截图、部署文档 |
| 第 10 周 | 简历与面试化 | 重写 README、录制 2–3 分钟 Demo、整理技术决策和数字结果 | STAR 表达、系统设计复盘 | 项目主页、演示视频、简历要点、面试题库 |

## 7. 不同选项对计划的调整

| 选择 | 需要调整的重点 |
|---|---|
| A | 第 1–5 周严格跟随 5 个 Notebook 的能力顺序，但每学完一个模块就在独立工程中重写；第 7 周接 Deep Agents UI 或自建轻量 UI。 |
| B | 第 3 周重点做 session/业务上下文；第 4–5 周做 Guardrail、审批与客服升级；第 6 周接业务数据库和知识库。 |
| C | 第 0 周先验证本地模型能否稳定结构化输出；第 5 周可用“检索/核验/写作”三个角色；第 8 周必须对比至少两个本地模型。 |
| D | 延长到 12–16 周；前两周只读核心研究链路，不要全仓库逐文件阅读；先选一个子系统改造，再逐步接前端和部署。 |

## 8. 每周执行模板（防止再次“学不进去”）

每次学习 60–120 分钟，按以下顺序：

1. 写下今天唯一的可观察目标，例如“让 Agent 在工具超时后重试一次并记录原因”。
2. 先运行或修改代码 30–45 分钟，遇到概念障碍再定向查资料。
3. 用自己的话写 5 句话：输入是什么、状态是什么、模型决定了什么、代码决定了什么、哪里可能失败。
4. 提交一个小 Commit；Commit 信息说明“行为变化”，不写笼统的 `update`。
5. 每周末录制 3 分钟讲解：架构、一个失败案例、本周取舍。讲不清的地方就是下周补课点。

## 9. 最终项目验收标准

满足以下条件后再把项目写进简历：

- [ ] 项目名称和业务场景已经脱离原教程，不是简单 Fork 展示。
- [ ] 至少 3 个真实工具，其中一个包含失败、超时或权限控制。
- [ ] 有持久化状态/记忆，并说明为什么保存这些信息。
- [ ] 有人工审批或明确的安全边界。
- [ ] 有 30–50 条可重复评测用例和至少 3 个量化指标。
- [ ] 有单元测试、集成测试和至少一条端到端测试。
- [ ] 有成本、延迟、失败率记录，以及一次优化前后对比。
- [ ] 可用 Docker 或清晰脚本一键启动；密钥不进入 Git。
- [ ] README 包含架构图、Demo、设计取舍、局限与未来工作。
- [ ] 你能在 5 分钟内不用背稿解释一次完整任务的数据流。

## 10. 简历描述模板（完成后再填数字）

> 设计并实现面向 **[目标用户/场景]** 的多步骤 Agent 系统，基于 **[框架]** 编排 **[工具/角色]**，支持 **[规划、记忆、人工审批、恢复等能力]**；构建 **[N]** 条任务评测集，将 **[核心指标]** 从 **[基线]** 提升至 **[结果]**，平均任务成本/延迟降低 **[X%]**；通过 **[技术栈]** 完成可观测、可部署的端到端产品。

不要在项目未完成时虚构指标；评测数据本身就是面试中最有价值的材料之一。

## 11. 你现在只需要做的选择

请回复：

1. 选项 **A / B / C / D**；
2. 你目前的 **Python 水平**（未学过 / 会基础语法 / 做过 Web 项目）；
3. 是否会 **JavaScript/TypeScript**；
4. 每周大约能投入多少小时；
5. 更想做的领域（求职、学习、客服、研究、金融、其他）。

收到后，下一版计划会细化到每天的任务、要读的具体源码文件、每周 Commit/验收标准，以及最终项目的功能边界。

## 12. 调研来源

- [Deep Agents from Scratch](https://github.com/langchain-ai/deep-agents-from-scratch)
- [Deep Agents](https://github.com/langchain-ai/deepagents)
- [Deep Agents UI](https://github.com/langchain-ai/deep-agents-ui)
- [OpenAI Agents SDK](https://github.com/openai/openai-agents-python)
- [OpenAI Agents SDK Quickstart](https://github.com/openai/openai-agents-python/blob/main/docs/quickstart.md)
- [Local Deep Researcher](https://github.com/langchain-ai/local-deep-researcher)
- [GPT Researcher](https://github.com/assafelovic/gpt-researcher)
- [AI Agents for Beginners](https://github.com/microsoft/ai-agents-for-beginners)
- [Open Deep Research（已归档，仅作参考）](https://github.com/langchain-ai/open_deep_research)
