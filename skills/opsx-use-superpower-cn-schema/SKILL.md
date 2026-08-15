---
name: opsx-use-superpower-cn-schema
description: >-
  安装并使用简体中文的 superpowers-bridge-cn OpenSpec schema（融合本仓库 SDD 工作流
  prompt）。当用户想开始一次变更（/opsx:new、/opsx:ff、/opsx:propose），或在既有
  change 中继续 / apply / verify / archive，或想了解该 schema 的安装与使用方式时使用。
  与 openspec-* 系列 skill 互补：本 skill 负责 schema 的安装与全流程导航。
---

# opsx-use-superpower-cn-schema

将 [superpowers-bridge](https://github.com/JiangWay/openspec-schemas/tree/main/superpowers-bridge)
（Superpowers 技能体系 ↔ OpenSpec 的桥接 schema）整理为**简体中文版**
`superpowers-bridge-cn`，并融合本仓库 `prompts/stdd-*.md`（SDD：Spec-Driven
Development）工作流行为。本 skill 的目录结构：

```
skills/opsx-use-superpower-cn-schema/
├── schema.yaml                          # OpenSpec schema 定义（全部简体）
├── SKILL.md                             # 本文件
├── README.md                            # 简体中文使用文档
├── VERSION                              # 1.0.0
└── templates/
    ├── brainstorm.md                    # 头脑风暴（入口 artifact）
    ├── proposal.md                      # 变更提案
    ├── design.md                        # 技术设计
    ├── spec.md                          # Delta spec
    ├── tasks.md                         # 任务清单
    ├── plan.md                          # 实施计划
    ├── verify.md                        # 验证报告
    ├── retrospective.md                 # 回顾
    └── adopters/
        └── CLAUDE.md.fragment.md        # 给采纳方 CLAUDE.md 的 routing 片段
```

## 何时使用

- 用户想要发起一个新变更（`/opsx:new <name>`、`/opsx:ff`、`/opsx:propose`）
- 用户在既有 change 中继续（`/opsx:continue`）或推进 apply / verify / archive
- 用户想要安装、升级、验证本 schema，或了解它和 spec-driven / 原版 bridge 的区别

## 安装（在目标仓库内）

```bash
# 1. 复制 schema 包到项目
mkdir -p openspec/schemas/superpowers-bridge-cn
cp -r <本技能目录>/schema.yaml \
      <本技能目录>/templates \
      <本技能目录>/VERSION \
      openspec/schemas/superpowers-bridge-cn/

# 2. 验证 schema 可被解析
openspec schema validate superpowers-bridge-cn
# 中文 CLI 等价：openspec-cn schema validate superpowers-bridge-cn

# 3. 可选：把 routing 片段并入项目 CLAUDE.md（或首个文档）
#    templates/adopters/CLAUDE.md.fragment.md
```

> 若项目已有 `openspec/schemas/` 下的其他 schema，`openspec schemas` 可查看
> 全部可用项；项目级 schema 优先于内置 schema。

## 工作流概览

```
brainstorm ─→ proposal ─→ specs ─┐
                                  ├─→ tasks ─→ plan ─→ [apply] ─→ verify ─→ retrospective
                    design  ─────┘
```

- **入口**：`brainstorm`（对话收敛、输出重定向到 change 目录，不写 `docs/superpowers/`）
- **design 与 specs 并行**：二者都依赖 proposal，先于 tasks
- **apply 之后还有两步**：verify（含 7 项检查 + Overall Decision）与 retrospective
  （evidence-first，PR 之前写）
- 整个流程按 `openspec instructions --change <name>` 注入的 artifact 指引推进，
  依赖关系由 CLI 强制，不可跳步

## CLI cheat sheet

| 目的 | 命令 |
|---|---|
| 列出可用 schema | `openspec schemas` |
| 查看当前变更状态 | `openspec status --change <name> --json` |
| 新建变更 | `openspec new change <name>` |
| 生成下一 artifact | `openspec instructions --change <name>` |
| 校验全部 spec | `openspec validate --all --json` |
| 归档变更 | `openspec archive -y` |
| 验证 schema | `openspec schema validate superpowers-bridge-cn` |

> 全部命令可用 `openspec-cn` 替代以获得中文输出（等价）。

## 与内置 spec-driven schema 的差异

| 维度 | spec-driven | superpowers-bridge-cn |
|---|---|---|
| 入口 | 直接 `proposal` | `brainstorm`（先收敛对话） |
| plan 粒度 | 任务列表 | 微步 checklist（可执行粒度） |
| apply | 仅 `plan` 前置 | `plan` 前置 + worktree / subagent / TDD / code-review 集成 |
| apply 之后 | — | verify + retrospective（证据驱动） |
| 语言 | 简体（openspec-cn） | 简体 |

## 详细文档

见同目录 `README.md`（安装、升级、入口/出口 gates、SDD prompt 映射、
版本识别、兼容性）。
