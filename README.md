# Open Agent Role-Skill Spec (OARSS)

**让 AI Agent 真正像“人”一样工作**

OARSS 是一个开放规范，旨在让 AI Agent 生态从“散乱的最佳实践集合”升级为**可切换的专业角色技能栈**。

- **核心理念**：同一个 Runtime（Claude Code / Cursor / OpenClaw / Gemini CLI 等），可以随时切换不同“人设”——资深全栈工程师、温柔体贴的丈夫、严谨的代码审查专家……
- **第一公民**：**Role / Persona Bundle**（角色包），它不是简单堆砌技能，而是**人设 + 协调逻辑 + 子技能依赖**。
- **完全兼容**：底层 100% 遵循 [Anthropic Agent Skills 开放标准](https://agentskills.io/specification)（SKILL.md 文件夹格式），通过 `metadata` 扩展实现清晰分层。
- **模块化设计**：Persona 只做“人设与大脑”，真实能力封装在独立的 **sub_skills**（子技能）中，实现高复用、低 token 消耗、可独立迭代。

当前日期：2026 年 3 月  
灵感来源：Vercel skills.sh、Anthropic Agent Skills、OpenClaw 生态、社区数万 skill 实践

## 为什么需要 OARSS？

现有 Agent Skills 生态虽然开放，但存在明显问题：

- 大部分 skill 是原子工具或流程，缺少“人”的整体感
- 角色扮演（persona）与具体技能混在一起，难以复用、维护、搜索
- 企业/社区 mirror 场景下，缺乏清晰的分层与依赖管理

OARSS 解决这些痛点：

- **Role/Persona 第一**：registry 里搜索“前端工程师”就能拉到一个完整人设包
- **sub_skills 独立**：React 最佳实践可以被多个角色复用，单独升级
- **渐进加载友好**：Persona 本身轻量，只在需要时递归加载子技能
- **Mirror / 私有化友好**：企业可以轻松搭建私有 registry，审核、签名、离线同步

## 四层结构（清晰区分）

| 层级          | metadata.type       | 核心价值                               | 典型例子（dev 场景）                     | 是否适合 sub_skill |
|---------------|---------------------|----------------------------------------|------------------------------------------|---------------------|
| Tool          | `tool`              | 原子能力、脚本、API 调用               | browser-navigation, git-commit           | 是（底层依赖）      |
| Procedure     | `procedure`         | 详细步骤、checklist、工作流            | code-review-checklist, refactor-pattern  | 是（高频复用）      |
| Skill         | `skill`             | 领域最佳实践 + 判断逻辑                | react-best-practices, typescript-patterns | 是（核心模块）      |
| Persona Bundle| `persona-bundle`    | 完整人设 + backstory + 子技能协调      | senior-fullstack-engineer, loving-husband | —（顶层组装）       |

## 快速上手

1. **安装 CLI**（未来计划基于 skills.sh 扩展，目前可手动 clone）

```bash
# 临时方式：clone 本仓库使用示例脚本
git clone https://github.com/your-org/oarss.git
cd oarss
# 后续会发布 npx oar / npm install -g @oarss/cli
```

2. **安装一个 Persona 示例**

```bash
# 未来命令（规划中）
oar install senior-fullstack-engineer --with-dependencies --mirror our-registry.example.com

# 当前手动方式
git clone https://github.com/your-org/oar-skills
# 复制 senior-fullstack-engineer 文件夹到 ~/.agent-skills/ 或项目 .skills/
```

3. **在 Agent 中使用**

- Claude Code / Cursor 等会自动发现 SKILL.md
- 描述中包含触发词 → 自动激活对应 Persona

## 规范核心：SKILL.md 扩展约定（OARSS v1.0）

所有内容仍是一个标准 SKILL.md 文件夹，扩展点在 `metadata`：

```yaml
---
name: senior-fullstack-engineer
description: 处理全栈 Web 项目（React/Next.js + Node/Supabase）时激活。8 年经验全栈视角，注重性能、安全、DX。
metadata:
  type: persona-bundle
  role_category: ["开发-全栈", "工程师"]
  persona_backstory: |
    你是一位 8 年经验的全栈工程师，写代码干净、高性能、可维护……
  trigger_scenarios:
    - "React / Next.js / TypeScript / Supabase"
    - "写组件 / API / 数据库设计 / review"
  sub_skills:
    - name: react-best-practices
      version: "^1.0"
      required: true
    - name: supabase-postgres-best-practices
      version: ">=2.0"
    - name: code-review-checklist
      optional: true
  compatible_runtimes: ["*", "claude-code", "cursor"]
  estimated_tokens: { brief: 150, full: 3200 }
---
# 正文：角色协调指令
当任务涉及前端 → 优先注入 react-best-practices
涉及数据库 → 注入 supabase-postgres-best-practices
输出代码前 → 运行 code-review-checklist 检查
……
```

## 子技能（sub_skills）示例结构

```text
react-best-practices/
├── SKILL.md
├── references/
│   └── hooks-guidelines.md
└── scripts/
    └── perf-check.ts     # 可选辅助脚本
```

## 生态定位与 Roadmap

- **v1.0（当前）**：规范定义 + 示例 Persona/ sub_skill + CLI 草稿
- **v1.1**：完整 CLI（install / search / deps / mirror sync）
- **v1.2**：Registry 参考实现（基于 agentregistry fork 或轻量 Go 服务）
- **v2.0**：签名（cosign）、IPFS 内容寻址、社区 leaderboard

## 如何贡献

1. 遵循 [CONTRIBUTING.md](CONTRIBUTING.md)
2. 新增 Persona 或 sub_skill → 提交 PR 到 examples/ 或 skills/ 目录
3. 改进规范 → 在 issues 中提出 → 讨论后合并到 spec.md

## 相关项目 & 感谢

- [Anthropic Agent Skills Spec](https://agentskills.io) —— 基础格式标准
- [Vercel skills.sh](https://github.com/vercel-labs/skills) —— 灵感来源 & CLI 参考
- OpenClaw / Cursor / Codex / Gemini CLI 社区 —— 真实使用场景驱动

**让我们一起构建一个真正“以终为始”的 Agent 世界：从“角色”出发，而不是从工具出发。**

欢迎 star、fork、issue、PR！


如果你需要调整语气（更正式 / 更技术 / 更社区化）、添加徽章、logo、demo GIF、具体仓库链接等，随时告诉我，我们继续完善。
