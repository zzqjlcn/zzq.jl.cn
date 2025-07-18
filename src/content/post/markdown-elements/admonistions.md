---
title: "Markdown提示框使用指南"
description: "使用Markdown提示框功能，展示其突出显示重要信息、技巧、警告等关键内容类型的可视化呈现方式"
publishDate: "2024-08-25"
seriesId: "markdown-elements"
orderInSeries: 2
tags: ["markdown"]
draft: true
---

## 什么是提示框

提示框（也称为"侧边说明"）非常适合用于提供与内容相关的辅助性和/或补充性信息。

## 使用方法

在Astro Citrus中使用提示框，只需用三重冒号`:::`包裹您的Markdown内容。第一对冒号后需注明提示框类型。

例如：

```md
:::note
突出显示即使用户快速浏览也应考虑的信息。
:::
```

效果展示：

:::note
突出显示即使用户快速浏览也应考虑的信息。
:::

## 提示框类型

当前支持以下提示框类型：

- `note` 普通说明
- `tip` 实用技巧
- `important` 重要提示
- `warning` 警告信息
- `caution` 注意事项

### 普通说明

```md
:::note
突出显示即使用户快速浏览也应考虑的信息。
:::
```

:::note
突出显示即使用户快速浏览也应考虑的信息。
:::

### 实用技巧

```md
:::tip
帮助用户更高效完成操作的实用建议。
:::
```

:::tip
帮助用户更高效完成操作的实用建议。
:::

### 重要提示

```md
:::important
用户成功操作所必需的关键信息。
:::
```

:::important
用户成功操作所必需的关键信息。
:::

### 警告信息

```md
:::warning
因潜在风险需要用户立即关注的关键内容。
:::
```

:::warning
因潜在风险需要用户立即关注的关键内容。
:::

### 注意事项

```md
:::caution
操作可能带来的负面后果说明。
:::
```

:::caution
操作可能带来的负面后果说明。
:::

## 自定义标题

您可以通过以下语法自定义提示框标题：

```md
:::note[自定义标题]
这是一个带有自定义标题的说明框。
:::
```

效果展示：

:::note[自定义标题]
这是一个带有自定义标题的说明框。
:::

注：所有提示框样式均采用Astro Citrus主题预设的视觉设计，确保与文档整体风格协调统一。不同类型的提示框使用差异化的颜色和图标系统，便于用户快速识别信息重要性级别。