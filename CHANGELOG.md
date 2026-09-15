# Changelog / 更新记录

## 1.1.0 — 2026-09-16

### 中文

- 发布说明补充现有能力：安装后无需额外 MCP/API Key 配置；内部读取结合缓存与重复过滤，减少主模型接收界面和重复内容的 token 开销。

- 较大独立任务主动分配给4–6个普通极高 Chat，保留1–6总数及替换计数。
- 每轮一个主要可验收成果，允许必要关联操作与自查；拆分工序不削减原采集目标，不固定每轮步数。
- 各负责人独立验收续轮；保存总目标、已接受成果、缺口和下一动作，避免齐步等待与目标漂移。
- **迁移变化：内部读取，网页发送与控制。** 内部发送不能再作兜底；原文件上传与生成文件下载仍走网页。
- 每个派发版本第10分钟首读，随后每5分钟；第15分钟首查网页，随后每10分钟；90分钟最终检查仍无完整回复则保存进度并中断整个任务。
- 网页报错按指示重试；无结果且无处理迹象才刷新；重试与恢复不重置时钟，一小时旧设置被取代。
- 新增外部采集留给负责 Chat；主端仅基于已交回材料整合和本地检查，来源必须在 Chat 实际权限内。
- 完整成果首次接收、原文缓存、后续重复过滤与定向回读；不改成全程摘要验收。
- 根据 Astra skill 指南精简描述、按需加载说明、明确完成边界；Sol 任务包保留直接目标、必要约束、返回契约与相关变化。
- 同步六个运行文件、中英文指南和插件元数据，提供本版独立安装包。

### English

- Clarify existing capabilities: no extra MCP/API-key setup after installation, and lower host token overhead through internal reply reads, caching, and duplicate filtering.

- Prefer 4–6 workers for substantial independent scopes, within the existing 1–6 total limit.
- One primary stage result per prompt; preserve full collection targets and advance owners independently.
- Persist the goal, revisions, accepted evidence, gaps and next actions.
- **Migration: read internally; send every message and operate Chat through the webpage.** Browser file routes remain unchanged.
- Per revision: reply checks at 10/5 minutes, webpage checks at 15/10 minutes, and a final 90-minute check followed by task-wide interruption if a complete reply cannot be verified.
- Conditional refresh and page-directed retries preserve the original clock; no automatic host takeover.
- Keep new external collection within the assigned Chat's actual tools and permissions.
- Preserve full first receipt, cached originals, duplicate suppression and targeted later reads.
- Apply concise, outcome-focused, progressively loaded Astra guidance and focused Sol packets.
- Synchronize the six skill files, bilingual guides and plugin metadata, with a standalone installation ZIP.

## 1.0.0 — 2026-09-12

First public release / 首次公开发布。

- Astra-led coordination of 1–6 ordinary ChatGPT Sol Extra High workers.
- Direct text read/send, concurrent independent work, one owner per scope, full-result receipt with duplicate suppression, and targeted acceptance checks.
- Browser routes for original uploads and newly generated file downloads.
- Detailed English and Simplified Chinese guides, examples, opt-in Sol High adaptation instructions, MIT license, source ZIP, and plugin packaging.
- The six runtime skill files match the installed version; this release adds bilingual distribution material and an explanation of the host-allowance saving strategy.

- Astra 统筹 1–6 个普通 ChatGPT Sol 极高子智能体。
- 文字直接读写、独立任务并行、每块工作一个负责人、完整结果接收与重复内容过滤、按问题验收。
- 原文件上传与新生成文件下载固定走浏览器。
- 提供完整中英说明、示例、用户主动选择的 Sol High 适配说明、MIT 许可证、源码 ZIP 和插件封装。
- 六个技能运行文件与已安装版一致；本次增加双语发布材料和节约主任务额度的原理说明。
