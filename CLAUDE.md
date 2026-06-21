# CLAUDE.md — Wiki Schema & Operating Manual

> 这是 Claudian（LLM Agent）维护本 Vault 时的"操作手册"。
> 你（用户）负责采集源、提问、思考；LLM 负责所有维护工作。

---

## 1. Vault 角色定位

本 Vault 是一个**LLM 维护的个人知识库**（Personal LLM Wiki），主要服务于：
- **技术学习**（CS / AI / Graphics / Game Dev）
- **研究 / 论文阅读**

工作流：**单源精读模式（Single-Source Deep Read）**——每次只摄取一个源，深入讨论后再写入 wiki。

---

## 2. 三层架构

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: Raw Sources (原始源，不可变)                   │
│  - raw/articles/   ← 网页文章（Web Clipper）             │
│  - raw/papers/     ← PDF 论文 / 书籍章节                 │
│  - raw/notes/      ← 代码笔记、视频字幕                  │
│  - raw/assets/     ← 图片资源                            │
├─────────────────────────────────────────────────────────┤
│  Layer 2: Wiki (LLM 维护的合成层)                        │
│  - wiki/summaries/   ← 每个 raw 源对应一个摘要页         │
│  - wiki/entities/    ← 人、组织、项目、产品              │
│  - wiki/concepts/    ← 抽象概念、技术、方法              │
│  - wiki/syntheses/   ← 跨源综合、对比、分析              │
│                                                          │
│  已有的 concept 域（视为 wiki 的一部分）：               │
│  - animation/   ← 图形 / 动画                            │
│  - llm/         ← LLM / AI                               │
│  - 01 - C++/    ← C++                                    │
│  - 02 - UnrealEngine/                                    │
│  - 03 - D3D11/                                           │
│  - 04 - Math/                                            │
├─────────────────────────────────────────────────────────┤
│  Layer 3: Schema & Index                                 │
│  - CLAUDE.md    ← 本文件，操作规范                       │
│  - index.md     ← 内容索引（content-oriented）           │
│  - log.md       ← 时间日志（chronological，append-only） │
└─────────────────────────────────────────────────────────┘
```

**核心原则：**
- `raw/` **只读**——LLM 永远不修改、不删除
- `wiki/` **LLM 独占写入**——用户只阅读，不直接编辑（除非纠错）
- `index.md` & `log.md` **每次操作后由 LLM 更新**

---

## 3. 命名规范

### 文件命名
- **kebab-case**：`reinforcement-learning.md`、`attention-mechanism.md`
- **源摘要**：保持源标题简洁，如 `wiki/summaries/attention-is-all-you-need.md`
- **避免**：空格、特殊字符、过长名称

### Frontmatter（YAML 标准模板）

**Summary 页**（`wiki/summaries/`）：
```yaml
---
type: summary
source: raw/papers/attention-is-all-you-need.pdf
source_type: paper        # paper | article | note | video
title: "Attention Is All You Need"
authors: [Vaswani et al.]
year: 2017
ingested: 2026-05-25
tags: [transformer, attention, deep-learning]
related: [[concepts/self-attention]], [[concepts/transformer]]
---
```

**Concept 页**（`wiki/concepts/` 或已有领域文件夹）：
```yaml
---
type: concept
aliases: [Self-Attention, 自注意力]
tags: [deep-learning, transformer]
sources: 
  - [[summaries/attention-is-all-you-need]]
  - [[summaries/the-illustrated-transformer]]
last_updated: 2026-05-25
---
```

**Entity 页**（`wiki/entities/`）：
```yaml
---
type: entity
entity_type: person       # person | org | project | product
aliases: []
tags: []
mentioned_in: 
  - [[summaries/...]]
---
```

**Synthesis 页**（`wiki/syntheses/`）：
```yaml
---
type: synthesis
question: "What are the trade-offs between LoRA and full fine-tuning?"
created: 2026-05-25
sources: [[summaries/lora-paper]], [[summaries/qlora-paper]]
tags: [fine-tuning, comparison]
---
```

---

## 4. 操作：Ingest（摄取新源）

当用户说"**摄取这篇文章 / 论文**"或拖入一个 raw 源时：

### 步骤
1. **Read** 原始文件，理解全文要点
2. **Discuss**：用 2-3 句话向用户口头报告核心要点，问"重点关注哪些方面？"
3. **Write Summary**：在 `wiki/summaries/` 创建摘要页
   - YAML frontmatter（必填）
   - 章节：背景 / 核心论点 / 关键发现 / 方法 / 数据 / 局限 / 我的疑问
   - 用 `[[wiki-links]]` 链接到现有 concept/entity 页
4. **Update or Create Concept/Entity Pages**：
   - 找出文中提及的关键概念/实体
   - 已存在 → 更新该页（追加新视角、新数据、矛盾标记）
   - 不存在但重要 → 创建新页
5. **Update `index.md`**：在对应分类下添加新条目
6. **Append to `log.md`**：写一行 `## [YYYY-MM-DD] ingest | <title>`
7. **Report**：告诉用户：触及了哪些页面、新增了什么、是否发现矛盾

### 矛盾处理
若新源与现有页面矛盾：
- 在现有页加 `> ⚠️ **矛盾**：[[summaries/new-source]] 提出 ...` 块
- 不删除旧内容；让用户裁决

### 单源精读模式（默认）
- 一次只摄取一个源
- 摄取过程中与用户对话，征询意见
- 不要批量并行处理

---

## 5. 操作：Query（提问）

当用户提问时：

1. **先读 `index.md`** 定位相关页
2. **读 2-5 个最相关页面**（不超过，避免上下文爆炸）
3. **回答时引用来源**：`根据 [[summaries/xxx]]，... `
4. **如答案有价值** → 询问"是否归档到 `wiki/syntheses/`？"
5. **若归档**：
   - 新建 `wiki/syntheses/<question-slug>.md`
   - 更新 `index.md`
   - 追加 `log.md` 记录 `## [YYYY-MM-DD] query+archive | <question>`

### 答案格式可选
- 默认：markdown 文本
- 对比类问题：表格
- 量化分析：代码块 + matplotlib
- 框架类：mermaid 图 / 大纲

---

## 6. 操作：Lint（健康检查）

用户说"**lint wiki**"或"**健康检查**"时：

### 检查项
- [ ] 孤立页（无 inbound link）→ 列出
- [ ] 死链 `[[xxx]]` 但目标不存在 → 列出
- [ ] 概念被提及 ≥3 次但无独立页 → 建议建页
- [ ] frontmatter 缺失或不规范 → 标注
- [ ] 同义词散页（如 "Transformer" 和 "transformer-architecture"）→ 建议合并
- [ ] 标 `> ⚠️ 矛盾` 但未裁决的 → 列出
- [ ] `last_updated` 字段超过 6 个月且有新源涉及 → 建议刷新

### 输出
- 一份 markdown 报告，给用户审阅
- 用户批准后再执行修改
- 操作完追加 `## [YYYY-MM-DD] lint | <summary>` 到 `log.md`

---

## 7. Wiki Links 规范

**总是用 wiki-link 格式**：`[[wiki/concepts/self-attention]]` 或简写 `[[self-attention]]`

**优先级**：
1. 链接到 `wiki/concepts/` 或已有领域文件夹（animation/、llm/ 等）
2. 链接到 `wiki/entities/`
3. 链接到 `wiki/summaries/`（次要——避免读者陷入源细节）

**Mention 但不深入** → 用普通 markdown：`OpenAI 在 GPT-3 论文中...`

---

## 8. 图片处理

- 网页配图：用户用 Obsidian 的 "Download attachments for current file" 下载到 `raw/assets/`
- LLM 引用：`![[raw/assets/transformer-arch.png]]`
- 摘要页可嵌入关键图示
- 复杂图表 → LLM 用 mermaid 重画到 concept 页

---

## 9. 与用户的协作约定

- **永远不主动批量改文件**。每次大动作前先口头报告计划，等待确认。
- **保留用户原创内容**。若 `animation/Skinning.md` 已经是用户精心写的，LLM 只追加 `## 相关源` 章节，不动主体。
- **遇到模糊指令** → 问，不猜。
- **写作风格**：参考 `animation/Skinning.md` 的风格——概念清晰、数学严谨、有对比表、有代码示例、有"为什么"。
- **语言**：默认中文，技术名词保留英文原文（如 Self-Attention、LoRA）。

---

## 10. 工具与快捷

- **Obsidian Web Clipper**：抓网页 → `raw/articles/`
- **图谱视图（Graph View）**：可视化 wiki 拓扑，发现孤岛
- **Dataview 插件**：基于 frontmatter 跑动态查询
- **git**：vault 已是 git repo，所有变更可追溯

---

## 11. 启动检查清单（每次会话开始时 LLM 自检）

1. [ ] 读 `index.md` 了解 wiki 现状
2. [ ] 瞄一眼 `log.md` 最近 5 条，了解最近做了什么
3. [ ] 根据用户当前的 `<current_note>` 推断意图
4. [ ] 准备好响应：ingest / query / lint / 其他

---

## 12. 演化

本 schema 不是一成不变的。当工作流暴露问题或机会时，**直接和 LLM 讨论并修订本文件**。版本由 git 管理。

**当前版本**：v1.0 — 2026-05-25
**领域焦点**：技术学习（CS/AI/Graphics）+ 研究阅读
**工作流**：单源精读
