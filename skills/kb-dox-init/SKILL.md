---
name: kb-dox-init
description: >-
  为当前项目初始化 kb 知识库并启用 DOX 文档纪律：确保 `.pi/dashboard/knowledge_base.json` 存在（自动执行 `kb init` / `kb index`），写入 doctrine 配置，在根 AGENTS.md 追加 READ + WRITE 纪律指针块，运行 `kb dox init` 脚手架目录级 AGENTS.md 树（幂等）。当用户要求"启用 DOX"、"初始化文档纪律"、"kb + dox 初始化"时使用。
license: MIT
---

# kb-dox-init — 知识库 + DOX 文档纪律初始化

> 提取自 pi-agent-dashboard 的 `project-init` 技能（Step 3a / Step 5）中
> 关于 kb 与 DOX 的初始化流程，适配本技能包。

本技能做两件事：

1. **kb**：确保 `.pi/dashboard/knowledge_base.json` 存在且健康（本技能已完整内嵌 `kb init` / `kb index` 逻辑），并写入 DOX 所需的配置键。
2. **DOX**：向 `./AGENTS.md` 追加指针块，运行 `kb dox init` 生成目录级
   AGENTS.md 树，可选调用 `dox-describe` 填充内容。

**关键认知**：不要复制任何 doctrine 正文。逐轮注入的 DOX doctrine
（READ + WRITE 纪律）由 `pi-dashboard-kb-extension` 自动注入；项目的
`AGENTS.md` 只承载一个**标记 + 指针块**，通过
`.pi/dashboard/knowledge_base.json` 的 `doctrine` 键调优。

## 何时使用

- 用户想启用 DOX 文档纪律（"启用 DOX"、"初始化文档纪律"）。
- 项目已有 kb 配置但没有 `doctrine` 键，或没有 `directoryLevelAgents`。
- 用户想生成/补全目录级 `AGENTS.md` 树（`kb dox init` / `dox-describe`）。
- project-init 式的项目脚手架流程中，用户确认要启用 DOX。

## 前置条件

- `kb` CLI 可用（来自 `@blackbelt-technology/pi-dashboard-kb`）。
  不可用时引导用户先使用 npm 安装 CLI；不要继续。
- 目标目录是项目根（所有写入落在当前工作目录）。

---

## Step 1 — 检测并初始化 kb 配置（含 kb-init 逻辑）

```bash
kb config
ls .pi/dashboard/knowledge_base.json 2>/dev/null || echo "No kb config yet"
```

- 若项目配置不存在（`knowledge_base.json` 不存在或 `sources` 为空）：
  1. 执行 `kb init --source .`（或 `--global` 根据用户选择）
  2. 执行 `kb index --force`
  3. 运行一次冒烟测试：`kb search "test" --limit 3 --doc-type agents`
- 若配置已存在：检查 `sources` 是否至少包含当前目录（`.` 或项目根），缺少则合并添加。
- 始终运行 `kb config` 确认 `sources`、`dbAbsPath`、`doctrine` 正确。

## Step 2 — 初始化 KB-DOC 配置

**读取-合并-写入** `.pi/dashboard/knowledge_base.json`：文件存在则先读，
只设置下列键，**保留其余所有键不变**，写出合法 JSON。

设置：

```json
"doctrine": { "inject": "kb", "write": true }
```

当文件此前不存在时，写入完整骨架：

```json
{
  "sources": [{ "kind": "filesystem", "ref": "." }],
  "indexAgentsFiles": true,
  "directoryLevelAgents": { "enabled": true },
  "doctrine": { "inject": "kb", "write": true }
}
```

文件已存在但缺少 `indexAgentsFiles` / `directoryLevelAgents` 时，同样
补齐这两个键（合并，不覆盖其它内容）。

## Step 3 — 初始化 根AGENTS.md

**幂等检查**：若 `./AGENTS.md` 已包含标记 `<!-- kb-dox-doctrine -->`，
跳过本步，整段不重复追加。

```bash
grep -q '<!-- kb-dox-doctrine -->' ./AGENTS.md && echo PRESENT || echo ABSENT
```

ABSENT 时追加（若 `./AGENTS.md` 不存在则新建）：

```markdown
<!-- kb-dox-doctrine -->
## 查找文档（READ 纪律）

`kb` CLI 返回每个文件的一行目的（purpose）+ 关键导出（key exports），
而非原始字节 —— 比原始搜索更快、更省 token。**本纪律由动作触发，而非
由意图触发**：在你要 `grep`/`rg` 一个符号、`cat`/读文件去了解它是干什么的、
或追一个 import 之前，先跑 `kb` —— 即使任务中途、你已经知道该文件时也是
如此。当你的直觉是左列动作时，先跑右列：

| 你正要…… | 先这样做 |
|---|---|
| `grep -rn "SymbolName" src/` —— 找某个 fn / type / const 定义在哪 | `kb search "SymbolName" --doc-type agents`（kb 未接入时，读最近的目录 `AGENTS.md`） |
| `grep -rn "feature\|topic" src/` —— X 怎么工作 / X 在哪里处理 | `kb search "feature topic" --limit 5`（或 `kb agents <path>` 沿 根→最近 的 AGENTS.md 链） |
| `cat` / 读文件，只是为了在编辑前了解它的目的 | `kb agents <path>` —— 一行目的 + 导出 + 变更历史 |
| 跨文件追 imports / 调用方 | `kb neighbors "<path>"`；反查谁引用它用 `kb backlinks "<path>"` |
| 完整阅读某个文档章节 | `kb get <path> --section "<heading_path>"` |

**降级策略：** 若 `kb search`（或文档树）没有返回相关结果，
则允许 `rg` / 直接读源码 —— 然后按 DOX更新协议 补上缺失的
目录 `AGENTS.md` 行，并 `kb index` 重建索引。文档查找**不取代** grep；它只是**先行**。

## DOX更新协议（WRITE 纪律）

各目录的 `AGENTS.md` 组成一棵树。每个目录 `AGENTS.md` 是该目录下各文件的 **逐文件记录**。
根 `AGENTS.md` 只承载 DOX工作纪律 —— 绝不放逐文件索引。

**按文档类型路由每次文档更新：**

| 更新类型 | 写入位置 |
|---|---|
| 目录下的新文件，或该文件的逐文件细节 / 变更历史 | 最近的目录 `AGENTS.md`。添加 `\| `<basename>` \| <purpose> \|` 行，按路径字母序。 |
| 数据流、协议、架构论证 | `docs/architecture.md` 或某个 `docs/<topic>.md` |
| 终端用户 / 开发者安装配置 | `README.md` |
| 每个 agent 每轮都需要的横切规则（罕见） | 根 `AGENTS.md` |

**编辑前先读（链式查找）。** 编辑文件前，沿 根→叶 读最近的 `AGENTS.md` 链，了解该文件的已记录目的、契约与变更历史。不要盲目编辑。

**编辑后更新（收尾 pass）。** 改动文件后，更新其在最近目录 `AGENTS.md` 中的行：找到该文件的行，就地更新其 purpose；不存在则按路径字母序添加。
新目录 → scaffold 它的 `AGENTS.md`。每文件一行。purpose 包含：一行摘要、关键导出符号、契约/不变量，以及 `See change: <id>` 变更历史。

**行风格（caveman）。** 短声明式片段。去掉冠词。主语 → 动词 → 宾语，现在时。一行一事实。优先具体 token（路径、符号、环境变量）而非散文。标识符原样保留。

**文件大小限制。**
过大的目录 `AGENTS.md` 会导致上下文注入时不够专注。
在单个目录 AGENTS.md 跟踪超过 20 项时应推荐用户考虑拆分文件夹
<!-- kb-dox-doctrine-end -->
```

（标记 `<!-- kb-dox-doctrine -->` 保留原样不变——它是幂等检测的锚点。）

## Step 4 — 生成目录级 AGENTS.md 树

```bash
kb dox init
```

该命令按**仅路径行**（path-only rows）为源码树各目录 scaffold
`AGENTS.md` 骨架。运行后确认退出码为 0；非零退出时把错误原样报告给
用户，不要重试覆盖已有文件。

## Step 5 — 重建索引并验证

```bash
kb index
kb config
```

验证清单：

- `kb config` 输出中包含 `doctrine`、`directoryLevelAgents` 的新值，
  且原有 `sources` / `dbPath` 未被破坏。
- `./AGENTS.md` 恰好包含一个 `<!-- kb-dox-doctrine -->` 标记。
- `kb index` 报告 `N files … M chunks`（新生成的目录级 AGENTS.md 已被
  收录）。
- `kb search "<文档中的真实术语>" --limit 5` 返回带排名的
  `{path, headingPath, score, snippet}` 结果。

完成后告知用户：逐轮 DOX doctrine 将由 `pi-dashboard-kb-extension`
自动注入，日常检索使用 `kb-search` 技能。

## Step 6 — 处理 kb 索引数据库

当处在 git 仓库中时，为 kb 生成的 sqlite db 文件增加 gitignire

---

## 注意事项（Pitfalls）

- **绝不复制 doctrine 正文**到 `./AGENTS.md`——只写指针块；正文由
  extension 逐轮注入。
- `knowledge_base.json` 必须 **read-merge-write**：只动 `doctrine` /
  `indexAgentsFiles` / `directoryLevelAgents`，其余键原样保留。
- 幂等性：`<!-- kb-dox-doctrine -->` 标记存在即跳过 Step 3；
  `kb dox init` 不要在有未确认覆盖风险时重复运行。
- 未经用户确认不得覆盖已有 `./AGENTS.md` 或 kb 配置。
- **kb 初始化**：本技能已完整内嵌 `kb init` / `kb index` 逻辑，不再依赖 `kb-init` 技能。
- `dox-describe` 是可选项，永不强制；用户拒绝时告知可稍后运行即可。
