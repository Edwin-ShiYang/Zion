# 📥 Raw Sources

> **这是源材料层 —— LLM 只读，不修改。**

## 目录结构

| 子目录 | 用途 | 文件类型 |
| :--- | :--- | :--- |
| `articles/` | 网页文章 | `.md`（Web Clipper 抓取） |
| `papers/` | 论文 / 书籍 | `.pdf`、`.md` |
| `notes/` | 代码 / 视频笔记 | `.md`、`.txt` |
| `assets/` | 图片、附件 | `.png`、`.jpg`、`.svg` |

---

## 命名建议

- 论文：`<作者>_<年份>_<短标题>.pdf`  例：`vaswani_2017_attention.pdf`
- 文章：`<来源>_<标题>.md`  例：`karpathy_intro-to-transformers.md`
- 视频：`<平台>_<标题>.md`  例：`youtube_3blue1brown_neural-nets.md`

---

## 工作流

1. **采集源**：
   - 网页 → Obsidian Web Clipper → 自动落入 `raw/articles/`
   - 论文 → 手动放入 `raw/papers/`
   - 图片 → 用 Obsidian "Download attachments" 落入 `raw/assets/`

2. **通知 LLM**：
   - 在对话中说：「摄取 `raw/papers/xxx.pdf`」
   - 或：「我刚加了一篇新文章，请处理」

3. **LLM 自动**：
   - 阅读、讨论、写摘要到 `wiki/summaries/`
   - 更新 concept/entity 页
   - 更新 `index.md` 和 `log.md`

---

## ⚠️ 禁忌

- **不要直接编辑 `raw/` 中的文件**（保留原始形态，便于追溯）
- **不要把 LLM 生成的内容放这里**（属于 `wiki/`）
- **不要删除已摄取的源**（除非确认 wiki 中无引用）
