---
name: google-scholar-field-research-export
description: 获取 Scholar 全学科目录，按领域关键词和 h-index 发现学者，解析 outerHTML 学者页并导出 Zotero 兼容论文元数据
version: 0.1.0
---

# Google Scholar Field Research Export

## Trigger Conditions

- 用户要求同步全部 Scholar 学科分类、子分类或标签。
- 用户给出领域关键词/标签，要求按 h-index 检索或排序相关学者。
- 用户提交 Scholar 搜索页、作者检索页、作者详情页或 Metrics 页 outerHTML。
- 用户要求把学者论文导入 Zotero、EndNote、RefWorks 或其它文献管理器。

## Inputs

### Taxonomy input

- Scholar Metrics/category 页 outerHTML 或受控浏览器捕获结果。
- `captured_at`、`source_url`、`locale`、页面类型、content hash。

### Author discovery input

- `field_query: string`：自由关键词，例如 “cognitive science”。
- `labels: string[]`：Scholar profile label，例如 `machine_learning`。
- `h_index_metric: total | five_year`。
- `min_h_index`、`max_authors`、`sort`、locale。

### Snapshot input

- `outer_html: string` 或本地 snapshot 路径。
- 可选 `expected_page_type`：
  - `scholar_search_results`
  - `author_search_results`
  - `author_profile`
  - `author_publications`
  - `metrics_categories`
  - `metrics_subcategories`
- 捕获上下文不得包含 cookie、Authorization、API key 或表单秘密。

## Processing

### 1. Snapshot safety and classification

1. 计算 SHA-256，记录来源与捕获时间。
2. 移除 script、iframe、event handler、表单值和不需要的个人数据。
3. 在隔离 DOM parser 中解析，禁止执行输入 HTML。
4. 用结构特征判型；未知/consent/CAPTCHA 页面返回 typed status，不猜测。

### 2. Complete subject taxonomy

1. 从 Metrics/category 页面提取顶层学科、子分类、稳定标识、URL、locale。
2. 从学者 profile/search 页提取 label 与显示名。
3. 学科分类与人物标签分别建模，不混成同一概念。
4. 保存版本化 snapshot；新同步生成 added/changed/removed diff。
5. 支持列出全部分类、按关键词搜索分类、由分类展开标签。

### 3. Field and h-index author discovery

1. 把 `field_query` 解析为自由文本 query 与规范化 labels。
2. 从 author search outerHTML 提取姓名、profile id/url、affiliation、verified email domain、labels、cited-by（可得时）。
3. 为每个候选学者建立 profile capture job。
4. 从对应学者页提取：
   - total citations / five-year citations
   - total h-index / five-year h-index
   - total i10-index / five-year i10-index
   - labels/interests、affiliation、coauthors（可得时）
5. 按指定 h-index 指标过滤和稳定排序；同值按 citations、姓名、profile id 排序。
6. 每个排名结果保留 query、profile snapshot hash 和 metric extraction evidence。

### 4. outerHTML-compatible publication parsing

兼容旧文件已有的两种页面：

- 搜索结果 `.gs_r.gs_or.gs_scl`。
- 学者论文表 `#gsc_a_tw tbody tr`。

并扩展作者详情/论文详情字段：

- title、authors、year/date、venue/source、volume/issue/pages。
- abstract/snippet、citation count、Scholar cluster/result id。
- canonical URL、PDF URL、publication detail URL。
- DOI、ISBN、ISSN、publisher、language（页面或 enrichment 可得时）。
- profile id、source URL、snapshot hash、捕获时间、解析器版本。

缺失字段保持 null 并附 reason，不使用空字符串伪造。

### 5. Identity and enrichment

1. identity 优先级：DOI → Scholar cluster id → 规范化 title+year+first-author hash。
2. 可选通过 Crossref/OpenAlex/Semantic Scholar enrichment 补 DOI、完整作者、venue。
3. enrichment 不得覆盖已有冲突字段；冲突进入 `field_conflicts`。
4. 原始 Scholar 证据与 enrichment provenance 同时保留。

### 6. Export and Zotero interoperability

统一 Citation 模型至少支持：

- CSL-JSON。
- BibTeX UTF-8。
- RIS UTF-8。
- RefWorks Tagged（兼容旧输出）。
- CSV/JSONL 用于分析。

可选 Zotero Web API：

- 先 dry-run 验证 collection、item type、creator mapping。
- key 只从环境变量/系统密钥链读取。
- 以 DOI/URL/title identity 幂等 upsert，不重复推送。
- 记录 Zotero item key/version，不记录 API key。

## Outputs

- `taxonomy/<date>/subjects.json`、`labels.json`、`diff.json`。
- `authors.json` / `authors.csv`：领域 query、h-index 排名、profile metadata、provenance。
- `publications.csl.json`、`publications.bib`、`publications.ris`、`publications.refworks.txt`。
- 可选 Zotero push receipt：created/updated/skipped/failed，不含凭证。
- parse report：page type、records、warnings、blocked/error、snapshot hash。

## Requirement Registry

| ID | Priority | Requirement | Executable evidence |
|---|---:|---|---|
| `TAX-01` | P0 | 同步并版本化全部可见顶层学科与子分类，作者 labels 单独建模 | 两版 fixtures 的 coverage、idempotency、added/changed/removed 测试 |
| `AUTH-01` | P0 | 按领域关键词/labels 发现学者，并按 total/5-year h-index 过滤、稳定排序 | 三位以上 author/profile fixtures，覆盖缺值、同名和同分 |
| `CAP-01` | P0 | 用户显式捕获当前页 outerHTML，随后按队列处理对应学者页 | capture contract 不含 cookie/token/form secret；CAPTCHA 自动暂停 |
| `PUB-01` | P0 | 从旧/新 Scholar 页面提取完整 Citation 与字段 provenance | search/profile/publication fixtures 的 schema 与字段证据测试 |
| `EXP-01` | P0 | 导出 CSL-JSON、BibTeX、RIS、RefWorks、CSV/JSONL | serializers schema/snapshot 测试与 Unicode round-trip |
| `ZOT-01` | P0 | Zotero Desktop 无错位导入；Web API 可 dry-run、幂等 upsert | 三格式各三条手工导入回执与 mock API 重放测试 |
| `SAFE-01` | P0 | 不执行输入 HTML，不泄露凭证，不绕过 CAPTCHA | malicious HTML、secret scan、blocked-page regression |
| `OBS-01` | P1 | idle/running/empty/blocked/error/success 状态和来源可追溯 | CLI/API/Web contract tests 与 parse report |

## Acceptance Criteria

### Taxonomy

- [ ] Metrics fixtures 中的全部顶层/子分类均被提取，重复同步 idempotent。
- [ ] 分类与 author labels 分开存储，可相互关联但不混用。
- [ ] 两版本 fixture 能输出 added/changed/removed diff。

### Author and h-index

- [ ] 给定领域关键词和三位 author fixture，按 total/5-year h-index 得到确定排序。
- [ ] min h-index、缺 metric、同名作者、同分排序均有测试。
- [ ] 每位作者能追溯到 profile id、snapshot hash、metric 来源节点。

### outerHTML

- [ ] 旧 `.gs_r.gs_or.gs_scl` fixture 输出不少于 title/link/authors/year/source/snippet/citation。
- [ ] 旧 `#gsc_a_tw tbody tr` fixture 输出 title/authors/year/venue/citation。
- [ ] 输入中的 script/event handler 永不执行，输出已清理。
- [ ] CAPTCHA/consent/未知页面返回明确 status，不输出伪造记录。

### Export

- [ ] 同一 Citation 可输出 CSL-JSON、BibTeX、RIS、RefWorks。
- [ ] Unicode 中文作者/标题、多个作者、缺 DOI、网页/书籍/期刊类型均可导入。
- [ ] 自动化 parser/schema 校验全部通过。
- [ ] 手工 Zotero Desktop 导入 smoke：RIS、BibTeX、CSL-JSON 各至少 3 条，标题、作者、年份、venue、DOI 不错位。
- [ ] Zotero Web API dry-run 不泄露 key，重复执行不产生重复 item。

### Boundaries

- [ ] 默认测试完全离线。
- [ ] 不包含 CAPTCHA 绕过、cookie 持久化、住宅代理池。
- [ ] snapshot/log/fixtures secret scan 通过。
- [ ] blocked/error 在 CLI/API/Web 可见，不能算成功。
