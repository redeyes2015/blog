探索網頁 UAT 測試框架
=====================

CasperJS
=======

## 文件 ##

* 極佳，[Quick Start] 有用，API 文件也很完整
[Quick Start]: http://docs.casperjs.org/en/latest/quickstart.html

## API ##

* 測試需要的基本上都有 (`waitForSelector`... etc)
* fill 直接填整張表很方便
* API 的缺點是一些等待或是非同步的行為需要另外寫一段 `casper.then(...)`
* 「受測網頁」可以給 file path 或是指到 web server (like `http://localhost:9000/`)

## 安裝 ##

* 雖然可以用 npm 安裝，但其實是獨立的執行檔，不是跑在 node.js 上
* 有潔癖就下載後放在 PATH 內就對了
* BTW ... 沒辦法直接用 ES6 功能，不確定什麼時候才能用

## 執行 ##

    casperjs test test/casper/hello-test.js

一定要加參數 `test`

## 總之 ##

就是一個程式化操作 headless WebKit 的工具，順便可以作頁面測試，但這兩件事都作的不錯

Intern
======

## 文件 ##

* 只有一張極長的網頁，儘管有不少資訊，但與 Intern 提供的功能比起來還是太少
* API 的部份有 [leadfoot] 的文件
* 同時有一些跟不上改版的跡象，例如唯一可以當作 Quick Start 的 [tutorial]，就被註明與 v3 有出入
[leadfoot]: https://theintern.github.io/leadfoot/module-leadfoot_Command.html
[tutorial]: https://github.com/theintern/intern-tutorial

## API ##

* 整體感覺很像是公司內部 (dojo?) 的工具釋放出來之後沒有加上足夠的文件，也只預期在很限定的狀況下使用
* 操作網頁的 API 用 fluent style 來寫，沒有 `waitForSelector` 而是 `findByCssSelector` 可以 `setFindTimeout`
* 「受測網頁」一定要給檔案系統路徑，`intern-runner`會自行啟動 web server。
    * 但很大的問題是 `intern-runner`內部用的是 dojo 寫的 loader 和我們開發用網頁寫的 requirejs 湊不起來，執行的時候路徑又很難理解是相對路徑的參考點在哪邊
    * test case 裡`require(...)` 的路徑也有一樣的問題，leadfoot 裡提供的 pollUntil 怎樣也 `require()` 不到...
* `intern-runner` 之類的指令要先 `npm install -g intern` 才不用打一長串路徑，但是在文件裡完全沒看到

## 執行 ##

* 一開始用 selenium 給的 [standalone-firefox] 配 'NullTunnel' 不會動，要抓 Firefox ESR，塞到 PATH 最前面配 'SeleniumTunnel' 才會動...
* 我們的網頁是用 requirejs，各種無法使用，中間試過指定 loader，會出現 pre-execution error；basePath 和 loaderOption.baseUrl 很難搞
* 最後的作法是...
    1. 把 fixture 也編進 dist 裡
    2. 把 intern.js (config) 和測試都搬到 dist/ 底下
    3. `intern-runner config=intern.js functionalSuites=$PWD/intern-functional.js` functionalSuites 參數直接寫絕對路徑否則就是要自己試加幾次 `../` 才會動...

[standalone-firefox]: https://hub.docker.com/r/selenium/standalone-firefox/

## 總之 ##

是一個號稱什麼都可以作的測試框架，但如果環境跟預設的不同就會遇到各種讓人說出 [WTF] 的問題

[WTF]: http://www.osnews.com/story/19266/WTFs_m

---

* PhantomJS: mother of headless browser; raw API; using WebKit under the hood

* CasperJS: wrap up PhantomJS API

* SlimeJS: CasperJS counterpart but using Gecko (Firefox)

* Selenium: mother of everything manipulating the browser; raw API

* WebDriver / Selenium 2 : standardized wire protocol [w3c](https://w3c.github.io/webdriver/webdriver-spec.html)

* WebDriver.io : provide abstraction over WebDriver protocol

* PhantomCSS: provide pixel-diff-based test on PhantomJS; incompatible with PhantomJS 2.0+

* PhantomFlow: decision-tree-based test case; kind of a proof of concept to me

* BackstopJS: make responsive design test easier (pixel-diff-based); using CasperJS and Resemblejs ; pretty complete to me

* dalekJS: basically abandoned, "v0.0.9"

* CodeceptJS: in early alpha; self-test is not completed

* Intern: does-everything-about-test -- pretty crap, unless you are super into dojo structure; or to be fair, the document is not complete, considering its super rich feature set.

