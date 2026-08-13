---
name: ui-slot-first
description: Enforce slot-first UI implementation when generating component code from design files via MCP (Figma, MasterGo, Lanhu, Jijing) or design-to-code workflows. Use the component library's provided slots, props, and APIs (Element Plus #title/#footer, Ant Design Vue, Naive UI, Vuetify, PrimeVue slots) instead of rebuilding component internals with custom HTML. Trigger when writing Vue/React UI code from design specs, Figma MCP output, or whenever the agent tends to reconstruct component sub-structures (headers, footers, toolbars) that the library already exposes as slots.
---

# UI 实现规则:Slot 优先原则

> 当通过 MCP(Figma / MasterGo / 蓝湖 / 即时设计)读取 UI 设计图生成代码时,强制执行本规则。

## 核心原则

使用组件库(Element Plus / Ant Design Vue / Naive UI / Vuetify / PrimeVue 等)实现 UI 时,**必须优先采用组件库提供的 slot(插槽)、prop、API 满足设计需求**,而非用原生 HTML 元素重新实现组件的内部结构。

一句话:**先吃组件的 slot,吃不下再自己写 HTML。**

## 决策流程(严格按顺序)

| 步骤 | 动作 | 产出 |
|------|------|------|
| 1. 识别 | 对照设计稿,识别组件语义区域(标题→title slot、工具栏→header/toolbar slot、底部操作→footer slot、空状态→empty slot 等) | 语义映射表 |
| 2. 查文档 | 确认该组件提供了哪些具名插槽、prop、样式类名钩子 | 可用 slot 清单 |
| 3. 匹配 | 判断 slot 能否满足?能→用 slot;部分能→slot+局部自定义;完全不能→自行实现 | 实现方案 |
| 4. 样式穿透 | 样式差异(字号/颜色/间距/图标)通过 `:deep()` / slot 内包裹元素类名注入,**而非替换结构** | scoped style |

**关键认知**:设计稿的样式差异 ≠ 结构差异。样式差异一律用 CSS 穿透解决,不要因为"标题字号不同"就放弃 `#title` slot 另起炉灶。

## 判断"slot 确实无法满足"的合理场景(方可自行实现)

- 设计稿结构与组件 slot 提供的结构层级差异巨大(如 Drawer header 需嵌入 Tab 切换)
- 需要组件完全不支持的行为(交互/布局)
- slot 提供的样式钩子无法覆盖关键视觉需求,且 `:deep()` 无法穿透

> 自行实现时,必须在代码注释或提交说明中标注:"已确认无可用 slot,理由:XXX"。

## 示例对比(el-drawer 的 title)

### ❌ 错误:弃用 slot,自行实现 head

```vue
<el-drawer v-model="visible" :show-close="false">
  <div class="custom-header">
    <span class="title">自定义标题</span>
    <i class="icon-close" @click="visible = false" />
  </div>
  <div class="content">...</div>
</el-drawer>
```

**问题**:丢失 el-drawer 内置的 ESC 关闭、拖拽、ARIA、主题样式联动;与组件库主题脱节;后续组件升级需手动跟进。

### ✅ 正确:用 title slot + 样式穿透

```vue
<el-drawer v-model="visible">
  <template #title>
    <span class="custom-title">自定义标题</span>
  </template>
  <div class="content">...</div>
</el-drawer>

<style scoped>
:deep(.el-drawer__title) {
  font-size: 18px;
  font-weight: 600;
}
.custom-title {
  color: var(--el-color-primary);
}
</style>
```

**收益**:保留组件内置行为;样式可定制;升级无痛。

## 各组件库常见 slot 速查

| 组件库 | Drawer | Dialog | Table | Form | Cascader/Select |
|--------|--------|--------|-------|------|-----------------|
| Element Plus | `#title` `#footer` `#default` | `#header` `#footer` `#title` | `#default` `#header` `#append` `#empty` | `#default` `label` prop | `#default` `#prefix` `#empty` |
| Ant Design Vue | `title` `footer` | `title` `footer` `closeIcon` | `title` `footer` `summary` `expandedRowRender` | — | `default`(labelInValue) |
| Naive UI | `header` `footer` | `header` `footer` `action` `icon` | (通过 render 函数) | — | `render-label` `render-tag` |

> 具体以组件库官方文档为准。实现前**必须**查文档确认 slot 名。

## 自检清单(生成代码后逐项确认)

- [ ] 是否查阅了组件的 slot/prop 文档?
- [ ] 每个"看起来是组件一部分"的区域(标题/底部/工具栏/触发器),是否优先用了对应 slot?
- [ ] 样式差异是否用 `:deep()` 或 slot 内元素类名解决,而非替换结构?
- [ ] 若自行实现了结构,是否在注释中说明了 slot 无法满足的理由?
