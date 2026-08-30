# Report 模板（LDP 阶段 3）

所有 task 的原子 commit 串行集成后，须按 Candidate Head 验证循环形成独立测试证据，再由 fresh whole-branch reviewer 静态审查完整分支与对应证据；tester 的 FAIL 与 reviewer 的 Critical / Important 经独立 implementer 修复后，必须对新 Candidate 执行受影响测试、计划内 regression sanity、必要的定向回归和 whole-branch review。阻断项全部关闭、证据充分且最终完整 diff 已 review 后，才冻结 Final Code Head，并按本模板写 `YYYY-MM-DD-<topic>-report.md`，放目标仓 `docs/development/`，commit 后进入阶段 4 的终审判定。report 是唯一终审工件，既是自动终审四条件的自检依据，也是升级人工时呈现的工件；目标是审核者不用翻 trace 就能裁决，证据给原始片段，不给转述。写 report 前先把 commit 前的 HEAD 固定为 Report Parent；report commit 的实际 hash 只能在 commit 后由人工呈现消息记录，不得回写 report 形成自引用。

```markdown
# <需求名> — 终审 report

> 仓库 / 分支：<repo> `feature/<...>`
> Base `<base-hash>`
> Final Code Head `<final-code-head-hash>`
> 最终代码审查范围：`<base-hash>..<final-code-head-hash>`
> Report Parent `<report-parent-hash>`（本次 report commit 的 parent；写 report 时固定为
> commit 前的 HEAD。发审前应等于 Final Code Head，或等于仅追加下述已枚举审计元数据
> commits 后的 HEAD）
> report 前审计 commit 边界：`<final-code-head-hash>..<report-parent-hash>`（只枚举
> report commit 前已存在的审计 commits；空边界写「无」）

## 1. 代码与 commit 边界

- commit 列表：<hash + 一句话> ×N（标注与 plan task 的对应关系）
- 新增 / 修改文件摘要：<路径 + 一句话；行数等量化摘要逐项抄自
  `git diff --numstat <Base>..<Final Code Head>`，须与本节总计自洽，不按记忆或跨文件
  相加填写；大文件走对象存储的列 manifest 位置>
- 原子 task commit 核对：<逐 task 列 payload、必要使用文档、plan / 项目状态工件是否
  同 commit；并行 task 的独立分支 commit 与串行集成结果>
- Final 后、report 前审计 commit：<逐个列出 `Final Code Head..Report Parent` 内已存在
  commit 的 hash / parent / 文件路径 / 用途；只能是已枚举的审计元数据，空边界写「无」>
- Report Parent 核对：<确认它是待创建 report commit 的 parent，且等于 Final Code Head，
  或等于仅追加上项已枚举审计元数据 commits 后的 HEAD>
- report commit 预期文件：<本 report 及同批审计状态元数据路径；不得包含产品代码、依赖
  或影响运行态的配置>
- report commit 记录：<report 内容不填写自身不可预知的 hash；commit 后由人工呈现消息
  记录实际 hash 及其 Report Parent，不为补 hash 回写或 amend report>
- 边界结论：<确认 `Final Code Head..Report Parent` 中的 commit 已全部枚举，且不含产品
  代码、依赖或影响运行态的配置；report commit 也只含上项预期审计文件。任一不满足时
  Final 失效，停止发审并返回新 Candidate 验证周期>

## 2. Candidate 验证与修复记录

| 周期 | Candidate Head | 测试结论 | whole-branch review | 阻断项 | 修复 commit / 下一 Candidate |
| --- | --- | --- | --- | --- | --- |
| C1 | `<hash>` | <PASS / FAIL / INCONCLUSIVE> | `Base..<hash>`；<结论> | <编号或无> | <hash / C2 / 无> |

<tester 的 FAIL 与 reviewer 的 Critical / Important 必须指向同一独立 implementer 修复
路径。逐项记录修复影响面、受影响测试、regression sanity、定向回归及新 Candidate；
测试执行者和 reviewer 只记录 / 审查，不参与修复。>

## 3. 测试证据（原始输出片段，不转述）

| 测试项 | 命令 / 断言来源 | 证据对应 Head | 证据位置与原始关键行 | 结论 | 复用理由与 reviewer 依据 |
| --- | --- | --- | --- | --- | --- |
| <编号> | <命令；plan 条目> | `<Candidate hash>` | <路径；原文> | <PASS / FAIL / INCONCLUSIVE> | <未复用，或为何后续变化不影响代码、依赖和运行态> |

<每项证据必须记录实际被测 Head。旧 Candidate 证据只有在 reviewer 明确证明后续变化未
影响对应代码、依赖和运行态时才可复用；不能证明就执行定向回归。瞬时通道错误与功能
结论分开记录，同参数重试不算新 Candidate 周期。`INCONCLUSIVE` 不计为 PASS，须列出
任务复杂度、观察边界、进度事件并进入待裁决项；明确契约违例始终记 FAIL。>

## 4. Whole-branch reviewer 结论

- review package：`<base-hash>..<final-code-head-hash>`
- reviewer：<fresh 独立上下文标识；不记录或约束具体 model / effort>
- 静态审查依据：<最终完整分支 diff、改动文件与第 3 节已有测试证据的引用>
- 已有测试证据审查结论：<证据是否覆盖 plan、失败 / 缺口如何定性；附 reviewer 原文>
- 证据复用审查：<逐项确认第 3 节复用理由；无复用写「无」>
- 静态复核声明：<明确本节是静态 review，不是测试；证据缺失须回验证循环由 tester 补齐>
- 总结论：<PASS / BLOCKED；附 reviewer 原文>

### Critical

<无则写「无」。有则逐条记录发现、独立 implementer 的修复 commit、新 Candidate 的
受影响回归证据，以及 reviewer 对最终完整 diff 的静态复核原文。>

### Important

<无则写「无」。有则逐条记录发现、独立 implementer 的修复 commit、新 Candidate 的
受影响回归证据，以及 reviewer 对最终完整 diff 的静态复核原文。>

### Minor ledger

<无则写「无」。每条使用稳定编号：`M<n>` / 发现 / 文件与行号 / 影响 /
推荐处置（修复、转 TODO 或留档）/ 当前裁决状态。>

## 5. 问题与未决证据

<所有 FAIL、Critical、Important 在 Final Code Head 冻结前必须关闭。这里记录不阻断
证据充分性的 INCONCLUSIVE、Minor 或经流程允许留待用户裁决的事项；无则写「无」。每条：>

- **问题 <n>**：<现象（原始输出）> / 影响：<波及面> / 建议处置：<修复 或 转 TODO，
  附理由>

## 6. 与 plan 的偏差

- 未完成项：<task/step + 原因；无则写「无」>
- 超范围项：<plan 外做了什么 + 已并回 plan 或转 TODO 的位置；无则写「无」>
- 弹性点收口：<plan「已知弹性点」逐条实测结论>

## 7. 编排事件（可选）

| 项目 | 记录 |
| --- | --- |
| plan task 数 | <数量；并行组与串行 task 的划分> |
| subagent 派发 | <implementer / tester / reviewer / 修复 / execution 分列；无或无法从工件核验时明确写出> |
| 阶段 3 问题与修复 | <FAIL、Critical、Important、Minor、INCONCLUSIVE 的数量和处置；修复轮次> |
| 正式 BLOCKED | <次数、原因、恢复动作；无则写「无」> |
| 中断 / 接管 | <中断原因、已核验进度、遗留进程 / 产物、旧写入者释放与新宿主接管动作；无则写「无」> |
| 用户纠正 / 裁决 | <次数、原文或可定位位置、对 plan / report / 实施的影响> |
| 跳过阶段 | <无则写「无」；有则写阶段、依据与风险> |

<本节用于后续自进化的可审计输入，不替代第 1–6 节的产品验证证据；不要为凑数臆造派发、
BLOCKED 或用户纠正。>

## 8. 建议验收动作

<用户 5–15 分钟内可完成的抽查步骤，含可直接粘贴的命令与预期输出。>

## 9. 待裁决项

### 自动终审四条件自检

| 条件 | 状态 | 依据 |
| --- | --- | --- |
| plan 验收标准全 PASS | <满足 / 不满足> | <第 3 节对应证据项> |
| whole-branch review 零 Critical / 零 Important | <满足 / 不满足> | <第 4 节结论> |
| 零 INCONCLUSIVE | <满足 / 不满足> | <第 3、5 节> |
| 无预授权外待裁决项 | <满足 / 不满足> | <下方待裁决项与 plan 预授权清单逐条比对> |

- 终审策略：<按 plan「终审策略声明」写：默认自动终审 / 本需求强制人工终审>
- 判定结论：<四条件全满足且未声明强制人工终审 → 按 plan 预授权自动合并；任一不满足或
  已声明强制人工终审 → 升级人工，列出未满足的具体条件与建议处置>

### 待裁决项

<超出 plan 预授权清单、需要用户拍板的问题，每条给推荐选项；无则写「无，全部在 plan
预授权范围内」。第 5 节的 INCONCLUSIVE 与 Minor ledger 中建议「转 TODO」或「留档」的
条目：落在预授权清单覆盖范围内的按预授权处置并在此记账，超出范围的逐条列出待批——后者
即「预授权外待裁决项」，会使自动终审条件不满足而升级人工。用户拒绝或要求修改时，明确
回到阶段 3：独立 implementer 修复 → 新 Candidate Head → 受影响测试与 regression
sanity → whole-branch review → 新 Final Code Head → 更新 report 后重新判定。>

回复示例：「<给用户可整句照抄的最短回复，如：通过，问题 1 按推荐转 TODO>」
```

## 使用要点

- report 只在所有 FAIL、Critical / Important 关闭、证据充分且最终完整 diff 完成 whole-branch review 后生成；此时须已确定 Base、Final Code Head 与分级结论。
- 第 3 节只贴原始输出，禁止「测试通过」式转述；失败项照贴，不美化，并逐项写证据 Head。
- plan 按依赖、写冲突、共享运行态和证据文件划分测试组；明显独立、非微小且日志隔离的子项由多个 fresh tester 并行，小型或共享状态测试不强制拆分。每项独立留证和定性；测试执行者发现的问题只记录不修复。
- 账单、网络等瞬时通道错误只阻断受影响项，核对现场后以完全相同参数重试；不定性为功能失败，也不得停止其它独立项。
- harness 断言必须追溯到 plan，禁止增加计划外验收。模型 / agent 长任务按复杂度声明观察边界并保存进度事件；仅超时或未分类 runtime error 且无契约违例时写 `INCONCLUSIVE`，不得计为 PASS 或掩盖明确失败；须进入待裁决项。
- reviewer 只静态审查完整分支与已有测试证据，不亲自修复；静态审查、证据核对和静态复核均不得表述为测试。证据缺失时回验证循环由 tester 补齐。
- tester 的 FAIL 与 reviewer 的 Critical / Important 必须由独立 implementer 修复；修复后形成新 Candidate，执行受影响测试、计划内 regression sanity 和必要定向回归，再 review 最终完整 diff。仍未关闭时不得冻结 Final Code Head 或生成 report。
- Final Code Head 后若产品代码、依赖或影响运行态的配置变化，必须回到新 Candidate 验证周期；`Final Code Head..Report Parent` 只允许已枚举的审计元数据 commit，report commit 只允许本 report 与同批审计状态元数据，均不得夹带产品变化。
- Report Parent 在创建 report commit 前固定。report 所在 commit 的实际 hash 在 commit 后由人工呈现消息记录，不写回 report，也不为记录该 hash amend report。
- agent 类型、model 与 effort 由宿主按任务选择，不在 report 中固定或比较。
- report 自身直接承载 reviewer 结论、ledger 与人工裁决，不新增伴生终审工件。
- 阶段 4 先按第 9 节自检自动终审四条件（验收标准全 PASS、零 Critical / Important、零 INCONCLUSIVE、无预授权外待裁决项）：全满足且 plan 未声明强制人工终审时，按 plan 预授权清单直接合并；任一不满足或 plan 声明强制人工终审时，停下升级人工，取得用户对当前 report / Final Code Head 的明确批准才能合并。
- 自检结论与逐条依据必须落在 report（第 9 节），可事后审计；不得只在会话里口头断言，也不得自行放宽条件或先合并后补报。用户要求修改时回阶段 3，不得直接修改后合并。
- `docs/development/` 中同一任务只保留 plan / report；`closeout` 不属于允许工件。
- 末行回复示例必须让用户可整句照抄完成裁决。
