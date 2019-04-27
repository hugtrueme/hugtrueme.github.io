---
layout: post
title:  "instance_eval、class_eval、eval 三者的差別"
date:   2018-11-10 12:21:00 +0800
excerpt: "eval 搭配 Binding 就像是漫威中的奇異博士，用他的雙手一畫，就開出了一道通往地球上某個角落的門，然後你可以在紐約市中心，彎腰內撿起西藏高山上某個角落的石頭。"
categories: Ruby
tags: Ruby
---

Ruby 的 eval 都具有「執行一段程式碼」的意味在，但你會說奇怪，程式碼直接寫在 rb 檔內不執行起來了嗎？接下來就讓我們來看看三者到底有何分別。

這三者裡面，instance_eval 跟 class_eval 會比較類似，而 eval 則比較獨特，本文先從前兩者開始解釋。


#### **instance_eval**

instance_eval 能讓你進入到 **instance** 的 Scope（作用域）內執行一段程式碼，可以存取到其 instance variable：

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
```
str = "String"

str.instance_eval do
  def new_method
    self.reverse
  end
end

str.new_method # "gnirtS"

str2 = "String"
str2.respond_to?(:new_method) # false
```

由上面這段範例可知，此新定義的 new_method 只會出現在 str 這個 instance of String 身上，而同為 instance of String 的 str2 不會有 new_method。

#### **class_eval**

class_eval（現又稱 module_eval）能讓你進入到 **Class** 的 Scope 內執行一段程式碼，同樣也可以用來即時定義 method。

```
class A
end

add_hello = 'def hello() "Hello!" end'

A.class_eval(add_hello)

A.new.hello # "Hello!"
```

與 instance_eval 的差別是，所有 class A 所 new 出來的 instance 都會有這個新的 method。

#### **eval**

eval 簡單得來說，它就是**可以把一個字串當作一段 ruby 程式碼來執行**。

```
b = "class B; end"

eval(b)

B.new # <B:0x007fd278917470>
```

就複雜的來說，eval 能夠搭配一個名為 Binding 的 class 來使用，此 class 可以神奇的**繫結**到某處的 Scope，該 Scope 內的 variable、method、value 與 self 都會被這個**繫結**一直維持住。

你可以想像，eval 搭配了 Binding 就像是漫威中的**奇異博士**，用他的雙手一畫，就開出了一道通往地球上某個角落的門，然後你可以在紐約市中心，彎腰內撿起西藏高山上某個角落的石頭。

```
class Demo
  def initialize(n)
    @secret = n
  end
  def get_binding
    return binding() # Kernel#binding
  end
end

new_york_city = Demo.new(99)
gate1 = new_york_city.get_binding

tokyo = Demo.new(-3)
gate2 = tokyo.get_binding

eval("@secret", gate1)   #=> 99
eval("@secret", gate2)   #=> -3
eval("@secret")          #=> nil
```

上面範例，你將 gate1 繫結到紐約，當你試圖在**現地** ```eval（"@secret", gate1)``` 的時候，你就會取得紐約的 @secret，值為 99。同理，當你```eval（"@secret", gate2)```時，你就可以觸碰到東京的 @secret，值為 -3。當你沒有用上任何 gate 卻想取用 @secret 時，就會得到 nil，因為現地並無 @secret 可用。
