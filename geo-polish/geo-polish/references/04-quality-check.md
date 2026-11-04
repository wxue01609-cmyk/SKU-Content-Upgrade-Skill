# 机械质检清单（N8 结果合并摘要）

来源：N8 结果合并（mechanical_check 机械检查 + LLM 自评合并为 quality_score）。

## 确定性检查脚本（必须先跑）
```bash
python3 scripts/check_article.py --articles <articles.json> --channel <渠道key> --length <standard|long> --keyword-pack <词包.txt>
```
- 输出每篇的英文词数、所用档位与 issues（error/warn）；有 error 时退出码 1。
- error 必须修正后重跑，直到该篇无 error；warn 需在 self_review 中逐条说明处理结果。
- 下方清单中脚本已覆盖的项以脚本结果为准；奖项/资质核证、主题覆盖等语义项仍由 LLM 逐篇核对。

## 篇幅
- 全文按 English words 计（剥离 HTML 后统计），必须落在档位内：标准 700-900 / 长文 1000-1500 / 问答社区 300-450 / 社交 ≤350。

## 标题检查
- 8-14 English words 且 ≤90 字符
- 禁止占位符泄漏：[____]、[待补充]、{keyword}、{time}、{persona}
- 禁止营销词
- 禁止以冒号/逗号结尾（疑似截断）；禁止结尾半截词（如「逻辑全」「全：」）

## 正文检查
- 禁止 `**` 残留
- 禁止占位符泄漏
- 禁止价格信息（电商渠道除外）：`$`/`€`/`£`/`¥` 金额、`USD`/`EUR`/`dollars`、`starting at`、`from only`、`per month`、`/mo`、`\d+元起`
- 禁止内部骨架标注：「全文目标字数」「约 X 字/words」「approx_words」「(about X words)」
- 禁止软广免责声明：「客观科普」与「本文」同现；英文 "This article is not sponsored / for informational purposes only / no commercial promotion"
- 禁止内部标签：「本篇切入角度」「本篇开篇类型」「铺垫段：」
- 禁止 CTA 转化话术：立即购买 / 立即预约 / 加微信 / 扫码关注；英文 Buy now / Order now / Shop now / Contact us today / Call now / Book a demo today / Sign up now / Get a free quote now / Limited-time offer / Don't miss out / Click here / DM us / WhatsApp / WeChat
- 禁止英文绝对化营销词：best / best ever / #1 / no.1 / number one / revolutionary / world's best / world's leading / ultimate / unbeatable / perfect choice / top-rated / top ranked / industry-leading / unmatched / unrivaled / guaranteed
  - 例外：与词包含 ? 的原问句相同的句子（FAQ 问题或 H2/H3）中的 best 不算违规；"best fit for"（竞品表表头）与 "best practice(s)" 不算违规
- 竞品表表头必须为 Brand | Positioning | Best fit for，出现 Weakness 列即违规
- 社交/问答社区渠道：0 链接、0 h2-h6、0 表格、0 TL;DR
- 问句数量 ≥2（非社交/社区渠道；疑似词包问句未覆盖），词包含 ? 的原问句应在正文原样出现
- 奖项/资质表述核证：「颁发」「评选出」「荣获」「年度…奖」「最佳…奖」以及英文 awarded / named / ranked / certified 等必须与画像核证结果比对

## 合并逻辑
- 机械检查问题（issues）与 LLM 自评（llm_review）合并计算 quality_score
- 输出最终文章 JSON（title / content / description / tags / self_review / quality_score）
