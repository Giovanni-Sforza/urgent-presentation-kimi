# ARIS (Auto Research Intelligence System)

> 自动化科研智能系统 —— 覆盖从文献调研、实验规划、论文撰写到 Presentation 准备的全链路科研辅助工具集。
>
> 上游项目：[https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep)

---

## 一、项目定位

ARIS 是一个面向科研人员的最小可运行科研辅助框架，旨在加速以下场景：

- **文献调研**：快速了解一个新领域，产出结构化摘要
- **实验规划**：设计、跟踪和记录实验流程
- **论文撰写**：从实验结果到完整论文的辅助写作
- **Presentation 准备**：幻灯片、海报、演讲稿、配图的全链路生成
- **学术评审**：论文审稿、新颖性检查、实验审计、数学证明验证
- **专利申请**：从发明披露到权利要求书、说明书的全流程辅助

本项目包含 **Agent 配置**、**技能模块**、**工具脚本**和**工作流模板**四大组件，可根据需求独立或组合使用。

---

## 二、目录结构

```
aris/
├── README.md                          # 本文件：项目说明与使用指南
├── LICENSE                            # MIT 许可证
├── agents/                            # Agent 配置（Kimi CLI 用）
│   ├── aris-kimi.md                   # 主代理系统提示词
│   ├── aris-kimi.yaml                 # 主代理配置（含子代理注册表）
│   └── subagents/                     # 8 个专业子代理配置
│       ├── research-reviewer.yaml
│       ├── novelty-checker.yaml
│       ├── experiment-auditor.yaml
│       ├── paper-editor.yaml
│       ├── patent-examiner.yaml
│       ├── proof-reviewer.yaml
│       ├── hep-reviewer.yaml
│       └── hep-theory-reviewer.yaml
├── skills/                            # 核心技能模块（7 个）
│   ├── presentation-slides/           # 通用幻灯片生成
│   ├── paper-slides/                  # 从已编译论文生成会议演讲幻灯片
│   ├── paper-poster/                  # 从论文生成会议海报 (LaTeX → PDF/PPTX/SVG)
│   ├── lecture-script/                # 从幻灯片/综述生成演讲稿
│   ├── mermaid-diagram/               # 生成流程图/架构图/时序图等
│   ├── paper-illustration/            # AI 生成高质量论文插图
│   └── quick-lit-review/              # 快速文献调研（为 presentation 提供素材）
├── templates/                         # 工作流输入模板
│   ├── CLAUDE_MD_TEMPLATE.md          # 项目面板（Pipeline Status）
│   ├── RESEARCH_BRIEF_TEMPLATE.md     # 研究方向简报
│   ├── EXPERIMENT_PLAN_TEMPLATE.md    # 实验计划（Claim-driven）
│   ├── NARRATIVE_REPORT_TEMPLATE.md   # 研究叙事报告
│   ├── PAPER_PLAN_TEMPLATE.md         # 论文大纲
│   ├── INVENTION_BRIEF_TEMPLATE.md    # 发明披露书
│   ├── PATENT_CLAIMS_TEMPLATE.md      # 专利权利要求书
│   ├── PATENT_SPECIFICATION_TEMPLATE.md # 专利说明书
│   └── ...（更多模板详见 templates/README.md）
└── tools/                             # Python / Shell 工具脚本
    ├── arxiv_fetch.py                 # arXiv 搜索与下载
    ├── semantic_scholar_fetch.py      # Semantic Scholar 搜索
    ├── deepxiv_fetch.py               # DeepXiv 数据获取
    ├── exa_search.py                  # Exa 搜索引擎接口
    ├── figure_renderer.py             # FigureSpec → SVG 渲染器
    ├── research_wiki.py               # 研究 Wiki 工具
    ├── watchdog.py                    # 文件监控与自动处理
    ├── smart_update.sh                # 智能更新脚本
    └── save_trace.sh                  # Trace 保存脚本
```

---

## 三、Agent 与子代理

主代理 `aris-kimi` 负责编排复杂科研工作流，可将任务委派给以下专业子代理：

| 子代理 | 用途 |
|--------|------|
| `research-reviewer` | 对抗性论文评审（NeurIPS/ICML 级别） |
| `novelty-checker` | 针对现有技术验证新颖性 |
| `experiment-auditor` | 实验完整性审计与欺诈模式检测 |
| `paper-editor` | 学术论文写作与编辑 |
| `patent-examiner` | 专利新颖性与非显而易见性评估 |
| `proof-reviewer` | 数学证明严格审查 |
| `hep-reviewer` | 高能物理现象学评审 |
| `hep-theory-reviewer` | 高能物理理论评审 |

---

## 四、技能矩阵

### 核心技能（Presentation 相关）

| 技能 | 适用场景 | 输入 | 输出 | 特点 |
|------|---------|------|------|------|
| `presentation-slides` | 通用汇报、课堂展示、临时 talk | 任意文本/综述/大纲/论文 | Beamer PDF + PPTX + Markdown | **最灵活**，不依赖完整论文 |
| `paper-slides` | 会议 Oral / Spotlight / Poster-talk | 已编译的论文目录 (`paper/`) | Beamer PDF + PPTX + 完整讲稿 | **学术级**，严格按论文内容生成 |
| `paper-poster` | 会议 Poster Session | 已编译的论文目录 (`paper/`) | A0/A1 PDF + PPTX + SVG | **视觉优先**，支持多会场配色 |

### 辅助技能

| 技能 | 作用 | 调用时机 |
|------|------|---------|
| `quick-lit-review` | 10-15 分钟快速调研一个领域，产出结构化摘要 | **准备阶段**：当你对话题不熟，需要先收集素材 |
| `mermaid-diagram` | 生成流程图、架构图、时序图等 | **制作阶段**：幻灯片/海报需要示意图时 |
| `paper-illustration` | AI 生成出版级插图（架构图、方法示意图） | **制作阶段**：需要高质量视觉素材时 |
| `lecture-script` | 按页生成带时间标记的完整演讲稿 | **收尾阶段**：需要排练或给他人代讲时 |

---

## 五、工具脚本

| 工具 | 功能 | 典型用法 |
|------|------|---------|
| `arxiv_fetch.py` | arXiv 搜索、PDF 下载、源码获取 | `python3 tools/arxiv_fetch.py search "attention mechanism" --max 10` |
| `semantic_scholar_fetch.py` | Semantic Scholar 论文搜索与引用分析 | 集成于文献调研工作流 |
| `deepxiv_fetch.py` | DeepXiv 数据获取 | 补充论文源数据 |
| `exa_search.py` | Exa 语义搜索引擎 | 网络级文献与资源搜索 |
| `figure_renderer.py` | FigureSpec JSON → 出版级 SVG | `python3 tools/figure_renderer.py render spec.json --output out.svg` |
| `research_wiki.py` | 研究 Wiki 管理 | 构建个人研究知识库 |
| `watchdog.py` | 文件系统监控与自动触发 | 自动响应实验输出变化 |
| `smart_update.sh` | 智能增量更新 | 批量更新技能与模板 |
| `save_trace.sh` | 会话 Trace 保存 | 归档 Agent 交互记录 |

---

## 六、工作流模板

`templates/` 目录提供即用型 Markdown 模板，覆盖以下工作流：

### 研究管线（Research Pipeline）

| 模板 | 对应工作流 | 用途 |
|------|-----------|------|
| `RESEARCH_BRIEF_TEMPLATE.md` | Workflow 1 | 详细研究方向文档输入 |
| `RESEARCH_CONTRACT_TEMPLATE.md` | Workflow 1 | 定义问题边界、非目标、时间线 |
| `EXPERIMENT_PLAN_TEMPLATE.md` | Workflow 1.5 | Claim-driven 实验路线图 |
| `NARRATIVE_REPORT_TEMPLATE.md` | Workflow 3 | 研究叙事（Claims + 实验 + 结果） |
| `PAPER_PLAN_TEMPLATE.md` | Workflow 3 | 预置论文大纲 |
| `CLAUDE_MD_TEMPLATE.md` | 全部工作流 | 项目面板（Pipeline Status） |
| `MANIFEST_TEMPLATE.md` | 全部工作流 | 输出跟踪清单 |

### 专利管线（Patent Pipeline）

| 模板 | 用途 |
|------|------|
| `INVENTION_BRIEF_TEMPLATE.md` | 发明披露（技术问题、方案、优势、附图） |
| `PATENT_CLAIMS_TEMPLATE.md` | 权利要求层次表（CN/US/EP 示例） |
| `PATENT_SPECIFICATION_TEMPLATE.md` | 说明书骨架（含全部必要章节） |

---

## 七、推荐工作流

### 场景 A：「明天要汇报，但我只有一个话题」

```
quick-lit-review ──► presentation-slides ──► lecture-script
     (10-15 min)         (20-30 min)           (10 min)
```

1. 用 `quick-lit-review` 快速了解话题，产出 `QUICK_REVIEW.md`
2. 用 `presentation-slides` 基于 `QUICK_REVIEW.md` 生成幻灯片
3. （可选）用 `lecture-script` 生成逐页讲稿

### 场景 B：「论文已接收，需要准备会议报告」

```
paper-slides ──► lecture-script
   (30-40 min)      (15 min)
```

1. 用 `paper-slides` 基于 `paper/` 目录生成对应 talk 类型（oral/spotlight）的幻灯片
2. 用 `lecture-script` 生成完整讲稿 + Q&A 预案

### 场景 C：「需要 poster session 海报」

```
paper-poster
  (40-60 min)
```

1. 用 `paper-poster` 直接生成 A0/A1 海报，输出 PDF + PPTX + SVG
2. 如需自定义插图，中间可插入 `paper-illustration` 或 `mermaid-diagram`

### 场景 D：「幻灯片需要一张架构图」

```
mermaid-diagram  或  paper-illustration
      (5 min)            (10-15 min)
```

- 需要**简洁流程图/时序图** → `mermaid-diagram`（快、可编辑文本）
- 需要**高质量出版级插图** → `paper-illustration`（精美、支持迭代精修）

---

## 八、关键依赖

本套件假设运行环境具备以下条件（与主项目一致）：

| 依赖 | 用途 | 安装提示 |
|------|------|---------|
| `pdflatex` / `xelatex` | 编译 Beamer / Poster LaTeX | TeX Live / MacTeX / BasicTeX |
| `latexmk` | 自动编译 | 随 TeX Live 安装 |
| `python-pptx` | 生成可编辑 PPTX | `pip install python-pptx` |
| `pymupdf` (fitz) | PDF → PNG/SVG 转换、海报视觉审查 | `pip install pymupdf` |
| `pdf2image` | PDF 图片嵌入 PPTX | `pip install pdf2image` |
| `pandoc` | Markdown → PPTX 备选 | `brew install pandoc`（可选） |

> ⚠️ **提示**：`paper-poster` 的 SKILL.md 内含详细的 TeX Live 用户目录安装方案（无需 sudo），可直接参考。

---

## 九、快速索引

| 我想做... | 使用的技能 | 查看文件 |
|-----------|-----------|---------|
| 从大纲/综述快速做幻灯片 | `presentation-slides` | [`skills/presentation-slides/SKILL.md`](skills/presentation-slides/SKILL.md) |
| 从论文做会议演讲幻灯片 | `paper-slides` | [`skills/paper-slides/SKILL.md`](skills/paper-slides/SKILL.md) |
| 从论文做会议海报 | `paper-poster` | [`skills/paper-poster/SKILL.md`](skills/paper-poster/SKILL.md) |
| 写演讲稿/讲稿 | `lecture-script` | [`skills/lecture-script/SKILL.md`](skills/lecture-script/SKILL.md) |
| 画流程图/架构图 | `mermaid-diagram` | [`skills/mermaid-diagram/SKILL.md`](skills/mermaid-diagram/SKILL.md) |
| AI 生成精美插图 | `paper-illustration` | [`skills/paper-illustration/SKILL.md`](skills/paper-illustration/SKILL.md) |
| 快速调研一个领域 | `quick-lit-review` | [`skills/quick-lit-review/SKILL.md`](skills/quick-lit-review/SKILL.md) |

---

## 十、使用示例

```bash
# 1. 进入项目目录
cd aris

# 2. 各技能通过 ARIS agent 调用，例如：
# /skill:presentation-slides "QUICK_REVIEW.md" --pages 12 --minutes 15
# /skill:paper-slides "paper/" --talk_type spotlight --venue NeurIPS
# /skill:paper-poster "paper/" --venue ICML --size A0
# /skill:lecture-script "slides/slides.pdf" --minutes 15 --language zh
# /skill:mermaid-diagram "系统架构图：包含数据流、模型训练和推理三个模块"
# /skill:quick-lit-review "Neural Radiance Fields" --depth standard
```

---

## 引用与许可

本项目是 [ARIS (Auto-Research-In-Sleep)](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep) 的衍生/适配版本。

- **上游仓库**：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
- **许可证**：[MIT License](LICENSE)

```
@software{aris2026,
  author = {wanshuiyin},
  title = {ARIS: Auto Research Intelligence System},
  url = {https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep},
  year = {2026}
}
```
