---
layout: post
title:  "怎麼做出一個 Web Widget 服務（一）解決前端污染問題"
date:   2020-01-18 13:00:03 +0800
categories: Rails
tags: Rails Widget
---

[Web Widget]，是一種能嵌在網頁裡的一種**小插件**，例如現在流行的**第三方客服系統**，可以讓一些純前端（無後台）的網站能夠在網頁的 script 區塊嵌入一小段 Javascript code 後，就可以擁有一個即時客服聊天的功能。還有像是第三方網頁廣告服務、網頁內嵌入 Google Map 服務等... 使用的就是這類技術。

本篇向大家介紹製作 Web Widget 的技術架構、會需要突破的技術瓶頸（也就是說，身為工程師的您，會需要研究哪些東西）以及這些技術為何會出現在 Web Widget 中。


首先來看看 Web Widget 最簡單的形式，假設這是一個由 widget.com 提供的第三方客服服務：
```javascript
<script src="http://widget.com/script.js" type="text/javascript"></script>
```

我們立即可以想到一些隱含的技術問題，客戶的網頁裡面的 Javascript 程式碼會不會跟第三方服務商 widget.com 提供的 script.js 產生衝突？例如：

* 使用了同樣名稱的變數。
* 使用的 JQuery、Vue、或是 Bootstrap 版本不同。
* CSS style 相互污染。

要如何避免這些問題，最基本的方式是利用 Javascript 的語言特性 **Anonymous Fuction**（匿名函式）把己方的程式碼隔離出來，如下範例：

```javascript
var foo = "Hello World!";
document.write("<p>在我們的 Anonymous Fuction 外面，foo 是 '" + foo + '".</p>');

(function() {

    // 這段程式碼會被封裝在匿名函式內。

    var foo = "Goodbye World!";
    document.write("<p>在我們的 Anonymous Fuction 裡面，foo 是 '" + foo + '".</p>');

})(); // 定義完匿名函式後，直接呼叫之。

document.write("<p>在我們的 Anonymous 之後，foot 是 '" + foo + '".</p>');
```

上面可以解決變數命名污染的問題，但如果已方使用的 jQuery 版本與客戶端不同呢？一個老式的解法如下：

```javascript
# 假設己方使用 1.4.2 版本 jQuery

(function() {

// Localize jQuery variable
// 將 jQuery 變成區域變數
var jQuery;

/******** 瀏覽器如果沒有載入過 jQuery 1.4.2 則載入之 *********/
if (window.jQuery === undefined || window.jQuery.fn.jquery !== '1.4.2') {

    // 自行建立一個 <script></script> 把 src 指向 jQuery 的 CDN 網址
    var script_tag = document.createElement('script');
    script_tag.setAttribute("type","text/javascript");
    script_tag.setAttribute("src",
        "http://ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.min.js");

    // 我們待會要將 script tag 加進 HTML 內，一但加進去後，瀏覽將會幫我們下載 jQuery
    // 回來，這會需要一段時間（非同步異步的行為）我們希望一但一切載入完畢後，瀏覽器能通
    // 知我們，因為我們還需要做一些事，所以我們攔截要 onload 這個事件。
    if (script_tag.readyState) {
      // 舊版 IE 瀏覽器
      script_tag.onreadystatechange = function () {
          if (this.readyState == 'complete' || this.readyState == 'loaded') {
              scriptLoadHandler();
          }
      };
    } else {
      // 其他的瀏覽器
      script_tag.onload = scriptLoadHandler;
    }

    // 尋找 HTML 內的 <head></head> 標籤，不存在就找 documentElement，
    // 然後將 script tag 加在它底下。
    (document.getElementsByTagName("head")[0] ||
     document.documentElement).appendChild(script_tag);
} else {
    // 客戶端的 jQuery 版本剛好就是我們要的，則什麼事都不用做。
    jQuery = window.jQuery;
    main();
}

/******** 一但 jQuery 載入完畢後，此 function 會被呼叫 ******/
function scriptLoadHandler() {
    // 由於我們載入了 1.4.2 版本的 jQuery，這會把原本客戶使用的版本給覆蓋掉。
    // 所以此處我們使用一個 jQuery 提供的 noConflict() 函式，它可以將被我們覆蓋
    // 掉的 jQuery 版本還回給 window.jQuery，並且將我們後來下載的 1.4.2 版本給
    // return 出來，我們將之存放於位於我們匿名函式內的區域變數 jQuery，之後就可以
    // 使用它。

    jQuery = window.jQuery.noConflict(true);

    // 呼叫我們要做的正事
    main();
}

/******** 我們的主功能 ********/
function main() {
    jQuery(document).ready(function($) {
        // 我們可以開始一如往常地使用 jQuery 了
        // 這邊的 $ 符號即是 1.4.2 版本的 jQuery。
    });
}

})(); // We call our anonymous function immediately
```

但，Vue 與 Bootstrap 的版本不同的話，該怎麼辦？它們沒有如同 jQuery.noConflict 這種功能的 function（如果您知道，歡迎寫信告訴我）況且，每個框架都要這樣處理，也會很麻煩。所以，新式的做法是使用 [Webpack] 來將己方的 Javascript 程式碼**編譯**過，再提供出去。

jQuery、Vue、Bootstrap 都有提供 Webpack 專用的 npm install package，將它們 import 後就可以一起編譯進來，由於 Webpack 的編譯行為之一就是會重新命名所有的 Javascript 變數名稱，這會讓已方使用的 jQuery 與 Vue 等套件與客端網頁獨立出來，達到隔離的效果。

到此，已經解決了這兩種問題：

* 使用了同樣名稱的變數。
* 使用的 JQuery、Vue、或是 Bootstrap 版本不同。

但是，CSS style 污染就麻煩了。我們假設您了使用任一版本的 Bootstrap，並在已方程式碼內用 Vue 做了一個 component，裡面的 HMTL 使用了 Bootstrap 的 card style。不巧，客戶網頁也是用 Bootstrap，而且他也用了 card style，並且還直接地修改了該 style 的外觀。這時您會發現，要嘛，**不是你的 card style 把客戶的蓋掉了，就是客戶蓋掉了你的**。因為，Webpack 不會自動地幫我們把 Bootstrap CSS style 重新命名。

但這裡要特別提到的是，您還是可以使用 [CSS Module] 來讓已方的 css 命名不會與客端發生衝突（
相關的技術手法，可以參考  阮一峰先生的 [CSS Modules 用法教程]）但這可以防止自己去弄髒別人，卻不能防止別人弄髒自己。以 Bootstrap card style 來說，由於它並不是我們自己寫的 style，故不在 CSS Module 的掌控範圍內。


[Web Widget]:https://zh.wikipedia.org/wiki/Web_Widget
[Webpack]:https://webpack.js.org/
[CSS Module]:https://github.com/css-modules/css-modules
[CSS Modules 用法教程]:http://www.ruanyifeng.com/blog/2016/06/css_modules.html