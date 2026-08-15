# Google Scholar Research Exporter

独立的 Google Scholar 学科、学者与论文元数据研究工具。项目从旧版“粘贴 Google Scholar HTML → RefWorks Tagged”页面解析器演进，目标是：

1. 动态获取并版本化全部 Scholar 学科分类、子分类与学者标签。
2. 按领域关键词/标签发现学者，读取总 h-index、近五年 h-index、引用与 i10-index，并可过滤、排序。
3. 通过浏览器辅助捕获 `document.documentElement.outerHTML`，解析搜索页、学者列表页、学者详情页与论文表。
4. 输出论文名称、完整元数据与 provenance，并导出 CSL-JSON、BibTeX、RIS、RefWorks Tagged；可选推送 Zotero Web API。

当前只交付需求与实施计划，尚未实现业务代码。

- Owner: `godelgodel4everllm`
- Repository: `https://github.com/godelgodel4everllm/google-scholar-research-exporter`
- Visibility: private（需求与 snapshot 安全审查完成前）

## 需求结构

- [real.md](real.md)：真实现状与约束
- [cog.toml](cog.toml)：领域认知模型
- [spec/core.skill.md](spec/core.skill.md)：机器可执行需求
- [实施计划](docs/plans/2026-08-15-google-scholar-research-exporter.md)
- [iteration-0](work/iteration-0.md)：本次需求工作记录
- [旧解析器说明](references/legacy-parser-behavior.md)

## 安全与合规边界

- 不绕过 CAPTCHA，不实现住宅代理池，不抓取登录 cookie。
- outerHTML 默认仅保存在本机，保存前移除 script、cookie、token、表单值和无关个人信息。
- Zotero API key 只从环境变量/系统密钥链读取。
- 遵守来源站点条款、robots/rate limit；blocked 是明确状态，不伪装成功。

## Attribution

旧 HTML 页面标注作者：IA003 吕明明、IA001 林浩涛教练。本仓库保留行为说明和 SHA-256，用于兼容性重写，不抹去原署名。
