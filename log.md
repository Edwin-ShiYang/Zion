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

## [2026-06-28] query+archive | Z-Aligned Capsule Overlap

- 归档 collision 学习笔记：[[04 - Math/Collision.Z-Aligned-Capsule-Overlap]]
- 主题：Unreal-style capsule half height、bone start/end、Z-aligned capsule overlap、kissing 不算 overlap。

---

## [2026-06-27] query+archive | Thesis reference scout

- 归档今日 build-first thesis reference scout：[[thesis/reference-scout/2026-06-27]]
- 主题：Animation-Driven Third Person Melee Combat。
- 重点：UE Root Motion、Animation Notifies、Traces、Behavior Tree、fighting game AI、game feel、Soulslike challenge。

---

## [2026-06-25] query+archive | Thesis reference scout

- 归档今日 thesis reference scout：[[thesis/reference-scout/2026-06-25]]
- 主题：Animation-Driven Third Person Melee Combat。
- 覆盖方向：game feel、impact feel、motion matching、behavior trees、game AI、utility AI。

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

## [2026-07-03] refine | 阅读 + 学习模式

- 新增 [[wiki/syntheses/reading-learning-mode]]：定义阅读 + 学习模式的触发方式、讲解结构、学习深度档位与归档规则。
- 更新 [[AGENTS]]：加入 `3.5 阅读 + 学习模式`，作为之后阅读书籍、文章、课堂字幕、代码文档时的默认工作流。
- 更新 [[index]]：登记该综合工作流页面。

## [2026-07-04] query+archive | Position / Velocity / Speed / Displacement

- 归档阅读 [[raw/books/Game Physics Engine Development.pdf]] 时整理出的 movement / physics integration 基础。
- 新增 [[04 - Math/Physics.Position-Velocity-Speed-Displacement]]。
- 核心三行：`velocity = direction * speed`、`displacement = velocity * deltaSeconds`、`newPosition = oldPosition + displacement`。
- 更新 [[index]]。

## [2026-07-14] query+archive | Built-in raw shader 与 inline 变量

- 新增 [[rendering/BuiltIn-Raw-Shader-And-Inline-Variable]]。
- 主题：Tone Mapping 这类 engine-level post-process shader 可以作为 built-in raw shader source；header 中定义全局 shader source 时应使用 `inline` 避免 multiple definition。
- 更新 [[index]]。

## [2026-07-23] query+archive | D3D11 Depth Buffer / Depth Stencil Texture

- 整理代码片段中 `D3D11_TEXTURE2D_DESC` 创建 depth-stencil texture 的含义。
- 丰富 [[03 - D3D11/D3D11.DepthBuffer]]：补充 `DXGI_FORMAT_D24_UNORM_S8_UINT`、`D3D11_BIND_DEPTH_STENCIL`、Texture 与 DepthStencilView 的关系、`OMSetRenderTargets` 绑定流程和常见坑。
- 更新 [[index]]。

## [2026-07-23] refine | D3D11 descriptor 清零

- 更新 [[03 - D3D11/D3D11.DepthBuffer]]：补充 `D3D11_TEXTURE2D_DESC depthTextureDesc = {};` 为什么要先清零。
- 重点：避免未显式填写的 `CPUAccessFlags`、`MiscFlags`、`SampleDesc.Quality` 等字段保留随机垃圾值，导致 `CreateTexture2D` 参数组合异常。

## [2026-07-23] refine | D3D11 DepthBuffer 字段解释格式

- 更新 [[03 - D3D11/D3D11.DepthBuffer]]：将 `D3D11_TEXTURE2D_DESC` 字段解释从表格改为带注释的 C++ code block，方便直接对照代码阅读。

## [2026-07-23] refine | D3D11 code block 单行调用格式

- 更新 [[03 - D3D11/D3D11.DepthBuffer]]：将 code block 中的多行函数调用改为单行调用，例如 `CreateDepthStencilView(...)`、`OMSetRenderTargets(...)`、`ClearDepthStencilView(...)`。
- 记录偏好：整理代码笔记时，函数调用参数不要拆成多行，优先写在一行里方便快速阅读。

## [2026-07-23] refine | D24_UNORM_S8_UINT 像素布局

- 更新 [[03 - D3D11/D3D11.DepthBuffer]]：补充 `DXGI_FORMAT_D24_UNORM_S8_UINT` 的每像素布局：32 bits / 4 bytes，其中 24-bit UNORM 用于 depth test，8-bit UINT 用于 stencil test。
- 补充 depth 与 stencil 的使用时机：depth 解决前后遮挡，stencil 作为整数 mask 控制绘制区域。

## [2026-07-23] refine | R16G16B16A16_FLOAT 与 HDR

- 更新 [[rendering/Render-Target]]：补充 `DXGI_FORMAT_R16G16B16A16_FLOAT` 的含义：RGBA 四通道、每通道 16-bit float、每像素 64 bits / 8 bytes。
- 重点：`FLOAT` 不像 `UNORM` 一样限制在 `0.0 ~ 1.0`，因此可以保存超过 1 的 HDR 光照值，后续再通过 tone mapping 压回屏幕显示范围。

## [2026-07-23] refine | Code comment language preference

- 更新 [[03 - D3D11/D3D11.DepthBuffer]] 和 [[rendering/Render-Target]]：将 code block 里的 `//` 注释统一改为英文。
- 记录偏好：以后代码块中的 comment 使用英文；正文解释可以继续使用中文。

## [2026-07-23] query+archive | D3D11_TEXTURE2D_DESC

- 新增 [[03 - D3D11/D3D11.D3D11_TEXTURE2D_DESC]]：单独整理 `D3D11_TEXTURE2D_DESC` 的作用、清零原因、字段解释、`MipLevels`、`Format`、`BindFlags`、depth buffer 示例和 render target 示例。
- 更新 [[03 - D3D11/D3D11.DepthBuffer]]：把通用 descriptor 说明链接到新笔记。
- 更新 [[index]]。
