---
name: project-loop
description: Use when binding the LDP five-phase loop to a concrete project — declaring scope, repo routing, worktree isolation, connectors, human gate and closeout destinations — or when running a development task under such a project binding. Triggers on 项目绑定, loop 绑定, 立项入口, project loop, L2 绑定, 项目开发入口.
---

# project-loop — LDP 的项目绑定层（L2 模式）

LDP（`ldp` skill）是与项目无关的开发引擎；本 skill 是把它绑定到具体项目的 L2
模式：一个项目绑定应当声明什么、怎么路由，以及所有绑定都必须遵守的通用不变量。
阶段状态机、角色合同、并行隔离、原子 task 收口与证据规则的唯一出处是 `ldp`；
绑定只写差量，不复制流程正文。

每个采用 LDP 的项目维护自己的薄绑定 skill（按本模式裁剪）；本文件自身也可在
无绑定项目中直接按通用不变量使用。

## 绑定必须声明的六项

1. **作用域**：哪些仓库 / 目录归本项目管（allowlist，不得扩大）；基建仓、学习仓、
   第三方仓等例外逐个显式声明，含各自的主线分支与合并纪律。
2. **仓库路由**：总路由文件的唯一位置（跨需求状态与「进行中」登记的权威入口），
   以及从入口到目标仓需求源（TODO / 当前 plan）的定位顺序；路由解析用已核实的
   绝对路径，不从业务仓 cwd 相对拼接。
3. **连接器**：项目的外部资源通道注册表（模式与纪律见 `connectors` skill）；
   需要但未配置的通道在开发前提醒用户配置，不硬来、不静默降级。
4. **人工闸门**：plan 闸门与终审升级人工使用哪个宿主人机交互通道（纪律见
   `connectors` 的「人工通道纪律」节）。
5. **收尾去处**：5a 知识沉淀、5b 自进化复盘、5c 审计清场分别绑定哪个组件或项目
   自有去处（默认依次为 `knowledge` / `self-evolution` / `neat-freak`）。
6. **宿主 adapter**：skill 发现、独立 subagent 派发、动态路由与人工交互如何映射
   到宿主原生能力；adapter 不复制流程正文。

## 入口判定（先于一切流程）

- **只读审查 / 解释 / 诊断**：只做回答所需的读取与汇报；不建分支、不写工件、不
  进入合并或沉淀。用户明确要求实施时才从 LDP 阶段 0 启动。
- **实施变更**：按绑定声明的路由逐仓定位后启用完整 LDP；明确范围外的目标不启用；
  混合目标只对作用域内子集启用，Git 操作始终在已定位的目标仓内执行。
- 单文件且无歧义的小改可使用最小 single-task plan，但隔离、plan 闸门与阶段 3
  均不可省略。

## 通用不变量（任何绑定都遵守）

- **权威读取链**：跨会话接续按绑定声明的固定顺序只读核验（典型为
  `路由文件 → 目标仓 TODO / 当前 plan → git status/log/diff → report / knowledge /
  decisions`）；会话 transcript 与平台 Memory 只作补充，不承载唯一事实，不覆盖
  文件与 git 证据。载体规范见 `knowledge` skill。
- **单一写入者**：同一工作树任一时刻最多一个写入宿主。切换宿主前由原写入者完成
  正在收口的 task commit；不能提交时保持未完成状态，记录停止点、未提交文件集、
  验证缺口与下一条安全动作，不把完成状态写入路由；新宿主按权威读取链核验后才
  接管。
- **基建仓 worktree 模式**（项目含自身基建仓时采用）：主 checkout 固定检出主线，
  只承受跨需求状态工件的精确提交、需求 worktree 的创建 / 核验 / 清理与终审后
  合并；需求的 plan、实现、测试与 report 全部在主 checkout 同级独立 worktree
  （`feature/<slug>`）完成，禁止嵌套 worktree。冻结 Candidate 前核对主线是否
  前进，含已合并的实现变更时先把主线集成回 feature 再重跑阶段 3。运行中的宿主
  只读主 checkout 的已合并版本，需求 worktree 内的改动合并前不即时生效属预期。
  合并后的 worktree / 分支清理按 plan 预授权清单执行，不使用 `--force`。
- **状态工件隔离**：跨需求共享的路由 / 状态文件不进入 feature 分支的原子 task
  commit；其投影协议与看板密度由 `knowledge` skill 的「路由文件维护」节约束。
- **Red Flags 分工**：绑定 skill 只收项目特有的违规模式（越出作用域、绕过连接器、
  状态工件越界等），且每条附依据出处；流程类违规（闸门、角色、证据、收口）以
  `ldp` 的 Red Flags 为唯一出处，不镜像。

## 迁移性

迁到新项目：复用 `ldp` 与各组件 skill，按本模式重写一个薄绑定（六项声明 +
必要例外），不复制任何流程正文。迁到新宿主：由薄 adapter 映射入口与宿主能力，
绑定与引擎正文保持不变。
