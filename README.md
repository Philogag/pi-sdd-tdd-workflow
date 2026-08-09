# sdd-tdd-workflow

用于分享 **prompts** 和 **skills** 的 [pi package](https://pi.dev/docs/latest/packages)，围绕 SDD（Spec-Driven Development）+ TDD（Test-Driven Development）工作流。

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

> 安全提示：pi 包拥有完整系统权限，安装第三方包前请先审查源码。

## 开发

1. 在 `prompts/` 添加 prompt 模板：

   ```markdown
   ---
   description: 一句话描述该模板的作用
   ---
   模板内容，支持 $1、$@ 等参数占位符。
   ```

2. 在 `skills/<skill-name>/SKILL.md` 添加技能（frontmatter 必含 `name` 与 `description`）。

3. 本地验证：

   ```bash
   pi install -l ./path/to/sdd-tdd-workflow   # 写入项目配置
   pi list                                    # 查看已安装的包
   ```

## 发布

```bash
npm publish    # 发布到 npm
git push       # 或直接通过 git URL 分享
```

发布后可在 [pi.dev/packages](https://pi.dev/packages) 图库中被检索（依赖 `pi-package` keyword）。
