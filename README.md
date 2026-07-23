# AI Persona Weekly Digest

用于打磨、测试和沉淀「按读者角色定制 AI 周报」Codex skill 的工作区。

## 目录

- `ai-persona-digest/`：可安装的 skill；包含执行说明、角色/行业参考资料和 HTML 模板。
- `artifacts/`：每次生成周报的 Markdown、HTML 等制品；按 skill 名归档。

## 当前 skill

`ai-persona-digest` 会基于同一批 AI 周度资讯，为 AI 小白、产品经理、技术研发或特定行业从业者生成相应版本的周报。每个版本同时输出 Markdown 源稿和独立 HTML 页面。

可直接使用以下提示开始：

```text
Use $ai-persona-digest to create this week’s AI digest for product managers.
```

## 产物约定

角色版周报输出至 `artifacts/ai-persona-digest/`。行业从业者场景会将可复用的搜索关键词和关注机构记录在 skill 的 `references/industries.md`。
