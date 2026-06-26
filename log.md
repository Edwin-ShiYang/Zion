---
type: log
created: 2026-05-25
---

# 📜 Wiki Activity Log

> 时间倒序的操作记录（最新在上）。Append-only。
> 格式约定：`## [YYYY-MM-DD] <operation> | <subject>`
>
> 操作类型：`ingest` | `query` | `query+archive` | `lint` | `refactor` | `meta`

---

## [2026-06-25] meta | Zion 知识迭代系统 v1.1

- 新增 [[AGENTS]]：面向 Codex / Claudian / 通用 LLM Agent 的 Vault 维护手册。
- 新增 [[wiki/syntheses/knowledge-iteration-system-for-zion|Zion 知识迭代系统]]：结合当前 UE、C++、D3D11、Rendering、Animation、Math 笔记结构，定义 Source Summary / Atomic Note / Domain Synthesis / Entity Page 四类知识单元。
- 更新 [[index]]：加入 Control Plane 与新的 synthesis 条目。

---

## [2026-05-25] meta | Wiki 框架初始化

- 建立三层架构（raw / wiki / index+log）
- 创建 [[CLAUDE]] schema 文件 v1.0
- 创建 [[index]] 内容索引
- 创建本日志文件
- 创建目录结构：
  - `raw/{articles,papers,notes,assets}/`
  - `wiki/{summaries,entities,concepts,syntheses}/`
- 已有内容纳入体系：
  - [[animation/Skinning]]（用户预先编写）
  - [[llm/LLM-Wiki]] 及 4 个子章节（LLM 协助生成）
  - `01 - C++/`、`02 - UnrealEngine/`、`03 - D3D11/`、`04 - Math/`（保持原状）
- 工作流确认：**单源精读模式**
- 领域焦点：技术学习（CS/AI/Graphics）+ 研究 / 论文阅读
- 新增模板文件：
  - `raw/README.md`、`wiki/README.md`
  - `wiki/summaries/_TEMPLATE.md`
  - `wiki/concepts/_TEMPLATE.md`
  - `wiki/entities/_TEMPLATE.md`
  - `wiki/syntheses/_TEMPLATE.md`

---

## [2026-05-25] query+archive | Render Target / Deferred Rendering / Portal Rendering

对话式学习，归档三个概念页到新建的 `rendering/` 领域：
- [[rendering/Render-Target]] — RT、MRT、D3D11 API
- [[rendering/Deferred-Rendering]] — Pass 概念 + Pass 链 + GBuffer + 完整 D3D11 代码
- [[rendering/Portal-Rendering]] — 传送门渲染原理

新增领域文件夹：`rendering/`（与 `animation/`、`llm/` 平级）
更新：[[index]]

## [2026-05-26] query+archive | D3D11 Shader Pipeline 基础

对话式学习，归档进已有的 `03 - D3D11/` 体系：
- **丰富** [[03 - D3D11/D3D11.InputLayout]] — `D3D11_INPUT_ELEMENT_DESC` 详解、`APPEND_ALIGNED_ELEMENT`、Semantic 对接机制
- **丰富** [[03 - D3D11/D3D11.ConstantBuffer]] — `register(bN)` 槽、内存布局匹配、16字节对齐规则
- **丰富** [[03 - D3D11/D3D11.Texture]] — SamplerState、Filter 类型、AddressMode、与 RT 的关系
- **新建** [[03 - D3D11/D3D11.HLSL-FX-Structure]] — FX 文件结构、数据流全景、UV 翻转原因

<!-- 新条目追加在此分隔线下方 ↓ -->
