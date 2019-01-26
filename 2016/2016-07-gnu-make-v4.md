GNU make 新知三則
=================

擁抱了 Ubuntu 16.04 LTS Xenial Xerus 的我忽然發現 gcc 跳到第五版了！ GNU make 也從 Ubuntu 14.04 的 3.81 進到 4.2.1 版，在這裡分享一下一些改變…

make -O
-------

`make --jobs=n` 同時執行任務雖然很好用，但是輸出交錯在一起總是讓 debug 變得很麻煩，在 make v4.0 之後，新增了選項 --output-sync (-O)，可以把個別 recipe 的輸出集中在一起，這樣 debug 的時候就更簡單囉！詳細用法請參考使用說明。

混用 impicit 與 normal rule 的狀況
-----------------------------------

make v3.82 之後對於同一個 rule 中包含 “implicit” 和 “normal” 的狀況處理方式不同。也就是說像這樣的寫法 :

```Makefile
config %config:
    echo $@
```

會讓 make v4.2.1 直接報錯誤訊息:

> \*\*\* mixed implicit and normal rules. Stop.

解法… 就是同一個 recipe 多寫幾次這樣:

```Makefile
config:
    echo $@
%config:
    echo $@
```

這個問題還滿怪異的，因為 `single single%` 這樣的寫法目前還不會出錯，可以在 GNU make 的 source code 直接搜尋 “mixed implicit” 會發現只有一個地方是當作 warning 處理，其他都還是 fatal… 在 GNU make 的 bug report 裡面可以看到官方認定這是未註明在文件上的行為，因此未來將禁止這樣的寫法…

wildcard 回傳不再是排序好的
---------------------------

make v4.0 之後… wildcard的回傳不再是排序好的，也就是以下寫法都可能會受到影響 (如果順序有影響的話):

```Makefile
-include */module.mk
-include $(wildcard *.d)
解法: 用 $(sort ...) 或是直接展開寫死…
```

Ending
------

果真有在改變的軟體就是不停的在挖坑增加工作機會阿… v3.82 的 NEWS 洋洋灑灑列了六個不相容通知… 恐怕未來很可能會再踩到這些問題了吧 (遠目)

Originally publiched on [tumblr](http://rein-notes.tumblr.com/post/147446394293/gnu-make-%E6%96%B0%E7%9F%A5%E4%B8%89%E5%89%87-%E6%93%81%E6%8A%B1%E4%BA%86-ubuntu-1604-lts-xenial-xerus)
Date: JULY 15, 2016 (9:59 PM)
