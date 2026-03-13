### 1. CLI 配置文件格式（TOML 推荐，易读易写）

**推荐路径**：`~/.oarss/config.toml`（或项目级 `.oarss/config.toml`）

**示例内容**：

```toml
# 默认 registry（可多个，按顺序 fallback）
[registries]
default = "https://skills.oarss.org"
mirrors = [
  "https://internal.company.com/oarss-mirror",
  "https://github.com/open-agent-role-skill/oarss-examples/raw/main/index.json"
]

# 安装目标目录（优先级：命令行参数 > 项目 .oarss/ > 全局配置 > 默认）
[install]
default_target_dir = "~/.agent-skills"
# 常见 runtime 路径映射（CLI 可自动检测或提示）
runtime_paths = { "cursor" = "~/.cursor/skills", "claude-code" = "~/.claude-code/skills", "openclaw" = "~/.openclaw/skills" }

# 依赖行为
[dependencies]
auto_install_required = true          # 默认自动安装 required sub_skills
prompt_for_optional = true            # 对 optional sub_skills 询问用户
conflict_resolution = "highest-compatible"  # highest-compatible | prompt | fail

# 安全与验证
[security]
require_signature = false             # 是否强制 cosign/sigstore 验证（默认关闭，便于早期）
trusted_publishers = ["open-agent-role-skill", "your-company"]  # 白名单用户名/组织

# 缓存与性能
[cache]
enabled = true
max_age_hours = 24                    # 缓存过期时间
cache_dir = "~/.oarss/cache"

# 日志与调试
[logging]
level = "info"                        # debug | info | warn | error
```

**字段说明**（为什么这样设计）：

- registries：支持多源 fallback，企业 mirror 优先级最高
- install.runtime_paths：适配不同 runtime 的 skills 目录，避免用户手动指定
- dependencies：精细控制依赖安装行为（类似 cargo / npm 的 --no-deps）
- security：未来强制签名时可开启，早期保持灵活
- cache：避免频繁请求 registry，类似 Go proxy 或 npm cache

CLI 启动时会自动读取这个文件（fallback 到默认值）。
