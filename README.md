# LUI Development Plugins (LDP)

LDP 是一组面向 coding agent 的通用项目开发 skill，用一次 plan 人工闸门组织需求、
开发、验证、终审与收尾。它包含 8 个 skill 和 3 个任务模板，不绑定具体项目或宿主，
也不依赖 hooks 或 session-start 注入。

## Quick start

### Claude Code

```bash
claude plugin marketplace add 0614lsn/lui-development-plugins
claude plugin install lui-development-plugins@lui-development-plugins
```

安装后新建会话，直接描述需求，或显式调用：

```text
/lui-development-plugins:ldp 用 LDP 实现：为 CLI 增加 JSON 输出
```

### Codex 与其他宿主

```bash
git clone https://github.com/0614lsn/lui-development-plugins.git
```

将 `skills/` 下的目录安装到宿主的 skill 发现路径。完整插件包含全部 8 个 skill；
只需要专项能力时也可单独安装。

- Codex：`~/.agents/skills/` 或 `~/.codex/skills/`
- Claude Code 独立 skill：`~/.claude/skills/`
- Cursor：按本地插件方式加载仓库根目录（legacy adapter）

优先使用目录链接，避免本地副本与仓库版本漂移。安装后重启宿主或新建会话。

## 开发闭环

LDP 先在 feature 分支或独立 worktree 隔离需求，再执行五个阶段：

1. **Plan**：批量澄清，写 plan，取得本需求唯一一次常规人工批准。
2. **开发**：由隔离的 implementer 执行，主控按 task 形成原子提交。
3. **验证**：测试 Candidate，并对整个变更分支做 review。
4. **终审**：条件满足时按 plan 预授权自动合并；否则升级人工裁决。
5. **收尾**：知识沉淀、自进化复盘、事实审计与清场。

完整状态机、角色边界和终审条件见 [`ldp`](skills/ldp/SKILL.md)。

## Skills

| Skill | 用途 |
| --- | --- |
| [`ldp`](skills/ldp/SKILL.md) | 五阶段开发闭环与状态机 |
| [`leader`](skills/leader/SKILL.md) | 为 goal / 无人值守执行准备获批 plan 与启动指令 |
| [`project-loop`](skills/project-loop/SKILL.md) | 把 LDP 绑定到具体项目的作用域、仓库与收尾去处 |
| [`connectors`](skills/connectors/SKILL.md) | 维护外部资源通道及其权限纪律 |
| [`knowledge`](skills/knowledge/SKILL.md) | 整理 TODO、知识库、路由与跨会话记忆 |
| [`self-evolution`](skills/self-evolution/SKILL.md) | 在任务结束后产出可审查的流程改进提案 |
| [`neat-freak`](skills/neat-freak/SKILL.md) | 收尾事实审计、清理预览与清场 |
| [`readme-writer`](skills/readme-writer/SKILL.md) | 创建、精简或审查可验证的 README |

`ldp` 同目录包含 [plan](skills/ldp/plan-template.md)、
[implementer prompt](skills/ldp/implementer-prompt.md) 和
[report](skills/ldp/report-template.md) 三个模板。

## 项目绑定

LDP 只定义通用流程。项目自己的作用域、仓库路由、连接器、人工通道与收尾目录，
由基于 [`project-loop`](skills/project-loop/SKILL.md) 的薄绑定声明，不复制 LDP 正文。

## License

[MIT](LICENSE)。`leader` 与 `neat-freak` 改编自 `KKKKhazix/khazix-skills`，文件内保留
上游版权与许可告知。
