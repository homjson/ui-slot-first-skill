# ui-slot-first

> Agent Skill: when generating UI code from Figma MCP, MasterGo MCP, Lanhu, screenshots, or other design sources, prefer component-library slots/props/APIs before rebuilding component internals with raw HTML.

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-spec-2ea44f)](https://agentskills.io)

## What Problem This Solves

When an AI reads a UI design through MCP, it often recreates component internals directly with HTML. For example, Element Plus `el-drawer` already provides a `#title` slot, but if the design title has custom styles, the AI may build a new `head` or `header` area by hand.

That usually breaks the component-library contract: built-in keyboard behavior, ARIA semantics, close controls, theme integration, spacing, and upgrade compatibility can be lost.

This skill gives the agent a strict implementation rule:

1. Identify the library component.
2. Check its slots, props, render hooks, and style hooks.
3. Use those extension points first.
4. Treat style differences as styling work, not a reason to replace structure.
5. Use raw HTML only when the official API cannot express the design, and record the reason.

## Install

```bash
npx skills add homjson/ui-slot-first-skill
```

For local development:

```bash
npx skills add ./ui-slot-first-skill
```

The CLI detects installed agents and places the skill in the appropriate directory.

## Structure

```text
ui-slot-first-skill/
├── README.md
├── LICENSE
└── skills/
    └── ui-slot-first/
        └── SKILL.md
```

## Trigger Scenarios

- Reading a UI design through Figma MCP, MasterGo MCP, Lanhu, Jijing, screenshots, or design-to-code specs
- Implementing UI with Element Plus, Ant Design Vue, Naive UI, Vuetify, PrimeVue, Ant Design, MUI, Chakra UI, or similar component libraries
- Building component regions such as titles, headers, footers, actions, prefixes, suffixes, empty states, table cells, drawer headers, dialog footers, and toolbars
- Preventing the agent from replacing library component internals with custom HTML just because the design has different spacing, color, font, or icon styling

## Core Rule

Do not rebuild a component's internal region with raw HTML until the component library's extension points have been checked and ruled out.

Preferred order:

1. Component prop/API
2. Named/default slot, render prop, or render hook
3. Theme token, CSS variable, class hook, or scoped style override
4. Small wrapper inside an official slot
5. Raw HTML replacement only after the above cannot satisfy the design

If raw HTML is necessary, the agent should leave a short note:

```text
Slot-first fallback: checked <component> <slot/prop/API>; using custom HTML because <reason>.
```

## Example: Element Plus Drawer Title

Avoid recreating the header:

```vue
<el-drawer v-model="visible" :show-close="false">
  <div class="drawer-head">
    <span class="drawer-title">Order details</span>
    <button class="drawer-close" @click="visible = false">x</button>
  </div>
</el-drawer>
```

Prefer the built-in slot:

```vue
<el-drawer v-model="visible">
  <template #title>
    <div class="drawer-title">
      <span>Order details</span>
      <el-tag type="info">Draft</el-tag>
    </div>
  </template>
</el-drawer>
```

Then adjust visuals with scoped styles, theme tokens, CSS variables, or documented class hooks.

## License

MIT
