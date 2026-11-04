# 链接校验与替换规则（L-* 摘要）

来源：L-extract 链接提取、L-verify 链接联网校验、L-parse 校验结果解析、L-replace 失效链接替换。

## L-extract 链接提取
从清洗后的文章提取所有 href + 素材池 URL，去重且只保留 http/https（`scripts/check_links.py --articles` 已内置此步）。

## L-verify 可用性校验（实际请求，不靠搜索判断）
搜索结果只能说明某个页面曾被收录，不能证明它现在能打开，因此可用性一律以实际请求为准：

```bash
python3 scripts/check_links.py --articles <articles.json>
```

脚本判定：
- ok：最终状态码 2xx，跳转后仍在同一站点，且页面不是软 404（标题/正文含 not found、domain for sale 等）
- dead：4xx/5xx，或软 404
- redirect_offsite：跳转到另一个注册域名（常见于域名过期、停放页）
- unreachable：DNS/连接/超时/网络受限，或 401/403/429 等反爬拦截——无法下结论，不得当作 ok，标记待人工确认

## 替代 URL 查找（仅对 dead / redirect_offsite）
> 你是链接替代专家，已开启联网搜索。对每个失效 URL，用 web_search 找一个替代 URL：必须与文章上下文关键词匹配，优先同域名，其次品牌官网或权威来源。

铁律：
1. 必须联网搜索，不用训练记忆；只信 web_search 返回内容
2. url 字段必须原样回填输入值，禁止改写或输出占位词
3. 找到的替代 URL 必须再用 `python3 scripts/check_links.py --url <替代URL>` 复核为 ok 才能写入 replacement；复核不是 ok 的一律不用
4. 无法给出可靠替代时 replacement 给空字符串
5. 输出严格 JSON，无 markdown
6. 每次 web_search 的查询词必须 ≥ 3 个实质词（如「品牌名 + 文章关键词 + 地区/来源类型」），禁止 1-2 词极短查询

输出格式：
```json
{"results":[{"url":"<原样回填>","status":"ok|dead|redirect_offsite|unreachable","replacement":"<已复核为 ok 的替代URL或空>","reason":"一句话依据"}]}
```

## L-parse 校验结果解析
解析校验响应，兼容截断 JSON：逐段提取完整对象；url 非空且 status 归一化（ok/dead/redirect_offsite/unreachable）。

## L-replace 失效链接替换
- ok 保留原样
- dead / redirect_offsite：优先用已复核为 ok 的替代 URL（需与文章上下文关键词匹配），其次用同域名已验证链接，都没有则去掉链接只留文字
- unreachable：保留原链接但在质检报告中列为「待人工确认」；交付前必须由人工确认或替换，不得默认视为可用
