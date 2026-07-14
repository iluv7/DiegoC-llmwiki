---
description: "处理 raw/ 中的新来源，编译为 wiki 页面（含图片语义理解）"
---

# Ingest 操作

处理 `raw/` 中的新来源，将其编译为结构化的 wiki 页面。**包含图片语义理解**：扫描文章中引用的图片及 `raw/assets/` 下的独立图片，调用 Read（Vision）在 ingest 阶段完成视觉理解，生成 Image 页。

## 关键约束

- **视觉理解仅发生在 ingest 阶段**，Query 阶段禁止再次调用视觉模型
- **raw/ 完全不可变**：不修改、不移动、不重命名、不删除 raw 中的任何文件（含图片）
- 图片文件本身保留在 `raw/assets/` 中，所有语义理解结果写入 `wiki/images/`
- 处理状态由 `wiki/log.md` 追踪，不在 raw 层面做任何标记

## 执行步骤

### 阶段一：扫描

1. **查阅 `wiki/log.md`**：确认哪些 raw 文件尚未被 ingest（log 中有 ingest 记录的跳过）
2. **扫描 `raw/` 各子目录**：`articles/`、`videos/` 中查找待处理文件
3. **确定 ingest unit**：
   - 用户明确指定的文件
   - 同一主题的相关文件作为一组
   - 默认：单个文件为一个 unit（推荐 batch size 1-5）

### 阶段二：图片处理（在生成 wiki 页面之前）

> 图片处理必须在 source/concept/entity 页生成之前完成，以便这些页面引用 Image 页。

4. **扫描图片资源**：
   - 读取 raw 文章全文，提取所有图片引用（markdown `![](path)` 和 wikilink `![[path]]` 语法）
   - 扫描 `raw/assets/` 下所有图片文件（`.png`、`.jpg`、`.jpeg`、`.gif`、`.webp`、`.svg`）
   - 对比 `wiki/images/` 已有页面，找出尚未生成 Image 页的图片
   - 对比 `wiki/log.md` 中的 image ingest 记录，跳过已处理的

5. **视觉理解（调用 Read Vision）**：
   对每张未处理的图片，调用 `Read` 工具读取图片，完成视觉理解。需要产出：
   - **caption**：一句高质量摘要，包含检索关键词（禁止无意义描述，如"一张关于 XX 的图片"）
   - **visual_description**：完整视觉语义
     - 架构图：描述组件、连接关系、数据流、执行顺序
     - 流程图：描述步骤、分支、决策节点
     - 数据图表：描述变量、趋势、关键数据点
     - 截图/照片：描述场景、关键元素、上下文
   - **extracted_text**：OCR 提取的全部可见文字（如无文字则标注"无可见文字"）

6. **生成 Image 页**（`wiki/images/`）：
   为每张图片创建独立的 Markdown 页，使用 image 模板：
   - 文件名：kebab-case，建议 `{source-slug}-{image-description}.md` 或 `{image-description}.md`
   - 完整 frontmatter（`type: image`，含 `caption`、`visual_description`、`extracted_text`、`asset_path`）
   - 页面顶部嵌入原图：`![[raw/assets/xxx.png]]`
   - 填写 `sources` 字段指向对应的 source 页（如有）
   - 填写 `related` 字段指向相关 concept/entity 页
   - 追加 `wiki/log.md`：`## [YYYY-MM-DD] ingest | image:{图片文件名}`

### 阶段三：文本处理

7. **完整阅读来源**：读取 raw 文件全文（注：图片语义已在前一步获取，阅读时可关联 Image 页）

8. **与用户讨论**：分享关键发现（含图片发现），确认重点方向，询问用户想强调什么

9. **创建 source 页**（`wiki/sources/`）：
   - 使用 source 模板
   - 填写 frontmatter（含 `raw_note`、`external_url`）
   - 核心摘要 200-500 字
   - 列出可抽取实体和概念（带 `[[]]` 链接）
   - **在正文中引用 Image 页**：如 `see [[images/xxx]] for the architecture diagram`
   - 标注与现有 wiki 的连接
   - 追加 `wiki/log.md`：`## [YYYY-MM-DD] ingest | source:{标题}`

10. **创建/更新 entity 页**（`wiki/entities/`）：
    - 实体在 2+ 来源中出现 → 完整页面
    - 首次出现 → stub 页面
    - **添加「## 图解」段落**：如存在相关 Image 页，嵌入原图 `![[raw/assets/xxx.png]]` 并链接 `[[images/xxx]]`

11. **创建/更新 concept 页**（`wiki/concepts/`）：
    - 概念在 2+ 来源中出现 → 完整页面
    - 首次出现 → stub 页面
    - **添加「## 图解」段落**：如存在相关 Image 页，嵌入原图 `![[raw/assets/xxx.png]]` 并链接 `[[images/xxx]]`

### 阶段四：交叉引用与收尾

12. **维护交叉引用**：
    - 更新所有相关页面的 `sources`、`related`、`supports`、`contradicts` 字段
    - **Image 页与 concept/entity 页建立双向链接**：在 Image 页的 `related` 中填入关联的 concept/entity；在 concept/entity 的 `related` 中反向填入 Image 页
    - 确保无 dangling wikilink
    - **有方向地建链**：在旧页面里也加上新页面的链接

13. **更新 `wiki/index.md`**：添加所有新页面条目（source / entity / concept / image，链接 + tldr/caption）

14. **追加 `wiki/log.md`**：
    ```
    ## [YYYY-MM-DD] ingest | 标题
    - 处理详情
    - raw 文件：filename.md
    - 图片处理：N 张（详见 image: 记录）
    ```

## 处理 raw/assets/ 独立图片

当 `raw/assets/` 下有独立图片（不隶属于任何 raw 文章）时：

- 调用 Read（Vision）生成 Image 页，步骤同阶段二
- Image 页的 `sources` 字段可为空，`related` 指向相关 concept/entity
- 如有相关 concept/entity 页，需更新其「图解」段落和 `related` 字段
- 在 `wiki/log.md` 中记录：`## [YYYY-MM-DD] ingest | image:{文件名} (standalone)`

## 强制 re-ingest（`--force`）

当需要重新处理已有图片时，使用 `--force` 参数：
- 跳过 `wiki/log.md` 的已处理检查
- 覆盖已有 Image 页（更新 `date_modified`）
- 其他流程不变

## 质量检查

完成前自检：
- [ ] 所有新页面包含完整 frontmatter（含 tldr/caption）
- [ ] 所有 `[[]]` 链接指向已存在的页面（无 dangling）
- [ ] **每张引用图片均有对应 Image 页**（raw/article 中引用的 + raw/assets/ 中的）
- [ ] Image 页 `caption` 包含检索关键词，无无意义描述
- [ ] Image 页与 concept/entity 页已建立双向链接
- [ ] concept/entity 页的「图解」段落已添加（如适用）
- [ ] source 页面包含反方观点/数据缺口（如适用）
- [ ] index.md 和 log.md 已更新
- [ ] **raw 文件未被修改**（不移动、不改名、不改内容、不修改图片）
- [ ] **log.md 为 append-only**，未修改已有条目

## 重要约束

- raw 文件**完全不可变**，不修改、不移动、不重命名
- 处理状态由 `wiki/log.md` 的 ingest 记录追踪，不在 raw 层面做任何标记
- raw 文件按类型存放：`articles/`、`videos/`、`assets/`。新建 source 页面时 `raw_note` 字段需包含完整子目录路径（如 `[[raw/articles/filename]]`）
- **图片语义理解在 ingest 阶段一次性完成**，Query 阶段禁止调用 Read（Vision）
