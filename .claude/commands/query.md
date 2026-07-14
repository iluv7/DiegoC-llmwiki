---
description: "基于 wiki 内容回答问题，附带引用"
---

# Query 操作

基于已有的 wiki 内容回答用户的问题。优先从编译好的 wiki（含 Image 页）综合答案，而非从 raw 重新检索。**支持图文并茂的回答**：当问题命中 Image 页的语义内容时，自动嵌入图片。

## 关键约束

- **禁止在 Query 阶段调用 Read（Vision）分析图片**：图片语义理解已在 ingest 阶段完成，Query 阶段仅使用 `wiki/images/` 中已编译的 Image 页进行检索和推理
- **图片检索依赖 Image 页的 frontmatter 和正文文本**：`caption`、`visual_description`、`extracted_text` 字段提供可检索的文本
- **永远优先从 wiki 回答**，而不是从 raw 重新检索

## 执行步骤

1. **读取 `index.md`**：通过 tldr/caption 快速扫描所有页面（含 `wiki/images/`），定位与问题相关的页面
2. **拉取相关页面**：读取相关 wiki 页面的完整内容，包括：
   - concept/entity 页（概念定义、关联知识）
   - **Image 页**（图片语义描述、OCR 文字）
   - source 页（原始资料摘要）
3. **检查是否需要回溯 raw**：
   - 如果 wiki 信息不足以回答，查看相关 source 页面的 `raw_note`，回溯阅读原始资料
   - 如果 wiki 中存在矛盾标注，需要展示双方观点
   - **不要回溯 raw 图片**：如需更多图片信息，应从 Image 页获取，而非重新分析原图
4. **判断是否需要嵌入图片**：
   - 用户明确询问图表/架构/流程/思维导图 → 必须嵌入相关图片
   - 图片能显著增强回答的可理解性 → 嵌入图片
   - 问题与 Image 页的 `visual_description` 或 `extracted_text` 高度匹配 → 嵌入图片
   - 纯概念定义/简单问答 → 可仅引用 Image 页链接，不嵌入
5. **图文并茂的综合回答**：
   - 每个关键论断**必须引用 `[[]]` 页面**作为出处
   - 标注置信度（high/medium/low）
   - **嵌入图片格式**：
     ```markdown
     ![[raw/assets/xxx.png]]

     > 图：{caption 内容}

     来源：[[images/xxx]]
     ```
   - 如果存在矛盾，明确展示
6. **建议归档**：
   - 如果这个问答有长期价值，建议用户用 `/save` 归档
   - 如果发现 wiki 中缺少某个应该有页面的概念/实体/图片，建议下次 ingest 时补充
   - 如果 Image 页的描述不足以回答问题（如缺少细节），标注为 low 置信度并建议重新 ingest 该图片

## 注意事项

- **永远优先从 wiki 回答**，而不是从 raw 重新检索
- **图片内容从 Image 页获取，不从 raw 图片重新分析**
- 如果 wiki 完全没有覆盖这个问题，明确告知用户，建议添加相关来源
- 如果 wiki 信息过时（status: stub 且 date_modified 较久），提醒用户可能需要更新
- 如果 Image 页的 `visual_description` 缺乏用户所需的细节（如"APT 杀伤链第三步的具体操作"），标注为 low 置信度并给出已有信息
