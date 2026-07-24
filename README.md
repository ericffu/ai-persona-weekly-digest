# AI Persona Weekly Digest

一个按读者角色定制 AI 周报的 Codex skill。同一批 8 类 AI 资讯素材池，按角色筛选、加权、改写，一次搜索产出多份定制版周报。

![周报预览](assets/screenshot.png)

## 支持的角色

| 角色 | 定位 | 输出风格 |
|------|------|---------|
| **产品经理** | 提炼交互设计、商业化打法、增长启示 | 3 个主题分节 |
| **技术研发** | 关注架构、基准、部署门槛 | 模型/论文/开源分节 |
| **AI 小白** | 大白话说明本周变化，"跟我有什么关系" | 简单口语化结构 |
| **行业从业者** | 聚焦具体行业，叠加专项搜索 | 行业动态 + 通用板块（模型/AI 产品/投融资等） |

每个版本同时产出 Markdown 源稿和独立 HTML 页面，支持亮暗主题切换，移动端粘性导览。

## 设计原则

- **搜一遍，裁多份**：8 类 × 中英双语 WebSearch 一次完成，角色定制不重复搜索
- **行业记忆**：`references/industries.md` 记录每个行业的关键词和关注公司，下次直接复用
- **信源可追溯**：优先一手来源，每条标注原始链接，排除公关稿和 SEO 垃圾站
- **质量下限**：每个通用板块 ≥4 条，行业专项 ≥6-7 条

## 安装到 Codex

```text
请将 https://github.com/ericffu/ai-persona-weekly-digest 安装为 Codex skill，名称为 ai-persona-digest。

目标目录是 ${CODEX_HOME:-$HOME/.codex}/skills/ai-persona-digest：如果目录不存在，请 git clone 该仓库到这个目录；如果已经存在，请进入该目录并拉取最新 main 分支。完成后确认 SKILL.md 位于目标目录根部，并告诉我如何调用它。
```

使用：

```text
Use $ai-persona-digest to create this week's AI digest for product managers.
```

详细流程与输出规则见 [SKILL.md](SKILL.md)。
