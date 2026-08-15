# sdd-tdd-workflow

用于分享 **prompts** 和 **skills** 的 [pi package](https://pi.dev/docs/latest/packages)，围绕 SDD（Spec-Driven Development）+ TDD（Test-Driven Development）工作流。

由 OpenSpec 和 Superpowers 驱动。

> **推荐路径**：使用下方「openspec schema 工作流」（`superpowers-bridge-cn`，简体中文），
> 由技能（`opsx-init` / `opsx-use-superpower-cn-schema`）+ OpenSpec CLI 驱动。
> 原有的 `prompts/stdd-*.md` 工作流**即将弃用**（见文末）。

## 结构

```
sdd-tdd-workflow/
├── package.json    # pi 清单：声明 prompts 与 skills 目录
├── prompts/        # ⚠️ 即将弃用：stdd-*.md prompt 模板（见文末）
└── skills/         # 技能（每个技能是一个含 SKILL.md 的目录，按需加载）
    ├── opsx-init/                        # 初始化 OpenSpec 工作流
    └── opsx-use-superpower-cn-schema/    # 简体中文 openspec schema 技能包
```

- `prompts/`：每个 `.md` 文件是一个 prompt 模板，文件名（不含 `.md`）即命令名，编辑器里输入 `/name` 展开。
- `skills/`：每个技能是一个包含 `SKILL.md` 的目录，按需加载。

---

## 推荐工作流：openspec schema（superpowers-bridge-cn）

一个变更（change）走完整生命周期：

```
brainstorm ─→ proposal ─→ design ─┐
                        └── specs ─┴→ tasks ─→ plan ─→ [apply] ─→ verify ─→ retrospective ─→ archive
```

DAG 依赖由 CLI 强制（`openspec instructions` 会拦截缺依赖的产出物），
每个阶段产出物位于 `openspec/changes/<change-name>/`。

### 完整过程示范（以 `add-user-auth` 为例）

以「用户输入 → 技能调用 → 产出物」的形式描述一个完整 cycle。

#### ① 初始化仓库（一次性）

- **用户输入**：`/opsx-init`（或「帮我把这个仓库初始化成 openspec 工作流」）
- **技能调用**：agent 加载 `opsx-init` → 先确认 openspec CLI 可用（缺失则引导安装，无权限自动回退
  `~/.local/bin`）→ 运行 `openspec init --tools agents` → 询问是否接入中文 schema，确认后复制
  `superpowers-bridge-cn` 到 `openspec/schemas/` 并设为默认 → `openspec schema validate` 验证
- **产出**：`openspec/`（config.yaml）、`.agents/skills/`（6 个 openspec 技能）

#### ② 提议变更（brainstorm + proposal）

- **用户输入**：`/opsx:propose` 或「我想给系统加用户认证，支持邮箱+密码登录」
- **技能调用**：agent 加载 `opsx-use-superpower-cn-schema` 获取流程导航 → 运行
  `openspec new change add-user-auth` → 生成 `brainstorm` 指令 → 加载 `superpowers:brainstorming`
  （PRECHECK 校验技能可用）→ 用 `ask_user_question` 逐个澄清需求 → 给出 2-3 个备选方案与取舍
  → 收敛后写 `brainstorm.md` → 继续生成 `proposal` 指令 → 写 `proposal.md`
- **产出**：`openspec/changes/add-user-auth/{brainstorm.md, proposal.md}`

#### ③ 设计与规格（并行）

- **技能调用**：agent 生成 `design` 指令 → 把 brainstorm 重组为结构化设计
  （Context / Goals / Non-Goals / Decisions / Risks / Migration）→ 写 `design.md`；
  同时生成 `specs` 指令 → 写 `specs/auth/spec.md`（delta 规格：ADDED/MODIFIED/REMOVED）
- **产出**：`design.md`、`specs/auth/spec.md`（Requirement 须含 SHALL/MUST，Scenario 须 4 井号）

#### ④ 计划（tasks + plan）

- **技能调用**：agent 生成 `tasks` 指令 → 写 `tasks.md`（任务清单）→ 生成 `plan` 指令 →
  写 `plan.md`（微步实施计划，含 change/design-doc/base-ref 文件头，供 subagent 执行）
- **产出**：`tasks.md`、`plan.md`

#### ⑤ 实现（apply）

- **用户输入**：`/opsx:apply` 或「开始实现」
- **技能调用**：agent 生成 `apply` 指令 → pre-flight 检查必需技能 → 加载
  `superpowers:using-git-worktrees` 建 `feat/add-user-auth` 隔离分支 → 加载
  `superpowers:subagent-driven-development` 按 plan 微步派发子代理（传递激活
  `test-driven-development` 与 `requesting-code-review`）→ 提交
- **产出**：`feat/add-user-auth` 分支上的实现提交

#### ⑥ 验证

- **技能调用**：agent 生成 `verify` 指令 → 运行 `openspec validate --all --json` 结构校验 →
  逐项完成 7 项检查（结构校验 / tasks 完成度 / delta spec 同步 / design-specs 一致性 /
  实现信号 / front-door 路由泄漏 / deferred dogfood 等价性）→ 填 Overall Decision
- **产出**：`verify.md`（PASS / PASS WITH WARNINGS / FAIL）

#### ⑦ 回顾 + 归档

- **技能调用**：agent 生成 `retrospective` 指令 → evidence-first 写 `retrospective.md`
  （§0 Evidence + Wins/Misses + 技能合规 + promote candidates，PR 之前完成）→
  运行 `openspec archive -y` 归档到 `openspec/specs/` → 加载
  `superpowers:finishing-a-development-branch` 开 PR、合入 main
- **产出**：归档后的 specs、合入的 PR

### 各阶段用到的技能

| 阶段 | 技能 |
|---|---|
| 初始化 | `opsx-init` |
| 全流程导航 / schema 说明 | `opsx-use-superpower-cn-schema` |
| brainstorm | `superpowers:brainstorming`（PRECHECK 校验，缺失即 STOP） |
| 实现 | `superpowers:using-git-worktrees`、`subagent-driven-development`、`test-driven-development`、`requesting-code-review` |
| 收尾 | `superpowers:finishing-a-development-branch` |

---

## 技能包

- `opsx-init/`：在当前仓库初始化 OpenSpec 工作流。默认运行 `openspec init --tools agents`
  （6 个 openspec skills 装进 `.agents/skills/`），可选接入 `superpowers-bridge-cn`
  schema 并设为默认；含幂等处理与端到端验证。
- `opsx-use-superpower-cn-schema/`：**简体中文版** OpenSpec schema `superpowers-bridge-cn`，
  由 [superpowers-bridge](https://github.com/JiangWay/openspec-schemas/tree/main/superpowers-bridge) 整理而来，
  并融合本仓库 `prompts/stdd-*.md` 的 SDD 工作流细节。安装：把该目录复制到项目
  `openspec/schemas/superpowers-bridge-cn/` 后运行 `openspec schema validate superpowers-bridge-cn`。
  详见该技能内 `README.md`。

---

## ⚠️ 即将弃用：stdd prompt 工作流

`prompts/stdd-*.md`（`/stdd-propose`、`/stdd-plan`、`/stdd-apply`、`/stdd-archive`）
是早期的 prompt 模板工作流，**即将弃用**，理由：

- 流程约定散落在 prompt 文本里，无法由 CLI 强制 DAG 依赖（可跳步、缺产出物不自检）；
- 没有 apply 之后的 verify / retrospective 闭环；
- 实现方式未集成 worktree / subagent / TDD / code-review。

其能力已完整并入 openspec schema 工作流（`superpowers-bridge-cn` 的 artifact
instructions 融合了 SDD 行为细节：对话纪律、apply 暂停条件、提交纪律等）。

**迁移路径**：新工作一律走 openspec schema 工作流；`opsx-init` 可一键初始化。
计划在下一个大版本移除 `prompts/stdd-*.md`。

---

## 安装

```bash
# 通过 git 安装（发布到 GitHub 后）
pi install git:github.com/Philogag/sdd-tdd-workflow

# 通过 npm 安装（发布到 npm 后）
pi install npm:sdd-tdd-workflow

# 本地安装（开发调试）
pi install /absolute/path/to/sdd-tdd-workflow

# 临时试用，不写入配置
pi -e /absolute/path/to/sdd-tdd-workflow
```
