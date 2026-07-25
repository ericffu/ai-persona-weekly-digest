---
name: ai-persona-digest
description: Generates role-tailored weekly AI news digests — AI industry practitioner (comprehensive), AI newbie (小白), or specific industry practitioner. Runs 8-category bilingual WebSearch as material pool, re-selects and rewrites per persona. For industry practitioners, runs extra targeted search and persists keywords in references/industries.md for reuse. Outputs Markdown plus styled HTML per persona. Use when user asks for a persona-specific AI weekly digest, e.g. "AI 周报 小白版", "AI 行业版", "教育行业 AI 周报".
---

# AI Persona Digest · 角色定制周报

同一批 AI 资讯搜索结果,按读者角色重新筛选、加权、改写语言,产出更贴近该角色需求的定制版周报。**不是重新搜 N 遍,是搜 1 遍素材池裁多份**(行业从业者角色多搜 1 轮专项内容)。

## 何时触发

- "AI 周报小白版 / 帮我生成一份 AI 行业版周报 / xx 行业的 AI 周报 / 帮 xx 行业出一份本周 AI 动态"
- 用户说"帮我定制一份周报"或"生成本周 AI 周报" → 先了解其从业背景，再在内部选定合适的内容策略

不触发:用户要的是不分角色的通用全量周报 —— 那是另一个独立 skill,不归这里管。

## 三个可选角色

一次可以多选,同时产出多份。

| 角色 | 关心什么 | 需要追问的参数 |
|---|---|---|
| **AI 行业从业者** | 本周 AI 全行业动态的全景纵览——模型/产品/投融资/交互范式/商业增长等全方位覆盖,最详尽的版本 | 无 |
| **AI 小白** | 看得懂、有代入感,"跟我有什么关系" | 无 |
| **XX 行业从业者** | 本行业相关 AI 动态 + 跨界启示 | **必须**追问具体行业名(如"教育""金融"),可选追问关注的具体公司/机构名单(不一定是竞品,合作方、监管对象、任何想跟踪的公司都算) |

每个角色的详细取舍规则、语言风格、输出模板见 [references/personas.md](references/personas.md)——生成哪个角色就读哪个角色的章节,不用整份读完。

## 跨平台决策交互

每次要让用户选择版本时,按以下规则调用平台的原生决策 UI 工具,不支持时退回纯文字:

**平台工具映射**:

| 平台 | 工具名 | 说明 |
|------|--------|------|
| WorkBuddy / Claude Code | `AskUserQuestion` | 直接调用,参数一致 |
| Cursor | `AskQuestion` | **仅 Plan Mode 有效**,普通 Agent 模式不生效 |
| Codex | `ask_user_question` 或 `request_user_input` | 参数结构略有差异,包含 `id` 字段 |

**执行策略**:
- 优先调用当前平台的决策 UI 工具。如果工具不存在或调用失败（如 Cursor 非 Plan Mode、Codex 模式不支持）,**立即退回纯文字询问**（直接输出文字让用户选择）,不要让用户看到报错或卡住
- 回退的文字版本直接说:"我可以生成下面几个版本,你选哪个？1. AI 行业从业者版(最详尽的 AI 全行业全景) 2. AI 小白版(大白话,普通人看得懂) 3. 行业从业者版(告诉我行业,做针对性内容)"

## 前置:选择周报版本与参数

第一次生成时,按「跨平台决策交互」规则弹出选择弹窗或文字询问,让用户从三个固定版本中选择,并提供"行业从业者"选项。示例配置如下(WorkBuddy / Claude Code):

```
AskUserQuestion({
  questions: [{
    question: "请选择想要生成的 AI 周报版本（可多选）",
    header: "周报版本",
    multiSelect: true,
    options: [
      { label: "AI 行业从业者版",
        description: "本周 AI 全行业动态的全景纵览——模型/产品/投融资/交互范式/商业增长等,最详尽的版本" },
      { label: "AI 小白版",
        description: "大白话说明本周变化与实际影响，看得懂、有代入感" },
      { label: "行业从业者版",
        description: "提供行业名，针对性搜一轮专项内容再出周报" }
    ]
  }]
})
```

用户选完后:
- 选了"AI 行业从业者版"→ 按 AI 行业从业者模板生成,这是最详尽的版本,全模块覆盖
- 选了"AI 小白版"→ 按小白版规则生成
- 选了"行业从业者版"→ **必须**追问具体行业名（可以再弹一个单选的 `AskUserQuestion` 或者直接文字询问）,不能自己瞎猜
- 同时选了多个版本 → 都生成,按各自的流程走

1. 若用户选择行业版:**必须**确认具体行业(不能自己瞎猜)
2. 拿到行业名后,先 Read [references/industries.md](references/industries.md) 查这个行业有没有记录过:
   - **有记录** → 直接复用里面的专项搜索关键词和关注公司名单,不用再问一遍"关注哪些公司"
   - **没有记录** → 正常追问一句"有没有特别想跟踪动态的具体公司/机构"(不局限于竞品,合作方、监管对象、上下游都算),没有就跳过公司雷达小节;这是第一次遇到这个行业,生成完之后要回来记录(见「行业专项搜索」)
3. 确定日期窗口:默认最近 7 天,用户可另外指定;`date +%F` 取今天,`date -v-7d +%F` 取 7 天前

## 素材池:自己搜

直接按下面「通用素材搜索」跑一遍 8 类 × 中英双语 WebSearch,产出内部素材池(不必写文件落盘)。

## 通用素材搜索(8 类 × 中英双语,并行发起)

这一步是给**多个角色**供料,不是只给一份通用周报供料,所以关键词要尽量铺开、宁可多搜不要漏搜——每类至少按下表的英文/中文关键词各搜一轮,能多加变体就多加,不要只挑 1-2 个词应付:

| 类别 | 英文关键词示例 | 中文关键词示例 |
|-----|--------------|-------------|
| 模型发布 | `OpenAI release <YYYY-MM>`, `Anthropic Claude release`, `Google Gemini update`, `Meta Llama`, `DeepSeek model`, `Mistral new model`, `xAI Grok`, `model benchmark comparison`, `open weight model license` | `大模型发布 <日期>`, `DeepSeek 新版本`, `通义千问 Qwen 发布`, `智谱 GLM`, `字节豆包 / 扣子 新版本`, `月之暗面 Kimi`, `模型跑分 benchmark`, `模型部署 显存` |
| AI 产品/Agent | `AI product launch this week`, `new AI agent release`, `AI coding tool launch`, `Cursor / Windsurf / Devin update`, `AI agent pricing model`, `AI app growth metrics` | `AI 产品发布 <日期>`, `AI Agent 上线`, `AI 创业公司 新品`, `AI 编辑器 发布`, `AI 产品 定价 / 商业化`, `AI 产品 增长数据` |
| 论文/研究 | `arxiv trending AI this week`, `notable AI papers <month>`, `AlphaXiv top`, `Hugging Face papers trending` | `arXiv 热门论文 AI`, `<实验室/机构> 发布研究` |
| 深度博客 | `Simon Willison blog`, `Hacker News AI frontpage`, `Stratechery AI`, `One Useful Thing Ethan Mollick`, `Interconnects` | `机器之心 本周`, `量子位 本周`, `Founder Park AI`, `远川研究所 AI` |
| X/Twitter | `viral AI twitter thread <week>`, `@sama tweet`, `@karpathy tweet`, `@miramurati tweet`, `@jimfan tweet` | `AI 小红书 热门 <日期>`, `AI 抖音 热门`, `AI 推文 热搜` |
| 行业动态 | `AI regulation <week>`, `AI policy`, `AI lawsuit`, `AI industry trends` | `AI 监管 政策`, `AI 合规`, `AI 行业趋势`, `AI 市场动态` |
| 投融资 | `AI funding round <week>`, `AI startup investment`, `AI acquisition <week>`, `AI company valuation` | `AI 融资 <日期>`, `AI 投资`, `AI 收购`, `AI 估值` |
| 开源项目 | `GitHub trending AI <week>`, `new AI open source repo`, `top AI repo this week` | `GitHub 热门 AI 项目 <日期>`, `开源大模型 新项目` |

**信源质量与验证**:优先使用一手来源——官方公告(模型发布页、产品博客)、权威媒体(科技日报、界面新闻、量子位、机器之心)、学术来源(arXiv、Hugging Face)。避免低质量二次转载站、聚合站、SEO 垃圾站和纯粹的公关稿。搜索结束后,对信息源逐一验证:每条内容必须有可追溯的原始链接,无法溯源的内容不纳入素材池。搜索结果中若发现明显为 AI 创业公司付费软文/通稿的内容,予以排除。

**收录数量不设上限**:这里是内部素材池,不是最终交付物,宽进一点没关系,交给后面每个角色各自的筛选规则去收窄。

时间过滤、去重、只保留有链接的条目——按常识执行即可。

## 行业专项搜索(仅"行业从业者"角色触发,避免重复劳动)

在通用素材池之外,**额外**为该行业跑一轮中英双语 WebSearch,不要用通用 8 类的关键词凑数。先看 [references/industries.md](references/industries.md) 里这个行业有没有已记录的关键词:

- **有记录**:直接用记录里的中英文关键词发起搜索(可以补一下日期范围),不用重新现想;如果搜索过程中发现明显更好用的新词,顺手加进去更新那一节
- **没有记录**(第一次遇到):用通用模式现想关键词,例如:
  - `<行业> AI 应用 <日期范围>` / `<industry> AI adoption case study`
  - `<行业> AI 产品 融资` / `<industry> AI startup funding`
  - `<行业> AI 政策 监管` / `<industry> AI regulation`
  - 如果用户给了具体公司名单:`<公司名> AI` 逐个查一遍最新动态
  - 搜完之后,在 references/industries.md 末尾新建一节,把这次实际用的关键词 + 用户给的公司名单记录下来(格式见 references/industries.md「记录格式」),下次同一行业就能直接跳过这一步现想

这轮搜索结果是该角色版本**"一、<行业>AI 动态"**小节的核心素材。其他通用板块（模型发布、AI 产品、行业动态、投融资、开源、深度观点全部从通用 8 类素材池出，内容为 AI 行业本身的信息，不混入行业专项搜索结果。详见 references/personas.md 模板。

## 输出文件与命名

每个角色**同时**产出 md + html 两个文件,风格必须和通用版一致——不是另起一套视觉语言:

- `artifacts/ai-persona-digest/AI周报-<角色标签>-<START>_<END>.md`
- `artifacts/ai-persona-digest/AI周报-<角色标签>-<START>_<END>.html`

角色标签:`小白` · `行业-AI` · `行业-<行业名>`(如 `行业-教育`)

多角色一起生成就写多组文件,互不覆盖。v1 不强制建索引文件,角色版本多起来后再考虑加。

## HTML 输出:复用同一套设计系统

本 skill 目录下的 [assets/template.html](assets/template.html) 是一套 Claude 官网风 HTML 模板(暖米色纸感 + Fraunces + Claude 橙 `#CC785C`),内嵌设计系统,是单文件自足的本地副本。

生成规则:

- **不改 `<style>`**:配色、字体、间距跨角色保持一致;要改样式就统一改这份 `assets/template.html`,不要每次生成时顺手改样式
- 复用 `.hero` / `.shell` / `article` / `aside.toc` / `section.category` / `.item` / `.why` / `.sources` / `footer` 这套现成组件,内容替换,DOM 结构不变
- **每个 `.item` 必须加 `id` 属性**(如 `id="item-models-01"`),供本周速览的 `hl-ref` 精确跳转
- **Hero 大字规则(关键)**:`<h1 class="hero-title">` 必须用模板的 `clamp(44px, 6.6vw, 84px)` 字号,不要写死;`AI` 和行业关键词都用 `<em>` 包裹,统一用 `var(--accent)` 强调色。详见 references/personas.md「通用 · Hero 大字规则」节。
- **分节数量按该角色的实际结构走**:小白版通常 1–2 个 `section.category`(比如"详情"一节);AI 行业从业者版覆盖全部模块,顺序为:模型发布 / AI 产品 / 论文研究 / 开源项目 / X小红书热点 / 行业动态 / 投融资 / 交互范式设计 / 商业模式增长 / 深度观点与趋势,共 10 个板块;行业从业者版按通用板块结构展开(行业专项 + 模型发布 / AI 产品 / 行业动态 / 开源 / 深度观点等,可选的关注公司动态)
- `.topnav-meta` 和 `.hero-label` 要标出这是哪个角色版本,例如 `角色版 · 小白 · <窗口>` / `AI Weekly · 从业者版`,让人一眼看出这不是通用版
- 单文件自足:所有 CSS 内嵌,只从 Google Fonts 加载字体,不引入其他外部依赖

## 输出质量要求

- **条数下限**:每个分类板块(模型发布/AI 产品/投融资/行业动态/开源/深度观点等)不少于 **4 条**;行业从业者版的行业专项板块不少于 **6–7 条**。素材池里实在不够就如实呈现,不要编造条目凑数。
- **信源质量**:优先使用一手来源——官方公告(模型发布页、产品博客)、权威媒体(科技日报、界面新闻、量子位、机器之心)、学术来源(arXiv、Hugging Face)。避免低质量二次转载站、聚合站、SEO 垃圾站。排除明显为 AI 创业公司付费软文/通稿的内容。
- **信源可追溯**:每条内容必须有可追溯的原始链接。最终输出中每条都需标注来源链接,无法溯源的内容不纳入。
- **行业解读用语**:行业从业者版的每条解读统一使用 **"对<行业>的潜在影响"** 格式,不要用"对我们业务的启示"这种第一人称表述。

## 工作流

```
Progress:
- [ ] 1. 询问用户选择 AI 行业从业者版、AI 小白版,或提供行业(行业从业者版);可多选
- [ ] 2. 若用户选择"行业从业者版",确认具体行业;Read references/industries.md 查是否已有记录,有则复用关键词+公司名单,没有才追问关注公司名单
- [ ] 3. 确定日期窗口(START/END)
- [ ] 4. 跑 8 类 × 双语 WebSearch 作为通用素材池
- [ ] 5. 若含"行业从业者":额外跑一轮行业专项 WebSearch(复用或现想关键词,见「行业专项搜索」)
- [ ] 6. Read references/personas.md 中所选内部策略的章节,按其取舍规则筛选、按其模板改写
- [ ] 7. 写入 artifacts/ai-persona-digest/AI周报-<角色标签>-<START>_<END>.md(每角色一个文件)
- [ ] 8. Read 本 skill 目录下的 assets/template.html,按「HTML 输出」一节替换内容区,写入同名 .html(锚点数量按该角色实际分节数,不是固定 8 个)
- [ ] 9. 若含"行业从业者"且这是该行业第一次记录:在 references/industries.md 末尾新建一节记录关键词与公司名单;若已有记录且这轮有更新,回来追加/修订
- [ ] 10. 一句话汇报:产出了哪几份(md+html)、每份大致条数、文件路径、(若涉及行业)references/industries.md 是否新建/更新了记录
```

## 反模式

- ❌ 每个角色重新跑一遍 8 类搜索 → ✅ 搜 1 遍素材池,裁多份(行业专项搜索例外)
- ❌ 把行业版和固定版本混成难懂的角色菜单 → ✅ 明确提供 AI 小白、AI 行业从业者版两个固定选项；行业版只需让用户说出所在行业
- ❌ "行业从业者"角色不问具体行业,自己瞎猜 → ✅ 必须先问清楚,没问清楚不要往下走
- ❌ 把角色版做成"更短的摘要" → ✅ 是"筛选 + 改写语言 + 额外一句解读",不是简单缩写原文
- ❌ 给小白版也保留参数堆砌的技术细节(如"2.8 万亿参数 MoE") → ✅ 彻底改写成感受性描述
- ❌ 编造条目凑数 → ✅ 素材池里没有就不硬凑,某角色版本条数少也可以
- ❌ 只出 md 忘了 html,或者 html 另起一套视觉风格 → ✅ md + html 两份同步,样式保持模板一致
- ❌ HTML 里照抄 8 个固定锚点 → ✅ 锚点数量跟着该角色的实际分节数走
- ❌ 同一行业每次都从零现想关键词、重新问一遍关注哪些公司 → ✅ 先查 references/industries.md,有记录就复用
- ❌ 生成完"行业从业者"版本忘了回填 references/industries.md → ✅ 第一次记录、后续有新发现都要顺手更新,这是这个机制的价值所在
