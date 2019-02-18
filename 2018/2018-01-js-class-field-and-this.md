在 Class field 正式成為 ECMAScript 一部分之後， this 的解讀可能會更簡單… 或是更複雜，視你的觀點而定
=====

Class field 目前是 stage 3，而且的確很有用，所以我估計在這兩年內很可能就會成為 ECMAScript 的一部分

在 ES5 的年代，this 關鍵字的解讀很單純，就是「這個 function 被呼叫時的 context」所以預設是由 call site 決定 this 會是什麼，如果需要提前決定，就用 bind 產生一個新的 function。

這個特性也導致像一些 API 設計，如 jQuery，會把 this 用來當作 callback function 的一個不用宣告的參數用。當年似乎是有避免 memory leak 的好處，但在 IE 終於不再獨霸市場的今天，這樣的設計已經被視為 anti-pattern。[註1]

由 call site 決定 this，實在是製造各種困擾，bind() 語法也不直觀(看到尾巴才知道會綁定哪一個 this)，常見的作法是在造 function的時候，幫 this 取個別名，然後在 callback 裡面改用別名取代 this，像這樣:

```js
{
 foo: function foo () {
  var that = this;
  setTimeout(function () {
    console.log('hello, ' + that.name); // 'hello, baz'
  }, 0)
 },
 name: 'baz'
}
```

於是在 ES2015 新增的 arrow function，就改成區塊內的 this 與造 arrow function 時相同。看起來很複雜，但有一個我想的簡單解讀法是「往外找到最近的 function 關鍵字就可以決定 this。」

而 Babel 對 arrow function 的轉換手段其實也就是上面提的幫 this 取別名，再去換掉 this 的做法。

同時在 ES2015 也帶來了 class 的語法，但其實還是符合 JavaScript 原本的 prototype 機制:

```js
class Foo {
  constructor () {
    this.name = 'baz';
  }
  foo () {
    console.log(this.name);
  }
}
// 差不多相當於
function Foo () {
  this.name = 'baz';
}
Foo.prototype.foo = function () {
  console.log(this.name);
}
// 兩個版本宣告的 Foo 對於下面用法的結果都一樣
var f = new Foo();
f.foo(); // 'baz'
f.foo.call({name: 'doh'}) // 'doh' (1)
```

從 (1) 可以看到，用 class 語法宣告的 method 遵從「funtion 預設由 call site 決定 this」的規則，所以改變 this 的手段，對 method 也有作用。[註2]

但，這個預設規則當年製造的問題又回頭造成困擾，像是不能寫 setTimetout(this.foo, 0)，你得要寫 setTimeout(() => this.foo, 0)。而後面這個語法每一次執行到又會重新造一次 function，所以一些 React 的教學推薦使用 class field 的語法，在宣告時就產生綁定好的 method－－儘管你會因此需要 Babel 幫你改寫，但 JSX 本來就需要改寫了…

就像一開始提到的 class field 還沒有正式成為 ECMAScript 的標準，所以還有可能會改變，不過先用我的認知介紹一下:
```js
class Foo () {
  constructor () {
    this.name = 'baz';
  }
  foo = () => console.log(this.name); //(2)
}
var foo = new Foo();
setTimeout(foo.foo, 0); // 'baz'
// 因為 class field 的效果是:
class Foo () {
  constructor () {
    this.foo = () => console.log(this.name); // (3)
    this.name = 'baz';
  }
}
```
因為 (2) 如 (3) 那樣被搬到 constructor 裡，所以搭配 arrow function，我們就有了一個語法可以簡單的造出預設綁好 this 的 method 們。

看起來很棒，對吧? 如果我覺得事情這麼美好，就不會有這篇了 (?)

Class field 的用法的確是符合使用上的直覺，但是對照從 ES5 以來的歷程，會發現，現在又多了一個 this 的解讀規則: 「這個 this 會綁定成 constructor 裡的 this」，不是「由 call site 決定」，也不是「往上找最近的 function」。

另外，當我們把 ES2015 定義的 method 用法和 class field 放在一起:
```js
class Foo () {
  constructor () {
    this.name = 'baz';
  }
  foo = () => {
    console.log(this.name);
  };
  bar () {
    console.log(this.name);
  }
}
```

… 你知道哪一個是哪一個嗎? 同時因為 class field 會遇到 ASI 的問題，有可能要強制加上分號，而 method 是不用加的 (雖然加上去好像不會錯…)。

而且因為 this 已經綁定好了，所以 class field 的 method 不能透過改變 this 的方式進行 code reuse－－儘管通常看起來像是在惡搞什麼。

寫到這裡，我也開始覺得，不如說是 JavaScript 一開始設計的機制太反直覺了 (「像 Python 那樣 class method 預設都綁好了不是很好嗎?」)，導致後來一直補洞，只能說我有點「老兵症候群」地覺得事情不像以前那麼單純美好了… (遠目) 至少不要在 production 裡用上還沒進標準的語法吧 Orz

---

註1: 但這個 pattern 還活著，例如 mocha 設定 timeout 的作法 還是用 this.timeout(10000)，所以有用到的話就不能用 arrow function 寫 test。

註2: 不過我為 Arrow Function 想的那個「往外找到最近的 function 關鍵字」解讀法就失效了，因為這裡沒有 function 關鍵字，哈哈哈…

Originally posted on [medium](https://medium.com/@redeyes2015/%E5%9C%A8-class-field-%E6%AD%A3%E5%BC%8F%E6%88%90%E7%82%BA-ecmascript-%E4%B8%80%E9%83%A8%E5%88%86%E4%B9%8B%E5%BE%8C-this-%E7%9A%84%E8%A7%A3%E8%AE%80%E5%8F%AF%E8%83%BD%E6%9C%83%E6%9B%B4%E7%B0%A1%E5%96%AE-%E6%88%96%E6%98%AF%E6%9B%B4%E8%A4%87%E9%9B%9C-%E8%A6%96%E4%BD%A0%E7%9A%84%E8%A7%80%E9%BB%9E%E8%80%8C%E5%AE%9A-abeadfb0b4b9)
