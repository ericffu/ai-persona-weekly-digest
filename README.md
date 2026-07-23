# AI Persona Weekly Digest

一个为不同读者角色生成 AI 周报的 Codex skill。它先建立一份共用的 AI 资讯素材池，再按角色筛选和改写，避免为每个版本重复搜索。

支持四类读者：

- AI 小白：用大白话说明本周变化与实际影响。
- 产品经理：提炼交互、商业化、增长和竞品启示。
- 技术研发：关注架构、基准、部署门槛和可复现性。
- 行业从业者：增加行业专项搜索，并给出业务启示。

每个版本都会产出 Markdown 源稿和独立 HTML 周报页。

## 安装到 Codex

将下面整段内容复制给具备终端权限的 Agent，即可安装或更新此 skill：

```text
请将 https://github.com/ericffu/ai-persona-weekly-digest 安装为 Codex skill，名称为 ai-persona-digest。

目标目录是 ${CODEX_HOME:-$HOME/.codex}/skills/ai-persona-digest：如果目录不存在，请 git clone 该仓库到这个目录；如果已经存在，请进入该目录并拉取最新 main 分支。完成后确认 SKILL.md 位于目标目录根部，并告诉我如何调用它。
```

安装完成后，可直接这样使用：

```text
Use $ai-persona-digest to create this week’s AI digest for product managers.
```

## 生成内容

生成周报时，skill 会根据用户选择的角色确定日期范围、复用或搜索资料，并按角色规则输出内容。行业版会记住已使用的行业搜索关键词和关注公司，供后续周报复用。

详细流程与输出规则见 [SKILL.md](SKILL.md)。
