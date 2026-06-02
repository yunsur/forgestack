# Forge

离线 AI 工作站 — 一个自包含的 AI 开发环境，支持内网迁移。

## 快速开始

```bash
# 1. 检查可用更新
./forge update

# 2. 下载工具包（仅下载）
./forge download

# 3. 初始化运行环境（解压+配置+skills+mcp+链接）
./forge init

# 4. 加载环境
source shell/env.sh

# 5. 检查环境
./forge doctor
```

## 目录结构

```
forge/
├── forge                # CLI 入口
├── shell/
│   ├── env.sh           # 环境变量（source 加载）
│   └── forge/*.sh       # 命令模块
├── registry/            # 工具清单（每个工具一个 .sh）
├── config/
│   ├── claude/          # CLAUDE/agents/mcp 配置
│   └── openspec/        # OpenSpec schema 配置
├── download/            # 下载缓存与 manifest
├── versions.lock        # 已安装版本记录
└── ai/                  # 运行时（gitignore）
    ├── bin/             # 工具符号链接
    ├── tools/           # 工具安装目录
    ├── runtimes/        # 运行时（pyenv, python）
    ├── mcp/             # 合并 MCP 暂存
    └── cache/           # 缓存（pip, cargo, npm）
```

## forge 命令

| 命令 | 说明 |
|------|------|
| `forge` | 检查并提示更新 |
| `forge -a` | 检查并更新全部 |
| `forge list` | 显示工具状态 |
| `forge update` | 仅检查可用更新 |
| `forge download [tool...]` | 下载工具到 `download/`（不解压） |
| `forge init [tools|config|skills|mcp|bins]` | 初始化运行环境 |
| `forge uninstall <tool>` | 卸载工具 |
| `forge skills install <owner/repo/skill>` | 下载 skill |
| `forge skills install <owner/repo>` | 下载整个 skill 仓库 |
| `forge skills list` | 显示已安装 skills |
| `forge mcp install` | 安装 MCP server 包 |
| `forge mcp list` | 显示 MCP server 配置 |
| `forge doctor` | 环境检查 |
| `forge pack [file.tgz]` | 打包整站（含二进制）用于内网迁移 |
| `forge new <name>` | 生成新工具的 manifest 模板 |

## 工具清单

| 工具 | 说明 |
|------|------|
| rg (ripgrep) | 快速搜索 |
| fd | 文件查找 |
| fzf | 模糊搜索 |
| jq / yq | JSON/YAML 处理 |
| bat | 语法高亮 cat |
| eza | 现代 ls |
| delta | git diff 增强 |
| lazygit | git TUI |
| ast-grep (sg) | AST 搜索 |
| just | 任务运行器 |
| uv | Python 包管理器 |
| node / npm / npx | Node.js |
| python (pyenv) | Python（源码编译） |
| go | Go 工具链 |
| rust / cargo | Rust 工具链 |
| claude | Claude Code CLI |
| codex | OpenAI Codex CLI |
| bun | JavaScript 运行时（GStack 依赖） |
| openspec | 规范驱动开发框架（proposal/design/tasks） |
| gstack | AI 全生命周期工具集（12 skills 选择安装） |
| superpowers | AI 开发最佳实践（5 skills 选择安装） |

## 代理配置

编辑 `shell/env.sh` 取消注释对应的代理行：

```bash
export HTTP_PROXY="http://127.0.0.1:7890"
export HTTPS_PROXY="http://127.0.0.1:7890"
```

已配置的国内镜像：

| 生态 | 镜像 |
|------|------|
| PyPI | mirrors.aliyun.com |
| npm | registry.npmmirror.com |
| Go | goproxy.cn |
| Rust | rsproxy.cn |

## 内网迁移

```bash
# 有网机器：打包
./forge pack              # 含二进制（~500MB）

# 传输到内网
scp forge-*.tgz target:~/

# 内网机器：解压
tar xzf forge-*.tgz
cd forge
source shell/env.sh
```

## AI 开发工具选择安装

GStack、Superpowers 采用**选择安装**策略，只安装需要的 skills：

### GStack（4 skills）

| 类别 | Skills |
| --- | --- |
| 需求/方案挑战 | office-hours |
| 设计评审 | review |
| 问题调查 | investigate |
| QA/验收 | qa-only |

```bash
forge download gstack
forge init tools
forge init skills       # 自动链接到 ~/.claude/skills/gstack-*
```

### Superpowers（4 skills）

| 类别 | Skills |
|------|--------|
| 测试驱动开发 | test-driven-development |
| 系统化调试 | systematic-debugging |
| 完成前验证 | verification-before-completion |
| 工程代码审查 | requesting-code-review |

```bash
forge download superpowers
forge init tools
forge init skills            # 自动链接到 ~/.claude/skills/sp-*
```

### OpenSpec（精简工作流）

使用自定义 schema `config/openspec/schema.yaml`，仅保留：
- `proposal` — 需求提案
- `design` — 设计文档
- `tasks` — 任务清单

```bash
forge download openspec
forge init tools
forge init config
# 离线使用：OPENSPEC_TELEMETRY=0 已在 shell/env.sh 中配置
```

## 团队协作

### 架构

```
┌─────────────┐     HTTP MCP      ┌─────────────────┐
│  开发者 A    │ ───────────────→  │  GBrain 服务器    │
│  (客户端)    │                   │  gbrain serve    │
└─────────────┘                   │  :3131           │
                                  │  共享知识库        │
┌─────────────┐     HTTP MCP      └─────────────────┘
│  开发者 B    │ ───────────────→         ↑
└─────────────┘                          │
                                  内网 IP
┌─────────────┐     HTTP MCP      ┌─────────────────┐
│  开发者 C    │ ───────────────→  │  内部 GitLab      │
└─────────────┘                   │  specs 仓库       │
                                  │  OpenSpec 协作     │
                                  └─────────────────┘
```

### 服务端部署（一台机器）

```bash
# 1. 安装工具
./forge update && ./forge download && ./forge init
source shell/env.sh

# 2. 初始化 GBrain
bash scripts/gbrain-server.sh init

# 3. 后台启动（不需要 root）
bash scripts/gbrain-server.sh start-bg

# 4. 管理
bash scripts/gbrain-server.sh status     # 查看状态
bash scripts/gbrain-server.sh logs       # 查看日志
bash scripts/gbrain-server.sh stop       # 停止
bash scripts/gbrain-server.sh restart    # 重启

# 5. 开机自启（二选一，都不需要 root）
# 方案A: 用户级 systemd
mkdir -p ~/.config/systemd/user
bash scripts/gbrain-server.sh user-systemd > ~/.config/systemd/user/gbrain.service
systemctl --user enable gbrain
sudo loginctl enable-linger $(whoami)   # 允许未登录时运行

# 方案B: crontab
crontab -e
# 添加: @reboot bash /path/to/scripts/gbrain-server.sh start-bg
```

### 客户端配置（每台开发机）

```bash
# 1. 解压工具包
tar xzf forge-*.tgz && cd forge && source shell/env.sh

# 2. 连接 GBrain 服务器
GBRAIN_SERVER=http://<服务器IP>:3131 source scripts/team-setup.sh gbrain

# 3. 配置 OpenSpec GitLab 仓库
GITLAB_URL=http://gitlab.internal TEAM_GROUP=team/specs \
  source scripts/team-setup.sh gitlab

# 4. 生成项目 CLAUDE.md
source scripts/team-setup.sh claude-md /path/to/project

# 5. 重启 Claude Code 生效
```

### 团队工作流

| 步骤 | 命令 | 说明 |
|------|------|------|
| 1. 拉取最新 | `git pull` | 同步代码和 specs |
| 2. 创建提案 | `/opsx:propose` | OpenSpec 创建 proposal |
| 3. 设计 | `/opsx:propose` → design | 编写设计文档 |
| 4. 任务拆分 | `/opsx:propose` → tasks | 拆分任务清单 |
| 5. 实现 | `/opsx:apply` | 按任务清单实现 |
| 6. 审查 | `/review` 或 `/requesting-code-review` | 代码审查 |
| 7. 提交 | `git add . && git commit && git push` | 共享给团队 |
| 8. 回顾 | `/retro` | 每周工程回顾 |
| 9. 保存记忆 | `/context-save` | 保存到 GBrain |

### 共享内容

| 内容 | 存储位置 | 共享方式 |
|------|---------|---------|
| 代码 | GitLab 仓库 | git push/pull |
| specs (proposal/design/tasks) | GitLab specs 仓库 | git push/pull |
| 知识/记忆 | GBrain 服务器 | HTTP MCP 实时共享 |
| 开发经验 | GBrain learnings | gbrain put/search |
| 进度/timeline | GBrain timeline | 自动记录 |

## Skills

从 [skills.sh](https://skills.sh) 下载 Agent Skills：

```bash
# 单个 skill
forge skills install anthropics/skills/frontend-design

# 整个仓库
forge skills install obra/superpowers

# skills 自动链接到 ~/.claude/skills/，Claude Code 和 Codex 均可加载
```

内置技能（仓库自带）通过 `forge init skills` 部署：

```bash
# 部署内置 skill 到 ~/.claude/skills/
forge init skills

# 比赛协作流程技能（分支/提交/final-approve 约束）
~/.claude/skills/team-workflow
```

## MCP 配置

MCP Server 配置在 `config/claude/mcp.json`，通过 `forge init mcp` 合并到 `~/.claude/mcp.json`。

```bash
# 编辑仓库内 MCP 配置
$EDITOR config/claude/mcp.json

# 合并到 ~/.claude/mcp.json
forge init mcp
forge mcp list
```
