---
description: 提案新变更 - 输出提案和设计
argument-hint: "<提案概述>"
compatibility: 需要 openspec CLI 和 superpower 技能包。
---

提案新变更 - 一步创建变更并生成所有产出物。

准备好需求后，运行 /stdd-apply 开始执行变更

---

**输入**：用户输入部分的内容是变更名称（kebab-case），或用户想要构建内容的描述。

**步骤**

0. **准备步骤**
   确认 openspec 工具或 openspec-cn 工具已全局安装
   > openspec-cn 为 openspec 的完全等价替代

1. **如果没有提供输入，询问他们想要构建什么**

   使用 **tool://ask** 或类似工具 进行澄清（开放式问题无预设选项时，直接在对话中询问）：
   > "您想要处理什么变更？请描述您想要构建或修复的内容。"

   根据他们的描述，推导出一个 kebab-case 名称（例如："add user authentication" → `add-user-auth`），作为 <change-name>。

   **重要提示**：在不了解用户想要构建什么的情况下，请勿继续。
   **在充分了解项目当前状态前，使用 **skill://openspec-explore** 和 **skill://brainstorming** 探索**

2. **创建变更目录**
   ```bash
   openspec new change "<change-name>"
   ```
   这将在 `openspec/changes/<change-name>/` 创建一个带有 `.openspec.yaml` 的脚手架变更。
   如果在工作区路径到实际工作模块路径存在多层子 openspec 工作目录，使用 **tool://ask** 提示用户选择合适的 openspec 工作目录。

3. **获取产出物构建顺序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以获取：
   - `applyRequires`: 实现前所需的产出物 ID 数组（例如：`["tasks"]`）
   - `artifacts`: 所有产出物及其状态和依赖项的列表
   如果存在 **tool://todo** 或类似工具，则进行进度跟踪

4. **按顺序创建基础Spec**

   按依赖顺序循环遍历产出物（没有待处理依赖项的产出物优先）：

   a. **对于每个 `ready`（依赖项已满足）的产出物**：
      - 获取指令：
        ```bash
        openspec instructions <artifact-id> --change "<change-name>" --json
        ```
      - 指令 JSON 包括：
        - `context`：项目背景（对你的约束 - 不要包含在输出中）
        - `rules`：产出物特定规则（对你的约束 - 不要包含在输出中）
        - `template`：用于输出文件的结构
        - `instruction`：此产出物类型的 Schema 特定指导
        - `outputPath`：写入产出物的位置
        - `dependencies`：已完成的产出物，用于读取上下文
      - 读取任何已完成的依赖文件以获取上下文
      - 使用 `template` 作为结构创建产出物文件
      - 应用 `context` 和 `rules` 作为约束 - 但不要将它们复制到文件中
      - 显示简短进度："✓ 已创建 <artifact-id>"
      - 使用 **tool://todo** 更新状态

   b. **继续直到所有 `applyRequires` 产出物完成**
      - 创建每个产出物后，重新运行 `openspec status --change "<change-name>" --json`
      - 检查 `applyRequires` 中的每个产出物 ID 在 artifacts 数组中是否具有 `status: "done"`
      - 当所有 `applyRequires` 产出物完成时停止

   c. **如果产出物需要用户输入**（上下文不清楚）：
      - 使用 **tool://ask** 进行澄清
      - 然后继续创建

5. **显示最终状态**
   ```bash
   openspec status --change "<change-name>"
   ```

**输出**

完成所有产出物后，总结：
- 变更名称和位置
- 已创建产出物的列表及简要描述
- 准备就绪："提案已就绪！准备好实现。"
- 提示："运行 `/stdd-apply` 开始输出实现提案。"

**产出物创建指南**

- 遵循每个产出物类型的 `openspec instructions` 中的 `instruction` 字段
- Schema 定义了每个产出物应包含的内容 - 遵循它
- 在创建新产出物之前阅读依赖产出物以获取上下文
- 使用 `template` 作为输出文件的结构 - 填充其各个部分
- **重要提示**：`context` 和 `rules` 是对你的约束，而不是文件内容
  - 不要将 `<context>`、`<rules>`、`<project_context>` 块复制到产出物中
  - 这些引导你编写内容，但不应出现在输出中

**护栏**

- 创建实现所需的所有产出物（由 Schema 的 `apply.requires` 定义）
- 在创建新产出物之前始终阅读依赖产出物
- 如果上下文极其不清楚，询问用户 - 但倾向于做出合理的决定以保持势头
- 如果同名变更已存在，询问用户是否要继续它或创建一个新的
- 在继续下一个之前，验证写入后每个产出物文件是否存在

---

**用户输入**
$@
