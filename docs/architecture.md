# Pain Point Monitor — 项目架构设计

> 本文档定义 pain-point-monitor 仓库的架构规范与目录组织原则。

---

## 1. 项目概述

### 1.1 项目定位

Pain Point Monitor 是一个 **AI 编码技能与插件集合**，为多种 AI 编码代理工具（Claude Code、Cursor、Codex、OpenCode）提供结构化的生产级开发指导，涵盖前端、后端、移动端、图形 Shader 及多媒体内容生成等领域。

### 1.2 核心价值

| 能力 | 说明 |
|------|------|
| 全栈开发指导 | React/Next.js、REST API、Auth、WebSocket |
| 移动端开发 | Android (Kotlin/Jetpack)、iOS (Swift/SwiftUI)、Flutter、React Native |
| AI 多媒体生成 | PDF、PPTX、XLSX、DOCX、音乐、视频、图像 — 全部基于 MiniMax API |
| Shader 图形 | GLSL 着色器：光线追踪、SDF、流体模拟、粒子系统 |
| 写作框架 | AIDA 文案框架、AI 文本 humanizer |

---

## 2. 架构原则

### 2.1 设计目标

1. **多代理兼容** — 同一套 skill 可同时服务于 Claude Code、Cursor、Codex、OpenCode
2. **原子化 Skill** — 每个 skill 职责单一，可独立安装使用
3. **可验证质量** — 所有 skill 必须通过 pr-review 自动化校验
4. **跨平台一致** — Windows/macOS/Linux 行为一致

### 2.2 核心规范

- **Skill 结构标准化**：每个 skill 必含 `SKILL.md`，可选含 `design/`、`scripts/`、`references/`
- **零魔法数字**：所有阈值、限制、超时参数必须在 `SKILL.md` 开头以常量形式声明
- **版本锁定依赖**：所有外部工具链（Node.js、Python、FFmpeg）版本必须声明
- **CI 前置校验**：PR 必须通过 `python .claude/skills/pr-review/scripts/validate_skills.py`

---

## 3. 目录架构

```
pain-point-monitor/
│
├── README.md                      # 英文总览
├── README_zh.md                   # 中文总览
├── CONTRIBUTING.md                # 贡献指南
├── LICENSE                        # MIT License
├── .gitignore
│
├── docs/                          # 文档中心 [新增]
│   ├── architecture.md            # 本文档 — 项目架构规范
│   ├── changelog.md               # 变更日志
│   ├── roadmap.md                # 路线图
│   ├── skin-standards.md          # Skill 皮肤/格式规范
│   └── multi-agent-support.md     # 多代理兼容说明
│
├── assets/                        # 全局静态资源
│   ├── logo.png                   # 项目 Logo
│   └── images/                   # 文档用图
│
├── .claude/                       # Claude Code 自身配置
│   └── skills/
│       └── pr-review/            # PR 自动审查 skill
│           ├── SKILL.md
│           ├── references/
│           └── scripts/
│               └── validate_skills.py  # Skill 校验脚本
│
├── .cursor-plugin/               # Cursor 插件配置
│   └── INSTALL.md                # Cursor 安装说明
│
├── .codex/                        # Codex 配置
│   └── INSTALL.md                # Codex 安装说明
│
├── .opencode/                    # OpenCode 配置
│   └── INSTALL.md                # OpenCode 安装说明
│
├── .claude-plugin/               # Claude Code 官方插件集成 [重构自根目录]
│   └── ...
│
├── skills/                       # AI 编码代理 Skills 集合
│   │
│   ├── _skeleton/               # Skill 模板 [新增]
│   │   ├── SKILL.md
│   │   ├── design/
│   │   ├── scripts/
│   │   └── references/
│   │
│   ├── ai/                      # AI 内容生成类 [重组]
│   │   ├── minimax-multimodal-toolkit/
│   │   ├── minimax-music-gen/
│   │   ├── minimax-music-playlist/
│   │   ├── buddy-sings/
│   │   ├── vision-analysis/
│   │   └── gif-sticker-maker/
│   │
│   ├── document/                # 文档生成类 [新增分类]
│   │   ├── minimax-pdf/
│   │   ├── minimax-docx/
│   │   ├── minimax-xlsx/
│   │   └── pptx-generator/
│   │
│   ├── writing/                 # 写作/文案类 [重组]
│   │   └── (humanizer + AIDA framework)
│   │
│   ├── dev/                    # 开发技能类
│   │   ├── frontend-dev/
│   │   ├── fullstack-dev/
│   │   ├── shader-dev/
│   │   └── gif-sticker-maker/   # (移动到 ai/，此处保留引用)
│   │
│   ├── mobile/                 # 移动开发类 [新增分类]
│   │   ├── android-native-dev/
│   │   ├── ios-application-dev/
│   │   ├── flutter-dev/
│   │   └── react-native-dev/
│   │
│   └── _archived/              # 已废弃 skill [新增]
│       └── (迁移中的旧 skill)
│
├── plugins/                     # 插件 (非 skill 格式的扩展)
│   ├── pptx-plugin/
│   └── (其他插件)
│
├── scripts/                     # 全局工具脚本 [新增，提升到根目录]
│   ├── setup.sh                # 仓库初始化
│   ├── sync-skills.js          # 多代理同步工具
│   └── bootstrap.py            # Skill 脚手架生成器
│
└── tests/                       # 全局测试 [新增]
    ├── unit/
    │   ├── validate_skills_test.py
    │   └── skill_structure_test.py
    └── integration/
        └── multi_agent_test.sh
```

---

## 4. Skill 结构规范

每个 Skill 必须遵循以下目录结构：

```
<skill-name>/
├── SKILL.md                      # [必填] Skill 主定义文件
├── design/                       # [可选] 设计资源
│   ├── cover.png                # 封面图
│   └── assets/                  # 内部设计文件
├── scripts/                      # [可选] 自动化脚本
│   ├── make.sh                  # 构建脚本
│   └── *.py / *.js              # 辅助脚本
├── references/                   # [可选] 参考文档
│   ├── api-spec.md             # API 规格说明
│   └── glossary.md              # 术语表
└── tests/                       # [可选] Skill 自身测试
    └── test_skill.py
```

### 4.1 SKILL.md 结构

```markdown
---
name: <skill-name>
description: <一句话描述>
version: <语义化版本>
agent: [claude-code | cursor | codex | opencode | all]
minimax-api: [true | false]
---

# <Skill 显示名称>

## 1. 概述
<Skill 的完整描述>

## 2. 能力边界
<明确此 Skill 能做什么 / 不能做什么>

## 3. 前置要求
- 工具链要求
- API Key 配置
- 环境要求

## 4. 使用限制
| 参数 | 值 | 说明 |
|------|-----|------|
| MAX_TOKENS | 8192 | 单次输出上限 |
| TIMEOUT_MS | 30000 | API 超时 |

## 5. 核心工作流
<Step-by-step 使用流程>

## 6. 常见错误排查
| 错误 | 原因 | 解决方案 |
|------|------|----------|
```

---

## 5. 分支策略

```
main                  # 稳定版，始终可安装
├── develop           # 开发版，下一版本特性
├── skill/<name>      # 单个 skill 的独立开发分支
├── fix/<issue-id>    # Bug 修复分支
└── docs/<topic>      # 文档更新分支
```

**合并规则**：
- Skill 分支 → develop：需要 1 个 reviewer 通过 + pr-review CI 绿灯
- develop → main：需要 2 个 reviewer 通过 + 完整 CI
- 紧急修复可走 fast-track，但事后需补 review

---

## 6. CI/CD 流程

```
PR 提交
  │
  ├─→ validate_skills.py       # Skill 结构校验
  ├─→ lint_markdown.py         # SKILL.md 格式校验
  ├─→ check_dependencies.py   # 依赖版本校验
  │
  └─→ pr-review skill          # AI 自动审查
        │
        └─→ Review 结果
              ├─ PASS → reviewer 收到通知
              └─ FAIL → PR 作者收到修改建议
```

---

## 7. 多代理支持矩阵

| Skill | Claude Code | Cursor | Codex | OpenCode |
|-------|:-----------:|:------:|:-----:|:--------:|
| frontend-dev | ✅ | ✅ | ✅ | ✅ |
| fullstack-dev | ✅ | ✅ | ✅ | ✅ |
| android-native-dev | ✅ | ✅ | ❌ | ❌ |
| ios-application-dev | ✅ | ✅ | ❌ | ❌ |
| minimax-pdf | ✅ | ✅ | ✅ | ✅ |
| minimax-multimodal-toolkit | ✅ | ✅ | ✅ | ✅ |
| shader-dev | ✅ | ✅ | ✅ | ✅ |

---

## 8. 变更记录

| 日期 | 版本 | 变更内容 | 作者 |
|------|------|----------|------|
| 2026-04-24 | 1.0.0 | 初始架构文档 | Claude |
