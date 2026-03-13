
# AI Agent 生态核心概念澄清文档  
**版本：1.0**  
**日期：2026年3月**  
**目标读者**：agent开发者、框架维护者、registry mirror设计者、企业AI平台团队

## 1. 核心原则：分层抽象，避免一词多义
当前生态（Claude Code、Cursor、OpenClaw、LangGraph、CrewAI、Vercel skills.sh 等）中，同一个词在不同框架/社区常被赋予不同含义，导致最大混淆点在于：

- **底层执行层**（runtime） vs. **上层能力层**（skill/role/persona）
- **原子能力**（tool/function） vs. **人类式角色技能栈**（persona + 多工具 + 判断逻辑）

我们建议采用以下**严格分层**（从下到上）：

1. **Runtime / Agent Runtime / 执行环境**  
   → 对应传统编程的“runtime”（Node.js、JVM、Python解释器）

2. **Tool / Function / Capability**  
   → 原子、可调用的动作（类似函数）

3. **Skill / Procedure / Workflow**  
   → 带指导、步骤、上下文的“专家技能”（Anthropic Agent Skills 标准的核心）

4. **Role / Persona / Specialist**  
   → 完整的人设 + 技能栈（最接近人类“角色”）

## 2. 逐个概念精确定义（2026年3月主流用法）

| 概念          | 定义（当前事实标准）                                                                 | 对应层级 | 典型实现/格式                              | 谁最常用这个叫法                              | 与人类类比                          | 例子（贴近“同一个男人不同角色”）                  |
|---------------|-------------------------------------------------------------------------------------|----------|--------------------------------------------|-----------------------------------------------|-------------------------------------|---------------------------------------------------|
| **Runtime** / **Agent** (狭义) | 负责循环（observe → think → plan → act → observe）、状态管理、工具调度、内存、错误重试的“空壳执行器”。本身无领域知识，仅提供基础设施。 | 底层     | Claude Code / Cursor / OpenClaw / LangGraph executor / CrewAI crew runner | 所有框架底层文档、OpenClaw、LangGraph         | 操作系统 / JVM                      | Cursor + Claude Opus 4.5 组成的“运行大脑”         |
| **Tool** / **Function** / **Action** | 原子、可执行的接口。模型输出结构化参数 → runtime 执行 → 返回结果。无复杂判断逻辑。 | 第2层    | OpenAI function calling / LangChain tools / MCP connectors | LangChain、OpenAI SDK、Composio               | 单个API / 命令行工具                | browser_use、write_file、git_commit               |
| **Skill**     | 可复用、模块化的“专家指导包”。通常是一个文件夹，核心文件为 SKILL.md（YAML前置 + Markdown指令），支持渐进加载（progressive disclosure），可包含脚本、参考文档、资产。触发时注入上下文/规则。 | 第3层    | Anthropic Agent Skills 标准（SKILL.md） + Vercel skills.sh 生态 | Anthropic、Cursor、Claude Code、Vercel、OpenClaw 兼容层 | “专业技能证书”或“专家手册”          | code-review（代码审查流程 + 最佳实践）<br>family-budget-advisor（家庭理财判断逻辑） |
| **Role** / **Persona** / **Specialist** | 完整的人设封装：system prompt（性格、目标、行为准则）+ 一组Skills/Tools + 记忆初始化 + 触发条件。代表“某个身份/职业/关系角色”的整体能力。 | 顶层     | CrewAI role + goal + backstory<br>Anthropic Skill bundle（带persona字段）<br>OpenClaw soul.md + skills | CrewAI（最典型）、AutoGen、部分中文社区      | “人”本身（丈夫 / 工程师）          | loving-husband（温柔沟通 + 家务分工 + 情感支持）<br>senior-backend-engineer（架构决策 + debug + review） |

## 3. 为什么会出现混淆？常见误区对照

| 误区（常见叫法）                  | 实际对应层级 | 为什么错？                                                                 | 正确叫法建议                              |
|-----------------------------------|--------------|----------------------------------------------------------------------------|-------------------------------------------|
| “Skill 就是 tool”                 | 错           | 很多框架把原子tool叫skill，导致与Anthropic开放标准冲突                     | Tool = 原子；Skill = 带流程的专家包       |
| “Agent 就是带role的那个东西”      | 部分错       | Agent狭义= runtime；广义才= runtime + role/persona                         | 说“agent runtime”或“agent instance”更准   |
| “OpenClaw的skill ≈ 人的技能栈”    | 接近但不准   | OpenClaw skill偏向role bundle，但仍基于SKILL.md标准                        | 叫“role-level skill”或直接“persona bundle”|
| “同一个男人不同角色 → 多个agent”  | 不推荐       | 实际是同一个runtime加载不同persona/skill bundle                           | “同一个agent runtime，切换不同persona”    |

## 4. 推荐的统一术语（供registry mirror设计使用）

- **Registry / Mirror 存储的三类 artifact**：
  1. **Tool packages**：原子能力（e.g. browser、pdf-parser），manifest重点：inputs/outputs/security
  2. **Skill packages**：SKILL.md标准文件夹（Anthropic/Vercel事实标准），manifest重点：triggers、compatible_runtimes、token_estimate
  3. **Persona / Role bundles**：扩展SKILL.md，增加persona_backstory、role_category、preferred_model、sub_skills数组

- **CLI 示例统一命令**（借鉴skills.sh + cargo）：
  ```
  agent-cli install husband-role --from my-mirror.example.com
  agent-cli install code-review-skill --runtime cursor,claude-code
  agent-cli search "家庭 丈夫" --category persona
  ```

- **Manifest 推荐扩展字段**（加在SKILL.md YAML）：
  ```yaml
  name: loving-husband
  description: 当涉及家庭关系、情感支持、家务分工时激活。模拟温柔体贴的丈夫角色。
  role_category: ["家庭-丈夫", "亲密关系"]
  persona_backstory: 你是一个已婚5年的温柔丈夫，擅长倾听、共情、主动分担家务…
  compatible_runtimes: ["claude-code", "cursor", "openclaw", "*"]
  sub_skills: ["emotional-communication", "home-chores-division", "family-finance-advice"]
  trigger_scenarios: "用户提到老婆/孩子/家务/争吵/纪念日"
  ```

## 5. 总结一句话定位
- **Runtime (Agent)** 是“电脑/大脑”，空壳执行器。
- **Tool** 是“手脚”，原子动作。
- **Skill** 是“专业证书”，可复用专家流程。
- **Role / Persona** 才是“人”，带性格、目标、完整技能栈。

只有把 **Role/Persona** 当作第一公民（顶层商品），才能真正实现“同一个runtime，不同人生阶段/职业的技能切换”，避免锁定单一生态，也最贴近人类直觉。
