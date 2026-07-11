# LLM Wiki

> 基于 [karpathy/llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 理念搭建的个人知识库脚手架 — 让 LLM 增量构建和维护一个持久化、结构化、持续积累的 wiki。

## 核心理念

**传统 RAG**：每次查询从原始文档碎片拼凑答案，查 100 次同一个主题也没有积累。  
**LLM Wiki**：LLM 在摄入时主动消化、归纳、建立交叉引用，wiki 越用越密、越用越聪明。

> "Obsidian 是 IDE，LLM 是程序员，wiki 是代码库。" — Andrej Karpathy
>
> "你和 LLM 一起，随时间共同演化。" — Karpathy

### 三层架构

| 层 | 说明 |
|---|---|
| **Raw sources** | 精选的源文档（文章、论文、图片等），LLM 只读不写 |
| **Wiki** | LLM 生成的结构化 markdown 文件（摘要、实体页、概念页、对比、索引），LLM 全权维护 |
| **Schema** | 类似 CLAUDE.md，定义 wiki 的结构、命名约定、工作流 |

## 目录结构

```
vault/
├── CLAUDE.md                  # Schema — LLM 的行为规范和 wiki 约定
├── .claude/commands/          # Slash 命令定义
│   ├── ingest.md              # /ingest — 摄入资料，编译为 wiki 页面
│   ├── query.md               # /query  — 基于 wiki 结构化回答
│   ├── lint.md                # /lint   — wiki 健康巡检
│   └── save.md                # /save   — 归档有价值的对话
├── raw/                       # 原始资料 — LLM 只读
│   ├── articles/              #   博客、论文、文档
│   ├── videos/                #   视频/播客文字稿
│   └── assets/                #   图片、附件
└── wiki/                      # 知识库 — LLM 维护
    ├── sources/               #   source 摘要页（每篇 raw 对应一个）
    ├── entities/              #   实体页（框架、工具、组织、人物）
    ├── concepts/              #   概念页（方法、模式、术语）
    ├── synthesis/             #   综合分析页（跨源比较、阶段性结论）
    ├── outputs/               #   问答归档页
    ├── index.md               #   内容索引（tldr 摘要，检索用）
    └── log.md                 #   操作日志（append-only，可 grep）
```

## 快速开始

### 前置要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup) CLI
- [Obsidian](https://obsidian.md/)（可选，用来看 wiki、Graph View）

### 1. 克隆项目

```sh
git clone <repo-url> my-vault
cd my-vault
```

### 2. 在 Obsidian 中打开（可选）

Open folder as vault → 选择 `my-vault/`

### 3. 丢入第一篇文章

把你想整理的文章放进 `raw/articles/`（支持 .md 文件）

### 4. 启动 Claude Code

```sh
cd my-vault
claude
```

### 5. 开始 ingest

在 Claude Code 对话中输入 `/ingest`，LLM 会：

- 阅读 `raw/articles/` 中的文章
- 在 `wiki/sources/` 创建 source 摘要页
- 提取实体和概念，创建/更新 entity、concept 页
- 建立页面间的交叉引用
- 更新 `index.md` 和 `log.md`

### 6. 后续

- `/query <问题>` — 基于 wiki 回答
- `/lint` — 定期给 wiki 做健康检查
- `/save` — 把有价值的对话内容归档进 wiki

## 四步命令

| 命令 | 作用 | 使用频率 |
|------|------|----------|
| `/ingest` | 摄入 raw/ 中新资料，编译为结构化 wiki 页面 | 每次有新资料 |
| `/query` | 基于 wiki 回答问题，带 `[[引用]]` 和置信度 | 随时 |
| `/lint` | 扫描断链、孤儿页、矛盾、过时内容 | 定期（每周/月） |
| `/save` | 把对话中有价值的分析/问答归档 | 有需要时 |

## 核心原则

1. **raw 不可变** — LLM 只读 raw/，绝不修改原文
2. **双链要求** — 正文 `[[wikilink]]` + frontmatter `related` 字段，双向维护
3. **kebab-case 命名** — 文件名全小写连字符
4. **冲突不抹平** — 两份资料有矛盾用 callout 标注，不假装一致
5. **一次 ingest 可触及 10-15 页** — 新知识要和旧知识接上网

## 自定义

CLAUDE.md 和 commands 都不是一次定死的。用着用着觉得约定不清晰，直接让 Claude 改：

> "帮我把 entity 和 concept 合并成一个 topics 目录，更新 CLAUDE.md 和所有相关页面"

## 参考

- [karpathy/llm-wiki.md](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — 原始想法
- [Obsidian](https://obsidian.md/) — 本地 markdown 知识库
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code/setup) — LLM CLI
