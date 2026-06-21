# 🧠 Wiki (LLM-Maintained Synthesis Layer)

> **这里是 LLM 的写作领地。用户读，LLM 写。**

## 目录结构

| 子目录 | 内容 | 触发创建 |
| :--- | :--- | :--- |
| `summaries/` | 源摘要页（1:1 对应 `raw/`） | 摄取源时自动 |
| `concepts/` | 抽象概念、技术、方法 | 摄取过程中识别到 |
| `entities/` | 人、组织、项目、产品 | 摄取过程中识别到 |
| `syntheses/` | 跨源综合分析、深度回答 | 用户提问后归档 |

---

## 已有领域笔记的关系

以下文件夹**不在 `wiki/` 内**，但被视为 wiki 的概念域延伸：

- `animation/` — 图形 / 动画
- `llm/` — LLM / AI
- `01 - C++/`、`02 - UnrealEngine/`、`03 - D3D11/`、`04 - Math/` — 已有的技术学习区

LLM 在创建新 concept 页时，若主题归属于这些已有领域，**优先放入对应文件夹**，而非 `wiki/concepts/`。例如：
- "RoPE（Rotary Position Embedding）" → 放入 `llm/`
- "Quaternion Slerp" → 放入 `04 - Math/`
- "Inverse Bind Matrix" → 与 `animation/Skinning.md` 关联或新建 `animation/IBM.md`

---

## 写作风格指南

参考 `animation/Skinning.md` 的风格：
- ✅ 概念清晰、由浅入深
- ✅ 数学公式严谨
- ✅ 对比表 + 代码示例
- ✅ 回答"为什么"，不只是"是什么"
- ✅ 中文为主，技术名词保留英文

---

## 页面模板

详见 [[CLAUDE]] 第 3 节"命名规范"中的 frontmatter 模板。
