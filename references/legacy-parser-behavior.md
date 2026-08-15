# Legacy Google Scholar Parser Reference

## Source

- Original path: `/Users/wickedmc/Downloads/google_scholar_parser.html`
- SHA-256: `cf74451190458113ed1636acdc03cd199c4e4e0a455653a95be6b93abcab0fc8`
- Size/encoding: 5,336 bytes; GB18030 with CRLF
- Attribution in source: IA003 吕明明、IA001 林浩涛教练

不直接复制原文件到 production bundle。实现时先保存一份 UTF-8、只读 fixture，并保留 attribution/hash。

## Existing behavior

### Search result page

Selector: `.gs_r.gs_or.gs_scl`

| Legacy field | Selector/logic | RefWorks tag |
|---|---|---|
| title | `h3 a`，缺链接时回退 `h3` | `T1` |
| link | `h3 a[href]` | `LK` |
| authors/year/source | regex parse `.gs_a` | `A1/YR/DS` |
| abstract/snippet | `.gs_rs` | `AB` |
| citation | 第三个 `.gs_fl > a` 并按中文冒号切分 | `T2` |

### Author publication table

Selector: `#gsc_a_tw tbody tr`

| Legacy field | Selector/logic |
|---|---|
| title | `.gsc_a_at` |
| link | 用 title 构造 Scholar search URL |
| authors | 第一个 `.gs_gray` |
| venue | 第二个 `.gs_gray` |
| year | `.gsc_a_y` |
| citation | `.gsc_a_c` |

## Known limitations to preserve as regression cases

- 年份 regex 依赖特定标点和英文布局。
- citation 依赖链接顺序与中文冒号。
- 作者可能是一个拼接字符串，未拆分 creators。
- 学者论文表没有 abstract/DOI/直接论文 URL。
- 隐藏 DOM 使用 `.html(untrustedInput)`，存在脚本/事件注入风险。
- 依赖 HTTP Baidu jQuery，不能用于新实现。
- 只有 RefWorks Tagged，没有 CSL/BibTeX/RIS/Zotero API。
