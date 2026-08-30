---
name: ldp
description: Use when starting, planning, executing, or finishing a development requirement through the LDP 5-phase loop — feature-branch isolation, a single plan gate with human approval plus a pre-authorization list, isolated implementer subagents, Candidate validation and report, automatic final review that escalates to a human when its conditions fail, then closeout (knowledge, retrospective, audit and cleanup). Applies to any repository or project. Triggers on LDP 开发, 五阶段, plan 闸门, 自动终审, 收尾, 开发闭环, feature 分支开发流程.
---

# LDP — LUI Development Plugins 五阶段开发闭环

## 定位

L1 通用开发引擎：任何仓库、任何项目的「一个开发需求从立项到合并到收尾」的编排流程。LDP 只定义流程骨架与纪律，不绑定具体项目或 coding agent；项目专属信息（作用域、连接器、plan 闸门通道、收尾去处）由上层（如 L2 mars-loop）绑定，宿主能力由薄 adapter 映射，详见「环境绑定」节。

五阶段闭环（外加前置的阶段 0 隔离）：

| 阶段 | 名称 | 一句话 | 进入下一阶段的条件 |
| --- | --- | --- | --- |
| 0 | 隔离 | 目标仓建 feature 分支 | 分支已建并检出 |
| 1 | Plan 闸门 | 批量澄清后写 plan，落盘 commit，直接提交人工审核 | **审核通过**（唯一常规人工闸门） |
| 2 | 开发 | 同工作树串行，或在独立 worktree / task 分支并行；原子 task commit | 全部 task 已串行集成 |
| 3 | 测试与 report | Candidate Head 验证循环；冻结 Final Code Head 后写 report | report 已落盘 commit 且自动终审四条件已自检 |
| 4 | 自动终审与合并 | 四条件全满足即按 plan 预授权合并；任一不满足停下升级人工 | 合并完成 |
| 5 | 收尾 | 固定顺序 5a 知识沉淀 → 5b 自进化复盘 → 5c 收尾审计与清场 | 三个子步骤均已收口 |

## 使用方式

- 收到一个开发需求（新功能、bug 修复、重构、基建改造）即从阶段 0 起步，按序推进；阶段之间的推进条件见上表，**不允许跳阶段**。
- 本 skill 的执行者是主控（编排者）。主控可做只读核验、分支 / worktree 管理、精确暂存与 commit、状态工件回写和人工工件呈现；实现、重型测试、SSH、环境安装、训练、数据处理与长日志调试必须由宿主原生 orchestrator 派到独立 subagent。
- subagent 的 agent 类型、model 与 effort 由宿主按任务动态选择；LDP 不固定具体值，也不要求子任务与父任务配置相同或不同。
- 小改动同样走完整闭环，只是各阶段收窄：至少保留最小单 task plan 且必须过 plan 闸门 （可由人工当面裁决快速过闸）；阶段 3 测试不得省略。
- **plan 批准是唯一常规人工闸门**：阶段 1 之前把技术选型、范围与细节问题一次性批量提问并附推荐答案，裁决写进 plan「预裁决记录」；plan 内的「预授权清单」一次性授权本需求全部原需中途确认的动作，获批后不再产生常规中途确认。闸门通道由环境绑定提供；无绑定时使用当前宿主的人机交互界面呈现 plan 并等待用户裁决。
- 阶段 3→4 默认走自动终审：四条件全满足即按预授权合并，任一不满足则停下升级人工。plan 可声明「本需求强制人工终审」，声明后该需求必须取得用户对当前 Final Code Head 的明确批准才允许合并。
- 常规闸门之外仍保留的停止点（不是常规闸门，是异常升级）：连接器缺配置、正式 BLOCKED、plan 外破坏性或不可逆动作、范围溢出、自动终审条件不满足。命中任一即停下报备并请用户裁决。
- 等待人工裁决或无可监听完成事件的外部输入期间，不设置定时后台自唤醒，保持闸门停止直至收到输入。
- 主控宣告一项无需等待新的人工裁决或外部输入、可立即执行的下一步动作时，必须在同一 turn 发出对应工具调用；纯文字宣告不得单独成 turn，也不得作为 turn 结尾。该要求贯穿阶段 0–5，不限于阶段 2 的原子 task 收口。主控 turn 意外终止或会话恢复后，先以只读手段（`git log` / `git status`、权威状态工件）核对上次宣告的动作是否已落地；未落地则直接执行，不重复宣告。状态答复结尾不挂悬空宣告。

## 五阶段流程

### 阶段 0 — 隔离

- 为需求在目标仓建 feature 分支：`git checkout -b feature/<需求>`。
- 已处于 linked worktree 时，只在对应 worktree 内建分支，**禁止嵌套创建 worktree**。
- 阶段 2 如获准并行，独立 task worktree 必须由主控从仓库 common dir 创建到彼此独立的位置，不能嵌套在当前 worktree 中；其位置和 task 分支须先写入 plan。
- 产出：干净的 feature 分支，作为唯一集成分支；串行 task commit 直接进入该分支，获准并行的 task commit 只进入 plan 声明的 task 分支并按序集成，任何开发 commit 都不碰主线。

### 阶段 1 — Plan 闸门

- **写 plan 前先批量澄清**：把技术选型、范围边界、实现细节上一切需要用户拍板的问题一次性列出，每问附推荐答案与理由，一轮问完；用户裁决逐条写进 plan「预裁决记录」节。实施期不再就已裁决项复问；无待澄清项时在该节写明「无剩余待澄清项」。
- 按 [`plan-template.md`](plan-template.md) 写 `YYYY-MM-DD-<topic>-plan.md`，放目标仓 `docs/development/`，commit 到 feature 分支。
- plan 必须含：背景与目标、验收标准、预裁决记录、预授权清单、终审策略声明、task 列表（每 task 的目标 / 文件集 / 完成判据、并行组标注）、外部工具入口 dry-run sanity、已知弹性点、测试计划。每个并行 task 还必须写明独立 worktree 位置、task 分支、串行集成顺序和冲突处置；缺一即按串行执行。
- **预授权清单**是把「原本要在实施中途逐次确认」的动作提前枚举、随 plan 一次批准：commit / 合并纪律、要执行的运行态变更、要落盘的收尾工件、清场动作等，逐条写清对象与边界。清单外的破坏性或不可逆动作（删历史工件、`--force` 类操作、未声明的远端 push）不在授权范围内，出现时停下报备。
- **终审策略声明**：默认自动终审；涉及真实数据迁移、生产环境、安全边界或其它高风险面时，plan 显式声明「本需求强制人工终审」。该声明属合同项，实施期不得由主控自行降级。
- 修改流程语义的 task 必须在文件集中枚举权威 SKILL、关联模板及复制该语义的活动 adapter；完成判据须包含活动文件旧措辞 / 相反语义扫描与 source/template 逐项核对。去重 / 瘦身类改动还须反查被删约束的承接者，逐条给出新出处的命中，全库零命中即判内容丢失而非位置迁移。历史文档只作显式白名单，不因一致性扫描批量改写。
- **验收标准的承接核对**：plan 定稿前逐条核对每条验收标准点名或隐含的文件与语义面，都能指到某个 task 的文件集与某个测试项的实际范围，三者同宽；验收标准用范围词（活动面、全库）时测试项不得收窄为示例文件。核对不通过就改范围或补 task / 测试项。
- plan commit 后直接通过环境绑定的人工通道呈现 plan 本身；plan 是唯一审批工件，不另建伴生摘要文件。
- 批准事实写在 plan 顶部状态行，记录日期与裁决通道；获批后更新该行并 commit，不为批准事实新增 development 工件。
- **必须等审核通过才能进入阶段 2**；驳回则修改 plan 后重新发审。
- **plan 修订分类**：获批后改 plan 分两类。**Contract change**（改验收标准、范围、安全边界、预授权清单、终审策略）必须重新取得人工批准，批准前不得按新内容执行；证据细化、状态更新与合同内的实现细节调整由主控直接记录，不重新发审。分类不明时按 Contract change 处理。

### 阶段 2 — 开发

- 按 plan 的 task 派 implementer subagent，派发词用 [`implementer-prompt.md`](implementer-prompt.md) 模板填充（占位符：plan 路径、task 章节名、必要的前序 task 接口等额外上下文）。
- 实现、运维执行与其它实质工作都必须进入独立上下文；主控使用宿主原生 orchestrator 按任务选择 agent 类型、model 与 effort，不在 LDP 或派发词中硬编码，也不比较其与父任务配置是否相同。
- **同一工作树任一时刻最多一个写入者**。默认在 feature worktree 串行执行 task；文件集不相交本身不构成并行授权，未完整声明并行隔离信息的 task 一律串行。
- 同仓并行只允许 plan 显式标注的并行组，并同时满足：每个 task 使用独立 worktree 和独立 task 分支、implementer 文件集两两不相交、plan 写明串行集成顺序与冲突处置。主控先完成 worktree / 分支核验，再把每个 implementer 派到其唯一隔离位置；一个 task worktree 内仍只允许一个写入者。
- implementer 契约（模板已内置）：先核对自己的 worktree / 分支与 plan 一致；只读 plan 对应章节为需求源；只改声明文件集内的文件；实现 + 按完成判据自测；**不 commit**、不勾选完成状态；不得再派下级 agent。边界不符即报 `NEEDS_CONTEXT`，不得自行切分支或改造 worktree。
- **implementer 一律不自己 commit**：实现 + 自测后报回 status、改动文件清单、测试证据、concerns。
- implementer 停止写入并交还工作树后，主控在对应 worktree 核对报告，精确暂存 task payload、必要使用文档、该 task 的 plan 状态，以及环境绑定要求的项目路由 / 状态工件，形成一个**原子 task commit**。不得先 commit payload、再单独勾 checkbox 或补状态。
- 原子 task commit 失败时，主控必须把 plan checkbox 与关联状态恢复为未完成后再重试或报告，不能留下“已完成”假象。task 只有在该 commit 成功且（若使用 task 分支）已按 plan 顺序集成到 feature 分支后才算收口。
- **跨仓 task 的原子性代偿**：当同一 task 的 payload 与状态工件按环境绑定必须分属不同仓库、物理上无法进入单个 commit 时，允许拆为「payload commit + 状态 commit」两个提交。允许条件：plan 已预先声明该跨仓结构与两仓 Base，且两个 commit 在同一收口动作内连续完成。代偿要求：report 的原子 task commit 核对必须逐项列出两个 hash 与各自内容，并显式说明不存在「payload 已提交而状态未收口」或其反向的失真；任一 commit 失败时按上一条恢复未完成状态。本例外只适用于物理跨仓，不得用于同仓内为方便而拆分提交。
- 并行 task 由主控在对应 worktree 提交后，再按 plan 声明的顺序串行集成到 feature 分支。遇到冲突立即停止并执行 plan 的冲突处置；主控可处理状态工件的机械合并，涉及实现判断的冲突须交给独立 implementer，禁止把并行集成变成并发写入。
- 宿主若不能提供独立 subagent，则报告 BLOCKED；不得由主控接管实质工作，也不得通过嵌套 agent 绕过。
- 对正在运行的程序或 subagent，使用宿主原生的事件式等待 / 完成通知，不以固定时长闹钟代替；执行方记录可核验的进程、日志与产物位置，并向派发者返回成功、失败或中断状态。subagent 派发因外部瞬时错误（账单、网络等）或通知链路中断时，派发者先读中断 transcript，并核对遗留进程、日志与产物，再续接或重派；重派派发词注明已确认进度与需修正项，不臆断产物损坏，也不无信息从零猜测。

### 阶段 3 — 测试与 report

- 所有 task 的原子 commit 串行集成后进入显式验证循环。每个周期先冻结并记录精确的 **Candidate Head**；测试、证据与 whole-branch review 都必须绑定这个 Head。任何会影响产品代码、依赖或运行态的修改都会使当前 Candidate 失效，必须形成新 Candidate 周期。
- 对当前 Candidate，按 plan 中声明的依赖、写冲突、共享运行态和证据文件划分测试组。明显独立、非微小且日志互相隔离的子项，应同时派多个 **fresh 测试 subagent**（不带开发上下文）并行执行；小型或共享状态测试可由一个 tester 串行完成。每项证据独立记录 Candidate Head、原始输出位置与结论；测试执行者发现问题只记录不修复。
- 账单、网络等外部瞬时通道错误只阻断受影响测试项；核对现场后使用完全相同参数重试，只记录为通道阻断，不定性为功能失败，也不得停止其它独立项；同参数重试不产生新 Candidate 周期。同一未变化 Candidate Head 不得无理由重复测试。
- 每个测试 harness 断言必须追溯到 plan 的验收标准或测试项，禁止增加 plan 外验收。模型 / agent 长任务须按任务复杂度声明观察边界并保存进度事件；只有超时或未分类 runtime error、且没有明确契约违例时，结论为 `INCONCLUSIVE` 而非 `FAIL`。该结论必须附进度证据和观察边界并进入待裁决项，不能计作 PASS；已有明确契约违例始终是 `FAIL`。
- 当前 Candidate 的测试证据形成后，派 **fresh whole-branch reviewer**（不带实现上下文），以 `Base..Candidate Head` 审查完整分支和该 Head 的测试证据，按 Critical / Important / Minor 分级。reviewer 默认不亲自重跑测试；静态审查、证据核对与修复后静态复核均不得表述为测试或测试复跑。
- 测试执行者与 reviewer 均不得再派下级 agent。
- tester 的 `FAIL` 与 reviewer 的 Critical / Important 进入同一修复路径：主控另派独立 implementer 修复并形成原子 task commit；tester 与 reviewer 都不得参与修复。修复后冻结新的 Candidate Head，至少重跑受影响测试和 plan 内声明的 regression sanity，完成必要的定向回归，再对 `Base..新 Candidate Head` 及其证据执行 fresh whole-branch review。
- 旧 Candidate 的证据只有在 reviewer 明确证明后续修改未影响对应代码、依赖和运行态时才可复用；report 必须逐项记录证据对应 Head、复用理由与 reviewer 依据。无法证明时必须执行定向回归，不得用静态判断冒充测试证据。Minor 全量进入 report ledger。
- 当所有 `FAIL`、Critical 与 Important 已关闭，计划内证据均已充分记账，且最终完整 diff 已完成 whole-branch review，才把当前 Candidate 冻结为 **Final Code Head**。之后若产品代码、依赖或影响运行态的配置变化，Final Code Head 立即失效并回到新 Candidate 验证周期；Final 之后只允许 report、裁决与状态元数据等审计工件 commit，并须记录 commit 边界。
- Final Code Head 确定后，主控才按 [`report-template.md`](report-template.md) 写 `YYYY-MM-DD-<topic>-report.md`，放目标仓 `docs/development/`，commit，再进入阶段 4 的终审判定。report 是唯一终审工件，不另建伴生摘要文件；强制人工终审或自动条件不满足时，通过阶段 1 的人工通道直接呈现该 report。
- report 必须含：Base / Final Code Head、Candidate 周期与修复清单、每项测试证据对应 Head 及复用理由、whole-branch reviewer 分级结论、Minor ledger、审计 commit 边界、问题清单、与 plan 的偏差、建议验收动作、自动终审四条件自检、待裁决项、末行最短回复示例。

### 阶段 4 — 自动终审与合并（异常升级人工）

- **自动终审四条件**（对当前 Final Code Head 与其 report 逐条自检）：plan 验收标准全 PASS；whole-branch review 零 Critical / 零 Important；零 INCONCLUSIVE；无预授权外待裁决项。四条件全满足时，按 plan 预授权清单把 feature 合入主线并 push（无远端的私有仓仅本地合并），不再等待新的人工裁决。
- **任一不满足则停下升级人工**：不得自行放宽条件、不得把 AI 判断或测试结论当作批准，也不得先合并后补报。升级时呈现当前 report、未满足的具体条件与建议处置，等用户裁决。
- plan 声明「本需求强制人工终审」时不走自动通道：必须取得用户对当前 report 与 Final Code Head 的明确批准才允许合并。
- 自检结论与其依据（四条件逐条状态、走自动还是升级人工）必须落在 report 或 plan 状态行，可事后审计；口头结论不算记账。
- 用户拒绝或要求修改时，不得在阶段 4 直接改完即合并；须回到阶段 3，由独立 implementer 修复，形成新 Candidate Head，完成受影响测试、计划内 regression sanity 与 whole-branch review，重新冻结 Final Code Head 并更新 report 后重新判定。
- 非阻断项转 TODO 另立项属待裁决项：在预授权清单覆盖范围内可直接按预授权处置并记账；超出预授权范围时按上条升级人工。
- **合并授权只有两个来源：满足四条件的自动终审，或用户对当前 Final Code Head 的明确批准**；两者皆无时绝不允许合并 push。

### 阶段 5 — 收尾

push（或本地合并）完成后按固定顺序执行 5a → 5b → 5c，三个子步骤都要收口，不得跳过或互相替代。每个子步骤先用环境绑定的去处；无绑定时用下述默认去处，LDP 独立使用也能完整执行。

#### 5a 知识沉淀

- 为本需求写一篇完成结果记录，四要素齐全：**做了什么**、**结论**（达成什么、关键数据）、**产出位置**（路径 / 分支与 commit / 外部存储位置，具体到未来的人能直接定位）、**踩过的坑**（以后注意的死路与约定）。只写「做了什么」而缺其余三项即为不合格。
- 只写已发生的事实：记录写入时点已成立的状态，不预写尚未执行的收尾动作（如 5c 还没做就不写「已清场」）。同一事实在不同状态工件中不得给出相反状态。
- **无绑定默认去处**：目标仓 `docs/knowledge/YYYY-MM-DD-<topic>.md`，并在同目录 `INDEX.md` 登记一行（日期 | topic | 一句话结论 | 链接）。有环境绑定时按绑定的 knowledge 去处与索引。
- `docs/development/` 中每个任务只保留 plan / report 二件套；`closeout` 不属于允许的任务工件，完成结果一律进 knowledge 去处。

#### 5b 自进化复盘

- **必须由独立 subagent 复盘，不能是本次实现者**：材料按「plan → git status/log/diff → report / knowledge」的权威链取证，会话 transcript 与平台自动 Memory 只作补充，不承载唯一事实。
- 诊断反复踩的坑、用户纠正过的偏好、流程与模板的缺口（该触发没触发、指引不清、与实际冲突），并附编排度量（task 数、各类 subagent 派发数、阶段 3 问题数与处置、返工轮次、BLOCKED 次数、用户纠正次数、跳过的阶段及理由）。
- **分级门槛**：只立案 P0 / P1。P0 = 阻断流程或丢失工作；P1 = 高代价、反复出现或用户明确纠正过的行为模式。P2 及以下只记录出现次数，不立案。
- **确定性缺陷不得长期停留在 P2**：某项观察若在同一环节近乎每次复现（不是偶发抖动），即使每次都有绕行且未丢工作，累计出现 **3 次**后必须升级 P1 并产出「定位根因」的提案（可另立需求，不必当场修）。「有绕行、无损失」不构成继续按 P2 计数的理由。
- 提案异步 review、绝不自动改流程规则：提案落盘并标记 `status: 待审`即完成本阶段闭环，不在此设人工闸门；用户异步批量裁决后才落盘采纳，并保留裁决原文可回溯。skill / rule 类提案须给具体 diff + 理由 + trace 依据。
- **语义不漂移**：落盘采纳项时不得让被改的 skill / rule / 模板偏离其原本用途与职责边界；确需改变用途的按新需求立项，不在复盘落盘里顺带改写。
- **小步可回溯**：每次落盘改动尽量小、只承载已裁决的提案，并保持可用 `git` 单独回退；不做一次性大改，多条采纳项拆成可分别回退的改动。
- **无绑定默认去处**：目标仓 `docs/evolution-log/YYYY-MM-DD-<topic>.md`，文件头 frontmatter 至少含 `date` / `trigger` / `status`（`待审` / `已采纳` / `部分采纳` / `驳回`）/ `targets`。有环境绑定时按绑定的自进化 skill 与其记录去处。

#### 5c 收尾审计与清场

- 逐面盘点本次需求的影响面并给出状态、证据、动作或未闭合原因，六面为：**代码**、**运行态**、**文档**、**规则**、**记忆**、**工作区**。状态只用 `verified-current` / `changed-and-verified` / `pending` / `out-of-scope` / `not-applicable`，不硬凑完成。
- 清场（删分支 / worktree / 临时产物）授权来自 plan 预授权清单：六面汇报落盘后按预授权执行，删除只能由主控执行，禁止 `--force`；清单未覆盖的删除对象停下报备。
- 未闭合项必须落进环境绑定的跨会话登记入口（无绑定时落 5a 的 knowledge 条目并注明未闭合），不停留在汇报文本里。
- **无绑定默认去处**：无审计入口时使用轻量兜底——对 README / docs 做对齐检查、清点会话残留与工作区临时物，如实标记证据和缺口，不编造运行态或外部系统结果。绑定了收尾审计件（如 `neat-freak`）时由它组织六面汇报与清场状态机。

## Red Flags

出现以下任一情况即流程违规，须立即停下纠正。本节只收复盘（docs/evolution-log/）确认实际发生的失败模式；流程规则以「五阶段流程」正文为唯一出处，不在此镜像。新增条目必须附复盘出处。

- **主控越界**：主控接管实现、运维或其它实质工作，含以「收口」「应急」为名的下场。（2026-06-24 enforce-subagent-orchestration）
- **闸门僭越**：plan / report 未落盘 commit 就提交人工审核；把测试结论、AI 判断、通道连接状态或消息展示当作人工批准；自动终审四条件未全满足仍合并；plan 声明强制人工终审却未取得明确批准就合并。（2026-07-09 发审时序复盘 E2/E3、2026-07-27 loop-gate-slim）
- **授权越界**：执行 plan 预授权清单外的破坏性或不可逆动作；Contract change 未重新取得人工批准就按新内容执行；实施期就 plan 已裁决项反复复问。（2026-07-27 loop-gate-slim）
- **收口失真**：宣告的收口动作未与工具调用在同一 turn 完成；原子 task commit 失败后仍保留完成状态；完成状态与事实不符。（2026-07-08 remote-review P1、2026-07-21 host-entry-layering P1-T1/T3）
- **收尾缺项**：阶段 5 跳过 5a / 5b / 5c 中任一子步骤，或用其中一个替代另一个；知识条目缺结论 / 产出位置 / 踩坑；复盘由本次实现者自己做；未闭合项只留在汇报文本里。（2026-07-27 loop-gate-slim）
- **证据失真**：测试证据未绑定 Candidate Head；无 reviewer 影响分析就复用旧证据；`INCONCLUSIVE` 计作 PASS；把静态复核表述为测试。（2026-07-10 superpowers-v611 E2/E4）
- **流程语义漂移**：修改流程语义只改权威 SKILL、模板或活动 adapter 的其中一处，未完成枚举核对与旧措辞 / 相反语义扫描。（2026-07-02 loop-audit-and-corrections）

## 环境绑定

项目环境可选挂载 `neat-freak` 作为阶段 5c 的收尾审计与清场件；它不新增 LDP 阶段、
不复制 LDP 角色合同或状态机。未绑定时按普通 LDP 流程执行。需求明确要走 goal /
无人值守执行时，可选挂载 `leader` 作为阶段 0–1 的规划与 goal 交接件；它同样不新增
LDP 阶段、不接管状态机，未挂载时照常直接走 LDP。

LDP 与项目无关；项目专属信息由上层（如 L2 mars-loop）绑定，包括：

- **作用域**：哪些仓库 / 目录归此项目管，主线分支与合并纪律的例外约定。
- **连接器**：git 远端、MR / issue、文档库、模型库、对象存储等外部通道及其入口。
- **人工闸门**：阶段 1 的 plan 闸门（及自动终审升级人工时）使用哪个宿主人机交互通道呈现工件并接收明确裁决。
- **收尾去处**：阶段 5 三个子步骤的绑定对象——5a 的 TODO / knowledge 目录与索引、5b 的自进化 skill 与记录目录、5c 的收尾审计件与跨会话遗留登记入口。
- **宿主 adapter**：把 skill 发现、独立 subagent 派发、动态路由和人工交互映射到宿主原生能力；adapter 不复制 LDP 流程正文。

LDP 独立使用（无上层绑定）时，直接在当前宿主人机交互界面呈现 plan 并等待明确裁决，阶段 5 按各子步骤的「无绑定默认去处」执行，其余阶段与纪律不变。

## 典型时间线示例

一个需求「给 foo 仓加导出功能」的完整走法：

1. 阶段 0：在 foo 仓 `git checkout -b feature/export`。
2. 阶段 1：先把选型与范围问题批量问完（每问附推荐答案），再写 `docs/development/2026-07-13-export-plan.md`（含预裁决记录、预授权清单、终审策略声明、2 个 task、并行 task 的 worktree / 分支 / 集成顺序 / 冲突处置、测试计划），commit，直接呈现 plan → 人工批准。这是本需求唯一一次常规人工中断。
3. 阶段 2：主控为 task 1 / task 2 建独立 worktree 与 task 分支，各派 1 个 implementer；implementer 交还写入权后，主控在对应 worktree 各建原子 task commit，再串行集成 ×2。
4. 阶段 3：冻结 Candidate Head C1，fresh tester 按 plan 留证，再由 fresh whole-branch reviewer 审查 `Base..C1`；测试 FAIL 由独立 implementer 修复为 C2，重跑受影响测试和 regression sanity，并完成定向回归与 whole-branch review。阻断项关闭、证据充分记账后冻结 C2 为 Final Code Head，写 report 并 commit。
5. 阶段 4：自检四条件（验收全 PASS、零 Critical / Important、零 INCONCLUSIVE、无预授权外待裁决项）——全满足则按预授权合并 push；若 C2 留下一条 INCONCLUSIVE，则停下升级人工，用户要求调整就回阶段 3 形成新 Candidate。
6. 阶段 5：5a 写 `docs/knowledge/2026-07-13-export.md` 并登记 INDEX；5b 派独立复盘 subagent 落盘 `docs/evolution-log/2026-07-13-export.md`（`status: 待审`）；5c 六面汇报后按预授权清理 worktree 与分支（禁 `--force`），未闭合项登记留档。

## 模板文件索引

同目录三个模板，配合各阶段使用：

- [`plan-template.md`](plan-template.md) — 阶段 1 plan 的结构模板 （背景与目标 / 验收标准 / 预裁决记录 / 预授权清单 / 终审策略声明 / task 列表与并行 worktree 字段 / 外部工具入口 dry-run sanity / 已知弹性点 / 测试计划）。
- [`implementer-prompt.md`](implementer-prompt.md) — 阶段 2 派发 implementer 的 prompt 模板（占位符 `{{PLAN_PATH}}` / `{{TASK_SECTION}}` / `{{EXTRA_CONTEXT}}`）。
- [`report-template.md`](report-template.md) — 阶段 3 report 的结构模板 （Base / Final Code Head / Candidate 与修复清单 / 测试证据 Head 与复用理由 / whole-branch reviewer 分级结论 / 审计 commit 边界 / Minor ledger / 问题清单 / 与 plan 的偏差 / 建议验收动作 / 自动终审四条件自检与待裁决项 / 最短回复示例）。
