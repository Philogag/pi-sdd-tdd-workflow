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

**① 初始化仓库**（用 `opsx-init` 技能，或手动）：

```bash
openspec init --tools agents                # 默认：6 个 openspec skills 装进 .agents/skills/
# 可选：接入本包的中文 schema 并设为默认
mkdir -p openspec/schemas/superpowers-bridge-cn
cp -r skills/opsx-use-superpower-cn-schema/{schema.yaml,templates,VERSION} \
      openspec/schemas/superpowers-bridge-cn/
sed -i 's/^schema: spec-driven/schema: superpowers-bridge-cn/' openspec/config.yaml
openspec schema validate superpowers-bridge-cn
```

**② 建变更并推进产出物**（每步先看指令再写文件）：

```bash
openspec new change add-user-auth            # 已设默认 schema 则无需 --schema
openspec status --change add-user-auth --json    # 查看 DAG 状态

# 按依赖顺序逐个生成并填写产出物：
openspec instructions --change add-user-auth brainstorm      # → brainstorm.md（对话收敛）
openspec instructions --change add-user-auth proposal       # → proposal.md（什么和为什么）
openspec instructions --change add-user-auth specs          # → specs/<capability>/spec.md（delta 规格）
openspec instructions --change add-user-auth design         # → design.md（技术设计，与 specs 并行）
openspec instructions --change add-user-auth tasks          # → tasks.md（任务清单）
openspec instructions --change add-user-auth plan           # → plan.md（微步实施计划）
```

**③ 实现**（apply 走查，SDD + Superpowers 技能集成）：

```bash
openspec instructions --change add-user-auth apply
```

apply 流程：pre-flight 技能检查 → `using-git-worktrees` 建 `feat/add-user-auth` 隔离分支 →
`subagent-driven-development` 按 plan 微步实现（传递激活 TDD 与 code-review）→ 提交。

**④ 验证**：

```bash
openspec instructions --change add-user-auth verify
openspec validate --all --json    # 结构校验（Requirement 须含 SHALL/MUST、Scenario 须 4 井号等）
```

verify 产出 `verify.md`：7 项检查（结构校验 / tasks 完成度 / delta spec 同步 /
design-specs 一致性 / 实现信号 / front-door 路由泄漏 / deferred dogfood 等价性）+
Overall Decision（PASS / PASS WITH WARNINGS / FAIL）。

**⑤ 回顾 + 归档**：

```bash
openspec instructions --change add-user-auth retrospective   # evidence-first，PR 之前写
openspec archive -y                                          # 归档到 openspec/specs/
```

归档后收尾（finishing-a-development-branch）：开 PR、合入 main。

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
