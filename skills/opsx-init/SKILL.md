---
name: opsx-init
description: >-
  在当前仓库初始化 OpenSpec 工作流：运行 `openspec init`（默认 `--tools agents`，
  即把 6 个 openspec skills 装进 .agents/skills/），可选接入本技能包自带的
  superpowers-bridge-cn 简体中文 schema 并设为默认。当用户想给一个新仓库/现有仓库
  初始化 openspec（openspec init）、或不确定 openspec 是否已初始化、或想切换默认
  schema 时使用。
---

# opsx-init — 在当前仓库初始化 OpenSpec 工作流

> 面向本技能包（sdd-tdd-workflow）的采纳者。产出：`openspec/` 目录 +
  `.agents/skills/`（6 个 openspec 技能）+ 可选的 `superpowers-bridge-cn` schema。

## 何时使用

- 用户想在新仓库初始化 OpenSpec（「初始化 openspec」「搭 openspec 工作流」）
- 现有仓库缺少 `openspec/` 目录或 `.agents/skills/`
- 用户想把默认 schema 换成 `superpowers-bridge-cn`（本技能包的中文 schema）

## 工作目录

在**目标仓库根目录**（含 `.git`，推荐）执行。若当前目录不在仓库根，先 `cd`。

---

## Step 0 — 确认 openspec CLI 可用（缺失则引导安装）

**这是第一步，先做可用性确认，再谈初始化。**

1. **检查 openspec 是否可用**：

   ```bash
   openspec --version
   ```

   - 本环境 `openspec` 是 `openspec-cn`（npm 包 `@studyzy/openspec-cn`，中文 CLI）的符号链接，
     `openspec-cn --version` 等价；输出形如 `1.8.0` 即可用。
   - 也检查一下别名：`command -v openspec openspec-cn`，两个都找不到才判定为未安装。

2. **判定未安装**（`command -v openspec openspec-cn` 均无输出）→ **引导用户安装**：

   用 `ask_user_question` 确认安装方式（不要擅自全局安装），提供选项：

   - **简体中文版（推荐）**：`npm install -g @studyzy/openspec-cn`
     （本技能包所有命令均兼容，输出为中文）
   - **英文原版**：`npm install -g openspec-cli`
   - **用户自行安装**：等待用户装好后再继续

   用户确认后执行安装，然后**重新验证**：

   ```bash
   openspec --version   # 必须能输出版本号才继续
   openspec -h | head -20   # 能看到命令列表（new/init/status/…）
   ```

   - 若安装后仍报 command not found：提示用户检查 npm 全局 bin 目录是否在 PATH
     （`npm prefix -g` 查看），必要时新开终端会话再试。

3. **检查是否已初始化**：

   ```bash
   ls openspec/openspec.yaml openspec/config.yaml .agents/skills 2>/dev/null
   ```

   - 已存在 `openspec/`（有 config.yaml 或 openspec.yaml）→ 跳到 Step 2 的「已有初始化」
     分支，**不要**盲目重跑 `openspec init`（会刷新 .agents skills，通常无害，但先问用户）。
   - 都不存在 → 正常走 Step 1。

4. **（提示，非阻塞）** 仓库推荐是 git 仓库（change 需要提交），但不是硬性要求。

---

## Step 1 — 运行 `openspec init`

默认安装行为：**`--tools agents`**（把 openspec 工作流技能装进 `.agents/skills/`，
供 agents/pi 等 harness 发现；`openspec init -h` 中 `--tools` 的可选值含
`claude`、`codex`、`pi`、`oh-my-pi`、`agents`、`all`、`none` 等）。

```bash
openspec init --tools agents
```

- 预期输出：创建 OpenSpec 结构 → 设置 Shared .agents skills（6 个 skills 位于
  `.agents/`）→ 配置 `openspec/config.yaml`（schema：spec-driven）。
- `已跳过命令：agents（无适配器）` 是**正常现象**：agents 工具没有生成
  AGENTS.md/CLAUDE.md 指令文件的适配器，不影响 skills 安装。
- 若用户想配其他工具（如 Claude Code 要 CLAUDE.md 指令），询问后改用
  `openspec init --tools claude`（可逗号分隔多个，如 `--tools agents,claude`）。
- 若 `.agents/skills` 已有 openspec 技能，CLI 会幂等刷新，不会破坏现有文件。

**已有初始化**：用户确认要重跑时加 `--force`（自动清理旧文件），否则保留现状，
只做 Step 2 的 schema 接入。

---

## Step 2 — （可选，推荐）接入 superpowers-bridge-cn schema

用 `ask_user_question` 询问是否接入本技能包自带的简体中文 schema
（它是 `openspec init` 默认 `spec-driven` 的中文化 + Superpowers/SDD 融合版）：

1. **定位 schema bundle**（相对本技能目录 `../opsx-use-superpower-cn-schema`）：

   ```bash
   ls ../opsx-use-superpower-cn-schema/schema.yaml 2>/dev/null
   ```

   - 找不到时，用 `find` 在技能包根目录搜索 `schema.yaml`（如
     `node_modules/<pkg>/skills/opsx-use-superpower-cn-schema/`），或让用户提供路径。

2. **复制到项目**：

   ```bash
   mkdir -p openspec/schemas/superpowers-bridge-cn
   cp -r <bundle目录>/schema.yaml <bundle目录>/templates <bundle目录>/VERSION \
         openspec/schemas/superpowers-bridge-cn/
   ```

3. **设为默认 schema**（把 `openspec/config.yaml` 首行改为）：

   ```yaml
   schema: superpowers-bridge-cn
   ```

4. **验证**：

   ```bash
   openspec schema validate superpowers-bridge-cn
   ```

---

## Step 3 — 验证初始化结果

```bash
# 1. schema 可解析（若装了 cn schema）
openspec schema validate superpowers-bridge-cn

# 2. 可用 schema 列表（应能看到 superpowers-bridge-cn（项目））
openspec schemas

# 3. 冒烟：建一个临时 change 并生成首个 artifact 指令
openspec new change smoke-check --schema superpowers-bridge-cn
openspec instructions --change smoke-check brainstorm   # 应输出含 PRECHECK 的指令
openspec status --change smoke-check --json             # 查看 artifact 依赖状态

# 4. 清理临时 change
rm -rf openspec/changes/smoke-check
```

全部通过 → 初始化完成。

---

## Step 4 — 汇报与下一步

向用户汇报：

- `openspec/` 与 `.agents/skills/`（6 个 openspec 技能：openspec-propose、
  openspec-apply-change、openspec-archive-change、openspec-update-change、
  openspec-sync-specs、openspec-explore）已就绪；
- 默认 schema 是哪个（spec-driven 或 superpowers-bridge-cn）；
- **建议重启 IDE / 重新打开会话**让 `.agents/skills/` 新技能生效；
- 下一步入口：
  - 走本技能包全流程：`/opsx:new <name>` 或 `/opsx:propose`
    （配合 `opsx-use-superpower-cn-schema` 技能的说明文档）
  - 或 CLI 自带入口：`/openspec-propose "你的想法"`
- 可选：把 `opsx-use-superpower-cn-schema/templates/adopters/CLAUDE.md.fragment.md`
  的内容并入项目 `CLAUDE.md`（给 agent 的分流指引）。

---

## 常见问题

| 现象 | 说明 |
|---|---|
| `已跳过命令：agents（无适配器）` | 正常。agents 工具无 AGENTS.md 生成适配器，skills 照常安装 |
| `openspec new change` 显示 `（schema 'spec-driven'）` 但实际用了新 schema | CLI 显示瑕疵；以 `Schema：<name>` 行为准 |
| `.agents/skills` 与 `~/.pi/agent/skills` 都有 openspec 技能 | 正常共存；`.agents/` 是项目级，全局的是用户级 |
| 重跑 `openspec init` | 幂等，安全；`--force` 清理旧文件 |
| 项目不在 git 仓库 | init 仍可用；但 change 提交/归档需要 git，建议先 `git init` |
