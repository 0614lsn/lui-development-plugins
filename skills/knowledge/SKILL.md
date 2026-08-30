---
name: knowledge
description: Use when recording project progress, maintaining TODO / knowledge base / router files, or preparing cross-session handoff — typically after completing a task or when TODO starts accumulating finished items. Triggers on 知识沉淀, TODO 整理, 知识库, 路由维护, ROUTER, 跨会话记忆.
---

# knowledge — 跨会话记忆：TODO、知识库与路由

项目的跨会话记忆怎么落盘：TODO 只留未完成，已完成沉淀进知识库，路由文件指路。
目标是「Agent 会忘，仓库不会忘」。本 skill 是 LDP 阶段 5a 的默认组件，也可独立
用于任何项目的日常记忆维护。

## 文件载体的边界

- `<repo>/docs/development/TODO.md`：**只放当前未完成待办**，是结构性索引，不是
  单次任务工件；每个开发仓至多一份，仓没有结构性 TODO 时不虚构。
- `<repo>/docs/development/YYYY-MM-DD-<topic>-plan.md` 与 `*-report.md`：单次任务
  的设计 / 验收与测试证据。每个任务只允许这二件套，不生成 brief / closeout。
- `<repo>/docs/knowledge/`：已完成事项的结论、产出位置与踩坑。
- 项目级路由文件、决策目录、自进化记录目录：具体位置由项目绑定声明（典型为基建
  仓 `docs/ROUTER.md`、`docs/decisions/`、`docs/evolution-log/`）。

判断归属：任务进行中的设计与验收证据进 plan / report；完成结果进对应仓
`docs/knowledge/`；跨仓或流程级决策进决策目录；流程改进提案进自进化记录（见
`self-evolution` skill）。

## 权威读取链

任何新会话或新写入者按固定顺序只读核验后再接续：

1. 项目路由文件；
2. 当前任务 plan（或目标仓 TODO）；
3. 目标仓 `git status` / `git log` / `git diff`；
4. 当前 report，以及相关 knowledge / decisions。

会话 transcript 与平台自动 Memory 只帮助理解，不保存唯一事实，也不覆盖文件与
git 证据；会影响接续的信息必须写回上述权威链。

切换写入者前，当前方优先完成正在收口的 task commit；不能提交时保持未完成状态，
诚实记录停止点、未提交文件集、验证缺口与下一条安全动作，不把完成状态写入路由。

## TODO.md 规范

- 只列未完成。完成一项立即移除并沉淀进知识库，不得把已交付长期堆在 TODO。
- 每条待办最小结构：

  ```markdown
  ## <待办标题>
  - 现状/目标：<一两句说清要做什么、为什么>
  - 验收/卡点（可选）：<完成判据，或当前被什么阻塞>
  ```

- 权威源、环境约定、命令细节不写死在 TODO；真正执行时按需从路由 / 连接器 /
  知识库临时取用。
- 保留顶部一行 `最后更新：YYYY-MM-DD`。

## 知识库规范

每完成一个需求写一篇 `<repo>/docs/knowledge/YYYY-MM-DD-<topic>.md`，四要素齐全：

```markdown
# <做了什么：任务/需求名>

- 结果/结论：<达成了什么，关键数据/结论>
- 产出位置：<代码路径 / 分支与 commit / 外部存储位置，具体到能直接定位>
- 踩过的坑（以后注意）：<避免重复踩的经验、死路、约定>
```

并在 `docs/knowledge/INDEX.md` 登记一行：
`YYYY-MM-DD | <topic> | 一句话结论 | 链接`。

三问自检：(1) 只写了「做了什么」却缺结论 / 产出位置 / 踩坑？补全。(2) 私有决策
是否误写进会随仓 push 的 docs？挪到项目私有位置。(3) 产出位置能否让未来的人
直接定位？

**只写已发生的事实**：记录写入时点已成立的状态，不预写尚未执行的收尾动作——
5a 早于 5c 清场，写入时分支 / worktree 一律写「保留待清场」，清场实际完成后才由
5c 补报改写。同一事实在不同状态工件中不得给出相反状态。

## 路由文件维护

路由文件是跨需求共享的控制面（任务看板，不是变更日志）：

- **立项即登记**：在「进行中」登记分支 / plan 路径 / 预计里程碑；收尾时移出，
  收尾入口只指向对应 knowledge 条目，不用 development 下的工件代替沉淀。
- **遗留项必须落路由**：LDP 5c 产生的未闭合项逐条写入「悬而未决」，标注立项
  状态与唯一恢复 / 复现入口；汇报文本、transcript 与平台 Memory 不作唯一载体，
  需求已合并不构成省略登记的理由。
- **写入纪律**：路由更新只在主 checkout 的主线上精确提交，写入前核验工作树除
  本次状态工件外无未提交差异；不得在 feature 分支改写路由。feature 的原子 task
  commit 不含路由；task commit 成功且发生需要跨会话公开的状态变化时，才在主线
  做只含路由的精确投影 commit。主线不可写或投影失败时记录「路由待同步」，并在
  下一 task、Candidate 冻结或切换写入者之前停下补齐，绝不回退到 feature 写状态。
- **payload 例外**：交付物本身就是路由文件结构 / 措辞的需求，plan 必须显式声明
  受影响章节；payload 随需求 worktree 走正常 LDP 流程，开工时只在主线做一次
  「进行中」登记投影，期间不再做其它状态投影，合并后由主控补齐收尾投影。
- **看板密度**：「最后更新」只一行，不串联历史；「进行中」只留真未完成；「最近
  完成」最多 5 条、每条一行只指向 knowledge；「悬而未决」只留未闭合项；自进化
  只留目录指针。存在其它未完成需求正在读写路由时，不主动做非必要的结构性重排。

## Red Flags

- 已完成事项还留在 TODO.md；知识条目缺结论 / 产出位置 / 踩坑。
- 私有决策写进会 push 的仓；产出位置含糊到无法定位。
- 新任务在 `docs/development/` 产出 plan / report 之外的工件；完成结果留在
  development 不沉淀。
- 在 feature 分支改写路由文件；「路由待同步」未清零就继续下一 task、冻结
  Candidate 或切换写入者。
- 未闭合项只留在汇报文本、transcript 或平台 Memory，没落进路由「悬而未决」。
- 把已完成需求的 hash 链、Candidate 史或验收数字写进路由看板。
