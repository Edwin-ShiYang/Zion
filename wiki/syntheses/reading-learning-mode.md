---
type: workflow
created: 2026-07-03
last_updated: 2026-07-03
tags: [learning, reading, game-dev, workflow]
sources: []
---

# 阅读 + 学习模式

这个模式用于：用户正在读一本书、一篇文章、一段课堂字幕、代码文档或技术材料时，让 Agent 不只是总结，而是像游戏开发导师一样带着用户学会。

## 触发方式

用户可以这样说：

- “进入阅读 + 学习模式”
- “我在读 `[[raw/books/xxx]]`，带我学”
- “解释我选中的这段”
- “把这章学完并归档”

如果用户给了 `<current_note>` 或 `<editor_selection>`，优先使用当前 note / selection。

## 工作目标

不是单纯压缩内容，而是完成四件事：

1. **读懂**：这段材料在讲什么。
2. **讲会**：用直觉、例子、图形/游戏开发语境解释。
3. **转化**：把内容转成可实现的 engine / gameplay / rendering / math 知识。
4. **沉淀**：把有长期价值的内容归档成 Obsidian 笔记。

## 标准回答结构

默认按这个顺序讲：

1. **一句话直觉**：先说这东西到底解决什么问题。
2. **核心概念**：列出必须懂的 2-5 个概念。
3. **逐段解释**：如果用户给了原文/选区，就跟着原文解释。
4. **游戏开发例子**：尽量用 engine、combat、animation、AI、physics、rendering、shader、math 例子。
5. **实现视角**：如果能落到代码/架构，就说明应该放在哪个 class / system / function。
6. **常见坑**：指出最容易误解的地方。
7. **检查问题**：最后问 2-4 个小问题，确认用户是否真的理解。
8. **是否归档**：如果内容有长期价值，建议归档到对应领域。

## 归档规则

阅读材料本身仍放在 `raw/`，默认不改。

归档位置按内容决定：

- C++ / STL / 编译：`01 - C++/`
- Unreal API / UE 概念：`02 - UnrealEngine/`
- D3D11 / HLSL / graphics API：`03 - D3D11/`
- 数学 / 物理 / collision：`04 - Math/`
- animation / skinning / root motion：`animation/`
- rendering / shader / lighting：`rendering/`
- 跨领域学习总结：`wiki/syntheses/`
- 单个 source 摘要：`wiki/summaries/`

## 学习深度档位

用户没指定时，默认用 **Level 2**。

### Level 1 — 快速理解

- 讲清楚大意。
- 少代码。
- 适合快速扫书/文章。

### Level 2 — 学会并能用

- 讲直觉 + 原理 + 实现。
- 给最小代码/伪代码。
- 适合游戏开发学习。

### Level 3 — 深挖到可实现

- 推导公式或架构。
- 指出数据结构、调用顺序、边界条件。
- 适合 thesis、engine feature、shader、physics、AI。

## 对用户的默认假设

- 用户最终目标是做出东西，不只是写读书笔记。
- 所以解释要尽量落到：如何实现、怎么调试、放在 engine 哪一层、和已有系统怎么连接。
- 如果材料来自 UE / Unity / Godot，可以参考它的思想，但优先抽象成 engine-agnostic 的做法。

## 常用输出模板

```md
## 一句话直觉

## 这段在解决什么问题

## 核心概念

## 逐步解释

## 游戏开发里的例子

## 如果你要自己实现

## 常见坑

## 检查你是否懂了

## 可归档内容
```
