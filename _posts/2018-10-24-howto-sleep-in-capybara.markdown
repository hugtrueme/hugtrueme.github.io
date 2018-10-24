---
layout: post
title:  "如何在 Capybara 內測試 Ajax 的網頁內容"
date:   2018-10-24 21:00:00 +0800
categories: Rails Rspec
tags: Rails Capybara Rspec
---

知名 Rails 前端測試工具 Capybara 在 2.0 版的時後把 `#wait_until` 這個 sleep 的功能移除了，有些新手朋友可能在 Ajax 的網頁內容測試上遇到困難，還是會在測試裡面用上 ruby 的 `sleep`，但請不要這樣做，因為您不管 sleep 多久，都有可能仍舊是不準（不夠）的，而且會拖慢全體測試跑完的時間。


舉例來說，筆者的新手同事曾在專案的測試中寫出這樣的測試：


```
# add 11 items to the memo list

within('.memo-input') do
  11.times do |index|
    sleep 1
    fill_in "description", with: "test#{index}"
    sleep 1
    click_button('Add')
  end
end
```

同事這樣做的原因是，程式規劃當 Add 鈕被按下後的那一瞬間會變成 disabled = true，所以不能按（按了沒用），要過一段時間才會恢復成 disabled = false 的狀態，所以如果不 sleep 的話，會因為 Capybara 的測試動作跑太快所以按不到。

這個例子為了要完成測試，也至少花費了 11 x 2 = 22 秒（若善用正確方法，不用 2 秒就跑完了）每跑一次測試就浪費了 22-2 = 20 秒，更別說這 20秒會長期留在專案裡... 如果同事們大家都這樣 sleep 的話， 後果不敢想像。

更別說在發 PR 的那幾天，為了要修被自己弄壞的（別人的）測試，你會需要跑幾次的完整測試...


其實 Capybara 目前還是有提供[測試 Ajax 內容的方法]，前述的問題可以改用如下寫法：

```
# add 11 items to the memo list

within('.memo-input') do
  11.times do |index|
    fill_in "memo_description", with: "test#{index}"
    click_button('Add')

    # 等待一個名為 Add 並且 disabled: false 的按鈕出現
    expect(page).to have_button('Add', disabled: false)
  end
end
```

其原理就是利用 `expect(page)` 讓 Capybara 等待一個**名為 Add 並且 disabled: false 的按鈕**出現後，才進行下一步動作，至於要等待多久才 timeout 呢？在專案內的 **capybara.rb** 內設定即可：

```
Capybara.default_max_wait_time = 30
```

上面的秒數設定太長的話，你在 local 測試等不到東西的時候會容易睡著，所以可以先設定短一點（例如 5 秒）


[測試 Ajax 內容的方法]:https://github.com/teamcapybara/capybara#asynchronous-javascript-ajax-and-friends