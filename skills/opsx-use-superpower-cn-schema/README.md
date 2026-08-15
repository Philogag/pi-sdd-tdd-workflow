# superpowers-bridge-cn（简体中文版）

将 [superpowers-bridge](https://github.com/JiangWay/openspec-schemas/tree/main/superpowers-bridge)
整理的**简体中文** OpenSpec schema，并融合本仓库 SDD（Spec-Driven Development）
工作流 prompt（`prompts/stdd-*.md`）的行为约束。

- **目标读者**：想通过 OpenSpec 引入「brainstorm → 提案 → 设计/规格 → 计划 → 实现
  → 验证 → 回顾」全流程、且希望全中文产出物的团队。
- **schema 名**：`superpowers-bridge-cn`
- **版本**：见 `VERSION`（`1.0.0`）。OpenSpec baseline 1.4.1，Superpowers v5.1.0。
- **安装方式**：把本目录复制到目标项目 `openspec/schemas/superpowers-bridge-cn/`，
  见 [SKILL.md](./SKILL.md#安装在目标仓库内)。

---

## 1. 为什么需要它

内置 schema（spec-driven / workspace-planning）把 OpenSpec 当「规格仓库」用，
对「怎么从想法走到实现」不做约定。Superpowers 提供了一套成熟的**技能工作流**
（brainstorming → writing-plans → subagent-driven-development → TDD → code-review
→ finishing-a-development-branch），但它与 OpenSpec 是两套独立体系。

bridge 把两者接起来：**OpenSpec 负责强制 artifact 依赖关系与变更追踪，Superpowers
负责实现时的方法论**。本中文版在 bridge 基础上，进一步融入了本仓库
`stdd-propose / stdd-plan / stdd-apply / stdd-archive` 的行为细节（对话纪律、
暂停条件、提交纪律、文件头元数据等）。

## 2. 工作流总览

```
brainstorm ─→ proposal ─→ specs ─┐
                                  ├─→ tasks ─→ plan ─→ [apply] ─→ verify ─→ retrospective
                    design  ─────┘
```

| 阶段 | 产出物 | 说明 |
|---|---|---|
| 1 | `brainstorm.md` | 对话收敛，输出重定向到 `openspec/changes/<name>/`（**不写 `docs/superpowers/`**） |
| 2 | `proposal.md` | 变更提案（Why 硬限制 50–1000 字符） |
| 3a | `specs/<capability>/spec.md` | Delta spec（ADDED/MODIFIED/REMOVED/RENAMED） |
| 3b | `design.md` | 技术设计（与 specs 并行，依赖 proposal） |
| 4 | `tasks.md` | 任务清单，与 plan 一一对应 |
| 5 | `plan.md` | 实施计划（微步 checklist，供 subagent 执行） |
| 6 | apply | 用 git worktree + subagent-driven-development 实现 |
| 7 | `verify.md` | 7 项检查 + Overall Decision |
| 8 | `retrospective.md` | 证据驱动的回顾（PR 之前写） |

> apply 后**还有** verify 与 retrospective，是 bridge 与 spec-driven 最大的时序差异：
> 规格闭环之后，实现闭环也要可验证、可回顾。

## 3. 快速流程（daily-driver）

| 指令 | 做什么 |
|---|---|
| `/opsx:new <name>` | 新建变更骨架 |
| `/opsx:ff` | 直达快速流程（spec 就绪时省去 brainstorm/proposal） |
| `/opsx:continue` | 在既有 change 中生成下一 artifact |
| `/opsx:apply` | 进入 apply 流程（worktree + subagent + TDD + code-review） |
| `/opsx:verify` | 生成并填写 verify.md |
| `/opsx:archive` | 归档变更（`openspec archive -y`） |

## 4. 进入与离开的判断（entry / exit gates）

### 何时**不**走 opsx（直接 PR）

| 情境 | 直接 PR? |
|---|---|
| 新功能 / 新 capability / 架构变更 / breaking change | ❌ 走 opsx |
| Bug fix（不变更合约）/ 测试补写 / linter 规则 / 非破坏性升级 / typo / 文档 / config 微调 | ✅ 直接 PR |

原则：**流程仪式跟风险成正比**。

### Verbal brainstorm 升级到 opsx 的 5 条判据

5 条**全满足**才升级（任一缺则继续 brainstorm）：

1. **Scope 锁定** —— 一句话讲清「包含/不包含什么」
2. **主要设计分歧已收敛** —— 替代方案选过，TBD 有明确 owner 与影响面
3. **跨系统依赖盘点过** —— 就绪 / 暂 mock / 真未知，三选一讲得清
4. **验收条件可陈述** —— 具体 pass 条件
5. **对话进入收敛** —— 最近几轮在 confirm 不在发散

### Front-door 反模式（别做）

- 让 brainstorming 写到 `docs/superpowers/specs/`
- 让 writing-plans 写到 `docs/superpowers/plans/`
- TBD 没收敛就升级到 opsx
- 对 bug fix / typo 也建 change

## 5. Apply 走查（6 步）

1. **Pre-flight**：确认 plan 已归档（artifact 状态），检查必需技能可用
   （brainstorming / writing-plans / using-git-worktrees / subagent-driven-development
   / test-driven-development / requesting-code-review / finishing-a-development-branch）
2. **using-git-worktrees**：从 main 建 `feat/<change-name>` 隔离工作区
3. **subagent-driven-development**：按 plan 微步执行；传递 TDD 与 code-review 依赖
4. **verify**：实现完成后运行 `openspec validate --all --json` + 7 项检查
5. **retrospective**：在开 PR **之前**写（§0 Evidence 先行）
6. **finishing-a-development-branch**：收尾、开 PR、合入

## 6. 设计触点（与 SDD prompts 的融合点）

1. **Skill-name PRECHECK**：brainstorm / apply 开头检查对应技能存在（安装性错误
   尽早暴露）
2. **Schema-level vs prompt-level**：技能工作流放 schema（artifact instruction），
   项目环境类约定放 prompt —— 本中文版把 SDD 的行为细节织入 schema instruction
3. **传递依赖显式化**：apply 显式传递 test-driven-development 与
   requesting-code-review（subagent 需要知道）
4. **Opinionated 边界**：桥接层 `opinionated: true` —— 需要 subagent 平台，
   无 manual fallback（OpenSpec 官方亦以 subagents 为执行主体）
5. **Evidence-based PRECHECK**：verify / retrospective 的 PRECHECK 用
   `git log` / `grep` 计数做量化判断，不靠主观
6. **时序错位 artifact**：verify 与 retrospective 在 apply 之后才产生，
   但在 proposal 阶段就存在于图中 —— 提前声明，避免实现完成后才想起要写

## 7. CLI cheat sheet

| 目的 | 命令 |
|---|---|
| 列出可用 schema | `openspec schemas` |
| 验证 schema | `openspec schema validate superpowers-bridge-cn` |
| 新建变更 | `openspec new change <name>` |
| 查看状态 | `openspec status --change <name> --json` |
| 生成下一 artifact | `openspec instructions --change <name>` |
| 校验全部 spec | `openspec validate --all --json` |
| 归档变更 | `openspec archive -y`（可用 `--skip-specs` 跳过 spec 校验） |

> 中文输出：`openspec-cn` 为等价替代（同一 CLI，本地化输出）。

## 8. 与 spec-driven 的差异

| 维度 | spec-driven | superpowers-bridge-cn |
|---|---|---|
| 入口 | 直接 `proposal` | `brainstorm`（先收敛对话） |
| plan 粒度 | 任务列表 | 微步 checklist（可执行粒度） |
| apply | 仅 `plan` 前置 | `plan` 前置 + worktree / subagent / TDD / code-review 集成 |
| apply 之后 | — | verify + retrospective（证据驱动） |
| 语言 | 简体（openspec-cn） | 简体 |

## 9. 版本识别

| 层 | 位置 | 语义 |
|---|---|---|
| schema major version | `schema.yaml` 的 `version: 1` | 变更图结构的破坏性级别 |
| bundle VERSION | `VERSION`（`1.0.0`） | 本技能包整体版本 |

**兼容性**：

| schema | OpenSpec | Superpowers | 日期 |
|---|---|---|---|
| v1 | 1.4.1 | v5.1.0 | 2026-06-10 |

## 10. 升级路径

- 拉取本技能包新版本 → 复制覆盖目标项目
  `openspec/schemas/superpowers-bridge-cn/`（模板与 schema.yaml 一起更新）
- schema major version 变更时，先跑 `openspec validate --all --json` 确认既有
  change 不受影响
