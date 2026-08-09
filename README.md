# sdd-tdd-workflow

用于分享 **prompts** 和 **skills** 的 [pi package](https://pi.dev/docs/latest/packages)，围绕 SDD（Spec-Driven Development）+ TDD（Test-Driven Development）工作流。

由 OpenSpec 和 Superpower 驱动

## 结构

```
sdd-tdd-workflow/
├── package.json    # pi 清单：声明 prompts 与 skills 目录
├── prompts/        # prompt 模板（/name 触发，见 docs/prompt-templates.md）
└── skills/         # 技能（每个技能是一个含 SKILL.md 的目录，见 docs/skills.md）
```

- `prompts/`：每个 `.md` 文件是一个 prompt 模板，文件名（不含 `.md`）即命令名，编辑器里输入 `/name` 展开。
- `skills/`：每个技能是一个包含 `SKILL.md` 的目录，按需加载。

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
