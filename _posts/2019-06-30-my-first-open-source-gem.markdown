---
layout: post
title:  "這是我的第一個開源 Gem"
date:   2019-10-16 23:34:00 +0800
categories: Rails
tags: Rails Gem
---

還記得剛學 Ruby on Rails 的時候，看到老師在台上示範 `aasm do ... end` 這樣的程式碼。因為我以前的背景是編譯式語言如：C/C++/Objective。在它們的世界裡面，寫在 function 內的程式碼才能執行，其餘的都是定義（Definition）、宣告（Declaration）或巨集（Marco）。

所以我很好奇舉手發問，為什麼 ruby 可以在 Class 的 Definition 裡面直接執行程式碼？

老師說，這叫做 Metaprograming，是 ruby 程式語言的是一種寫法。然後過陣子又聽同事輕描淡寫的說，他在當兵無聊時，把【Metaprograming Ruby】這本書唸完了。讓我有一種，這麼神秘的東西怎麼大家都會，唯獨我不會的感覺...

一年多後，在每天兩小時的捷運通勤時間中，我也把這本書念完了。當時的心得是，平常的專案中好像沒機會用到
Metaprograming，但是這本書讓我對 Ruby 的認識，有了另一個新的角度。

慢慢地，有時後在 debug 的時候，我會嘗試運用書裡面提到的一些觀念去「窺探」當下 ruby 程式的狀況，也漸漸發現有些 issue 可以用 Metaprograming 來優雅地解決，節省日後「工人智慧」的工時消耗。直到近期在一個工作任務中，我遇上了一個狀況，發現自己在 copy/paste 大塊的程式碼從一處到另一處，而且過幾個禮拜後，我可能還要這樣做「好幾次」。

所以就動起了把這個需求用 Metaprograming 寫成 gem 的念頭，而且還可以矯情的模仿那些受歡迎的 open source gem 的用法... 在完成工作任務後，利用下班時間把程式精煉了一下，噹啦，終於達成了一個人生成就，擁有了一個自己的 open source gem 了！

Here is my first own open source gem on Github：[ModelTranscribers]

[ModelTranscribers]:https://github.com/hugtrueme/model_transcribers