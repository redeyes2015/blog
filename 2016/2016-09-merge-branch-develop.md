Merge branch 'develop'
======================

標題來自 [CodeTengu Weekly#73][Code-Tengu-73]。一開始還看不懂梗是什麼，直到看到
[@WanCW][twitter-WanCW] 說他抱怨這件事很多次之後，才獲得[線索][driven-crazy-by-merge-into-dev]。

原來是在說，在 git 裡面看到 'Merge origin/develop into develop' 這件事。

因為有同事最近好像也踩到類似的狀況，想不自量力的給一點解釋和幫助。

[Code-Tengu-73]: http://weekly.codetengu.com/issues/73
[driven-crazy-by-merge-into-dev]: https://twitter.com/WanCW/status/347899187867840513
[twitter-WanCW]: https://twitter.com/WanCW

為什麼這會讓人想要崩潰?
------------------

因為這是一個有人沒有遵守 project 開發流程的訊號。

來解釋下這一個訊息: 'Merge origin/develop into develop':

1. 'Merge {foreign-branch} into {base-branch}' 是 git 預設的 merge commit message 的標題
2. 所以這裡的 foreign-branch 是 'origin/develop'；base-branch 是 'develop'
3. 'origin' 是 `git clone` 預設給 repository 來源 (remote) 的名字，所以 'origin/develop' 指的是 remote 'origin' 上的 branch 'develop' (// remote 解釋的不好...)
4. 而 develop 指的是本地端的 develop branch

所以，綜合來說，就是有人在「本地端」把來源上的開發 branch merge 到自己的開發 branch，之後又把這個 commit 推上共用的 repository。在 git 提供的功能內，這是完全正常的。問題出在一般開發流程不期待這件事發生。


一般開發流程?
----------

git 對比於 svn 強大許多，給使用者許多的彈性，但是多人協作時有一些規範才好作事，所以有人提出一些開發流程，像是最有名 (?) 的 [git flow][git-flow] 和 [github flow][github-flow]

[git-flow]: http://nvie.com/posts/a-successful-git-branching-model/
[github-flow]: https://guides.github.com/introduction/flow/

所以，說是一般開發流程，應該說是 github flow 這個流程。這個流程大概是這樣的(\*1):

1. 開發者在他的機器把 repository clone 下來，新增一個 feature branch (或叫 topic branch)
2. 開發者在本地端進行開發，造出一到多個 commit
3. 開發者把這個 branch 推到遠端的 repository (此時遠端 repository 會多一個新 branch)
4. 開發者新增一個 pull request 提出要把這些修改整合進去的請求。此時管理人可以和開發者以 pull request 為單位進行討論。
5. 完成後管理人按下 merge 把這個 feature branch 整合進 repository。

也就是說，這個流程裡開發者端是不會作 merge 的！但是這件事卻意外地容易發生...


悲劇是怎麼發生的?
----------------

1. 怎麼把 source code 抓下來? `git clone`
2. 修改，commit，修改，commit (在 develop branch 上)
3. 怎麼把修改推到共用的 repository 上? `git push origin`
4. 咦!? push confict!? 喔，有好心的提示耶，他說可以 `git pull`

    hint: Updates were rejected because the remote contains work that you do
    hint: not have locally. This is usually caused by another repository pushing
    hint: to the same ref. You may want to first integrate the remote changes
    hint: (e.g., '**git pull** ...') before pushing again.
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.

5. pull 好囉，再 push 一次看看
    * 這個 pull 預設就會產生 merge commit，於是...
6. 唉呀！ 'Merge origin/develop into develop' 出現在共用 repository 了 (拍頭)

這是我親身體驗過的流程... 真的還滿容易踩到的

可以怎麼作?
----------

1. 不要用 `git pull`，先作 `git fetch origin` 把來源的更新抓下來 (不會修改在本地端的 branch)，然後再視狀況作 `git rebase` 或是 `git cherry-pick`
2. 如果你大部份的時候需要的是 `git fetch` 加上 `git rebase`，可以打開 pull.rebase 的設定: `git config pull.rebase true`
3. 注意: rebase 是一個會改動歷史的行為，最好先確認會不會不小心把已經公開的 branch 歷史給改寫了
4. 尋找關於專案開發流程的文件，如果沒有就去找看起來知道流程的人，請他寫一份文件... 或是在團隊內提出是否需要定出開發流程的討論。
5. 閱讀文件了解以下概念: [git remote][pro-git-git-remote]，[rebase 和 merge][pro-git-git-merge]；或是直接把 [Pro git][pro-git] 啃一遍。
6. gitlab 可以針對受保護的 branch 限制哪些人可以直接推到這些 branch，[github][github-protected-branch] 也有類似的設定(\*2)。gerrit... 的設定比這更複雜。
7. 延伸閱讀: [Linus 大神][linus-on-rebase-merge] 談 rebase 還有 pull 的時機

[pro-git]: https://git-scm.com/book/en/v2
[pro-git-git-remote]: https://git-scm.com/book/zh-tw/v1/Git-%E5%9F%BA%E7%A4%8E-%E8%88%87%E9%81%A0%E7%AB%AF%E5%8D%94%E5%90%8C%E5%B7%A5%E4%BD%9C
[pro-git-git-merge]: https://git-scm.com/book/zh-tw/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E8%A1%8D%E5%90%88
[github-protected-branch]: https://help.github.com/articles/about-protected-branches/
[linus-on-rebase-merge]: https://gist.github.com/kstep/e28d1c762da4e97065fe

後記
-----

前陣子看到 twitter 上有人在說 git 設計的不好，太複雜了，當時心中頗有不平，覺得明明很好用啊! 寫了這一篇才覺得真的有很多無法用一兩句話解釋完的概念... 一方面個人才疏學淺啊...

\*1: 嚴格的說這裡的描述更像是 gitlab flow? 但是要把 fork 講進去就太複雜了...
\*2: 我不確定是誰抄誰，哈哈哈...


Originally publised on [logdown](http://redeyes2015-blog.logdown.com/posts/1224774-merge-branch-develop).
Date: December 19, 2016
