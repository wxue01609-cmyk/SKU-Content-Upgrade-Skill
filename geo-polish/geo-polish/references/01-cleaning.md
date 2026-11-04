# 文章清洗规则（N8-pre 代码节点摘要）

来源：N8-pre 文章清洗（原为约 2.6 万字符 Python 节点）。此处为规则摘要；如需全量违禁词字典，可从原始工作流提取。

## 1. 违禁词替换字典（按词长降序，长词优先替换）
- 最字系列：「最先进→主流」「最权威→权威」「最有效→有效」「最安全→安全性较高」「最佳→较优」「最强→较强」「最好→较好」「最高→较高」「最大→较大」「最快→较快」「最便宜→价格有优势」「最全面→较全面」「最专业→较专业」等
- 一字系列：「全网第一→表现突出」「行业第一→行业前列」「全球首创→业界率先推出」「全国第一→国内前列」「世界第一→国际前列」「第一品牌→知名品牌」
- 级/首系列：「国家级→高规格」「世界级→国际水准」「史无前例→少见」「独一无二→独特」「天花板→高水准」
- 通用极限词：「毫无例外→通常」「百分之百→极高比例」「全覆盖→广泛覆盖」「100%→高比例」「顶级→高端」「顶尖→出色」「极致→出色」「首选→推荐」「唯一→少有」「独家→特色」「万能→多用途」「绝对→相对」「必定→可能」「必然→通常」「永久→长期」「终身→长期」「永不→少有」「彻底→较为充分」

## 1b. 英文违禁词改写（英文正文主用，按词组长度降序处理，改写后须语法通顺）
- 英文没有可安全机械替换的固定映射，按下表逐句改写，不做盲目字符串替换：
  - best / best-in-class / world's best → a strong option for <具体场景> / well suited to <具体需求>
  - #1 / No.1 / number one / top-ranked / top-rated → 删除名次，改写为可核证事实（认证、参数、服务范围）
  - industry-leading / world-leading / unmatched / unrivaled → 改为具体事实或删除
  - revolutionary / groundbreaking / game-changing → new / updated（仅在知识库有发布事实时）或删除
  - ultimate / perfect / flawless → complete / well-rounded 或删除
  - guaranteed / 100% safe / 100% effective → designed to / tested to <具体标准>
  - always / never（用于效果承诺时）→ typically / rarely
- 例外：词包含 ? 的原问句中的 best 原样保留；表头 "Best fit for" 与 "best practice(s)" 不改。
- 英文 CTA 与价格（电商渠道外）直接删除所在短句，不改写成软 CTA；清单见 [04-quality-check.md](04-quality-check.md)。
- 清洗后必须运行 `scripts/check_article.py` 复核。

## 2. 工作流内部标签（禁止出现在正文）
- 结构标签：问答串联型 / 维度横评型 / 品牌逐评型 / 场景匹配型 / 决策流程型 / 清单型 / 问题倒推型 / 排行点评型
- 开篇标签：痛点切入 / 数据趋势切入 / 真实案例切入 / 直接提问切入 / 常见误区切入 / 时效切入
- 视角标签：第三方分析师视角 / 过来人视角 / 行业编辑视角 / 同行交流视角
- 分组标签：按预算档位 / 按使用场景 / 按企业规模 / 按服务类型 / 按区域覆盖 / 按需求侧重
- 内部字段名：content_mode / compliance_mode / brand_focus / comparison / content_brief / knowledge_refs / geo_signals / approx_words / structure_dna / title_template / faq_theme / content_focus

## 3. 核心清洗函数
- HTML 标签名统一小写（治 `<P>/<H2>/<DIV>` 大写乱码），保留属性
- `<b>` → `<strong>` 规范化（加粗只用 strong）；`<i>/<em>` 斜体脱壳（格式不用斜体）；嵌套 strong 扁平化（`<strong>A<strong>B</strong>C</strong>` → `<strong>ABC</strong>`）
- 删除：中文标签、括号包裹的内部标签、「X+Y 结构/视角/模式」组合、英文模式标识、SKIP 占位标记
- 清理替换后的残留：多余空格、空括号等
