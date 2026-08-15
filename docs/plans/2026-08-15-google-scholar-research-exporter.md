# Google Scholar Research Exporter Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 把旧版 HTML→RefWorks 页面重建为独立的 Scholar 学科目录、h-index 学者发现、outerHTML 解析和 Zotero 兼容导出项目。

**Architecture:** 浏览器扩展/受控浏览器只负责捕获 outerHTML；`src/core` 在隔离 DOM 中完成判型、解析、身份合并与导出；CLI、API 与 Web 共用核心模型。默认流程离线可测，live provider 可替换，blocked/error 被持久化。

**Tech Stack:** Node.js 20+、TypeScript、pnpm、Fastify、React/Vite、Cheerio/parse5、SQLite/Drizzle、Zod、Vitest、Playwright（仅 capture/live）、CSL-JSON、BibTeX/RIS serializers、Zotero Web API。

---

## Task 1: Legacy fixture 与安全 page classifier

**Requirements:** `CAP-01`, `SAFE-01`, `OBS-01`

**Files:** `references/legacy_google_scholar_parser.html`、`src/core/snapshot.ts`、`src/core/page_classifier.ts`、`tests/fixtures/*.html`、`tests/page_classifier.test.ts`

1. 将原 HTML 转 UTF-8 fixture，保留 hash/署名，不加载远程 jQuery。
2. 先写失败测试：识别 search results、author search、profile、publication table、metrics、CAPTCHA/consent/unknown。
3. 实现 snapshot hash、sanitization、无脚本 DOM parse。
4. 验证恶意 event/script 不执行，unknown 不猜类型。
5. Commit：`feat: classify sanitized scholar snapshots`。

## Task 2: 全学科 taxonomy 与标签版本

**Requirements:** `TAX-01`

**Files:** `src/core/taxonomy.ts`、`src/core/schema.ts`、`src/core/store.ts`、`tests/taxonomy.test.ts`

1. fixture 覆盖顶层分类、子分类、author labels 和两版本 diff。
2. 先验证测试因模块不存在而失败。
3. 分别存储 subject/category 与 label；保存 source URL、locale、captured_at、hash。
4. 实现 list/search/diff；重复同步幂等。
5. Commit：`feat: version scholar subjects and author labels`。

## Task 3: 领域关键词与 h-index 学者发现

**Requirements:** `AUTH-01`

**Files:** `src/core/authors.ts`、`src/core/jobs.ts`、`tests/author_ranking.test.ts`

1. fixture 三位以上学者，覆盖 total/5-year h-index、缺值、同名、同分。
2. 实现 query/label normalization、profile capture queue、metric parser。
3. 稳定排序：指定 h-index → citations → normalized name → profile id。
4. 每个结果携带 query、labels、snapshot hash 和 extraction evidence。
5. Commit：`feat: rank field experts by profile h-index`。

## Task 4: 浏览器辅助 outerHTML capture

**Requirements:** `CAP-01`, `SAFE-01`, `OBS-01`

**Files:** `extension/manifest.json`、`extension/src/capture.ts`、`src/providers/browser.ts`、`tests/capture_contract.test.ts`

1. 测试 capture payload 不含 cookie/token/form secrets。
2. 实现当前 tab `document.documentElement.outerHTML` 捕获、page type hint、URL/time/hash。
3. capture job 逐个打开对应 profile；尊重用户节奏、rate limit 和 stop。
4. CAPTCHA/consent 立刻暂停队列并提示人工处理，不自动绕过。
5. Commit：`feat: capture scholar outerhtml from user browser`。

## Task 5: 学者页与论文元数据

**Requirements:** `PUB-01`

**Files:** `src/core/parsers/*.ts`、`src/core/publication_identity.ts`、`tests/publications.test.ts`

1. 先为旧两个 selector 和新版 profile/publication fixtures 写失败测试。
2. 提取 Citation model 全字段；缺失写 null+reason。
3. DOI → cluster → title/year/first-author identity 合并。
4. 可选 enrichment 保留 source precedence 和 field conflicts。
5. Commit：`feat: parse author publications with provenance`。

## Task 6: CSL/BibTeX/RIS/RefWorks/Zotero

**Requirements:** `EXP-01`, `ZOT-01`, `SAFE-01`

**Files:** `src/core/exporters/*.ts`、`src/integrations/zotero.ts`、`tests/exporters.test.ts`、`tests/zotero.test.ts`

1. fixtures 覆盖期刊、书籍、网页、Unicode、多作者、无 DOI。
2. 先让四格式 snapshot/schema 测试失败。
3. 实现 serializers；RefWorks 保持旧 tag 兼容并修正 creators。
4. Zotero API 先 dry-run，再幂等 upsert；凭证只读环境/钥匙串。
5. 手工 Zotero Desktop smoke 记录到 `docs/acceptance/zotero-import.md`。
6. Commit：`feat: export citations to zotero-compatible formats`。

## Task 7: CLI、API 与 Web

**Requirements:** `TAX-01`, `AUTH-01`, `CAP-01`, `PUB-01`, `EXP-01`, `ZOT-01`, `OBS-01`

**Files:** `src/cli/index.ts`、`src/server/*.ts`、`web/src/*`、`tests/api.test.ts`

CLI 至少包括：

```text
gsre taxonomy sync|list|diff
gsre authors search --field <query> --label <label> --min-h-index <n>
gsre snapshots import <html>
gsre profiles parse <snapshot>
gsre export --format csl|bibtex|ris|refworks|csv
gsre zotero push --collection <key> --dry-run
```

Web 提供 taxonomy 浏览、query/filter/sort、capture queue、学者详情、论文勾选与导出。所有状态区分 idle/running/empty/blocked/error/success。

## Task 8: 验收与发布

**Requirements:** `TAX-01`, `AUTH-01`, `CAP-01`, `PUB-01`, `EXP-01`, `ZOT-01`, `SAFE-01`, `OBS-01`

1. `pnpm test`、两套 typecheck、Web build 全部通过，默认无公网。
2. secret/traversal/XSS/snapshot privacy 测试通过。
3. 受控 live：至少一个学科目录、一个领域 query、三位学者 profile、每位至少一篇论文。
4. RIS/BibTeX/CSL 各导入 Zotero Desktop 三条；字段人工核对。
5. 写 `docs/acceptance/2026-08-15-mvp.md`，逐需求记录命令、退出码、结果与 blocked。
6. 每 Task 独立提交，再申请验收。
