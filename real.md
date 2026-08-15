# Project Reality

## Background

用户提供 `/Users/wickedmc/Downloads/google_scholar_parser.html`，希望把它改造成独立项目仓库。旧文件：

- 大小：5,336 bytes，145 行，GB18030/CRLF。
- SHA-256：`cf74451190458113ed1636acdc03cd199c4e4e0a455653a95be6b93abcab0fc8`。
- 页面标题：Google Scholar 转换 RefWorks。
- 原始署名：IA003 吕明明、IA001 林浩涛教练。
- 输入：用户把 Scholar 页 HTML 粘贴到 textarea。
- 识别：搜索结果卡片 `.gs_r.gs_or.gs_scl`，或学者论文表 `#gsc_a_tw tbody tr`。
- 输出：RefWorks Tagged 文本，含 title、link、authors、year、source、abstract/snippet、citation。

## Current State

- 新项目创建时没有业务实现。
- 旧页面没有联网请求、没有 h-index、没有学科 taxonomy、没有分页/任务状态。
- 旧页面使用 HTTP Baidu jQuery，并通过隐藏 DOM `.html(untrustedHtml)` 解析输入，不可直接部署。
- 学者论文页中没有直接论文 URL；旧实现用标题构造 Scholar 搜索 URL。
- 当前 `0_AI_scholar` 的 Scholar 适配存在验收缺陷；本项目独立建设，未来只通过稳定 JSON/export contract 集成，不 vendor。

## Real-world Constraints

- Scholar 页面结构可能变化，且可能返回 403/CAPTCHA。
- 搜索结果页、作者检索页、作者详情页和 Scholar Metrics 页面结构不同。
- 页面文本存在多语言、实体编码、缺失 DOI/摘要/作者截断等情况。
- h-index 属于学者 profile 指标，不是搜索结果卡片的固有字段；需要访问对应学者页。
- “全部分类”不能永久硬编码，必须带抓取日期、来源与 diff。
- Zotero 等管理器要求标准格式与字段语义；仅生成旧 RefWorks 文本不够。
- 不记录登录 cookie、验证码或真实 API key。

## Goals

- 学科目录同步后，分类/子分类/标签覆盖率 100%，可查询版本和来源。
- 给定领域关键词或标签，输出学者列表并按 total/5-year h-index 排序、过滤。
- 给定受支持页面 outerHTML，解析器能判型并输出结构化 author/publication 数据。
- 每篇论文保留名称、作者、年份、venue、摘要、引用数、URL、DOI/cluster id（可得时）及来源证据。
- 同一结果能导出 CSL-JSON、BibTeX、RIS、RefWorks Tagged，并通过 Zotero 导入验收。
