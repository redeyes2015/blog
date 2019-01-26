[踩洞] display: table-XXX; position: relative;
==============================================

Example:

```html
<div id=‘someCell’ >
  <span id=someButton />
</div>
```

想直接在 `div#someCell` 用 `position: relative`, 然後 `span#someButton` 用 `position: absolute`, 結果底下的 `#somButton` 一整個跳過 `#someCell` 繼續往上找其他的 `position: relative`…

猜想是 `display: table-cell` 跑來壞事, 找了一會, 發現真是這樣:

From http://www.w3.org/TR/CSS21/visuren.html#propdef-position:

> The effect of 'position:relative’ on table-row-group, table-header-group, table-footer-group, table-row, table-column-group, table-column, table-cell, and table-caption elements is undefined.

解法? 在 `#someCell` 和 `#someButton` 中間插一個 `<div>` 或是 `<span>` , 然後在中間這個 element 設 `position: relative` 就成了 :Q

Originally published on [tumblr](http://rein-notes.tumblr.com/post/31107609228/%E8%B8%A9%E6%B4%9E-display-table-xxx-position-relative)
Date: SEPTEMBER 8, 2012 (1:32 PM)
Tags: #CSS, #HTML, #踩洞
