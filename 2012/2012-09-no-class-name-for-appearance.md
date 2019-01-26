不要使用對應外觀的class name
=============================

前幾天看了這篇: “[Naming and using id’s and classes properly][1]”, 對照目前在作的工作有些感觸。

[1]: https://2002-2012.mattwilcox.net/archive/entry/id/1058/

怎麼說呢? 對於很常用的 css 屬性, 寫一個 class 讓自己之後撰寫也省力一點, 聽起來是個很合理的作法, 像是這樣:

```css
.hidden {
  display: none;
}
```

之後想要預設隱藏的 tag, 加上 hidden 的 class 就行了..!

但是, 問題來了, 如果有另一個規則是:

```css
.someicon {
  display: inline-block;
  width: 10px;
  height: 10px;
  background: url(’/static/some_cute_icon.png’) no-repeat;
}
```

如果有一個希望預設隱藏的圖示, 可能就會寫出:

```html
<span class=‘someicon hidden’ />
```

所以這個 span 的 display 屬性會是什麼呢? “很難說”

因為前面提到的兩個 rule 他們的權重 (Specificity) 是一樣的: “0,0,1,0” (ref), 於是那一個會生效? 視出現的順序和使用的瀏覽器而定.

但是身為一個有懶惰美德的 programmer, 我們不該每次遇到一樣的屬性就 copy-paste 一遍. 幸好, 我們有 LESS 和 Sass, 這兩種產生 css file 的工具都支援 mixin 的概念, 使用 mixin 應該可以帶給我們更好的末來!?


Originally published on [tumblr](http://rein-notes.tumblr.com/post/31107654016/%E4%B8%8D%E8%A6%81%E4%BD%BF%E7%94%A8%E5%B0%8D%E6%87%89%E5%A4%96%E8%A7%80%E7%9A%84class-name)
Date: SEPTEMBER 8, 2012 (1:33 PM)
Tags: #CSS, #HTML
