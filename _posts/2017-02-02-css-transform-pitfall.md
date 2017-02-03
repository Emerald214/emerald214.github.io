---
layout: post
title: CSS Transform Pitfall
---

When working with [Vaadin TreeTable](https://vaadin.com/docs/-/part/framework/components/components-treetable.html), we run into [a serious scrolling issue](https://jira.magnolia-cms.com/browse/MGNLUI-4084). The tree scroll accidentally jumps to top every time we do some actions below the fold.

For example:  

- Inline-edit a row then click aside or Enter
- Right-click unselected row
- Click aside selected row to close context menu
- etc.

After investigating and testing around, we found that the issue comes from `CSS transform`. 

```css
.v-table-body {
  -webkit-backface-visibility: hidden;
  -webkit-transform: translateZ(0);
}
```

The snippet originates from [an improvement](https://jira.magnolia-cms.com/browse/MGNLUI-1328) that accelerates hardware rendering on iOS. Commenting or removing it out has fixed the problems.

We also notice that Firefox 49+ added support for `-webkit` prefixed versions of a number of CSS transform properties. That explains why the issue occurs severely on both Chrome and Firefox.