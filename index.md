---
type: index
last_updated: 2026-07-23
---

# 📚 Wiki Index

> 本文件由 LLM 自动维护。它是整个 Wiki 的目录，按类型组织。
> 操作规范见 [[CLAUDE]]；通用 Agent 规范见 [[AGENTS]]。

---

## 🗂️ 快速导航

| 区域 | 内容 | 数量 |
| :--- | :--- | :--- |
| [[#-summaries-源摘要]] | 每个 raw 源对应一篇摘要 | 0 |
| [[#-concepts-概念]] | 抽象概念、技术、方法 | - |
| [[#-entities-实体]] | 人、组织、项目、产品 | 0 |
| [[#-syntheses-综合分析]] | 跨源对比、深度回答 | 1 |
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

- [[wiki/syntheses/knowledge-iteration-system-for-zion|Zion 知识迭代系统]] — 基于当前 Vault 结构定制的 LLM 维护流程、页面分层、日常命令与演进路线。

---

## 📁 已有领域笔记

### 🎨 Animation
- [[animation/Skinning|Skinning（蒙皮）]] — glTF 蒙皮机制、骨骼动画、引擎实现

### 🖼️ Rendering（渲染）
- [[rendering/Render-Target|Render Target]] — RT、MRT、像素格式、D3D11 创建方法
- [[rendering/Shadow-Map|Shadow Map]] — shadow pass / main pass、DSV 写 depth、SRV 读 depth、`DXGI_FORMAT_R24G8_TYPELESS` 与 view format 的关系。
- [[rendering/Post-Processing-Pass|Post-Processing Pass]] — `DrawFullQuad()`、Bright Pass、Bloom blur、Tone Mapping 与 texture-to-texture pass 数据流。
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

## 🧭 Control Plane（系统规范）

- [[AGENTS|AGENTS.md]] — 通用 LLM Agent 操作手册，适配 Codex / Claudian / 其他 Agent。
- [[CLAUDE|CLAUDE.md]] — Claude Code 版本 Wiki Schema。
- [[log|log.md]] — 时间日志。

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

---

## Workflow Notes

- [[wiki/syntheses/reading-learning-mode|阅读 + 学习模式]] — 面向游戏开发学习的阅读工作流：直觉、原理、实现、常见坑、检查问题与归档规则。
- [[04 - Math/Physics.Position-Velocity-Speed-Displacement]] — Game physics movement 基础：`velocity = direction * speed`、`displacement = velocity * deltaSeconds`、`newPosition = oldPosition + displacement`。
- [[rendering/BuiltIn-Raw-Shader-And-Inline-Variable]] — engine 内置 raw shader source 的用法，以及 header 全局变量为什么要用 `inline`。
- [[03 - D3D11/D3D11.DepthBuffer]] — D3D11 depth-stencil texture 创建流程：`CreateTexture2D`、`DXGI_FORMAT_D24_UNORM_S8_UINT`、`CreateDepthStencilView`、`OMSetRenderTargets`。
- [[03 - D3D11/D3D11.D3D11_TEXTURE2D_DESC]] — D3D11 Texture2D descriptor 字段说明：尺寸、mipmap、format、bind flags、usage、MSAA 与常见创建模式。
- [[03 - D3D11/D3D11.Texture#上传图片数据到 Texture]] — `UpdateSubresource` 上传 CPU image data 到 GPU texture：`rowPitch`、mip level 0、`GenerateMips`。
- [[rendering/Shadow-Map]] — shadow map 的 D3D11 resource / DSV / SRV 关系：底层 `R24G8_TYPELESS`，写入 `D24_UNORM_S8_UINT`，读取 `R24_UNORM_X8_TYPELESS`。
- [[rendering/Post-Processing-Pass]] — post-process pass 的判断标准：处理已有 texture 用 `DrawFullQuad()`，画真实 geometry 用 mesh draw。
