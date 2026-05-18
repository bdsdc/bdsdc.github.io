#


## opencli介绍

OpenCLI 是一个 AI 原生的 CLI 运行时框架，能把任何网站或 Electron 桌面应用变成你的命令行工具。复用 Chrome 登录态，零风控、零凭证存储，一条命令搞定 B站、知乎、Twitter、Reddit、小红书、YouTube 等 30+ 平台的数据获取与操作。
我们国内的常用的小红书，抖音，B站，贝壳，BOSS等

## 为什么是CLI
 **文本本质**：AI 最擅长处理文本和逻辑，CLI 是最高效的文本交互界面，无需解析 GUI 视觉元素。
- **低延迟高精度**：AI 可像调用函数一样精准执行 CLI 命令，避免图形界面中的识别与点击误差。
- **天然可组合**：CLI 支持管道（`|`）和脚本化，AI 能轻松串联多个工具完成复杂自动化任务。
- **标准化通信**：CLI 可作为 AI 代理之间的通用协议（如 OpenClaw），方便多智能体协作。
- **轻量低开销**：无需加载图形界面，AI 可高频、低成本地调用，适合大规模自动化部署。
- **易于生成**：大模型生成 CLI 命令（而非 GUI 操作序列）是更自然、更准确的输出形式。
还有就是一切大概记得，模糊记忆，然后全靠AI大模型推理，这样也很浪费成本

## opencli功能
OpenCLI 把任意网站或本地工具变成命令行接口。它不是爬虫——爬虫要维护 cookie、解析 HTML、对抗反爬；OpenCLI 接管的是你已经登录的 Chrome Session，直接拿渲染后的数据，输出 JSON / CSV / Markdown，不走 LLM，不花 token，同一条命令的格式不会变。


## opencli支持
- 有官方 CLI 的：GitHub、npm、Docker、AWS、Stripe——这些服务有完整的官方命令行工具，功能不比 Web 端差。问题不是工具不存在，是很多人没有把这条路建进工作流里。
- 没有 CLI 的：文档站、技术论坛、各类工具的 Changelog 页、没有公开 API 的 SaaS 后台。这些地方有大量高频查询的信息，但没有任何命令行接口
- 桌面APP程序：可以通过CLI控制桌面app程序

对于有官方的CLI的，我们可以通过封装官方CLI，对于没有CLI的，我们通过opencli给它做一个接口，目前opencli支持的网站还是比较多的,支持列表如下：
```
External CLIs (12):
  discord(discord-cli), docker, dws(DingTalk Workspace), gh, lark-cli, longbridge, ntn(notion), obsidian, tg(tg-cli), vercel, wecom-cli(企业微信), wx(wx-cli)

App adapters (7):
  antigravity, chatgpt-app, chatwise, codex, cursor, discord-app, doubao-app

Site adapters (136):
  1688, 1point3acres, 36kr, 51job, aibase, amazon, apple-podcasts, arxiv, baidu-scholar, band, barchart, bbc, bilibili, binance, bloomberg, bluesky, boss, brave, chaoxing, chatgpt, claude, cnki, coingecko, coupang, crates, ctrip, dblp, deepseek, defillama, devto, dianping, dictionary, dockerhub, douban,
  doubao, douyin, duckduckgo, eastmoney, endoflife, facebook, flathub, gemini, gitee, google, google-scholar, goproxy, gov-law, gov-policy, grok, hackernews, hf, homebrew, hupu, imdb, indeed, instagram, jd, jianyu, jike, jimeng, ke, lesswrong, lichess, linkedin, linux-do, lobsters, maimai, maven, mdn, medium,
  mubu, notebooklm, nowcoder, npm, nuget, nvd, oeis, ones, openalex, openfda, openreview, osv, packagist, paperreview, pixiv, powerchina, producthunt, pubmed, pypi, quark, qwen, reddit, rednote, rest-countries, reuters, rfc, rubygems, sinablog, sinafinance, smzdm, spotify, stackoverflow, steam, substack,
  taobao, tdx, ths, tieba, tiktok, toutiao, tvmaze, twitter, uisdc, uiverse, v2ex, wanfang, web, weibo, weixin, weread, wikidata, wikipedia, wttr, xianyu, xiaoe, xiaohongshu, xiaoyuzhou, xueqiu, yahoo, yahoo-finance, yollomi, youtube, yuanbao, zhihu, zlibrary, zsxq
```
