---
description: "wiki 健康检查：查找孤儿页、悬空链接、缺失字段等问题"
---

# Lint 操作

对整个 wiki 执行健康检查，发现问题并生成报告。

## 检查项

### 1. 结构完整性
- [ ] **Dangling wikilinks**：扫描所有 `[[]]` 链接，检查目标页面是否存在
- [ ] **孤儿页面**：没有任何入站链接的页面（index.md、log.md 除外）
- [ ] **index.md 同步**：检查 wiki/ 下所有 .md 文件是否都在 index.md 中列出

### 2. Frontmatter 合规
- [ ] **必填字段**：每个页面是否有 title、tldr、type、status、date_created、date_modified
- [ ] **type 合法**：type 值是否为 source/entity/concept/image/synthesis/output 之一
- [ ] **explored 门控**：所有页面 explored 是否为 false（LLM 不应设置 true）
- [ ] **Image 页专用字段**：
  - `caption` 是否存在且非空
  - `caption` 是否包含检索关键词（不应为"一张关于 XX 的图片"等无意义描述）
  - `visual_description` 是否存在且包含实质性内容
  - `extracted_text` 是否存在
  - `asset_path` 是否存在且指向 `raw/assets/` 下的文件
  - `asset_path` 指向的图片文件是否存在

### 3. 内容质量
- [ ] **矛盾未标注**：检查是否有两个 source 页面提出冲突结论但未用 callout 标注
- [ ] **反方观点缺失**：concept/synthesis/source 页面是否缺少反方观点/数据缺口
- [ ] **stub 僵局**：status 为 stub 的页面是否长时间未扩充
- [ ] **过时内容**：date_modified 超过 6 个月的页面
- [ ] **Image 页质量**：
  - `visual_description` 内容是否具体（架构图是否描述了组件/连接/数据流，流程图是否描述了步骤/分支）
  - `extracted_text` 是否有内容（如原图有文字但标注"无可见文字"，标记为可疑）
  - 是否存在 `wiki/images/` 中内容相似的重复页面

### 4. 图片资源同步
- [ ] **缺失 Image 页**：`raw/assets/` 中的图片文件是否有对应的 `wiki/images/` 页面
- [ ] **冗余 Image 页**：`wiki/images/` 中的页面是否指向 `raw/assets/` 中已删除的文件
- [ ] **孤立图片**：`raw/assets/` 中的图片是否未被任何 wiki 页面引用
- [ ] **Image 页关联完整性**：Image 页的 `related` 字段是否为空（如为空，考虑是否需要建立关联）
- [ ] **concept/entity 图解遗漏**：concept/entity 页是否缺少对相关 Image 页的引用和嵌入

### 5. 队列状态
- [ ] **待处理 raw**：raw/ 中尚未被 ingest 的文件
- [ ] **待处理图片**：raw/assets/ 中尚未生成 Image 页的图片
- [ ] **log.md 完整性**：ingest 记录与 wiki 页面是否一致（含 image ingest 记录）

## 输出

1. 生成报告到 `wiki/lint-report-YYYY-MM-DD.md`
2. **低风险问题自动修复**：index.md 缺失条目、简单的 Image 页字段补充、简单 typo 等
3. **中风险问题建议修复**：缺失 Image 页（可建议运行 ingest --force 补建）、concept/entity 缺少图解段落
4. **高风险问题仅报告**：矛盾标注、内容过时、Image 页语义描述质量差等需要用户判断
5. 追加 `wiki/log.md`：`## [YYYY-MM-DD] lint | 检查结果摘要`

## 注意事项

- **不要修改 raw/**：包括 raw/assets/ 中的图片，lint 只检查不修改
- Image 页的 `asset_path` 如指向不存在的文件，标记为高优先级问题
- 如果某张图片的语义理解质量差（如描述模糊），建议用 `ingest --force` 重新处理该图片
