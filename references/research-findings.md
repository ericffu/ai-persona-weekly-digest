# 同类 AI 新闻摘要 Skill 调研报告

## 一、找到的同类 / 相似项目

### 1. `agents-radar`（GitHub: akula/agents-radar）
- 以 GitHub Actions 驱动，每日自动追踪 Claude Code、Codex、Gemini CLI、OpenClaw 生态及 GitHub AI 趋势
- 产出多级粒度：日报（daily digest）→ 周报（weekly digest）→ 月报（monthly digest）
- 中英双语，结构高度结构化（按领域分：Models & Research / Tools & Engineering / Industry / Opinion）
- **差异点**：纯自动化 GitHub Actions 跑，不是 SKILL.md，没有角色定制

### 2. `ai-weekly-digest`（GitHub: Ruofei-Li/ai-weekly-digest）
- Python 代码项目，非 SKILL.md 格式
- 多源采集（Hacker News + arXiv + RSS + NewsAPI），Claude API 分类摘要
- 定时发邮件，不是 Agent skill
- **差异点**：自动化发信，无交互定制

### 3. `Weekly Digests`（claudskills.com）
- 通用 weekly digest skill，但描述过于宽泛
- 只有 SKILL.md 框架，没有具体业务逻辑
- **差异点**：太抽象，没法直接用

### 4. `openmark-weekly-digest`（skills.lc）
- 专注于用户自己的书签/收藏的周回顾
- 非常具体——从 OpenMark 的个人书签数据中聚类生成 digest
- **差异点**：不搜外部新闻，只处理个人数据

---

## 二、高赞 Best Practices 对我们的启发

### 从 anthropic/docs 和 mdskills.ai 学到的：

| 最佳实践 | 我们当前状态 | 改进方向 |
|---------|------------|---------|
| **SKILL.md 是路由器，不是百科全书** | 我们直接把全文业务逻辑写在 SKILL.md 里 | ✅ 已经在做（personas/references 拆分） |
| **description 是最重要的字段** | 我们的 description 比较长 | 可以考虑精简触发词，提高匹配准确率 |
| **渐近式披露（progressive disclosure）** | 我们用了 references/ 拆分 | ✅ 做得不错 |
| **500 行以内** | SKILL.md 现在约 170 行 | ✅ OK，有扩展空间 |
| **前端 YAML metadata 要精简** | 1 行太长了 | ❌ 可优化，拆成多行 |
| **使用第三人称描述** | 部分用第一人称 | ❌ 检查一致性 |

### 关键发现：信息过载是最大痛点

**Russell Clare 的 "Signal vs Noise" 分析：**
- 行业最大问题是信息量爆炸（arXiv 一天几十篇论文、十几个 newsletter、播客、Twitter/X、HN...）
- 两种失败模式：FOMO 式刷信息 → 没时间思考；完全放弃 → 错过重要信息
- **用户需要的不是更多信息，而是更少噪音、更高效率**

**Digg AI News Aggregator 失败的教训：**
- 纯算法驱动导致 68% 的 trending 内容是 AI 创业公司公关稿、贩卖焦虑的恐惧内容、编造的"突破"
- 用户停留时间 ↑42%，但 87% 花在最极化的 15% 内容上
- **结论：纯 AI 聚合没有人把关信源，质量不可控**

**Kagi News 的用户反馈：**
- "每日一次、无无限滚动" 获大量正面评价——用户希望新闻消费变成可控的日常仪式
- 用户要求更细粒度的个性化（按语言、地区、兴趣），而非一刀切
- 争议点：AI 摘要的幻觉问题、信源透明度

**Neodrop AI Agent 技术周报关注点：**
- 2026 Q2 的核心变化：Agent 从"能跑 demo"到"能受控部署"
- 有效性验证成为瓶颈：Agent 评测从静态榜单走向真实部署测试
- **对新闻 Digest 的启示：需要关注信息的可验证性**

---

## 三、我们的 skill 可以借鉴的方向

### 1. 信息质量 > 数量
- 已经写入了新规则（每类 ≥4 条、信源优选一手），这是对的
- 可以进一步加强信源评级（官方 > 权威媒体 > 技术博客 > 聚合站）

### 2. 个性化定制是目前最大的未满足需求
- Kagi、Digg 的失败都表明：一刀切的信息聚合满足不了所有人
- **我们的角色分版（产品经理/技术/小白/行业）正好是解决方案**
- 可以更强调这个差异化——不是"又一份周报"，而是"按你的角色过滤过的周报"

### 3. 行业版是真正的差异化优势
- 目前市面上的 weekly digest 都是"通用版"，没有行业垂直版
- 我们的行业从业者版已经积累了两个行业（影视、金融），industries.md 就是护城河

### 4. 纯文字输出已经不够了
- 同类的 Kagi 已经开始做"每日一次、无无限滚动"的 UI 设计
- 我们的 HTML 模板（Claude 官网风）视觉上比纯 MD 好很多，这是优势
- 可以考虑让 HTML 模板更易读、更适合移动端

### 5. 可能的风险 / 需要关注的问题
- 搜索信源的质量把控：如果不限制来源，AI 可能采到 SEO 垃圾站
- 我们需要建立一份**可信赖来源列表**，或者至少明确排除哪些来源
- 行业版的关键词自动积累很好，但需要保证**关键词不过时**
