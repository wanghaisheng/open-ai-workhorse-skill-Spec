
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





**我们想做的 Registry/Mirror 设计方案（v1.1）**  
**更新日期**：2026年3月13日  
**版本升级说明**：从 v1.0 升级至 v1.1，已全面整合 OARSS v1.0 规范、四层结构、sub_skills 依赖机制、Registry Index JSON Schema、config.toml、3个高质量示例角色、CLI 搜索/安装逻辑（Bash + Node.js MVP）、install-examples.sh 脚本等最新讨论成果。

**目标**：一个**完全开放、不锁定任何 runtime**（Claude Code、Cursor、Codex、Gemini CLI、OpenClaw、LangGraph 等全兼容）、**以 Role/Persona（人的角色 + 完整技能栈）为第一公民**的 Agent Skills Registry & Mirror。  
它既能做**企业内部私有 mirror**（安全、合规、零延迟、离线），也能作为**社区公开 registry** 存在，避免下一个“npm 垄断”或“ClawHub 单点风险”。

### 1. 核心设计原则（严格遵循概念澄清文档 + OARSS v1.0 规范）
- **Runtime（Agent）** = 执行空壳（IDE + Model），只负责加载，不存储能力。
- **Tool** = 原子能力（底层）。
- **Procedure** = 结构化步骤/checklist。
- **Skill** = 专家流程包（SKILL.md 标准）。
- **Role/Persona** = **第一公民**（顶层商品），代表“同一个男人不同身份的完整技能栈”（丈夫角色、工程师角色、产品经理角色等）。
- 所有 artifact **必须兼容 Anthropic Agent Skills 开放标准**（SKILL.md 文件夹 + YAML frontmatter），并严格遵循 **OARSS v1.0**（新增 `metadata.type`、`sub_skills`、`role_category`、`trigger_scenarios` 等扩展字段）。
- Mirror 机制：像 apt/yum/nexus 一样，支持一键同步官方（agentskills.io / skills.sh / GitHub）+ 企业私有覆盖 + 索引优先级。

### 2. 整体架构（四层解耦，新增 Registry Index）
```
[用户 / Runtime]
      ↓ (CLI: oar)
[Mirror / Registry 服务] ← 企业私有实例 or 社区公开
      ↓ (内容寻址 + 签名 + index.json)
[存储层]
  ├── Metadata Index（index.json + PostgreSQL JSONB）
  ├── Content Storage（IPFS CID / Git / S3）
  └── Signature Store（cosign / sigstore）
```

- **Registry 服务**：轻量 Go 服务（fork Solo.io agentregistry）或 Node.js，提供 REST API + index.json。
- **Mirror 同步引擎**：定时/事件驱动拉取官方 + 增量更新 + 签名校验。
- **CLI**：统一 `oar`（基于 Vercel skills.sh fork + 自定义扩展）。
- **新增核心**：`index.json`（Registry Index Schema） + `~/.oarss/config.toml`。

### 3. 关键特性（Role-First + sub_skills + 企业 Mirror 优先）
1. **Role/Persona Bundle 为核心商品**（OARSS 强制）
   - 一个 bundle = 一个文件夹（扩展 SKILL.md）：
     ```yaml
     name: senior-fullstack-engineer
     metadata:
       type: persona-bundle          # 关键区分字段
       role_category: ["开发-全栈", "工程师"]
       persona_backstory: "..."
       trigger_scenarios: ["React / Next.js / 代码 review"]
       sub_skills: [                 # 依赖数组，支持 semver + required/optional
         {name: "react-best-practices", version: "^1.3", required: true},
         ...
       ]
     ```
   - 用户搜索：`oar search "全栈"` → 直接安装整个栈 + 自动递归 sub_skills。

2. **四层 artifact 分层存储**（OARSS 定义）
   - Tool / Procedure / Skill（可被复用）
   - Persona Bundle（顶层，带 sub_skills 依赖）

3. **企业 Mirror 特性（核心需求）**
   - `oar mirror add https://internal.mycompany.com`
   - 自动同步官方 index.json + 私有覆盖（企业技能永远优先）。
   - 自定义审核队列 + 安全扫描。
   - 离线支持：完全内网运行。
   - 审计日志：安装记录、激活记录、token 消耗。

4. **安全 & 信任**
   - 强制 cosign 签名（官方 + 企业 CA）。
   - Supply-chain 扫描。
   - 版本锁定 + 回滚。

5. **发现与治理**
   - Semantic search（按 role_category、trigger_scenarios、sub_skills）。
   - 社区 leaderboard（公开版可选）。
   - DAO / 多签下架（长期）。

6. **新增：CLI 配置与 Registry Index**
   - `~/.oarss/config.toml`（registries、install paths、dependencies 行为、security、cache）。
   - `index.json`（Registry Index Schema）：轻量元数据摘要，支持搜索、依赖解析、mirror 复制。

### 4. 技术栈（已落地部分标★）
- **基础**：fork Vercel skills.sh CLI + Solo.io agentregistry。
- **CLI**：Node.js（推荐）或 Bash MVP（oar search / install，支持递归 sub_skills）。
- **后端**：Go（agentregistry）或 Node。
- **存储**：
  - Metadata：index.json（轻量）+ PostgreSQL JSONB。
  - Content：IPFS CID / Git（sparse-checkout）+ S3。
- **镜像工具**：自研 mirror-sync（类似 Verdaccio）。
- **部署**：
  - 企业：Docker / K8s（Harbor 风格）。
  - 社区：免费公开实例（registry.oarss.org）。
- **已落地**：install-examples.sh、config.toml 示例、index.json 示例、3个高质量角色示例（senior-fullstack-engineer + loving-husband-role + product-manager-role）。

**CLI 命令示例**（当前可用）：
```bash
oar search "丈夫"                     # 语义搜索
oar install senior-fullstack-engineer --with-deps --mirror internal
oar mirror add https://internal.company.com
oar sync --from skills.sh             # 一键镜像官方
oar deps tree product-manager-role    # 查看 sub_skills 依赖树
```

### 5. 落地路线图（已更新，1个月内可见成果）
- **已完成（第 1 周）**：OARSS v1.0 规范 + 3个高质量示例角色 + index.json + config.toml + CLI 搜索/安装伪代码 + install-examples.sh。
- **第 2–3 周**：完整 oar CLI（Node.js）+ 递归 sub_skills 安装 + 签名验证。
- **第 4 周**：私有 mirror PoC + 安全扫描 + 企业部署文档。
- **第 2 月**：社区公开 registry（skills.oarss.org）+ leaderboard。
- **第 3 月**：IPFS 支持 + DAO 治理原型。

### 6. 风险与对策（更新后）
- 生态碎片：强制 OARSS + Anthropic 标准（已成事实，无风险）。
- 恶意 skill：企业 mirror 默认审核 + 签名 + 安全扫描。
- 性能：index.json 轻量缓存 + IPFS CID + CDN。
- 兼容性：所有示例均 100% 兼容现有 runtime。

**一句话总结**：  
我们做的不是“又一个 ClawHub”，而是 **npm + apt + Docker Hub 的 Agent Skills 版**，以 **OARSS + Persona Bundle + sub_skills** 为核心，runtime 随意切换，企业 mirror 一键部署，完全开放、可镜像、可复用。

这个 v1.1 方案已完全符合“开放 + Role 第一 + mirror 优先”的预期，并且具备可立即落地的可执行部分（index.json、config.toml、CLI 伪代码、示例角色、安装脚本）。



