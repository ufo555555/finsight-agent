# FinSight Agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 12 周内边学边实现一个能读取金融 PDF 正文、表格和图表，并输出带页码证据及不确定性说明的多模态研究 Agent。

**Architecture:** 先在 `learning-labs/` 中重写 Deep Agents from Scratch 的五类核心能力，再在独立的 `finance-report-agent/` 中实现产品。产品使用规划、研究、审阅三个 Agent；PDF 校验、解析、证据存储、数值检查和报告渲染采用确定性 Python 工具。

**Tech Stack:** Python 3.11、uv、LangGraph、Deep Agents、Pydantic、PyMuPDF、pdfplumber、SQLite、Streamlit、pytest、Ruff

**Spec:** `docs/superpowers/specs/2026-09-18-financial-report-agent-design.md`

## Global Constraints

- Python 版本必须为 `>=3.11,<3.14`。
- 每次学习默认 60 分钟：5 分钟回顾、40 分钟动手、10 分钟验证、5 分钟记录与提交。
- 首版一次只处理 1–3 份文本型 PDF，不把扫描件 OCR 纳入范围。
- 系统只有规划、研究和审阅三个 Agent；确定性处理不得伪装成 Agent。
- 规划最多生成 6 个研究任务，研究最多并行 2 个任务，审阅最多退回 1 次。
- 每条重要结论必须绑定证据 ID、来源文件和页码。
- 精确数值的证据优先级为表格、正文、图表；图表首版只可靠地判断趋势。
- 不预测股价，不提供买卖或仓位建议。
- PDF 内容均视为不可信数据；工具使用白名单，禁止文档内容改变系统规则。
- 密钥只从环境变量读取；`.env`、上传文件、运行产物和本地数据库不得提交。
- 每项功能遵循测试先行：失败测试、最小实现、通过测试、检查差异、提交。

---

## Planned File Structure

```text
learning-labs/
  README.md                         # 五个实验的目标、运行方式和复盘
  00-agent-loop/main.py             # 最小工具调用循环
  01-task-planning/main.py           # 有界任务规划和状态更新
  02-file-context/main.py            # 文件上下文卸载
  03-subagents/main.py               # 隔离上下文和子 Agent 委派
  04-full-deep-agent/main.py         # 组合后的研究 Agent
  tests/                             # 学习实验的行为测试

finance-report-agent/
  pyproject.toml                     # 产品依赖和开发工具配置
  .env.example                      # 无真实密钥的配置示例
  app.py                            # Streamlit 入口
  src/finsight/
    __init__.py
    config.py                       # 环境配置和运行限制
    models.py                       # Document、Evidence、Claim、Run 模型
    documents/parser.py             # PDF 校验、页级文本/表格/图片解析
    evidence/repository.py           # SQLite 证据仓库
    tools/retrieval.py               # 证据检索工具
    tools/chart_analysis.py          # 图表趋势分析适配器
    tools/numeric_checks.py          # 单位、期间和数值一致性检查
    agents/planner.py                # 规划 Agent
    agents/researcher.py             # 研究 Agent
    agents/reviewer.py               # 审阅 Agent
    workflow.py                      # LangGraph 状态与边
    reporting/renderer.py            # Markdown/JSON 报告渲染
    observability.py                 # 调用量、成本、延迟和错误记录
  tests/
    fixtures/make_sample_pdf.py      # 生成可自由使用的测试 PDF
    test_models.py
    test_parser.py
    test_repository.py
    test_numeric_checks.py
    test_agents.py
    test_workflow.py
    test_renderer.py
    test_e2e.py
  evals/
    cases.jsonl                      # 30 个固定评测问题
    run_eval.py                      # 批量评测入口
    metrics.py                       # 引用、数值、拒答等指标
  README.md
```

---

## 12-Week Daily Schedule

### Week 1：Git、环境与阅读方法

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 1 | 练习 `git status`、`git log`、`git show 898c00c`，画出工作区/暂存区/本地仓库/远程仓库四层关系 | `notes/git-basics.md` |
| Day 2 | 确认 GitHub 账户名与仓库公开性；创建空仓库，配置 `origin`，首次 `push -u origin main` | GitHub 可见首个提交 |
| Day 3 | 安装并验证 Python 3.11 与 uv；学习虚拟环境、依赖和锁文件 | 环境检查记录 |
| Day 4 | Fork/浏览 Deep Agents from Scratch，阅读 README 和 `0_create_agent.ipynb` | 第一个源码导读 |
| Day 5 | 阅读 `state.py`、`research_tools.py`，标注输入、输出和依赖 | 模块关系笔记 |
| Day 6 | 建立 `learning-labs/README.md`，写明五个实验的验收问题 | 一次文档提交 |
| Day 7 | 用 5 分钟口述 Git 提交流程和最小 Agent 数据流；整理不理解的问题 | 周复盘与下周清单 |

### Week 2：最小 Agent 循环

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 8 | 写一个工具输入 Schema 的失败测试 | 红色测试 |
| Day 9 | 实现第一个纯 Python 工具并让测试通过 | 工具提交 |
| Day 10 | 实现模型决定是否调用工具的最小循环 | 可运行 Agent |
| Day 11 | 增加最大步骤和停止条件 | 防无限循环测试 |
| Day 12 | 模拟工具异常并转成结构化错误 | 失败路径测试 |
| Day 13 | 对照原 Notebook，记录自己的实现差异 | `00-agent-loop` 复盘 |
| Day 14 | 合并 `feature/agent-loop`，练习查看分支图 | 第一个功能分支合并 |

### Week 3：任务规划与结构化状态

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 15 | 阅读 `todo_tools.py` 与 `1_todo.ipynb` | 状态转换图 |
| Day 16 | 为任务条目状态写失败测试 | Pydantic 测试 |
| Day 17 | 实现 pending/running/completed/failed 状态 | 任务模型 |
| Day 18 | 实现最多 6 项的任务规划校验 | 边界测试 |
| Day 19 | 实现任务状态更新工具 | 规划实验 |
| Day 20 | 测试非法状态转换和重复完成 | 错误案例集 |
| Day 21 | 提交并口述“模型决策”和“代码约束”的区别 | `01-task-planning` 复盘 |

### Week 4：文件上下文与恢复

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 22 | 阅读 `file_tools.py` 与 `2_files.ipynb` | 文件工具接口表 |
| Day 23 | 测试路径越界与不存在文件 | 安全失败测试 |
| Day 24 | 实现限定工作目录的读写工具 | 文件上下文工具 |
| Day 25 | 将长工具结果写入文件，只返回摘要与路径 | 上下文卸载演示 |
| Day 26 | 保存 Agent 状态并从检查点恢复 | 恢复测试 |
| Day 27 | 比较卸载前后输入 Token 数量 | 小型对比报告 |
| Day 28 | 提交并用 `git show` 复查本周改动 | `02-file-context` 复盘 |

### Week 5：子 Agent 与上下文隔离

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 29 | 阅读 `task_tool.py` 与 `3_subagents.ipynb` | 委派时序图 |
| Day 30 | 为 Agent 注册表写失败测试 | 注册表测试 |
| Day 31 | 实现一个主 Agent 委派一个研究 Agent | 最小委派流程 |
| Day 32 | 验证子 Agent 看不到无关上下文 | 隔离测试 |
| Day 33 | 增加最多两个并行任务限制 | 并发边界测试 |
| Day 34 | 对比单 Agent 与子 Agent 的结果、Token 和延迟 | 对比记录 |
| Day 35 | 提交并说明什么时候不该使用多 Agent | `03-subagents` 复盘 |

### Week 6：组合实验与金融领域约束

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 36 | 阅读 `4_full_agent.ipynb` 和 `prompts.py` | 完整流程图 |
| Day 37 | 组合规划、文件上下文和委派 | 可运行深度 Agent |
| Day 38 | 增加预算、最大步骤和一次返工限制 | 运行限制测试 |
| Day 39 | 编写金融研究禁止事项和证据规则 | 领域规则文档 |
| Day 40 | 用一小段公开财报文本执行研究任务 | 首次领域演示 |
| Day 41 | 记录失败点并决定哪些步骤改为确定性工具 | 架构决策记录 |
| Day 42 | 提交五个实验并创建学习阶段标签 `learning-v1` | 学习阶段里程碑 |

### Week 7：产品骨架、数据模型与 PDF 解析

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 43 | 创建产品包、pytest/Ruff 配置和首个失败测试 | 产品骨架 |
| Day 44 | 实现 Document、Evidence、Claim、Run 模型 | 模型测试通过 |
| Day 45 | 生成测试 PDF；测试文件类型、页数和加密校验 | PDF 校验测试 |
| Day 46 | 实现页级文本和页码提取 | 文本解析器 |
| Day 47 | 实现表格提取并保留页码 | 表格解析器 |
| Day 48 | 实现图片提取和图片编号 | 图片解析器 |
| Day 49 | 运行全部解析测试并提交 `feature/pdf-parser` | 解析里程碑 |

### Week 8：证据仓库、检索与规划 Agent

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 50 | 为 SQLite 证据写入/读取写失败测试 | 仓库红色测试 |
| Day 51 | 实现 EvidenceRepository 与去重 | 仓库测试通过 |
| Day 52 | 实现按文档、页码、模态检索 | 检索工具 |
| Day 53 | 实现最多 6 项的规划结构化输出 | 规划 Agent |
| Day 54 | 测试缺少公司、期间或问题时拒绝启动 | 输入边界测试 |
| Day 55 | 加入用户确认研究范围的中断点 | 人工审批流程 |
| Day 56 | 提交并回放一次规划运行 | 规划阶段里程碑 |

### Week 9：研究、图表理解与审阅

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 57 | 实现研究 Agent 的 Claim/Evidence 合约 | 合约测试 |
| Day 58 | 测试无证据 Claim 必须失败 | 证据门禁 |
| Day 59 | 实现单位、币种、期间基础检查 | 数值检查工具 |
| Day 60 | 实现图表分析适配器和模型失败降级 | 图表工具测试 |
| Day 61 | 强制图表精确数值必须有表格/正文第二证据 | 多模态门禁 |
| Day 62 | 实现审阅 Agent 的通过/退回/不确定输出 | 审阅 Agent |
| Day 63 | 提交并比较审阅前后的无证据结论数量 | 审阅阶段里程碑 |

### Week 10：完整工作流、报告和界面

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 64 | 定义 LangGraph 状态和三 Agent 节点 | 工作流骨架 |
| Day 65 | 连接解析→规划→研究→审阅→报告边 | 端到端流程 |
| Day 66 | 实现检查点恢复和最多一次返工 | 恢复/循环测试 |
| Day 67 | 实现 Markdown 与 JSON 报告模板 | 报告渲染测试 |
| Day 68 | 创建 Streamlit 上传与提问界面 | 页面 1 |
| Day 69 | 增加进度、报告与证据展开视图 | 页面 2–3 |
| Day 70 | 运行端到端测试并提交 MVP 标签 `v0.1.0` | 可演示 MVP |

### Week 11：评测、可靠性与成本

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 71 | 选定 3–5 家公司和至少 6 份公开报告，只记录来源链接 | 数据来源清单 |
| Day 72 | 编写 10 个正文事实问题 | 评测集第一部分 |
| Day 73 | 编写 10 个表格/财务指标问题 | 评测集第二部分 |
| Day 74 | 编写 5 个图表趋势和 5 个拒答问题 | 完整 30 题评测集 |
| Day 75 | 实现证据覆盖率、引用定位和数值准确率 | 指标代码 |
| Day 76 | 运行基线并分类失败原因 | 基线报告 |
| Day 77 | 修复最高频失败并重跑，提交前后对比 | 评测优化提交 |

### Week 12：安全、文档、部署与简历

| 天 | 60 分钟任务 | 当天产物 |
|---|---|---|
| Day 78 | 增加文档提示注入测试和工具白名单测试 | 安全测试 |
| Day 79 | 增加超时、重试、费用和延迟记录 | 可观测性结果 |
| Day 80 | 编写 README：问题、架构、运行、评测、局限 | 项目主页 |
| Day 81 | 增加 Dockerfile 或一键启动脚本并从空环境验证 | 部署说明 |
| Day 82 | 录制 2–3 分钟演示，展示一个成功和一个拒答案例 | Demo 视频 |
| Day 83 | 创建 Pull Request，按测试证据进行自审并合并 | 完整 PR |
| Day 84 | 创建 `v1.0.0` Release，用真实指标撰写简历要点 | 项目发布 |

---

## Task 1: Repository and Learning-Lab Foundation

**Files:**
- Create: `learning-labs/README.md`
- Create: `learning-labs/00-agent-loop/main.py`
- Create: `learning-labs/tests/test_agent_loop.py`
- Modify: `.gitignore`

**Interfaces:**
- Consumes: Python 3.11 and a chat-model adapter implementing `invoke(messages) -> AIMessage`.
- Produces: `run_agent(model: ChatModel, user_input: str, tools: dict[str, Callable], max_steps: int = 6) -> str`.

- [ ] **Step 1: Create a feature branch**

```bash
git switch -c feature/agent-loop
```

- [ ] **Step 2: Write the failing maximum-step test**

```python
def test_agent_stops_at_max_steps(fake_looping_model):
    result = run_agent(
        fake_looping_model,
        "research revenue",
        {"search": lambda q: q},
        max_steps=2,
    )
    assert result == "stopped:max_steps"
    assert fake_looping_model.calls == 2
```

- [ ] **Step 3: Run the focused test and confirm failure**

Run: `uv run pytest learning-labs/tests/test_agent_loop.py -v`

Expected: FAIL because `run_agent` does not exist.

- [ ] **Step 4: Implement the bounded loop**

```python
def run_agent(
    model: ChatModel,
    user_input: str,
    tools: dict[str, Callable],
    max_steps: int = 6,
) -> str:
    messages = [{"role": "user", "content": user_input}]
    for _ in range(max_steps):
        response = model.invoke(messages)
        if not response.tool_calls:
            return response.content
        messages.extend(execute_whitelisted_tools(response.tool_calls, tools))
    return "stopped:max_steps"
```

- [ ] **Step 5: Run tests and inspect the diff**

Run: `uv run pytest learning-labs/tests/test_agent_loop.py -v`

Expected: PASS.

Run: `git diff --check && git diff --stat`

- [ ] **Step 6: Commit the experiment**

```bash
git add learning-labs .gitignore
git commit -m "feat(labs): add bounded agent loop"
```

## Task 2: Planning, File Context, and Subagent Labs

**Files:**
- Create: `learning-labs/01-task-planning/main.py`
- Create: `learning-labs/02-file-context/main.py`
- Create: `learning-labs/03-subagents/main.py`
- Create: `learning-labs/04-full-deep-agent/main.py`
- Create: `learning-labs/tests/test_planning.py`
- Create: `learning-labs/tests/test_file_context.py`
- Create: `learning-labs/tests/test_subagents.py`

**Interfaces:**
- Consumes: `run_agent(model, user_input, tools, max_steps)` from Task 1.
- Produces: `TaskItem`, `SafeWorkspace`, `delegate_task(...)`, and `run_deep_agent(...)`.

- [ ] **Step 1: Define and test task-state transitions**

```python
class TaskItem(BaseModel):
    title: str
    status: Literal["pending", "running", "completed", "failed"] = "pending"

def test_completed_task_cannot_return_to_running():
    task = TaskItem(title="extract revenue", status="completed")
    with pytest.raises(ValueError):
        transition(task, "running")
```

- [ ] **Step 2: Implement `transition` and the six-task limit**

Run: `uv run pytest learning-labs/tests/test_planning.py -v`

Expected: PASS for valid transitions and rejection of plans containing seven tasks.

- [ ] **Step 3: Test and implement the safe workspace**

```python
def test_workspace_rejects_parent_escape(tmp_path):
    workspace = SafeWorkspace(tmp_path)
    with pytest.raises(ValueError, match="outside workspace"):
        workspace.read("../secret.txt")
```

Run: `uv run pytest learning-labs/tests/test_file_context.py -v`.

- [ ] **Step 4: Test isolated subagent input**

```python
def test_delegate_passes_only_task_context(fake_agent):
    delegate_task(fake_agent, task="check margin", context={"evidence": ["e1"]})
    assert "unrelated_chat" not in fake_agent.last_input
```

Run: `uv run pytest learning-labs/tests/test_subagents.py -v`.

- [ ] **Step 5: Combine the four capabilities**

Implement `run_deep_agent(question: str, max_steps: int = 12) -> AgentResult` with planning, file offload, at most two delegated tasks, and checkpoint persistence.

- [ ] **Step 6: Verify and commit**

```bash
uv run pytest learning-labs/tests -v
git diff --check
git add learning-labs
git commit -m "feat(labs): combine planning files and subagents"
git tag learning-v1
```

## Task 3: Product Scaffold and Domain Models

**Files:**
- Create: `finance-report-agent/pyproject.toml`
- Create: `finance-report-agent/.env.example`
- Create: `finance-report-agent/src/finsight/__init__.py`
- Create: `finance-report-agent/src/finsight/config.py`
- Create: `finance-report-agent/src/finsight/models.py`
- Create: `finance-report-agent/tests/test_models.py`

**Interfaces:**
- Produces: `Settings`, `Document`, `Evidence`, `Claim`, `ResearchTask`, and `RunRecord` Pydantic models.

- [ ] **Step 1: Configure the package and test tools**

Set `requires-python = ">=3.11,<3.14"`; add runtime dependencies `pydantic`, `pydantic-settings`, `pymupdf`, `pdfplumber`, `langgraph`, `deepagents`, `streamlit`; add development dependencies `pytest`, `pytest-asyncio`, `ruff`, `mypy`.

- [ ] **Step 2: Write a failing evidence-bound claim test**

```python
def test_important_claim_requires_evidence():
    with pytest.raises(ValidationError):
        Claim(text="Revenue increased", importance="important", evidence_ids=[])
```

- [ ] **Step 3: Define exact models**

```python
class Evidence(BaseModel):
    evidence_id: str
    document_id: str
    page_number: int = Field(ge=1)
    modality: Literal["text", "table", "chart"]
    content: str
    locator: str

class Claim(BaseModel):
    claim_id: str
    text: str
    importance: Literal["important", "supporting"]
    evidence_ids: list[str]
    review_status: Literal["pending", "approved", "rejected", "uncertain"] = "pending"
```

Add a model validator that rejects important claims without evidence.

- [ ] **Step 4: Verify and commit**

```bash
cd finance-report-agent
uv sync
uv run pytest tests/test_models.py -v
uv run ruff check .
git add pyproject.toml .env.example src tests/test_models.py
git commit -m "feat(domain): add evidence-first research models"
```

## Task 4: PDF Parser

**Files:**
- Create: `finance-report-agent/src/finsight/documents/parser.py`
- Create: `finance-report-agent/tests/fixtures/make_sample_pdf.py`
- Create: `finance-report-agent/tests/test_parser.py`

**Interfaces:**
- Consumes: a local `Path` to one PDF.
- Produces: `parse_pdf(path: Path) -> ParsedDocument` with page-numbered text, tables, and image paths.

- [ ] **Step 1: Generate a deterministic two-page fixture**

Create a PDF containing a revenue paragraph on page 1 and a small margin table plus chart image on page 2. Generate it during tests so no third-party report is committed.

- [ ] **Step 2: Write validation and page-number tests**

```python
def test_parser_preserves_page_numbers(sample_pdf):
    parsed = parse_pdf(sample_pdf)
    assert parsed.pages[0].page_number == 1
    assert parsed.pages[1].page_number == 2

def test_parser_rejects_non_pdf(tmp_path):
    path = tmp_path / "report.txt"
    path.write_text("not a pdf")
    with pytest.raises(InvalidDocumentError):
        parse_pdf(path)
```

- [ ] **Step 3: Implement deterministic extraction**

Use PyMuPDF for validation, text, images, file hash and page count. Use `pdfplumber` for tables. Record a page-level extraction error and continue other pages.

- [ ] **Step 4: Verify and commit**

```bash
uv run pytest tests/test_parser.py -v
uv run ruff check src/finsight/documents tests/test_parser.py
git add src/finsight/documents tests/fixtures tests/test_parser.py
git commit -m "feat(parser): extract page-linked financial evidence"
```

## Task 5: Evidence Repository and Retrieval

**Files:**
- Create: `finance-report-agent/src/finsight/evidence/repository.py`
- Create: `finance-report-agent/src/finsight/tools/retrieval.py`
- Create: `finance-report-agent/tests/test_repository.py`

**Interfaces:**
- Consumes: `Document` and `Evidence` models.
- Produces: `EvidenceRepository.add_document`, `add_evidence`, `get_evidence`, `search_evidence`.

- [ ] **Step 1: Write persistence and deduplication tests**

```python
def test_duplicate_evidence_is_not_inserted(repository, evidence):
    first = repository.add_evidence(evidence)
    second = repository.add_evidence(evidence)
    assert first.evidence_id == second.evidence_id
    assert repository.count_evidence() == 1
```

- [ ] **Step 2: Implement the SQLite schema and repository**

Create `documents`, `evidence`, `claims`, and `runs` tables. Use parameterized SQL and a unique key over document, page, modality, locator and content hash.

- [ ] **Step 3: Implement bounded retrieval**

```python
def search_evidence(
    query: str,
    *,
    document_ids: list[str],
    modalities: set[str] | None = None,
    limit: int = 8,
) -> list[Evidence]: ...
```

Reject limits above 20 and queries without a document scope.

- [ ] **Step 4: Verify and commit**

```bash
uv run pytest tests/test_repository.py -v
git add src/finsight/evidence src/finsight/tools/retrieval.py tests/test_repository.py
git commit -m "feat(evidence): persist and retrieve page-linked sources"
```

## Task 6: Numeric and Chart Tools

**Files:**
- Create: `finance-report-agent/src/finsight/tools/numeric_checks.py`
- Create: `finance-report-agent/src/finsight/tools/chart_analysis.py`
- Create: `finance-report-agent/tests/test_numeric_checks.py`

**Interfaces:**
- Produces: `normalize_amount`, `check_period`, and `analyze_chart(image_path, context) -> ChartObservation`.

- [ ] **Step 1: Test financial normalization**

```python
@pytest.mark.parametrize(
    ("raw", "expected"),
    [("1.2亿元", Decimal("120000000")), ("350万元", Decimal("3500000"))],
)
def test_normalize_amount(raw, expected):
    assert normalize_amount(raw, currency="CNY") == expected
```

- [ ] **Step 2: Implement explicit units and periods**

Return a structured error when currency, unit or period is missing. Do not infer an unstated currency.

- [ ] **Step 3: Test chart failure fallback**

```python
def test_chart_timeout_returns_uncertain(fake_timeout_model, image_path):
    result = analyze_chart(image_path, context="revenue trend")
    assert result.status == "uncertain"
    assert result.exact_values == []
```

- [ ] **Step 4: Implement the vision adapter**

Allow title, axes, legend, direction, turning points and uncertainty. Keep `exact_values` empty unless a second text/table evidence ID is supplied.

- [ ] **Step 5: Verify and commit**

```bash
uv run pytest tests/test_numeric_checks.py -v
git add src/finsight/tools tests/test_numeric_checks.py
git commit -m "feat(tools): validate financial values and chart trends"
```

## Task 7: Planner, Researcher, and Reviewer Agents

**Files:**
- Create: `finance-report-agent/src/finsight/agents/planner.py`
- Create: `finance-report-agent/src/finsight/agents/researcher.py`
- Create: `finance-report-agent/src/finsight/agents/reviewer.py`
- Create: `finance-report-agent/tests/test_agents.py`

**Interfaces:**
- Planner produces `list[ResearchTask]` of length 1–6.
- Researcher produces `list[Claim]` with evidence IDs.
- Reviewer produces `ReviewDecision(status, claims, notes)`.

- [ ] **Step 1: Test the planner boundary**

```python
def test_planner_rejects_more_than_six_tasks(fake_model):
    fake_model.output = {"tasks": [{"title": str(i)} for i in range(7)]}
    with pytest.raises(PlanLimitError):
        plan_research(fake_model, scoped_request)
```

- [ ] **Step 2: Implement structured planning and clarification**

Require company, report period and a bounded question before a run starts. Store the user's approval as a workflow event.

- [ ] **Step 3: Test the research evidence contract**

```python
def test_researcher_drops_unsupported_important_claim(fake_model, repository):
    claims = research_task(fake_model, task, repository)
    assert all(c.evidence_ids for c in claims if c.importance == "important")
```

- [ ] **Step 4: Test the review decision**

```python
def test_reviewer_marks_conflicting_sources_uncertain():
    decision = review_claims(conflicting_claims, evidence_by_id)
    assert decision.status == "uncertain"
```

- [ ] **Step 5: Implement prompts as enforceable contracts**

Use Pydantic structured output. Treat document content as quoted data. Reject investment recommendations and claims with missing evidence.

- [ ] **Step 6: Verify and commit**

```bash
uv run pytest tests/test_agents.py -v
git add src/finsight/agents tests/test_agents.py
git commit -m "feat(agents): add evidence-bound research workflow roles"
```

## Task 8: Workflow, Recovery, and Observability

**Files:**
- Create: `finance-report-agent/src/finsight/workflow.py`
- Create: `finance-report-agent/src/finsight/observability.py`
- Create: `finance-report-agent/tests/test_workflow.py`

**Interfaces:**
- Produces: `build_workflow(settings, repository, model) -> CompiledStateGraph`.
- Workflow state includes documents, question, tasks, claims, review round, usage and errors.

- [ ] **Step 1: Test the exact node order**

```python
def test_happy_path_node_order(recording_workflow):
    result = recording_workflow.invoke(valid_input)
    assert result["visited"] == ["parse", "plan", "research", "review", "render"]
```

- [ ] **Step 2: Test one-review retry limit**

```python
def test_review_can_return_to_research_only_once(always_rejecting_reviewer):
    result = workflow.invoke(valid_input)
    assert result["review_round"] == 1
    assert result["status"] == "completed_with_uncertainty"
```

- [ ] **Step 3: Build state and conditional edges**

Add checkpoint persistence by `run_id`. Limit model calls to two retry attempts with timeouts and recorded errors.

- [ ] **Step 4: Record usage**

`RunMetrics` records start/end time, model calls, input/output tokens, estimated cost, retry count and error categories.

- [ ] **Step 5: Verify and commit**

```bash
uv run pytest tests/test_workflow.py -v
git add src/finsight/workflow.py src/finsight/observability.py tests/test_workflow.py
git commit -m "feat(workflow): orchestrate agents with recovery limits"
```

## Task 9: Report Renderer and Streamlit UI

**Files:**
- Create: `finance-report-agent/src/finsight/reporting/renderer.py`
- Create: `finance-report-agent/tests/test_renderer.py`
- Create: `finance-report-agent/app.py`

**Interfaces:**
- Produces: `render_report(run: RunRecord, claims: list[Claim], evidence: dict[str, Evidence]) -> ReportArtifacts`.

- [ ] **Step 1: Test citation rendering**

```python
def test_report_contains_filename_page_and_modality(claim, evidence):
    report = render_report(run, [claim], {evidence.evidence_id: evidence})
    assert "annual-report.pdf, p.12, table" in report.markdown
```

- [ ] **Step 2: Implement deterministic Markdown and JSON output**

Sections are summary, verified facts, risks, conflicting evidence, uncertain items, sources and run metrics. Rendering must not invoke a model.

- [ ] **Step 3: Build three UI steps**

Use Streamlit file upload and form for input, a progress/status section, and a report section with expandable evidence. Reject files before workflow invocation.

- [ ] **Step 4: Verify and commit**

```bash
uv run pytest tests/test_renderer.py -v
uv run streamlit run app.py
git add app.py src/finsight/reporting tests/test_renderer.py
git commit -m "feat(ui): present reports with expandable evidence"
```

## Task 10: Evaluation and End-to-End Safety

**Files:**
- Create: `finance-report-agent/evals/cases.jsonl`
- Create: `finance-report-agent/evals/metrics.py`
- Create: `finance-report-agent/evals/run_eval.py`
- Create: `finance-report-agent/tests/test_e2e.py`

**Interfaces:**
- Produces: `evaluate_case(case, report) -> CaseMetrics` and aggregate JSON/Markdown results.

- [ ] **Step 1: Define the 30-case schema**

Each JSONL row contains `case_id`, `document_ids`, `question`, `expected_claims`, `expected_pages`, `expected_values`, and `must_refuse`.

- [ ] **Step 2: Implement metric tests**

```python
def test_citation_precision_counts_wrong_page_as_error():
    score = citation_precision(predicted_pages=[3, 9], expected_pages=[3, 8])
    assert score == 0.5
```

- [ ] **Step 3: Add prompt-injection and no-evidence cases**

The end-to-end test PDF contains text asking the system to ignore prior rules. Assert the workflow treats it as evidence text, does not alter tools, and refuses unsupported advice.

- [ ] **Step 4: Run baseline and save results**

Run: `uv run python evals/run_eval.py --output artifacts/eval-baseline.json`.

Expected output includes evidence coverage, citation accuracy, numeric accuracy, uncertainty detection, completion rate, cost and latency.

- [ ] **Step 5: Run the complete quality gate and commit**

```bash
uv run pytest -v
uv run ruff check .
uv run mypy src
git diff --check
git add evals tests/test_e2e.py
git commit -m "test(eval): add reproducible financial research benchmark"
```

## Task 11: Documentation, Deployment, Pull Request, and Release

**Files:**
- Create: `finance-report-agent/README.md`
- Create: `finance-report-agent/Dockerfile`
- Modify: `finance-report-agent/.env.example`

**Interfaces:**
- Produces: reproducible local setup, container startup, architecture explanation, evaluation table, demo and known limitations.

- [ ] **Step 1: Write README from verified evidence**

Include problem statement, architecture, setup, configuration, data policy, safety boundaries, test commands, evaluation method, measured results and limitations. Do not publish target metrics as achieved results.

- [ ] **Step 2: Add container startup**

The container runs `streamlit run app.py --server.address=0.0.0.0`. Mount uploads and artifacts outside the image; pass secrets only at runtime.

- [ ] **Step 3: Verify from a clean environment**

```bash
docker build -t finsight-agent:v1 .
docker run --rm -p 8501:8501 --env-file .env finsight-agent:v1
```

Expected: the upload form opens at `http://localhost:8501` and no secret is present in the image history.

- [ ] **Step 4: Create and review a release Pull Request**

```bash
git switch -c release/v1
git add README.md Dockerfile .env.example
git commit -m "docs: document reproducible FinSight release"
git push -u origin release/v1
```

The PR description contains the user problem, architecture changes, commands executed, evaluation results, screenshots and remaining limitations.

- [ ] **Step 5: Tag the verified release**

```bash
git switch main
git pull --ff-only
git tag -a v1.0.0 -m "FinSight Agent v1.0.0"
git push origin main --tags
```

Expected: GitHub displays the merged history and `v1.0.0` tag. Create a Release using the verified evaluation summary and demo video.

---

## GitHub Remote Setup Gate

The existing commit `898c00c` is local because `git remote -v` currently returns no entries. Before Week 1 Day 2, confirm the GitHub account name and choose public or private visibility. Then use one of these two explicit paths:

### GitHub CLI path

After authenticating with `gh auth login`, create and push the repository:

```bash
gh repo create finsight-agent --source=. --remote=origin --push
```

Choose Public or Private in the interactive prompt. Do not add another README, `.gitignore`, or license on GitHub because the local repository already has history.

### GitHub website path

Create an empty repository named `finsight-agent` without README, `.gitignore`, or license. Copy the exact HTTPS URL shown by GitHub, then run the first command, paste the URL, and press Enter:

```bash
read -r finsight_remote_url
git remote add origin "$finsight_remote_url"
git push -u origin main
```

Before pushing, run `git remote -v` and verify it points to the intended account and repository.

## Final Verification Checklist

- [ ] `uv run pytest -v` passes with zero failures.
- [ ] `uv run ruff check .` reports no errors.
- [ ] `uv run mypy src` reports no errors.
- [ ] `git diff --check` prints no output.
- [ ] `git status --short` prints no unintended files.
- [ ] All important claims contain resolvable evidence IDs.
- [ ] The 30-case evaluation report includes every required metric.
- [ ] README reports measured results separately from target thresholds.
- [ ] `.env`, PDFs, database files and run artifacts are absent from Git history.
- [ ] A clean-machine or clean-container setup reproduces the demo.
- [ ] GitHub shows the intended branch, Pull Request and `v1.0.0` tag.
