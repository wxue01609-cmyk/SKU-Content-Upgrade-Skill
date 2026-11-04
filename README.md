# SKU 内容升级改造 Skill 包

独立站电商场景下「商品详情页 / 商品数据 / 结构化数据」的 GEO 升级改造 skill 包，与 [Article-Generation-Skill](https://github.com/wxue01609-cmyk/Article-Generation-Skill) 平级：
前者负责**内容建设**（关键词 → 渠道判定 → 多资产成稿），本仓库负责**已有商品页的改造升级**（诊断 → 证据 → 改稿 → 质检）。

## 流水线

```
待改造商品页 URL
   ↓ geo-diagnose      页面诊断：AI 可读性、缺失属性、问题族
   ↓ geo-evidence      事实证据账本：逐条摘录原文，标注来源/日期/许可
   ↓ geo-pdp           改造交付：详情页正文 / 商品 feed / Product JSON-LD
   ↓ geo-polish        质检五关：事实、证据、可提取、渠道、分发
```

对应平台的「SKU 内容升级改造」模块：诊断 → 事实账本 → 交付物（成品页、模块片段、整页模板、改前改后对比）→ 质检 → 待确认回填。

## 目录

| 目录 | 阶段 | 作用 |
| --- | --- | --- |
| `geo-diagnose/` | 第二步 | 答案与来源诊断、站点 AI 可读性审计、竞品核验 |
| `geo-evidence/` | 第四步 | 事实证据账本，阻断证据不足的改动 |
| `geo-pdp/` | 生产 | 商品页改造、OpenAI / Google 双 feed、Product JSON-LD |
| `geo-polish/` | 质检 | 两层结构：外层 `SKILL.md` 为批量模式，`geo-polish/SKILL.md` 为改动卡模式（SKU 改造走这层） |

## 安装

```bash
cp -R geo-diagnose geo-evidence geo-pdp geo-polish ~/.codex/skills/
```

## 硬性规则

- 只写可核实事实；抓不到的价格 / 评分 / 库存留空并标「待确认」，禁止编造。
- 每个改后属性必须能追到证据账本里的来源，无来源不进正文与 feed。
- 改后正文不出现「待核验 / pending verification / 来源备注」等审核语；核验项只写进 `notes` / `changes`。
- 站外链接（CTA、商品页、政策页、来源）一律 `target="_blank" rel="noopener"`，避免在预览 iframe 中被目标站点拒绝嵌入。
- feed 与 JSON-LD 的字段必须与改后正文口径一致，缺失字段留空而不是猜值。

## 说明

- 上游 `geo-contract` / `geo-strategy`（选词与改动卡选择）不在本仓库：平台侧由「内容建设」模块与 SKU 改造表单直接给定改动范围。
- 本仓库内容与本地 `~/.codex/skills/` 下同名目录保持 1:1 镜像，便于版本回滚与青蛙平台打包。
