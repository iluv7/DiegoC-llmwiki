---
description: "将对话中有价值的内容归档到 wiki"
---

# Save 操作

将当前对话中有价值的内容（问答、分析、发现）归档到 wiki，使其成为持久知识。

## 执行步骤

1. **判断类型**：
   - **跨源综合分析**（综合了多个来源的比较、结论）→ `wiki/synthesis/`
   - **有价值问答**（基于 wiki 回答的问题）→ `wiki/outputs/`
2. **确定 slug**：从内容中提炼 kebab-case 文件名
3. **创建页面**：
   - 使用 synthesis 或 output 模板
   - 填写完整 frontmatter（含 tldr、sources、related）
   - 正文整理为结构化的 wiki 内容（不是聊天记录的简单复制）
4. **更新交叉引用**：
   - 在相关页面的 `related` 字段中添加新页面
   - 在 `wiki/index.md` 添加条目
5. **追加 `wiki/log.md`**：`## [YYYY-MM-DD] save | 标题`

## 质量检查

- [ ] 内容已结构化为 wiki 格式（标题、列表、表格），不是原始聊天记录
- [ ] 所有关键论断有 `[[]]` 引用
- [ ] frontmatter 完整（含 tldr）
- [ ] index.md 和 log.md 已更新
