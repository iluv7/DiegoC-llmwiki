---
description: "处理 raw/ 中的新来源，编译为 wiki 页面"
---

# Ingest 操作

处理 `raw/` 中的新来源，将其编译为结构化的 wiki 页面。

## 执行步骤

1. **查阅 `wiki/log.md`**：确认哪些 raw 文件尚未被 ingest（log 中有 ingest 记录的跳过）
2. **扫描 `raw/` 各子目录**：`articles/`、`videos/` 中查找待处理文件
3. **确定 ingest unit**：
   - 用户明确指定的文件
   - 同一主题的相关文件作为一组
   - 默认：单个文件为一个 unit（推荐 batch size 1-5）
4. **完整阅读来源**：读取 raw 文件全文
5. **与用户讨论**：分享关键发现，确认重点方向，询问用户想强调什么
6. **创建 source 页**（`wiki/sources/`）：
   - 使用 source 模板
   - 填写 frontmatter（含 `raw_note`、`external_url`）
   - 核心摘要 200-500 字
   - 列出可抽取实体和概念（带 `[[]]` 链接）
   - 标注与现有 wiki 的连接
7. **创建/更新 entity 页**（`wiki/entities/`）：
   - 实体在 2+ 来源中出现 → 完整页面
   - 首次出现 → stub 页面
8. **创建/更新 concept 页**（`wiki/concepts/`）：
   - 概念在 2+ 来源中出现 → 完整页面
   - 首次出现 → stub 页面
9. **维护交叉引用**：
   - 更新所有相关页面的 `sources`、`related` 字段
   - 确保无 dangling wikilink
   - 有方向地建链：在旧页面里也加上新页面的链接
10. **更新 `wiki/index.md`**：添加新页面条目（链接 + tldr）
11. **追加 `wiki/log.md`**：
    ```
    ## [YYYY-MM-DD] ingest | 标题
    - 处理详情
    - raw 文件：filename.md
    ```

## 质量检查

完成前自检：
- [ ] 所有新页面包含完整 frontmatter（含 tldr）
- [ ] 所有 `[[]]` 链接指向已存在的页面（无 dangling）
- [ ] source 页面包含反方观点/数据缺口（如适用）
- [ ] index.md 和 log.md 已更新
- [ ] **raw 文件未被修改**（不移动、不改名、不改内容）

## 重要约束

- raw 文件**完全不可变**，不修改、不移动、不重命名
- 处理状态由 `wiki/log.md` 的 ingest 记录追踪，不在 raw 层面做任何标记
- raw 文件按类型存放：`articles/`、`videos/`、`assets/`。新建 source 页面时 `raw_note` 字段需包含完整子目录路径（如 `[[raw/articles/filename]]`）
