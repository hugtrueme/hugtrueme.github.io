---
layout: post
title:  "instance_eval、class_eval、eval 三者的差別"
date:   2018-11-10 12:21:00 +0800
categories: Ruby
tags: Ruby
---

Ruby 的 eval 都具有「執行一段程式碼」的意味在，但你會說奇怪，程式碼直接寫在 rb 檔內不執行起來了嗎？接下來就讓我們來看看三者到底有何分別。

這三者裡面，instance_eval 跟 class_eval 會比較類似，而 eval 則比較獨特，本文先從前兩者開始解釋。


#### instance_eval

instace_eval 能讓你進入到 instance 的 Scope（作用域）內執行一段程式碼，可以存取到其 instance variable：

```
class Klass
  def initialize
    @secret = 99
  end
end
k = Klass.new
k.instance_eval { @secret }   #=> 99
```

也可以即時定義 instance method

#### class_eval

class_eval 能讓你進入到 Class 的 Scope 內執行一段程式碼，可以存取到其 class variable：