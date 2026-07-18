# LLM Wiki — Schema

> 你和 LLM 一起，随时间共同演化这份 Schema。
> 灵感来源：[karpathy/llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

---

## 核心定位

**主题**：这是一个通用知识库，用于积累和关联各个领域的知识。随着使用，主题边界会自然收窄和明确。

**角色分工**：
- **你（人类）**：筛选资料、提出问题、判断方向、思考意义
- **LLM（程序员）**：摘要、交叉引用、索引维护、一致性检查。你是 wiki 维护者，不是聊天机器人。主动维护交叉引用、更新索引，答完不算完。

---

## 三层架构

```
vault/
  raw/                  # 层 1：原始资料 — LLM 只读，永不修改
    articles/           #   博客、论文、文档、Web Clipper 抓取
    videos/             #   视频、播客的文字稿
    assets/             #   图片、附件、截图
  wiki/                 # 层 2：Wiki — LLM 完全拥有，创建/更新/重构
    sources/            #   source 摘要页（每个 raw 对应一个）
    entities/           #   实体页（框架、工具、组织、人物）
    concepts/           #   概念页（方法、模式、术语）
    images/             #   图片页（图片语义理解、OCR、关联知识节点）
    synthesis/          #   综合分析页（跨源比较、阶段性结论）
    outputs/            #   问答归档页
  CLAUDE.md             # 层 3：Schema — 本文件，人类与 LLM 共同演进
  .claude/commands/     #   slash 命令定义
```

**权限规则**：
- **raw/ 完全不可变**：LLM 只读。不改 body、不改 frontmatter、不移动、不改名、不删除。raw 是真相之源，一旦允许修改，就分不清「原文说了什么」和「LLM 理解成了什么」。
- **wiki/ 由 LLM 完全拥有**：创建、更新、重构页面由 LLM 执行。
- **CLAUDE.md 由人和 LLM 共同演进**：使用中发现约定不清晰时，更新本文件。

**ingest 状态追踪**：哪些 raw 已被 ingest，由 `wiki/log.md` 追踪。**不在 raw 层面做任何标记**。

---

## 音视频素材抓取

`raw/videos/` 存放视频和音频的文字稿（字幕、转写文本），来源包括 B 站技术分享、播客、教程录屏、会议录像等。视频/音频本身不可被 LLM 直接消费，需要先转换为文本再入库。

### 抓取工具

**场景一：有字幕轨的视频 → Bilibili Obsidian Clipper**

适用于 B 站等平台已有字幕轨的视频（UP 主上传字幕或平台 AI 自动生成）。直接从播放器提取字幕文本，又快又准，无需语音识别。

- **判断方法**：播放器是否有"字幕"选项 — 有就能抓，没有就抓不了
- **工具**：[Bilibili Obsidian Clipper](https://chromewebstore.google.com/detail/bilibili-obsidian-clipper/jokophbofiphenlplmohabdcmalcbenl?hl=zh-CN)（Chrome 扩展）
- **使用流程**：打开目标视频 → 点击扩展图标 → 自动识别字幕轨 → 预览确认 → 复制 Markdown 或一键保存至 Obsidian
- **Obsidian 集成**：配合 Obsidian 的 **Local REST API** 社区插件，可实现一键保存到本地 vault 的 `raw/videos/` 目录

**场景二：无字幕的音视频 → videotranscriber.ai**

适用于播客、访谈、YouTube 教程、TikTok 视频、会议录像等没有现成字幕的任意音视频。通过语音转文字（ASR）生成文字稿。

- **工具**：[videotranscriber.ai](https://videotranscriber.ai)（在线工具）
- **支持输入**：YouTube 链接、TikTok 链接、本地上传（MP4、MP3 等常见格式）
- **配额**：每天免费 10 次
- **使用流程**：贴链接或上传文件 → 等待转写完成（几分钟到十几分钟）→ 复制文字稿
- **导出格式**：支持多种文本格式导出，也可直接复制页面上的转写结果

### 文字稿预处理

无论使用哪种工具抓取，存入 `raw/videos/` 前需做基本整理：

- **删除冗余时间戳**：保留章节标记，去掉逐行时间戳（如 `00:01:23 --> 00:01:45`）
- **保留说话人标注**：如有 speaker diarization（说话人分离），保留标签（如 `Speaker A:`、`主持人：`）
- **标注元信息**：在文件头部标注视频标题、URL、作者/UP 主、发布日期
- **格式**：统一为 Markdown（.md），文件名 kebab-case

### 入库流程

1. 将整理好的文字稿存入 `raw/videos/{topic-slug}.md`
2. 运行 `/ingest`：走标准 ingest 流程，与文章处理别无二致：
   - 创建 `wiki/sources/` 下的 source 页（`raw_note` 指向 `[[raw/videos/filename]]`，`external_url` 填写视频链接）
   - 提取实体和概念，创建/更新对应 wiki 页面
   - 更新交叉引用、index.md、log.md
3. 到这一步，`raw/` 目录下的三个子目录就都跑通了：**articles/**（文章）、**assets/**（图片）、**videos/**（音视频台词）。文本、图片、音视频三类素材的入库流程完整闭环

---

## 页面类型

### source 页（`wiki/sources/`）
- **角色**：证据节点，一个 raw 文件对应一个 source 页
- **何时创建**：每次 ingest 一个 raw 文件
- **文件名**：`{author}-{year}-{short-title}.md`，kebab-case
- **模板**：

```yaml
---
title: "原文标题"
type: source
status: complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
tldr: "一句话摘要"
raw_note: "[[raw/articles/filename]]"
external_url: ""
author: ""
published: ""
sources: []
related: []
supports: []
contradicts: []
tags: []
---
# 标题

## 核心摘要
200-500 字的结构化摘要。

## 关键实体
- [[entities/xxx]] — 一句话说明

## 关键概念
- [[concepts/xxx]] — 一句话说明

## 与现有 wiki 的连接
- 支持/补充/矛盾了哪些已有页面

## 反方观点 / 数据缺口
（如适用）
```

### entity 页（`wiki/entities/`）
- **角色**：知识节点 — 框架、工具、组织、人物
- **何时创建**：
  - 实体在 2+ 来源中出现 → 完整页面
  - 首次出现 → stub 页面（status: stub）
- **文件名**：kebab-case，如 `langchain.md`、`karpathy-andrej.md`

```yaml
---
title: "实体名称"
type: entity
status: stub | complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
tldr: "一句话说明这是什么"
sources: []
related: []
supports: []
contradicts: []
tags: []
---
# 实体名称

## 概述
一段话说明。

## 来源
在不同 source 中如何被讨论。

## 关联
- [[concepts/xxx]]
- [[entities/xxx]]
```

### concept 页（`wiki/concepts/`）
- **角色**：知识节点 — 方法、模式、术语、理论
- **何时创建**：同 entity，2+ 来源 → 完整页，首次 → stub
- **文件名**：kebab-case，如 `retrieval-augmented-generation.md`

```yaml
---
title: "概念名称"
type: concept
status: stub | complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
tldr: "一句话说明"
sources: []
related: []
supports: []
contradicts: []
tags: []
---
# 概念名称

## 定义

## 来源
在不同 source 中如何被讨论。

## 关联
- [[entities/xxx]]
- [[concepts/xxx]]
```

### synthesis 页（`wiki/synthesis/`）
- **角色**：高阶判断 — 跨源比较、阶段性结论
- **何时创建**：当查询/分析综合了多个来源
- **文件名**：kebab-case

```yaml
---
title: "主题"
type: synthesis
status: complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
tldr: "一句话结论"
sources: []
related: []
supports: []
contradicts: []
tags: []
---
# 主题

## 问题/背景

## 关键发现

## 来源对比

## 结论

## 开放问题
```

### output 页（`wiki/outputs/`）
- **角色**：有价值的问答归档
- **何时创建**：当查询答案有长期保存价值
- **文件名**：kebab-case

```yaml
---
title: "问答主题"
type: output
status: complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
tldr: "问题的一句话答案"
sources: []
related: []
tags: []
---
# 问题

## 回答
```

### image 页（`wiki/images/`）
- **角色**：知识节点 — 图片语义理解、OCR 提取、架构/流程关系
- **何时创建**：每次 ingest 时，扫描 raw 文章中引用的图片及 `raw/assets/` 下的独立图片，调用 Read（Vision）生成
- **文件名**：kebab-case，如 `apt-attack-kill-chain.png.md`
- **视觉理解仅发生在 ingest 阶段**，Query 阶段禁止调用视觉模型

```yaml
---
title: "图片标题"
type: image
status: complete
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
caption: "一句高质量摘要，包含检索关键词"
visual_description: "完整视觉语义描述（架构图需描述组件、连接关系、数据流、执行顺序；流程图描述步骤和分支）"
extracted_text: "OCR 提取的全部可见文字"
asset_path: "[[raw/assets/xxx.png]]"
sources: []
related: []
tags: []
---
# 图片标题

![[raw/assets/xxx.png]]

## 视觉语义
（完整的视觉理解描述）

## OCR 文字
（提取的可见文字，如适用）

## 关联
- [[concepts/xxx]] — 一句话说明
- [[entities/xxx]] — 一句话说明
```

**约束**：
- 禁止生成无意义描述（如"一张关于 XX 的图片"）
- `caption` 必须包含检索关键词，确保 query 阶段能被搜索命中
- `visual_description` 对架构图必须描述组件、连接关系、数据流、执行顺序；对流程图必须描述步骤和分支
- 图片文件本身保留在 `raw/assets/` 中，Image 页仅存放语义理解结果

---

## 核心原则

### 1. raw 不可变
LLM 只读 raw/，绝不修改原始资料：
- 不修改 body
- 不修改 frontmatter
- 不移动文件
- 不重命名
- 不删除
- ingest 状态由 `wiki/log.md` 追踪，不在 raw 上做标记

### 2. 双链要求
页面之间必须用 `[[wikilink]]` 互链：
- **正文里**写行内 wikilink（给人看、给 Graph View 看）
- **frontmatter 里**填 `sources`、`related`、`supports`、`contradicts` 字段（给 Dataview 查询用）
- 两边都不能偷懒

### 3. 命名规范
- 文件名用 **kebab-case**（全小写，连字符分隔）：`llm-wiki.md`、`retrieval-augmented-generation.md`
- source 页文件名带作者和年份：`{author}-{year}-{short-title}.md`
- 中文标题不塞进文件名，放在 frontmatter 的 `title` 字段
- 好处：URL 友好，grep 方便

### 4. 冲突处理
两份资料对同一件事说法不同时，**永远不要静默抹平矛盾**。用 Obsidian callout 标注：

```markdown
> [!contradiction] 来源A vs 来源B
> [[sources/a]] 声称 X，但 [[sources/b]] 声称 Y。
> 当前状态：未解决 / 已解决（说明理由）
```

把双方观点、当前状态、判断理由都写清楚。这一条是 LLM Wiki 区别于普通笔记的关键。

### 5. 反方观点 / 数据缺口
每个 source、concept、synthesis 页面如果适用，需包含反方观点或数据缺口，避免 wiki 变成回音室。

---

## index.md 与 log.md

### index.md（内容索引）
- 按页面类型分节（sources / entities / concepts / images / synthesis / outputs）
- 每条 = `- [[path/to/page]] — tldr 一句话摘要`
- LLM 每次 ingest/save 后更新
- query 时 LLM 第一步读 index，靠 tldr 快速定位相关页面
- **index 是 wiki 的承重字段，tldr 写得好不好决定检索效率**

### log.md（操作日志）
- **append-only**（只追加不修改）
- 每条格式统一：`## [YYYY-MM-DD] op | 标题`
  - `op` = ingest / query / lint / save
- 可 grep：`grep "^## \[" log.md | tail -5` 看最近 5 条
- 用途：追踪 ingest 历史，避免重复处理 raw

---

## frontmatter 字段速查

| 字段 | 说明 | 页面类型 |
|------|------|----------|
| `title` | 页面标题 | 全部 |
| `type` | source / entity / concept / image / synthesis / output | 全部 |
| `status` | complete / stub | 全部 |
| `date_created` | 创建日期 YYYY-MM-DD | 全部 |
| `date_modified` | 最后修改日期 YYYY-MM-DD | 全部 |
| `tldr` | 一句话摘要（检索用） | 全部 |
| `raw_note` | 指向 raw 文件的 wikilink | source |
| `external_url` | 原始 URL | source |
| `author` | 作者 | source |
| `published` | 发布日期 | source |
| `caption` | 一句高质量摘要，包含检索关键词 | image |
| `visual_description` | 完整视觉语义描述 | image |
| `extracted_text` | OCR 提取的全部可见文字 | image |
| `asset_path` | 指向 raw/assets/ 图片文件的 wikilink | image |
| `sources` | 引用来源的 wikilink 列表 | 全部 |
| `related` | 相关页面的 wikilink 列表 | 全部 |
| `supports` | 支持的页面 | entity/concept/synthesis |
| `contradicts` | 矛盾的页面 | entity/concept/synthesis |
| `tags` | 标签列表 | 全部 |
