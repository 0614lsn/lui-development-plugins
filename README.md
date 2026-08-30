# LUI Development Plugins (LDP)

自研精简 L1 通用开发引擎，替代 Superpowers v6.1.1。提供 `ldp`、可选 `neat-freak` 与可选 `leader` 三个 skill + 三个模板，编排「一个开发需求从立项到合并到收尾」的完整闭环；`neat-freak` 只负责可选的阶段 5c 收尾审计与清场，不新增 LDP 阶段；`leader` 是可选的阶段 0–1 规划与 goal 交接件，把需求整理成可无人值守执行的 plan 与 goal 启动指令，同样不新增 LDP 阶段。不含 hooks，无 session-start 强注入。共享内核不绑定具体 coding agent，宿主只通过薄 adapter 负责发现 skill、派发独立 subagent 与呈现 plan 闸门。

**plan 批准是唯一常规人工闸门**：阶段 1 前批量澄清、裁决写进 plan「预裁决记录」，原需中途确认的动作写进 plan「预授权清单」，plan 获批即一并授权。阶段 3→4 默认自动终审，四条件（验收标准全 PASS、零 Critical / Important、零 INCONCLUSIVE、无预授权外待裁决项）全满足才按预授权合并；任一不满足则停下升级人工。plan 可声明「本需求强制人工终审」。获批后修改 plan 时，只有 **Contract change**（验收标准、范围、安全边界、预授权清单、终审策略）需要重新人工批准。

主控可执行只读核验、分支 / worktree 管理、精确暂存、commit、状态工件回写与人工工件呈现；实现、重型测试、SSH、环境安装、训练、数据处理和长日志调试必须进入独立 subagent 上下文。agent 类型、model 与 effort 由宿主原生 orchestrator 按任务选择，LDP 不固定具体值，也不要求子任务配置与父任务相同或不同；subagent 禁止再嵌套派发。

## 五阶段一览

| 阶段 | 名称 | 要点 |
| --- | --- | --- |
| 0 | 隔离 | 目标仓建 feature 分支；已在 linked worktree 则只在其内建分支 |
| 1 | Plan 闸门 | 批量澄清后按 plan-template 写 plan（含预裁决记录 / 预授权清单 / 终审策略声明），落盘 commit 后直接呈现 plan；唯一常规人工闸门，批准事实只写状态行 |
| 2 | 开发 | 同一工作树单写入；并行 task 各用独立 worktree / task 分支；主控原子 commit 后串行集成 |
| 3 | 测试与 report | Candidate Head 测试与 whole-branch review 循环；修复后定向回归，冻结 Final Code Head 才写 report |
| 4 | 自动终审与合并 | 四条件全满足按 plan 预授权合并；任一不满足或声明强制人工终审则升级人工；要求修改则回阶段 3 |
| 5 | 收尾 | 5a 知识沉淀 → 5b 自进化复盘（独立 subagent，提案 `status: 待审`异步 review）→ 5c 六面审计与按预授权清场 |

详细流程、纪律与 Red Flags 见 [`skills/ldp/SKILL.md`](skills/ldp/SKILL.md)。

## 工件约束

- `docs/development/` 中每个任务只保留 plan / report 二件套。
- plan 是唯一审批工件；report 是唯一终审工件，不生成伴生摘要文件。
- `closeout` 不属于任务工件；完成结果在阶段 5a 沉淀到 knowledge 目录（无环境绑定时默认 `docs/knowledge/` + `INDEX.md`）。
- 同一工作树任一时刻最多一个写入者。文件集不相交本身不构成并行授权；同仓并行必须由 plan 同时声明独立 worktree、task 分支、不相交文件集、串行集成顺序与冲突处置。
- implementer 不 commit、不改完成状态。主控在其交还写入权后，将 task payload、必要使用文档、plan 状态和项目绑定的路由 / 状态工件做成原子 task commit；commit 失败不得留下已完成 checkbox。
- 每项测试证据绑定 Candidate Head。tester 只记录问题不修复；FAIL 与 reviewer 的 Critical / Important 走独立 implementer 修复路径，新 Candidate 至少执行受影响测试、regression sanity 和必要定向回归，再重新 whole-branch review。
- 同一未变化 Candidate Head 不得无理由重复测试。账单、网络等瞬时通道错误核对现场后使用完全相同参数重试；同参数重试不产生新 Candidate 周期，只记录通道阻断。
- report 记录 Final Code Head、每项证据对应 Head、旧证据复用理由和 Final 后的审计 commit 边界；静态审查与静态复核不得表述为测试。

## Cursor legacy adapter（兼容入口）

Cursor 入口仅是 legacy adapter，不是 LDP 共享内核的一部分。它把本目录 source 复制到 Cursor 用户级 local 插件目录；其它宿主应通过各自薄 adapter 复用同一 source。

安装或更新 legacy adapter：

```bash
bash mars-loop/scripts/install-ldp.sh
```

然后在 Cursor 中 Reload Window 生效。脚本幂等：source 与已装副本一致时直接退出；有差异时只替换该 adapter 的受管目录，并自检 diff 为空。

回滚该 adapter：

```bash
rm -rf ~/.cursor/plugins/local/lui-development-plugins/
```

然后 Reload Window。此操作只移除 Cursor 兼容副本，不修改本仓 source；需要恢复时重新运行安装脚本即可。

## Superpowers 回滚入口

若需回退到 Superpowers v6.1.1（source 原样保留在 `~/.cursor/plugin-sources/obra-superpowers-v6.1.1`）：

```bash
cp -R ~/.cursor/plugin-sources/obra-superpowers-v6.1.1 ~/.cursor/plugins/local/superpowers
```

然后 Reload Window。

## 可移植性

LDP source 随 mars-loop 私有 git 仓版本管理。LDP 本身与项目无关：项目专属信息 （作用域、连接器、人工闸门、知识沉淀去处）由上层（如 L2 mars-loop）绑定。迁移到其它项目或宿主时，只替换上层绑定与薄 adapter；不得复制或分叉 LDP 流程正文。
