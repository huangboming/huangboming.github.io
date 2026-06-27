# Git 规则

## 分支策略

**单分支工作流**。所有提交直接推送到 `main` 分支，不使用 feature 分支或 Pull Request。保持线性历史，不产生 merge commit。

## Commit Message 格式

```
<type>: <description>
```

- **语言**：英文
- **语气**：祈使句（imperative mood），如 `add` 而非 `added`、`fix` 而非 `fixed`
- **长度**：简洁，一句话说清楚做了什么

### Type（提交类型）

| Type | 用途 | 示例 |
|------|------|------|
| `blog` | 新增博客文章 | `blog: add a new post on practice` |
| `feat` | 新增功能或页面（非博客内容） | `feat: add about page`、`feat: enable comment system by giscus` |
| `fix` | 修复错误（格式、路径、缺失内容等） | `fix: fix missing texts`、`fix: update code block formatting in using-cc post to use markdown syntax` |
| `chore` | 维护性工作（规则、配置、标准化、依赖、文档） | `chore: pin ruby version to 3.4.1`、`chore: standardize article tags` |

### 选择 Type 的判断逻辑

1. **新增了博客文章？** → `blog`
2. **新增了博客以外的功能/页面？** → `feat`
3. **修复了某个错误或不当之处？** → `fix`
4. **都不是（配置、规则、标准化、重构、依赖升级等维护工作）？** → `chore`

> **历史说明**：早期曾使用 `update` 类型（如 `update: add operation section`），后来统一并入 `chore` 和 `fix`。新提交不应再使用 `update`。

## 原子性

**一个 commit 只做一件事**。

- 新增博客文章 → 单独一个 commit
- 更新规则文档 → 单独一个 commit
- 批量标准化多个文章的标签/分类 → 一个 commit 覆盖所有受影响文件
- 修复某篇文章的格式问题 → 单独一个 commit

不要在同一个 commit 中混合不同类型的变更（例如不要在一个 commit 中既新增文章又修改配置）。

## 提交粒度参考

| 变更内容 | 提交方式 |
|----------|----------|
| 新增一篇博客 | 一个 `blog:` commit |
| 新增一个功能页面 | 一个 `feat:` commit |
| 修复单篇文章的格式/内容错误 | 一个 `fix:` commit |
| 批量更新多篇文章的标签/分类 | 一个 `chore:` commit（覆盖所有文件） |
| 新增/更新规则文档 | 一个 `chore:` commit |
| 配置变更（Gemfile、\_config.yml 等） | 一个 `chore:` commit |
| 升级 Chirpy 主题版本 | 一个 `chore: upgrade to X.Y.Z` commit |

## 辅助规则

### 不在 commit 中包含的内容

- 自动生成的文件（`_site/` 已在 `.gitignore` 中）
- 临时文件、系统文件
- 敏感信息（API Key 等）

### 文件重命名

Git 会自动追踪文件重命名。如果需要重命名文件（如修正拼写错误），直接在文件系统中操作后提交即可，无需特殊命令。

示例：`2025-08-09-ai-transalation.md` → `2025-08-09-ai-translation.md`（修正 "transalation" 拼写）

### 遵守项目已有规则

对博客文章的修改（标签、分类、图片、排版等）应符合 `docs/rules/articles.md` 中定义的规则。如果规则本身需要变更，先更新规则文档，再基于新规则修改文章。
