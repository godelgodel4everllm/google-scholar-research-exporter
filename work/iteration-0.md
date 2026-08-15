# Iteration 0 — Requirements Bootstrap

## Date

2026-08-15

## Produced

- Real/Cog/Spec/Work 需求结构。
- 对旧 GB18030 HTML 的行为、selector、RefWorks 字段和风险梳理。
- 全学科 taxonomy、领域关键词/标签、h-index 学者发现、outerHTML 捕获、论文元数据和 Zotero 导出的冻结需求。
- 独立实施计划。

## Trade-offs

- 基于 Scholar 页面可能阻断且用户明确要求 outerHTML，按照“浏览器辅助 capture + 隔离离线 parser”设计，不把 CAPTCHA 绕过写进项目。
- 基于 Zotero 互操作要求，内部以 CSL-compatible Citation model 为中心，同时保留旧 RefWorks 导出。
- 基于学科 taxonomy 与学者 label 语义不同，分别建模，再提供关联映射。
- 基于需求阶段可能包含敏感 snapshot，仓库先保持 private。

## Not Yet Produced

- 未实现 parser、browser capture、CLI/API/Web。
- 未运行 Scholar live 请求。
- 未配置 Zotero API key。
