---
name: self-evolution
description: Use at the end of a development task — after the requirement is finished, reviewed, and its knowledge is sunk — to review the session trace and propose durable improvements to docs, TODO, skills, or rules; also on explicit mid-task user request. Triggers on 复盘, 自进化, 改进 skill / rule, 任务收尾复盘.
---

# self-evolution — 自进化复盘

本 skill 是 LDP 阶段 5b 的默认组件，只定义触发时机、记录去处、frontmatter 格式与
收口验证。复盘方法（独立 subagent 取证、诊断口径与编排度量、P0 / P1 分级门槛与
确定性缺陷升级、提案异步 review）以 `ldp` 阶段 5b 为唯一出处，不在此复制。

## 触发时机

- **默认（收尾触发）**：LDP 阶段 5b——需求已合并、5a 知识沉淀完成后。
- **中途叫停触发（合法）**：用户在需求进行中明确发起复盘（如「先停下来进化一下
  skill 再续」）。此时只就被叫停的具体痛点产出窄范围提案，不替代收尾时的完整
  复盘；需求真收尾时仍要做一次默认触发的完整复盘。

需求未完成且用户未叫停时，不主动跑自进化。

## 记录去处与格式

- 每次复盘（无论采纳与否）都在项目声明的自进化记录目录落盘
  `YYYY-MM-DD-<topic>.md`（无绑定默认 `<repo>/docs/evolution-log/`），记录提案
  内容、trace 依据、裁决结果与理由。
- 提案目标分两类：(a) docs / TODO——按 `knowledge` skill 的规范落盘；(b)
  skill / rule——按项目绑定声明的规则位置落盘。
- 文件头 frontmatter：

  ```yaml
  ---
  date: YYYY-MM-DD
  trigger: 收尾 | 中途叫停 | 审核
  status: 待审 | 已采纳 | 部分采纳 | 驳回
  targets: [提案涉及的文件列表]
  ---
  ```

- **异步 review，不设闸门**：提案落盘并标记 `status: 待审` 即完成本阶段闭环，
  不在此等待人工裁决。用户异步批量裁决后才把明确批准的项写入对应文件并 commit、
  更新 status；裁决原文须在该记录的 trace 依据中可回溯。未收到明确裁决原文前不
  得改任何 skill / rule，也不得把 Agent 建议当作采纳。
- **语义不漂移**：落盘采纳项不得让被改的 skill / rule / 模板偏离其原本用途与职责
  边界；确需改变用途的按新需求立项。
- **小步可回溯**：每次落盘改动尽量小、只承载已裁决的提案，可用 git 单独回退；
  多条采纳项拆成可分别回退的改动。

## 收口验证

提案或采纳项落盘 commit 后，跑 `git status` 确认工作区干净、`git log -1` 确认
commit 存在——工作区不净即本步未完成。项目绑定声明了备份动作的（如本机 bundle
脚本）一并执行。

## 进阶预留

轻量版（人工复盘 + 提案）即可长期运转。frontmatter 的结构化记录为将来可选的
自动化诊断（读历史记录定位反复失败模式 → 针对性改 skill → 多候选评估）留接口；
启用阈值由项目自行决定（例如记录积累到一定数量且同类问题反复出现后），未达阈值
前保持人工提案制。

## Red Flags

- 复盘记录没落进自进化记录目录，或缺 frontmatter 的 `date` / `trigger` /
  `status` / `targets`。
- 落盘 commit 后未做 `git status` / `git log -1` 收口验证。
- 未收到用户明确裁决原文就改写 skill / rule，或把 Agent 建议当作采纳。
- 复盘由本次实现者自己做（违反 `ldp` 5b 的独立 subagent 要求）。
