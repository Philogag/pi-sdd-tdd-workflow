---
description: 提案新变更 - 输出提案和设计
argument-hint: "<提案概述>"
compatibility: 需要 openspec CLI 和 superpower 技能包。
---

提案新变更 - 一步创建变更并生成所有产出物。

我将创建一个包含以下产出物的变更：
- proposal.md（什么和为什么）
- design.md（如何实现）
- superpower-design.md (更细致的design)
- superpower-plan.md (详细的实现过程)
- tasks.md (实现状态跟踪)

准备好需求后，运行 /stdd-plan 开始生成计划

---

**输入**：$@

**步骤**

1. **如果没有提供输入，询问他们想要构建什么**

   使用 **ask_user_question tool**（开放式，无预设选项）询问：
   > "您想要处理什么变更？请描述您想要构建或修复的内容。"

   根据他们的描述，推导出一个 kebab-case 名称（例如："add user authentication" → `add-user-auth`），作为 <change-name>。

   **重要提示**：在不了解用户想要构建什么的情况下，请勿继续。
   **在充分了解项目当前状态前，使用 skill /openspec-explore 探索**

2. **创建变更目录**
   ```bash
   openspec new change "<change-name>"
   ```
   这将在 `openspec/changes/<change-name>/` 创建一个带有 `.openspec.yaml` 的脚手架变更。

3. **获取产出物构建顺序**

   按照如下顺序输出产物, 并使用 **todo tool** 追踪
   - proposal.md（什么和为什么）
   - design.md（如何实现）
   - superpower-design.md (更细致的design)
   - superpower-plan.md (详细的实现过程) **仅在todo tool中展示，不在本阶段输出**
   - tasks.md (实现状态跟踪) **仅在todo tool中展示，不在本阶段输出**

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
      - 更新 **todo tools状态**

   b. **继续直到所有 `applyRequires` 产出物完成**
      - 创建每个产出物后，重新运行 `openspec status --change "<change-name>" --json`
      - 检查 `applyRequires` 中的每个产出物 ID 在 artifacts 数组中是否具有 `status: "done"`
      - 当所有 `applyRequires` 产出物完成时停止

   c. **如果产出物需要用户输入**（上下文不清楚）：
      - 使用 **ask_user_question tool** 进行澄清
      - 然后继续创建

5. **使用 skill /brainstorming 细化需求**

   使用 /brainstorming 技能头脑风暴，深度技术设计，并传入以下上下文：

   ---
   Change: <change-name>
   上游需求（来自 OpenSpec，不要重写）：
   - 目标：<从 proposal.md 提取>
   - 架构约束：<从 design.md 提取>

   约束：
   1. 输出文件为 openspec/changes/<change-name>/superpower-design.md
   2. OpenSpec 是需求的事实源，不要重新定义需求，不要重写 proposal/spec
   3. 你的任务是基于已有需求做深度技术设计：实现方案、技术风险、测试策略、边界条件
   4. 如发现 delta spec 缺少验收场景，只能回写 OpenSpec delta spec，不要在 Design Doc 中创建第二份需求 spec
   5. 跳过上下文探索，直接进入设计提问
   6. 在 brainstorming 途中任何与用户的交互均使用 **ask_user_question tool** 
   ---

6. **显示最终状态**

   ```bash
   openspec status --change "<change-name>"
   ```

**输出**

完成所有产出物后，总结：
- 变更名称和位置
- 已创建产出物的列表及简要描述
- 准备就绪："任务目标和当前状态已经明朗，可以开始输出实现步骤拆解。"
- 提示："运行 `/stdd-plan` 开始输出实现步骤拆解。"

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
