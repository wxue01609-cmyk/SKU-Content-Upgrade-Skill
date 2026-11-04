---
name: geo-polish
description: 海外 GEO 质检与后处理 skill。两种模式：①改动卡模式——作为内容优化流水线的质检关卡，对 geo-pdp / geo-landing / geo-write / geo-offsite 的每张改动卡交付物跑事实、证据、可提取、渠道、分发五个关卡，生成质检报告并要求人工审批；②批量模式——对旧流水线批量生成的文章做合并容错、违禁词清洗、链接校验、社交帖子形态兜底与机械质检。当交付物已生成、需要质检与清洗时使用。
---

# GEO 质检与后处理（geo-polish）

来源：Dify 工作流「海外GEO生文-wx-360gpt-v18-13」最后阶段（articles-merge 合并输出 → N8-pre 文章清洗 → L-extract/L-verify/L-parse/L-replace 链接校验 → S-post 社交帖子兜底 → N8 结果合并）。

## 改动卡模式（内容优化流水线）

整套流水线：geo-contract → geo-diagnose → geo-strategy → geo-evidence → 生产类 → **geo-polish** → geo-measure。

对每张 `status=in_production` 的改动卡：
1. 运行 `python3 scripts/check_deliverable.py --project <项目目录> --change <change_id> --write`，按资产类型补跑对应脚本（博客跑 `check_article.py`；feed 与 schema 跑 geo-pdp 的 `check_feed.py` / `check_product_schema.py`；有外链时跑 `check_links.py`）。
2. 按 [references/06-channel-gates.md](references/06-channel-gates.md) 完成人工检查项，补入 `qa/<change_id>.json`。
3. 任一关卡 fail：改动卡置为 `qa_failed`，把问题清单退回对应生产 skill；修改后重跑。
4. 全部通过：改动卡为 `qa_passed`，提交人工审批（对比页、合规叠加层、法务要求的交付物另需法务审核）。审批完成后填写 `human_approval` 并把卡置为 `approved`，交给 geo-measure。脚本不会自动审批。

下面的「执行流程」为批量模式（旧流水线）。

## 执行流程（批量模式，严格按序）

### 步骤 1 合并与截断容错
- 聚合标准版 + 品牌版等全部文章为 all_articles；任一文章 JSON 截断未闭合时，用正则提取 title/content/description 兜底，并在 self_review 标记截断。
- 规则见 [references/05-output-merge.md](references/05-output-merge.md)。

### 步骤 2 文章清洗（N8-pre）
- 违禁词处理：中文按替换字典（最字系列/一字系列/级首系列/通用极限词，长词优先替换）；英文按改写表逐句改写（best/#1/industry-leading/guaranteed 等），并删除英文 CTA 与价格短句
- HTML 规范化：标签名小写、b→strong、斜体脱壳、嵌套 strong 扁平化
- 删除工作流内部标签（结构/开篇/视角/分组标签与内部字段名）与 SKIP 占位标记、残留清理
- 规则与示例见 [references/01-cleaning.md](references/01-cleaning.md)。

### 步骤 3 链接校验（L-*）
1. L-extract：提取文章全部 href + 素材池 URL，去重且只保留 http/https
2. L-verify：先运行 `python3 scripts/check_links.py --articles <articles.json>` 实际请求每个 URL（跟随跳转、记录状态码与最终域名、识别软 404）：ok → 保留；dead/redirect_offsite → 用内置 web_search 找替代 URL（与文章上下文关键词匹配，优先同域名，其次品牌官网或权威来源）；unreachable（网络受限、超时、反爬拦截）→ 标记待人工确认，不得当作 ok。替代 URL 必须再用 `check_links.py --url <替代URL>` 复核为 ok 才能使用
3. L-parse：解析校验结果，兼容截断 JSON，逐段提取完整对象
4. L-replace：ok 保留；dead 优先用已复核的替代 URL，其次同域名已验证链接，都没有则去掉链接只留文字
- 提示词原文与规则见 [references/02-link-check.md](references/02-link-check.md)。

### 步骤 4 社交帖子兜底（S-post）
- publish_media 命中社交/社区渠道时：整表删除、删 TL;DR 段、h1-h6 去标签留文字、链接去标签留锚文本、加粗斜体去标签、空段落收敛。
- 与写作提示词的帖子模式互为双保险；真人感 6 条（REAL6）见 [references/03-post-shaping.md](references/03-post-shaping.md)。

### 步骤 5 机械质检（N8 结果合并）
- 先运行 `python3 scripts/check_article.py --articles <articles.json> --channel <渠道key> --length <standard|long> --keyword-pack <词包.txt>`；有 error 的文章修正后重跑，直到无 error，warn 在 self_review 中说明。
- 篇幅：剥离 HTML 后的英文词数落在档位内（标准 700-900 / 长文 1000-1500 / 问答社区 300-450 / 社交 ≤350）
- 标题：8-14 English words 且 ≤90 字符；禁止 [____]/[待补充]/{keyword} 等占位符、禁止营销词、禁止冒号/半截词结尾
- 正文：禁止 ** 残留、占位符泄漏、价格信息（含 $/USD/per month 等，电商渠道除外）、内部骨架标注（approx_words/全文目标字数）、软广免责声明、内部标签、中英文 CTA 转化话术（Buy now/Contact us today/立即购买/加微信等）、绝对化营销词（best/#1/revolutionary/ultimate 等；词包原问句中的 best 例外）、竞品 Weakness 列
- 词包覆盖：正文问句 ≥2；奖项/资质表述与画像核证结果一致
- 主题覆盖（文章 JSON 含 topic 时）：核对文章所属主题的每个 query 在正文有实质覆盖，缺漏的在自评中标记并补足；质检报告注明每篇文章的主题归属
- 机械检查项与 LLM 自评合并为 quality_score
- 完整检查清单见 [references/04-quality-check.md](references/04-quality-check.md)。

## 输出
最终文章 JSON：title / content（HTML，已清洗）/ description / tags / self_review / fact_sources / quality_score，可直接发布。

### HTML 文件组装（保存 articles/第N篇.html 时必须执行）
- 完整 HTML 必须包含 <!DOCTYPE html> 与 <head>（meta charset / viewport / description / title），正文放在 <body><article>。
- <head> 内必须注入表格样式（固定内容，缺则补）：
<style>
table { border-collapse: collapse; width: 100%; margin: 12px 0; font-size: 14px; }
th, td { border: 1px solid #d0d7de; padding: 6px 10px; text-align: left; vertical-align: top; }
th { background: #f6f8fa; font-weight: 600; }
</style>

### 步骤 6 表格样式校验（追加在机械质检之后）
- 对每一篇含 <table> 的文章 HTML：<head> 内必须已注入上方表格样式，缺则就地补入，再保存。
