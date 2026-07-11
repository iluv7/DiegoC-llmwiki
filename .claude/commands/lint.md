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
- [ ] **type 合法**：type 值是否为 source/entity/concept/synthesis/output 之一
- [ ] **explored 门控**：所有页面 explored 是否为 false（LLM 不应设置 true）

### 3. 内容质量
- [ ] **矛盾未标注**：检查是否有两个 source 页面提出冲突结论但未用 callout 标注
- [ ] **反方观点缺失**：concept/synthesis/source 页面是否缺少反方观点/数据缺口
- [ ] **stub 僵局**：status 为 stub 的页面是否长时间未扩充
- [ ] **过时内容**：date_modified 超过 6 个月的页面

### 4. 队列状态
- [ ] **待处理 raw**：raw/ 中尚未被 ingest 的文件
- [ ] **log.md 完整性**：ingest 记录与 wiki 页面是否一致

## 输出

1. 生成报告到 `wiki/lint-report-YYYY-MM-DD.md`
2. **低风险问题自动修复**：index.md 缺失条目、简单 typo 等
3. **高风险问题仅报告**：矛盾标注、内容过时等需要用户判断
4. 追加 `wiki/log.md`：`## [YYYY-MM-DD] lint | 检查结果摘要`
