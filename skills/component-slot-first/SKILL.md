---
name: component-slot-first
description: Enforce slot-first implementation for UI code generated from design sources such as Figma MCP, MasterGo MCP, Lanhu, Jijing, screenshots, or design-to-code specs. Use when implementing Vue, React, or other frontend UI with component libraries such as Element Plus, Ant Design Vue, Naive UI, Vuetify, PrimeVue, Ant Design, MUI, Chakra UI, or similar. Prefer the library component's slots, props, render hooks, theme tokens, and documented APIs for headers, titles, footers, actions, prefixes, suffixes, empty states, cells, overlays, and toolbars before rebuilding those regions with custom HTML.
---

# Slot-First UI Implementation

When generating UI code from a design file, do not rebuild a component's internal regions with raw HTML until the component library's extension points have been checked and ruled out.

The default implementation order is:

1. Component prop/API
2. Named/default slot, render prop, or render hook
3. Theme token, CSS variable, class hook, or scoped style override
4. Small wrapper inside an official slot
5. Raw HTML replacement only after the above cannot satisfy the design

## Required Workflow

Before implementing any component that comes from a UI library:

1. Identify the target library component.
   Example: Figma drawer/modal panel maps to `el-drawer`, `a-drawer`, `n-drawer`, `Dialog`, or similar.

2. Map design regions to component extension points.
   Common mappings:
   - title/header -> `title`, `header`, `#title`, `#header`
   - footer/actions -> `footer`, `#footer`, `actions`
   - table cell/header/empty -> column slots, `#default`, `#header`, `#empty`, render callbacks
   - select/cascader option content -> option/default slots, label render hooks
   - prefix/suffix/icon -> prefix, suffix, icon, closeIcon, extra, addon slots or props

3. Check the component documentation, installed type definitions, examples, or source before choosing raw HTML.
   Do not guess slot names when they are uncertain. If documentation is not available locally, state the uncertainty and choose the closest documented API already known in the project.

4. Implement with the component API first.
   A style difference in the design is not a structural reason to abandon the slot. Use the slot and adjust typography, spacing, color, icons, and alignment with scoped CSS, theme tokens, CSS variables, `:deep()`, or a wrapper inside the slot.

5. Use raw HTML only when the library cannot express the required structure or behavior.
   Valid fallback reasons include:
   - The component has no relevant slot/prop/render hook.
   - The required layout changes the component hierarchy in a way the slot cannot represent.
   - The required interaction is unsupported by the component API.
   - Styling hooks and theme tokens cannot reach the required visual target.

6. When falling back to raw HTML, leave evidence.
   Add a short code comment or final note:
   `Slot-first fallback: checked <component> <slot/prop/API>; using custom HTML because <reason>.`

## Hard Rule

Never replace an existing library region only because the design has different visual styling.

If the design shows a drawer title with custom font, color, spacing, or icon treatment, use the drawer title/header slot first. Customize the title inside the slot or through the component's documented style hooks.

## Element Plus Drawer Example

Avoid this pattern:

```vue
<el-drawer v-model="visible" :show-close="false">
  <div class="drawer-head">
    <span class="drawer-title">Order details</span>
    <button class="drawer-close" @click="visible = false">x</button>
  </div>
  <section>...</section>
</el-drawer>
```

This recreates the drawer header and risks losing built-in behavior, accessibility, close affordances, theme integration, and future compatibility.

Prefer this pattern:

```vue
<el-drawer v-model="visible">
  <template #title>
    <div class="drawer-title">
      <span>Order details</span>
      <el-tag type="info">Draft</el-tag>
    </div>
  </template>

  <section>...</section>
</el-drawer>

<style scoped>
:deep(.el-drawer__header) {
  margin-bottom: 0;
  padding: 20px 24px 12px;
}

.drawer-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 18px;
  font-weight: 600;
}
</style>
```

If the design also customizes the close icon, check the component's close icon prop or close slot before hiding the built-in close button and creating a new one.

## Output Checklist

For each UI-library component you implement from a design source, verify:

- Did I identify the intended library component?
- Did I check for a slot, prop, render hook, theme token, or class hook for each title/header/footer/action area?
- Did I preserve the library component's built-in behavior instead of recreating it?
- Are visual differences handled with slot content or CSS rather than structural replacement?
- If raw HTML was used, did I document why the official API could not satisfy the design?
