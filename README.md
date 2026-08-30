# LUI Development Plugins (LDP)

`LDP` 是我自研的三层五阶段通用开发插件，内部包含7个 skill 和 3个模板，编排「一个开发需求从立项到合并到
收尾」的完整闭环。不含 hooks，无 session-start 强注入。内核不绑定具体 coding agent
或项目；宿主只通过薄 adapter 负责发现 skill、派发独立 subagent 与呈现 plan 闸门。

**plan 批准是唯一常规人工闸门**：阶段 1 前批量澄清、裁决写进 plan「预裁决记录」，
原需中途确认的动作写进 plan「预授权清单」，plan 获批即一并授权。阶段 3→4 默认自动
终审，四条件（验收标准全 PASS、零 Critical / Important、零 INCONCLUSIVE、无预授权外
待裁决项）全满足才按预授权合并；任一不满足则停下升级人工。plan 可声明「本需求强制
人工终审」。获批后修改 plan 时，只有 **Contract change**（验收标准、范围、安全边界、
预授权清单、终审策略）需要重新人工批准。

主控可执行只读核验、分支 / worktree 管理、精确暂存、commit、状态工件回写与人工工件
呈现；实现、重型测试、SSH、环境安装、训练、数据处理和长日志调试必须进入独立
subagent 上下文。agent 类型、model 与 effort 由宿主原生 orchestrator 按任务选择，
LDP 不固定具体值，也不要求子任务配置与父任务相同或不同；subagent 禁止再嵌套派发。

## 七个 skill

| skill                                              | 角色          | 何时启用                                                    |
| -------------------------------------------------- | ----------- | ------------------------------------------------------- |
| [`ldp`](skills/ldp/SKILL.md)                       | L1 核心引擎     | 任何开发需求的五阶段闭环：隔离 → plan 闸门 → 开发 → 测试与 report → 终审合并 → 收尾 |
| [`project-loop`](skills/project-loop/SKILL.md)     | L2 项目绑定模式   | 把 LDP 绑定到具体项目：声明作用域、仓库路由、连接器、人工闸门与收尾去处                  |
| [`knowledge`](skills/knowledge/SKILL.md)           | 阶段 5a 默认组件  | 知识沉淀、TODO 规范、路由文件维护、跨会话记忆                               |
| [`connectors`](skills/connectors/SKILL.md)         | L3 通道注册表模式  | 外部资源（git 远端 / MR / 文档库 / 模型库 / 对象存储）的通道纪律与注册表骨架         |
| [`self-evolution`](skills/self-evolution/SKILL.md) | 阶段 5b 默认组件  | 收尾复盘、提案落盘与异步 review                                     |
| [`neat-freak`](skills/neat-freak/SKILL.md)         | 阶段 5c 默认组件  | 收尾六面审计与清场；或用户明确发起的独立「大扫除」审计                             |
| [`leader`](skills/leader/SKILL.md)                 | 阶段 0–1 可选前端 | 需求要走 goal / 无人值守执行时，把需求整理成获批 plan + goal 启动指令           |

## 五阶段一览

| 阶段  | 名称         | 要点                                                                                           |
| --- | ---------- | -------------------------------------------------------------------------------------------- |
| 0   | 隔离         | 目标仓建 feature 分支；已在 linked worktree 则只在其内建分支                                                  |
| 1   | Plan 闸门    | 批量澄清后按 plan-template 写 plan（含预裁决记录 / 预授权清单 / 终审策略声明），落盘 commit 后直接呈现 plan；唯一常规人工闸门，批准事实只写状态行 |
| 2   | 开发         | 同一工作树单写入；并行 task 各用独立 worktree / task 分支；主控原子 commit 后串行集成                                   |
| 3   | 测试与 report | Candidate Head 测试与 whole-branch review 循环；修复后定向回归，冻结 Final Code Head 才写 report               |
| 4   | 自动终审与合并    | 四条件全满足按 plan 预授权合并；任一不满足或声明强制人工终审则升级人工；要求修改则回阶段 3                                            |
| 5   | 收尾         | 5a 知识沉淀 → 5b 自进化复盘（独立 subagent，提案 `status: 待审`异步 review）→ 5c 六面审计与按预授权清场                     |

详细流程、纪律与 Red Flags 见 [`skills/ldp/SKILL.md`](skills/ldp/SKILL.md)。

## 工件约束

- `docs/development/` 中每个任务只保留 plan / report 二件套。
- plan 是唯一审批工件；report 是唯一终审工件，不生成伴生摘要文件。
- 完成结果在阶段 5a 沉淀到 knowledge 目录（默认 `docs/knowledge/` + `INDEX.md`，
  规范见 `knowledge` skill）。
- 同一工作树任一时刻最多一个写入者。文件集不相交本身不构成并行授权；同仓并行必须
  由 plan 同时声明独立 worktree、task 分支、不相交文件集、串行集成顺序与冲突处置。
- implementer 不 commit、不改完成状态。主控在其交还写入权后形成原子 task commit；
  commit 失败不得留下已完成 checkbox。
- 每项测试证据绑定 Candidate Head。tester 只记录问题不修复；FAIL 与 reviewer 的
  Critical / Important 走独立 implementer 修复路径。
- report 记录 Final Code Head、每项证据对应 Head、旧证据复用理由和 Final 后的审计
  commit 边界；静态审查与静态复核不得表述为测试。

## 安装

### Claude Code 插件

仓库包含官方约定的 `.claude-plugin/plugin.json` 与同仓 marketplace 清单。公开安装：

```bash
claude plugin marketplace add 0614lsn/lui-development-plugins
claude plugin install lui-development-plugins@lui-development-plugins
```

安装后 skill 使用 `lui-development-plugins:<skill>` 命名空间，例如
`/lui-development-plugins:ldp`。开发中的本地仓库也可临时加载：

```bash
claude --plugin-dir <repo>
```

### 独立 skill

Codex 或不需要插件版本管理的 Claude Code，也可把 `skills/` 下的目录装入宿主的 skill
发现路径（七个 skill 可按需全装或只装核心 `ldp`）：

- **Codex**：装入 `~/.agents/skills/`（或 `~/.codex/skills/`，按本机约定）。
- **Claude Code**：装入 `~/.claude/skills/`。
- **Cursor（legacy adapter）**：把整个插件目录复制到
  `~/.cursor/plugins/local/lui-development-plugins/` 后 Reload Window；删除该目录即
  回滚。

为避免多份副本漂移，独立 skill 推荐用目录链接而非复制。Windows（PowerShell）：

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.agents\skills\ldp" -Target "<repo>\skills\ldp"
```

macOS / Linux：

```bash
ln -s <repo>/skills/ldp ~/.agents/skills/ldp
```

装完重启宿主或新建会话，让 skill 列表刷新。

## 项目绑定

LDP 与项目无关：作用域、连接器、人工闸门与收尾去处由上层绑定声明，模式见
[`skills/project-loop/SKILL.md`](skills/project-loop/SKILL.md)；外部通道注册表的模式
见 [`skills/connectors/SKILL.md`](skills/connectors/SKILL.md)。迁移到其它项目或宿主
时只重写薄绑定与 adapter，不复制或分叉 LDP 流程正文。

## 许可与来源

MIT，见 [LICENSE](LICENSE)。`leader` 与 `neat-freak` 的方法论改编自
`KKKKhazix/khazix-skills`（MIT），这两个文件内保留上游版权与许可告知。
