# Forge-Lite 工作流使用指南

GStack + Superpowers + OpenSpec 三位一体的 AI 驱动开发方法论。

---

## 架构总览

```
┌───────────────────────────────────────────────────────┐
│                   Forge (编排层)                        │
│              版本管理 + 离线打包 + 初始化                 │
├───────────────────────────────────────────────────────┤
│                                                       │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  OpenSpec    │  │   GStack     │  │ Superpowers  │ │
│  │  工作流结构   │  │  生命周期技能  │  │  工程纪律     │ │
│  │             │  │              │  │              │ │
│  │  proposal   │  │  office-hours│  │  tdd         │ │
│  │  design     │  │  review      │  │  debugging   │ │
│  │  tasks      │  │  investigate │  │  verification│ │
│  │             │  │  qa-only     │  │  code-review │ │
│  └─────────────┘  └──────────────┘  └──────────────┘ │
│                                                       │
│  ┌─────────────────────────────────────────────────┐  │
│  │  CLAUDE.md — 定义有序工作流                       │  │
│  │  proposal → design → tasks → implementation →    │  │
│  │  verification                                     │  │
│  └─────────────────────────────────────────────────┘  │
│                                                       │
│  ┌─────────────────────────────────────────────────┐  │
│  │  Agent Roles — 专职审查角色                       │  │
│  │  architect · backend · frontend · security       │  │
│  │  tech-lead · tester                              │  │
│  └─────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────┘
```

**三者分工：**

| 组件 | 职责 | 类比 |
|------|------|------|
| **OpenSpec** | 定义工作流结构和产物规范 | 项目管理流程 |
| **GStack** | 提供生命周期各阶段的技能 | 各阶段的专家顾问 |
| **Superpowers** | 提供工程纪律和最佳实践 | 工程规范和检查清单 |

---

## 安装

```bash
# 1) 检查更新（写入 download/update.manifest）
~/forge/forge update

# 2) 下载工具到 download/
~/forge/forge download

# 3) 压缩并部署到工作台
~/forge/forge pack

# 4) 解压并部署到 ai/
~/forge/forge init

# 加载环境变量（PATH、镜像源、别名）
source ~/forge/shell/env.sh
```

安装完成后，Forge 会自动：

- 克隆 GStack 和 Superpowers 到 `download/`，复制到 `ai/tools/`
- 从 `download/openspec-x.x.x.tgz` 离线安装 OpenSpec（仍需本地 npm），链接到 `ai/bin/openspec`
- 将选定的技能符号链接到 `~/.claude/skills/`
- 部署 CLAUDE.md 和 schema.yaml 到标准位置

---

## 核心概念

### OpenSpec 产物

Forge-Lite 只保留三个核心产物，不创建额外的规划文档：

| 产物 | 路径 | 说明 |
|------|------|------|
| **proposal** | `docs/forge/{change_id}/proposal.md` | 做什么、不做什么 |
| **design** | `docs/forge/{change_id}/design.md` | 怎么做、边界和风险 |
| **tasks** | `docs/forge/{change_id}/tasks.md` | 可执行、可验收的任务清单 |

### 角色（Roles）

角色是专职审查者，负责在对应阶段提出专业意见：

| 角色 | 参与阶段 | 职责 |
|------|---------|------|
| architect | design | 审查架构、检查模块边界和数据流 |
| backend | design, tasks | API 设计、数据库、并发、性能 |
| frontend | design, tasks | UI 流程、状态管理、边界情况 |
| security | design | 认证、权限、输入校验、敏感数据 |
| tech_lead | tasks | 任务拆分、依赖排序、实现顺序 |
| tester | proposal, design, tasks | 验收标准、测试用例、回归风险 |

### 技能（Skills）

Forge 从 GStack 和 Superpowers 中各选取 4 个技能：

**GStack 技能（生命周期）：**

| 技能 | 命令空间 | 用途 |
|------|---------|------|
| `office-hours` | `gstack:office-hours` | YC 式头脑风暴，诊断方案可行性 |
| `review` | `gstack:review` | 设计/PR 审查 |
| `investigate` | `gstack:investigate` | 系统化根因调试 |
| `qa-only` | `gstack:qa-only` | 仅报告的 QA（不自动修复） |

**Superpowers 技能（工程纪律）：**

| 技能 | 命令空间 | 用途 |
|------|---------|------|
| `test-driven-development` | `superpowers:test-driven-development` | 测试驱动开发纪律 |
| `systematic-debugging` | `superpowers:systematic-debugging` | 结构化调试方法 |
| `verification-before-completion` | `superpowers:verification-before-completion` | 完成前的最终检查 |
| `requesting-code-review` | `superpowers:requesting-code-review` | 工程级代码审查 |

**可选技能：**

| 技能 | 用途 | 使用时机 |
|------|------|---------|
| `brainstorming` | 发散思维 | 仅在 proposal 阶段，不可在 tasks/implementation 阶段使用 |

**内置团队技能：**

- `team-workflow`（路径：`config/claude/skills/team-workflow`）
- 用于比赛/冲刺场景下的分支与提交约束（`feat/*`、Conventional Commits、`FINAL_APPROVE` 门禁）
- 执行 `forge init skills` 后会链接到 `~/.claude/skills/team-workflow`

---

## 工作流程

### 标准流程（5 步）

```
proposal → design → tasks → implementation → verification
   ①         ②        ③         ④              ⑤
```

#### 第 1 步：Proposal（提案）

**目标：** 明确要不要做、做什么、不做什么。

```bash
# 在 Claude Code 中描述需求
> 帮我做一个用户导出功能，支持 CSV 和 JSON 格式
```

Claude 会：
1. 调用 `gstack:office-hours` 进行需求诊断
2. 产出 `docs/forge/{change_id}/proposal.md`
3. 包含：背景、问题、目标、非目标、范围、用户故事、验收标准、风险

**退出标准：** 范围清晰、非目标已定义、验收标准存在、风险已列出。

#### 第 2 步：Design（设计）

**目标：** 明确怎么做、边界在哪里、风险怎么处理。

Claude 会：
1. 调用 `gstack:review` 进行设计审查
2. 产出 `docs/forge/{change_id}/design.md`
3. 包含：架构、API 契约、数据模型、状态流转、错误处理、兼容性、安全、可观测性、测试策略、发布计划

**退出标准：** 架构清晰、API 契约明确、数据模型确定、失败模式已考虑。

#### 第 3 步：Tasks（任务拆分）

**目标：** 拆成可执行、可验收、可追踪的任务。

Claude 会：
1. 调用 `superpowers:test-driven-development` 制定测试策略
2. 产出 `docs/forge/{change_id}/tasks.md`
3. 包含：里程碑、后端任务、前端任务、测试任务、迁移任务、发布检查

**退出标准：** 任务原子化、有验收检查、依赖关系清晰、包含测试任务。

#### 第 4 步：Implementation（实现）

**目标：** 按照任务清单编码实现。

Claude 会：
- 遵循任务清单逐项实现
- 按需调用技能（如 TDD 纪律）
- 遵守工程规则：先检查现有代码、理解约定、最小变更、保持风格

#### 第 5 步：Verification（验收）

**目标：** 合并前的最终检查。

Claude 会：
1. 调用 `gstack:qa-only` 进行 QA 报告
2. 调用 `superpowers:requesting-code-review` 进行代码审查
3. 调用 `superpowers:verification-before-completion` 进行最终验收检查
4. 运行相关测试、说明变更内容、列出验证结果和剩余风险，并写入 `tasks.md` 的 `verification_log`

---

### 快速通道（Fast Track）

对于小型、低风险的变更，可以跳过 proposal → design → tasks：

**适用场景：**
- 错别字或措辞修正
- 配置值调整
- 单行 bug 修复
- 重命名或格式化变更
- 添加/移除简单依赖

**使用方式：**

```bash
# 方式一：明确说"快速通道"
> fast track: 修复 README 中的拼写错误

# 方式二：中文指令
> 直接改：把日志级别从 DEBUG 改成 INFO

# 方式三：描述的变更明显是琐碎的（1-3 行，无逻辑变更）
> 修正 config.yaml 中的端口号
```

**快速通道保留：**
- 检查现有代码再修改
- 做最小必要变更
- 修改后验证变更有效

---

### 调查命令（Investigate）

遇到 bug 或线上问题时，使用调查模式：

```bash
# 在 Claude Code 中描述问题
> investigate: 用户登录后偶尔会话丢失
```

Claude 会调用 `gstack:investigate` + `superpowers:systematic-debugging`，进行系统化的根因分析。

---

### 验收命令（Verify）

合并前的独立验收：

```bash
> verify: 检查当前分支是否可以合并
```

Claude 会调用 `gstack:qa-only` + `superpowers:requesting-code-review` + `superpowers:verification-before-completion`，产出完整的验收报告。

---

## 实际使用示例

### 示例 1：完整流程

```bash
# 1. 启动 Claude Code
claude

# 2. 描述需求（触发标准流程）
> 帮我实现一个文件上传功能，支持拖拽和点击上传，限制 10MB，支持图片和 PDF

# Claude 会依次产出：
#   docs/forge/file-upload/proposal.md
#   docs/forge/file-upload/design.md
#   docs/forge/file-upload/tasks.md
# 然后按照 tasks.md 逐项实现
```

### 示例 2：快速修复

```bash
# 直接说"快速通道"
> fast track: 修复 utils.ts 第 42 行的空指针错误

# Claude 跳过 proposal/design/tasks，直接检查代码并修复
```

### 示例 3：排查问题

```bash
# 使用 investigate 命令
> investigate: CI 构建在 Node 20 上失败，Node 18 正常

# Claude 进行系统化根因分析
```

---

## 技能调用矩阵

以下表格展示每个阶段会调用哪些技能和角色：

| 阶段 | GStack 技能 | Superpowers 技能 | 角色 |
|------|------------|-----------------|------|
| **proposal** | `office-hours` | `brainstorming`（可选） | tester |
| **design** | `review` | — | architect, backend, frontend, security, tester |
| **tasks** | — | `test-driven-development`, `requesting-code-review` | tech_lead, backend, frontend, tester |
| **implementation** | 按需 | 按需 | — |
| **verification** | `qa-only` | `requesting-code-review`, `verification-before-completion` | — |
| **investigate** | `investigate` | `systematic-debugging` | — |
| **verify** | `qa-only` | `requesting-code-review`, `verification-before-completion` | — |

---

## 工程规则

### 修改代码前

1. 检查现有文件
2. 理解当前约定
3. 做最小必要变更
4. 保持现有风格
5. 避免大范围重写

### 标记完成前

1. 运行相关测试（如果有）
2. 说明做了什么变更
3. 列出已执行的验证
4. 列出剩余风险（如果有）

### 角色行为准则

**角色必须：**
- 指出不清晰的假设
- 识别缺失的边界情况
- 检查实现风险
- 给出具体的修改建议
- 避免模糊的建议

**角色不得：**
- 编造需求
- 未经批准扩大范围
- 创建竞争性的 proposal/design/tasks 文档
- 未经用户批准覆盖现有产物

---

## 团队协作

对于团队使用，Forge 支持：

- **GBrain 服务器：** 共享知识/记忆的 HTTP MCP 服务（端口 3131）
- **GitLab specs 仓库：** 通过 git 共享 OpenSpec 产物（proposal/design/tasks）
- **多开发者连接：** 连接同一个 GBrain 服务器，通过 git 推拉 specs

---

## 文件参考

| 文件 | 用途 |
|------|------|
| `config/claude/CLAUDE.md` | 全局 Claude Code 指令（工作流定义） |
| `config/openspec/schema.yaml` | OpenSpec schema（阶段、角色、技能映射） |
| `config/claude/agents/*.md` | Agent 角色定义（6 个角色） |
| `registry/gstack.sh` | GStack 安装清单 |
| `registry/superpowers.sh` | Superpowers 安装清单 |
| `registry/openspec.sh` | OpenSpec 安装清单 |

---

## 状态流转

产物有以下状态，按顺序流转：

```
draft → review → approved → implemented → archived
         ↑         │
         └─────────┘  (可回退到 draft)
```

- **draft：** 草稿，可编辑
- **review：** 提交审查
- **approved：** 审查通过
- **implemented：** 已实现
- **archived：** 已归档
