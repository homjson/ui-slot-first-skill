# ui-slot-first

> Agent Skill:从 UI 设计稿生成组件代码时,强制优先使用组件库提供的 slot/prop/API,而非用 HTML 元素重新实现组件内部结构。

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-2ea44f)](https://agentskills.io)

## 解决什么问题

用 Figma MCP / MasterGo MCP 把设计稿喂给 AI 生成代码时,AI 倾向于直接用 `<div>` 拼一个 head 区域,而非用 `el-drawer` 的 `#title` slot。这会丢失组件内置行为(ESC 关闭、ARIA、主题联动),也与组件库主题脱节。

本 skill 给 AI 一条**可执行的决策流程**:先查组件 slot 文档 → 能用就用 → 样式用 `:deep()` 穿透 → 实在不行才自实现并标注理由。

## 安装

### 方式 A:skills CLI(推荐,自动适配多 Agent)

```bash
# 发布到 GitHub 后(把 <your-name> 换成你的 GitHub 用户名):
npx skills add <your-name>/ui-slot-first-skill

# 本地开发期直接用路径:
npx skills add ./ui-slot-first-skill
```

CLI 会自动检测本机已安装的 Agent(Claude Code / Codex / Cursor / GitHub Copilot / Windsurf / Cline / Zed / VS Code 等)并把 skill 放到对应目录。

### 方式 B:手动放置到各 Agent 的 skill 目录

| Agent | skill 目录 |
|-------|-----------|
| Claude Code | `~/.claude/skills/ui-slot-first/SKILL.md`(全局)或项目 `.claude/skills/` |
| Codex (OpenAI) | `~/.codex/skills/ui-slot-first/SKILL.md` |
| Cursor | `.agents/skills/ui-slot-first/SKILL.md`(项目级) |
| GitHub Copilot | `.github/copilot-instructions.md` 中引用,或 `.agents/skills/` |
| Windsurf | `.windsurf/skills/ui-slot-first/SKILL.md` |
| Cline | `.cline/skills/ui-slot-first/SKILL.md` |

把 `skills/ui-slot-first/` 整个目录复制过去即可。

## 目录结构

```
ui-slot-first-skill/
├── README.md
└── skills/
    └── ui-slot-first/
        └── SKILL.md        # skill 本体(frontmatter + 规则正文)
```

遵循 [Agent Skills Specification](https://agentskills.io),扁平布局 `skills/<name>/SKILL.md`,跨 Agent 通用。

## SKILL.md frontmatter

```yaml
---
name: ui-slot-first
description: Enforce slot-first UI implementation when generating component code
  from design files via MCP (Figma, MasterGo, Lanhu)... Trigger when writing
  Vue/React UI code from design specs...
---
```

- `name`:小写+连字符,符合规范
- `description`:Agent 据此判断何时激活本 skill。包含关键词(MCP/Figma/design-to-code/component library/slot/Element Plus),触发覆盖广
- **未使用** `context: fork`(仅 Claude Code 支持)和 hooks(多数 Agent 不支持),以保最大兼容性

## 触发场景

- 通过 Figma / MasterGo / 蓝湖 / 即时设计 MCP 读取设计稿生成代码
- 使用 Element Plus / Ant Design Vue / Naive UI / Vuetify / PrimeVue 实现 UI
- AI 出现"自己拼 div 实现 head/footer"倾向时

## 规则要点

1. **先查组件 slot 文档** → 确认可用 slot
2. **slot 能满足就用**,样式差异用 `:deep()` 穿透而非替换结构
3. **部分满足** → slot + 局部自定义组合
4. **完全不能满足**(结构层级差异巨大 / 组件不支持的行为 / 样式钩子无法穿透)→ 方自行实现,并在注释中标注理由

完整规则与 `el-drawer #title` 正反例见 `skills/ui-slot-first/SKILL.md`。

## 发布到 skills.sh

1. 本仓库设为 Public
2. 确保根目录或 `skills/<name>/` 下有 `SKILL.md`(已满足)
3. 推到 GitHub
4. 任何人执行 `npx skills add <your-name>/ui-slot-first-skill` 即安装
5. 通过 CLI 匿名安装遥测自动出现在 [skills.sh](https://skills.sh) 榜单(无需审核)

> 关闭遥测:`DISABLE_TELEMETRY=1`

## License

MIT
