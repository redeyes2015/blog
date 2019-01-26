我與 git-svn
============

背景
----

其實在公司的專案用的 VCS 是 svn, 但是在進公司之前就常聽到 git 的大名, 但沒有什麼實際使用過得經驗, 於是在一開始新人訓練階段寫的一些小程式就順便練習用 git, 發現果然強力, 對比很久以前在大學時代曾經有過得 svn 經驗相比, git 容易使用的多, 至少不用每次 commit 都提心吊膽.

正式開始加入專案的開發之後, 就開始想走旁門左道, 希望可以繼續享受 git 的好處, 剛好在看文件的時候得知有 git-svn 這個工具, 於是就試著用 git-svn 作我的 svn client. 本文便是我這幾個月的一些使用心得.

git vs. svn policy
------------------

團隊中使用的 svn policy 是標準(?)的 trunk/branches/tags 配置, 所以使用 `git svn clone --stdlayout` 就可以很完美(?)的複製成 local git repo 了.

完美是指之後如果 svn 上開了新 branch, git svn fetch 或是 git svn rebase 之後就會自動發現有新的 branch, `git checkout new_branch` 之後就可以如同 master 對應 trunk 一樣簡單同步.

我有碰到的 svn branches 使用的例子都是之後團隊中的成員會各自把 branch 上各自的 commit merge 回 trunk 上. … er… 但是我其實完全看不出來那樣叫 merge… 所以通常就是在 master 上 `git cherry-pick {我的commit}`, 再 `git rebase -i trunk` 把全部的 commit 融合成一個, 就可以 `git dcommit` 了! 輕鬆簡單. (雖然我還是看不出來這樣為什麼叫 merge ….)

喔, `git cherry-pick {我的commit}` 可以配合一個 shell script 來作, … 但是有點複雜改天再補 XD

初始, 設定
----------

* 從頭造一個:

```sh
git svn init *target_folder*
```

但是你都要從頭來了, 為什麼要用 git svn 呢?

* 複製已經有的 svn repo, 如同上面講的, 下面這行可以把大部分的事情做完:

```sh
git svn clone --stdlayout {svn_repo} *target_folder*
```

不過如果 svn 原本就有很多 revision 且/或 檔案量很大, 可能要等個幾十分鐘, 萬一要複製的對象顯然是這樣的話… 耐心等吧 orz

* 空資料夾: git-svn 最麻煩的就是你無法把空資料夾加到 git 的管制中, `git svn fetch` 下來的時候會對 svn 上的空資料夾特別作紀錄 (但好像不是很完美… 有幾次重新 clone 下來之後就踩到洞的印象)

    * svn.rmdir: 另一個方向是, 你想要刪掉一整個資料夾的時候, 即使你用 `git rm -r *folder*`, 這個時候 `git svn dcommit` 結束後, 那個空資料夾還是在 svn 裡, 畢竟 git 自己是沒在管空資料夾的, 這時候 除了 `git dcommit --rmdir` 明確指定要把空資料夾刪除以外, 還可以用　`git config --local svn.rmdir true` 讓之後的 dcommit, 都自動有這個效果.

日常使用
--------

* `git svn rebase` : 對應 `svn update`. 如果只想看看 svn 上有沒有新的 revision 可以用 `git svn fetch` 如同最上面說的, 如果 `git svn clone` 的時候有用 `--stdlayout` 選項, 在 master 上作 `git svn rebase` 會自動 rebase 到 svn 的 trunk. 如果是用 `git commit -b local_git_branch remote_svn_branch` 之後在 local_git_branch 上作 `git svn rebase` 會自動跟隨 `remote_svn_branch`

* dcommit: 對應 `svn commit`. 基本上我是當作 svn 沒有 merge 的觀念 (但好像其實有? 也許哪天會去查這件事吧), 所以在 dcommit 之前我一定是整理成在 git 裡面可以 ‘fast-forward’ 的狀況再敲 `git svn dcommit`

* 除此之外平常就當作 local git repo 來操作, 出於尊重團隊原本就有的原則, trunk 就是 trunk 會 commit 上去的, 就會被整理成單一一個 history. 稍微避免用 branch / merge, 免得要 commit 時太累.

想開 Branch 的時候
------------------

但有的時候想獨立開發/修改什麼功能, 又不太可能在 5 個 commit 之內做完, 這時候 “存一堆 commit” 在 master 上, 是一件很辛苦的事. 在摸索幾次之後我的行為大概會像這樣:
1. `git checkout -b my_exp`: 先從 master 開新的 branch 出去
2. 作一個小段落就 commit, 或是不時用 `git commit -p` 把一堆修改各自歸到有意義的 commit 上
3. 到一個段落但是還沒做完, 不想 git svn dcommit 的時候就 用 rebase merge 一下 trunk:

```sh
git checkout master
git svn rebase
git checkout my_exp
git rebase my_exp
```

其實這一步只是我個人用來分心的行為… 一般來說沒什麼必要, 在 master 上 git svn rebase 瞄一下最近的 commits 就夠了.

4. 終於作的差不多, 該推到 trunk 了! 這時候就要拿出利器 `git rebase -i trunk` 把開發的 commit 整理的漂亮一點, 想要全部變成一個 commit 其實也無所謂, 不過個人潔癖堅持(?) 還是會留下比較好理解的 commit history.

* `git rebase -i` 理論上只要 core.editor 有設定對, 照著提示應該知道怎麼使用了, 只能說想整理 commit history 的時候, 用過的都說讚!
* 如果很擔心 rebase 把歷史搞亂修回不去, 可以先開一個暫時的 branch (i.e. `git branch rebase_temp`, 整理完用 `git diff my_exp rebase_temp` 確認結果跟想要的一樣, 就安心可以把暫存 branch 砍掉: `git branch -D rebase_temp`)

5. 整理完, 就可以推上 trunk 了:

```sh
git checkout master
git rebase my_exp # 應該要可以無誤的 fast-foward 上去
git svn dcommit # 新 feature 上線!
```

特異功能
--------

svn info : 在專案中造 firmware 的流程中有用到 svn info 來把 svn 的版號資訊塞進去, 於是使用 git-svn 自然無法直接支援, 但是很幸運的是, git svn info 可以得到幾乎一樣的輸出, 於是塞一個檔名叫 svn 的 script 到 PATH 中, 內容大概長這樣:

```sh
#!/bin/sh
git svn $@
```

就可以囉.

結語
----

個人的感想是, 用 git-svn 當作 svn client 最大的好處大概就是:

1. svn server 因故連不上的時候, 你還是可以繼續開發 (雖然都只在公司內部開發, 但我真的遇過這種狀況!)
2. 在 svn commit / git svn dcommit 之前, 有更多機會把 commit 整理的更漂亮一點

git 相對於 svn 很多方面都強大許多, 為了配合 svn , 最後我使用的方法就像上面說的那樣, 也許還有很多可以修正改進的地方, 總之謝謝觀賞 XD

Originally published on [tumblr](http://rein-notes.tumblr.com/post/31925705401/git-svn-and-me)
Date: SEPTEMBER 21, 2012 (12:09 AM)
Tags: #GIT, #GIT-SVN
