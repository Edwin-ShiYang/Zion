---
type: index
last_updated: 2026-05-25
---

# 📚 Wiki Index

> 本文件由 LLM 自动维护。它是整个 Wiki 的目录，按类型组织。
> 操作规范见 [[CLAUDE]]。

---

## 🗂️ 快速导航

| 区域 | 内容 | 数量 |
| :--- | :--- | :--- |
| [[#-summaries-源摘要]] | 每个 raw 源对应一篇摘要 | 0 |
| [[#-concepts-概念]] | 抽象概念、技术、方法 | - |
| [[#-entities-实体]] | 人、组织、项目、产品 | 0 |
| [[#-syntheses-综合分析]] | 跨源对比、深度回答 | 0 |
| [[#-已有领域笔记]] | 用户预先积累的深度笔记 | 7+ |

---

## 📄 Summaries（源摘要）

> 位于 `wiki/summaries/`。每页对应 `raw/` 中的一个源文件。

_（暂无摘要——使用「摄取 / ingest」操作添加第一个源）_

### 按源类型
- **Articles**（网页）：暂无
- **Papers**（论文）：暂无
- **Notes**（代码/视频）：暂无

---

## 💡 Concepts（概念）

> 位于 `wiki/concepts/` 及已有领域文件夹。抽象概念、技术、方法。

### 通用概念
_（暂无——会随着源摄取自动建立）_

### 已有领域概念
- **🎨 Animation / Graphics**：见 [[#-animation]]
- **🤖 LLM / AI**：见 [[#-llm--ai]]
- **⚙️ C++**：见 `01 - C++/`
- **🎮 Unreal Engine**：见 `02 - UnrealEngine/`
- **🖼️ D3D11**：见 `03 - D3D11/`
- **📐 Math**：见 `04 - Math/`

---

## 👤 Entities（实体）

> 位于 `wiki/entities/`。人、组织、项目、产品。

_（暂无——会随着源摄取自动建立）_

---

## 🔗 Syntheses（综合分析）

> 位于 `wiki/syntheses/`。跨源对比、深度回答、个人见解。

_（暂无——通过「Query」时归档有价值的回答）_

---

## 📁 已有领域笔记

### 🎨 Animation
- [[animation/Skinning|Skinning（蒙皮）]] — glTF 蒙皮机制、骨骼动画、引擎实现

### 🖼️ Rendering（渲染）
- [[rendering/Render-Target|Render Target]] — RT、MRT、像素格式、D3D11 创建方法
- [[rendering/Deferred-Rendering|Deferred Rendering]] — Pass 概念、Pass 链、GBuffer、D3D11 完整实现
- [[rendering/Portal-Rendering|Portal Rendering]] — 传送门原理、递归渲染、斜裁剪面

### 🤖 LLM / AI
- [[llm/LLM-Wiki|LLM Wiki（总索引）]]
- [[llm/1-Fundamentals|1. LLM 基础概念]] — Transformer、Attention、Embedding
- [[llm/2-Architecture|2. 主流模型架构]] — GPT、BERT、LLaMA、Qwen
- [[llm/5-Fine-tuning|5. 微调方法论]] — LoRA、QLoRA、SFT
- [[llm/7-Inference-Optimization|7. 推理优化]] — KV-Cache、量化、PagedAttention

### ⚙️ C++ / Game Dev
- `01 - C++/` — C++ 学习笔记
- `02 - UnrealEngine/` — UE 笔记
- `03 - D3D11/` — D3D11 图形 API
- `04 - Math/` — 图形数学

### 📋 其他
- [[Cheatsheet]] — 备忘清单
- [[Perforce]] — 版本控制

---

## 📊 Wiki 健康度

| 指标 | 当前值 | 目标 |
| :--- | :--- | :--- |
| 总源数 | 0 | — |
| 总概念页 | ~10 (已有) | — |
| 死链数 | 0 | 0 |
| 孤立页 | 0 | < 10% |
| 最近 lint | 从未 | 每月一次 |

---

## 🎯 待办（LLM 建议）

- [ ] 摄取第一个源（论文 / 文章），体验完整工作流
- [ ] 把已有的 `animation/Skinning.md` 拆解出 entity（如 glTF）和 concept（如 IBM、LBS）页
- [ ] 把 `llm/` 系列拆解出 entity（如 OpenAI、Meta）和原子 concept 页
- [ ] 第一次 lint：检查现有链接完整性

---

**操作提示**：
- 想新增源 → 把文件放到 `raw/<type>/`，然后说"摄取 xxx"
- 想提问 → 直接问，会自动检索
- 想清理 → 说"lint wiki"
