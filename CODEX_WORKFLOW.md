# Codex 对话人工沉淀工作流

## 1. 目标与事实源

目标是把用户提供的 Codex 任务整理成可检索、可追溯、可离线携带的经验档案，同时保留逐轮关键原话。判断、分段和总结由 Codex 人工完成，不建设全自动解析系统。

事实优先级：

1. 原始 Codex 任务中的轮次、状态、时间、用户消息和 Codex 消息。
2. `knowledge-review-ui-prototype.html` 中当前统一数据源及 `window.__knowledgeDataIntegrity`。
3. `CURRENT_STATE.md` 中的最近一次交接快照。

状态文档与 HTML 不一致时，以 HTML 运行时完整性结果为准，并在本轮修正状态文档。

## 2. 开始前基线

开始任何新任务前：

- 阅读 `AGENTS.md`、`README.md` 和 `CURRENT_STATE.md`。
- 确认工作目录中存在且只维护 `knowledge-review-ui-prototype.html`。
- 记录 HTML 的字节数与 SHA-256。
- 通过 `file://` 打开页面，读取 `window.__knowledgeDataIntegrity`。
- 保存当前 items、realItems、demoItems、threads、turns、days、projectSpaces、projects 和警告作为变更前基线。
- 检查现有用户改动；不得清理、重置或覆盖来源不明的变化。

如果基线已有 integrity error，先只读定位；不要把新任务和旧错误混在同一批修改中。

## 3. 完整读取来源任务

使用 Codex 桌面的任务读取能力读取用户给出的 `codex://threads/...`。不得用网页搜索代替原任务。

读取规则：

- 从最新页或第一页开始，持续使用返回的 `nextCursor` 读取更旧轮次。
- 直到明确得到 `hasMore=false` 才算读取完成。
- 某些环境每页只返回一轮；数百轮任务可能需要较长时间，不得因慢而只取最近内容。
- 最终按 `startedAt` 恢复为从旧到新的顺序。
- 记录任务 ID、标题、工作目录提示、首末时间和总轮次。
- 无法访问来源、游标循环或读取中断时，不得写入半份历史；报告阻塞并保留现有文件。

为每轮建立临时核对账：

```text
order
turnId
startedAt / endedAt
status: completed | interrupted
userMessage
agentMessage phase: final | commentary
plan text
fileChange count
```

临时核对账只用于本轮工作，不另存为交付脚本或原始全文文件。

## 4. 人工分段与经验判断

先做覆盖账，再写内容：

- 完整阅读后先判断项目归属。已有项目且证据明确时自动归类；明确属于新项目时，在 `projectSpaces` 注册项目空间和子项目；命名或归属存在歧义时先询问用户。
- 项目空间用于归纳相关产品或业务域，子项目对应经验记录中的稳定 `project` 键；不增加第三级层级。
- 不得仅凭任务标题归类，也不得把无关项目塞入当前项目空间。

- 按目标、问题、方案转折和验收结果划分连续阶段，不按固定轮次数机械切分。
- 阶段必须使用连续且不重叠的轮次范围。
- 所有来源轮次恰好归入一条经验；覆盖数必须等于任务总轮次。
- 计划评审、实际实施、失败、用户否决、真实环境阻塞和最终验收要分清。
- 一个阶段跨日期时，会话时间线按日期和大于45分钟的间隔拆分。

经验类型只能使用：

- `incident`：事故、误判、失败或阻塞。
- `decision`：范围、架构、安全或产品取舍。
- `practice`：可重复使用的做法。
- `win`：经过证据支持的亮点或闭环成果。

高价值标记只用于关键事故、范围控制、接口治理、运行安全和架构转折，不得让多数记录都变成金色强调。

## 5. 摘要、原话与状态规则

每个逐轮记录必须区分：

- `summary`：Codex 人工整理的本轮摘要。
- `userQuote`：来自原任务的用户关键原话。
- `codexQuote`：来自原任务的 Codex 关键原话。
- `codexQuoteKind`：`final`、`process`、`plan` 或 `none`。
- `outcome`：本轮实际留下的结果，不能超出来源证据。

约束：

- 改写只能出现在摘要、结论和通俗团队表述中，不得标为原话。
- 读取接口没有暴露 final 时，commentary 只能标为 `process`；计划文本只能标为 `plan`。
- completed 但没有可核验回复时允许保留为空，并由完整性检查给出警告。
- interrupted 必须明确显示“本轮中断”，不得生成完成结论。
- 超长原话保留高信息量片段并加 `…〔原话节选〕`；默认单方节选不超过约1000字。
- 不复制627轮旧任务全文，不把新任务原文另存到独立文件。

## 6. 数据接口

不要依赖行号，使用下列稳定符号定位 HTML：

- `realThreadDefinitions`：真实任务 ID 与标题。
- `projectSpaces`：项目空间、子项目、显示名称和排序的唯一注册表。
- `realKnowledgeItems`：真实经验统一集合。
- `attachRealConversationTurns`：把逐轮记录挂到对应经验。
- `sanitizePortableConversationText`：运行时便携脱敏兜底。
- `validateKnowledgeItems` 与 `window.__knowledgeDataIntegrity`：完整性校验。
- `latestRealThreadId`：默认任务定位，不能硬编码旧任务。

新增任务时使用下一个连续 `REAL-NNN`，并把任务加入 `realThreadDefinitions`。可以沿用现有 stage config/builder 结构，但所有卡片、视图、日历、周报和对话档案仍必须消费同一 `knowledgeItems` 数据源。

项目注册结构：

```javascript
const projectSpaces = [
  {
    id: "stable-space-key",
    label: "项目空间显示名称",
    description: "该空间覆盖的业务或产品范围",
    order: 10,
    projects: [
      { key: "STABLE-PROJECT-KEY", label: "子项目显示名称", order: 10 }
    ]
  }
];
```

约束：项目空间 `id` 和子项目 `key` 必须全局唯一；每个子项目只能属于一个项目空间；经验中的 `project` 必须引用已登记的子项目键。新增注册项后，筛选、项目根系、搜索和完整性指标应自动更新，不得再硬编码下拉选项。

经验记录的最低结构：

```javascript
{
  id: "REAL-NNN",
  project: "已登记的稳定子项目键",
  type: "incident" | "decision" | "practice" | "win",
  status: "historical",
  date: "YYYY-MM-DD",
  title: "...",
  conclusion: "...",
  impact: "...",
  sourceSummary: "...",
  highValue: false,
  conversation: {
    sessions: [],
    dailySummary: {
      overview: "...",
      completed: [],
      corrections: [],
      nextActions: []
    },
    turns: []
  },
  timeline: [],
  excerpts: [],
  evidence: [],
  reusablePractices: [],
  avoidNextTime: [],
  reviewSignals: {},
  source: {}
}
```

真实来源必须包含：

```javascript
{
  kind: "codex-thread",
  threadTitle: "...",
  threadId: "...",
  threadUri: "codex://threads/...",
  turns: "001—005",
  turnIds: [],
  dateRange: "YYYY-MM-DD",
  threadVerified: true,
  currentProjectVerified: false,
  verified: true
}
```

逐轮行格式固定为：

```javascript
[
  threadTitle, order, turnId, startedAt, endedAt, status,
  userQuote, codexQuote, codexQuoteKind, summary, outcome,
  technicalNote, changedFiles, redactions
]
```

时间统一保存为带 `+08:00` 的 ISO 8601。`changedFiles` 是该轮 fileChange 中路径数量，必须为非负整数。

## 7. 便携脱敏

写入 HTML 前完成脱敏，不能只依赖运行时替换。新项目优先使用通用占位符：

- `<PROJECT_ROOT>`
- `<PROJECT_ARCHIVE>`
- `<LOCAL_PATH>`
- `<SERVER_LAN_IP>`
- `<EMPLOYEE_LAN_IP>`
- `<PUBLIC_IP>`
- `<SENSITIVE_ID>`

既有历史记录中的以下项目专用占位符继续有效，无需为统一命名而改写旧原话：

- `<AI_CS_ROOT>`
- `<AI_CS_CE_ROOT>`
- `<AI_CS_ARCHIVE>`

保留对判断有用的命令、错误码、端口角色和技术字段，但真实本机目录、局域网地址、公网出口、客户身份和凭据必须替换。脱敏后在 `redactions` 中留下说明。

动态内容继续使用安全 DOM 节点和 `textContent`；不得把原话、报错或证据作为未经处理的 `innerHTML` 插入。

## 8. 每日、周度与六维聚合

- 每条经验填写会话时段与 dailySummary，页面按同日经验自动聚合去重。
- 周度总结由每日总结按周一至周日聚合，不维护独立第二份周数据。
- 六维固定为推进产出、问题解决、决策质量、复用沉淀、风险控制、协作效率。
- 分数只能是整数1—5或 `null`；依据不足使用 `null`。
- 评分只描述留下的过程和证据，不能评价个人能力或绩效。
- 团队视图写普通同事能理解的影响与后续做法；深度视图保留技术证据和来源。

## 9. 修改与自检顺序

只使用可审计的局部编辑方式修改 HTML，优先 `apply_patch`。推荐顺序：

1. 加任务定义。
2. 加阶段经验配置与 builder 映射。
3. 将阶段加入 `realKnowledgeItems`。
4. 加逐轮数组。
5. 检查 JavaScript 语法和意外的行首 `+`。
6. 运行页面完整性检查。
7. 做真实 `file://` 回归。
8. 更新 `CURRENT_STATE.md`。

不得通过重写整个 HTML、批量格式化或创建中间脚本来完成接入。

## 10. 验收矩阵

### 数据完整性

- 新任务来源轮次总数等于对话档案轮次数。
- 轮次 ID 全局唯一；序号从1连续到任务总数。
- 完成/中断数量与来源一致。
- 每条经验的 source.turnIds 与 conversation.turns 一一对应。
- `window.__knowledgeDataIntegrity.errors` 必须为空。
- 预期 warning 可以保留，但必须在 `CURRENT_STATE.md` 准确记录。

### 功能回归

- 默认来源仍为真实线程。
- 默认项目范围为全部项目空间和全部子项目；两级项目筛选联动且影响所有页面、总结和图表。
- 新任务可从桌面列表和移动选择器打开。
- 搜索任务标题、原话、摘要、结果、轮次 ID、技术说明和六维依据均能命中。
- 团队/深度切换不改变搜索命中数量。
- 日历→对话、经验→对话、对话→经验正常并高亮关联轮次。
- 每日总结、周总结、雷达图和趋势图随筛选同步。
- 详情抽屉保留 dialog、焦点循环、Esc、遮罩关闭、滚动锁定和焦点返回。

### 离线与响应式

- 必须通过真实 `file://` 打开，不使用本地服务器代替。
- Console 无错误，Network 无请求，无外部资源。
- HTML 中不得新增 `fetch`、`XMLHttpRequest` 或动态数据的直接 innerHTML 赋值。
- 1440、1024、768、390、320 像素宽度均无页面级横向滚动。
- 如果环境有浏览器自动化能力，使用它验证；没有时直接双击并使用浏览器开发者工具人工核对，不创建永久测试脚本。

## 11. 更新交接状态与汇报

每次接入完成后，更新 `CURRENT_STATE.md`：

- 验收日期。
- HTML 字节数和 SHA-256。
- integrity 全部指标与警告。
- 项目空间、子项目及其经验数量汇总。
- 新任务标题、ID、轮次数、日期范围、经验范围和经验数量。
- 本轮 file://、网络、Console、响应式和跳转结果。

最终对用户只汇报：新增任务事实、经验范围、更新后总量、验收结论和 HTML 入口。不得把工作流内部临时账或长测试输出另存为文件。

## 12. 私有 GitHub 同步

当前文件夹可连接私有仓库 `666whatcanisay/precipitate`，但 GitHub 同步不是每次沉淀的默认步骤。

- 新任务验收完成后，默认只保留本地修改，不自动提交或推送。
- 只有用户明确要求上传、推送或同步时，才进入 GitHub 发布流程。
- 发布前确认仓库仍为 `PRIVATE`，重新运行完整性、脚本语法、外部资源和敏感信息检查。
- 使用 `git status -sb` 与差异检查确认范围，只显式暂存用户授权的文件，不使用不加判断的全目录暂存。
- 不把数据库、密钥、浏览器Profile、登录态、客户隐私、原始对话全文、临时截图或被沉淀项目源码提交到本仓库。
- 不自动改变可见性、启用 Pages、创建 Release、添加 CI 或配置定时同步。
- 推送完成后核对本地分支、远端分支和提交一致；需要便携性验收时，在独立临时目录进行干净克隆并比较 HTML SHA-256。
- GitHub README、预览图和仓库元信息只有在用户明确要求调整仓库展示时才修改。
