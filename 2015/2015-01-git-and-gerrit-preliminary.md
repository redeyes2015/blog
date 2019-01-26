Git 與 Gerrit 初步
==================


前言
====

這篇是寫給在部門內使用 Git/Gerrit 的教學文，假如你只是剛好路過，有些地方可能不適用。

Git 安裝與初始化
==============

1. 安裝，假設你在 ubuntu 或其他 debian 的延伸版上:
```sh
    $ sudo apt-get install git
```

2. 設定會出現在 Commit 上的 name 和 email
```sh
    $ git config --global user.name "Your Name Comes Here"
    $ git config --global user.email you@yourdomain.example.com
```

3. (可選) 一些讓 git 更方便使用的設定
```sh
    $ # 沒有可能會很不習慣的別名:
    $ git config --global alias.st status
    $ git config --global alias.ci commit --verbose
    $ git config --global alias.co checkout

    $ # git 的彩色輸出很不錯，而且設成 "auto" 時可以自動偵測接 pipe 就不要上色
    $ git config --global color.diff auto
    $ git config --global color.status auto
    $ git config --global color.branch auto

    $ # 若 git 版本在 1.8 之後(包含 1.8) 請用下面這個選項
    $ # 避免等一下 push 出現很長很可怕的警告訊息
    $ # (細節請參考 `git help config` 中的 `push.default` 節)
    $ git config --global push.default simple
```

Gerrit 註冊及設定
===============

1. 打開瀏覽器，連上 Gerrit:
![gerrit_start.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/zAaXSJxKTN2VXyZmRlpQ_gerrit_start.PNG)

2. 點右上角的 "Sign in":
![gerrit_signin.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/9U4pDxrUQ3ucGMQ7OVIz_gerrit_signin.PNG)

3. 填上公司的 AD 帳號/密碼，登入後點右上角的名字，然後點框框中的 "Settings" :
![user_setting.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/88vrfKk0STOxaashPuKk_user_setting.PNG)

4. 點 "HTTP Password":
![http_settings.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/H17qHvM9SAqfqWZZsB1e_http_settings.PNG)

5. 點 "Generate Password":
![user_setting_password.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/hCihxc3lRZy2aMSPAGxO_user_setting_password.PNG)

6. 回到 Linux 終端，打開編輯器，把下面這行加到 `~/.netrc` 中: (`<Gerrit IP>` 換成 Gerrit 站的 IP，不用加 Port，`<YOUR ID>` 換成 AD 帳號， `<YOUR PASSWORD>` 換成剛剛產生的密碼，**注意！不是 AD 密碼，而是剛剛在 "HTTP Password" 頁面產生的密碼！**
```
machine <Gerrit IP> login <YOUR ID> password <YOUR PASSWORD>
```

7. 修改 `~/.netrc` 的權限避免太容易被別人看到密碼... 以防萬一嘛:
```sh
> chmod 600 ~/.netrc
```

8. 因為很重要，所以再說一次，Gerrit 的網頁登入時，用的是 **AD 帳密**，但是存取 git repository 時用的是 "HTTP Password" 頁面中產生的密碼，有照著作的話，剛剛已經存在 `~/.netrc` 裡，所以你不必每次都去網頁上複製。或是，有另一種作法是用 ssh 的 public key 認證，個人比較喜歡用 public key，但是步驟稍微複雜一些，請等待打算 <del>富堅</del> 拖稿的我，或是自行問 Google 大神... ([P.S.1](#ps1))

開始玩沙盒
==========

1. 回到 Gerrit 的網頁，點左上角 "Project" -> "List" -> 找下面的　"Sandbox"
![project_list.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/fb2HFUdRpmm0PO2jpzH8_project_list.PNG)

2. 點 "Sandbox" 的文字部份 -> 點下面圖中我畫的紅框框起來的 "HTTP":
![sandbox_general.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/upXZqeTkT5COjacjH6gP_sandbox_general.PNG)

3. 把`git clone` 開頭那行，複製下來，回到 Linux 終端，貼上，按 Enter。**注意: 會把新的 Repository 建立在 `./Sandbox` 上**，可以先確認目前的目錄是不是你想要的。

4. 來造個 commit，然後 push 到 gerrit 上.
```sh
    $ cd Sandbox # 如果你還沒切換進 Sandbox 目錄...
    $ echo blahblah >> foo # 可以發揮一下創意 ;-)
    $ git add foo # 別忘了 git 需要先加到 staging area 再 commit
    $ git ci -m 'Hello world!' # 別忘了這時候只是 commit 到 local repository 上
    $ git push # 來把修改推到 gerrit server 上! 這裡假設有作 "push.default" 的設定...
```
5. 回到 gerrit 網頁，點右上角 "Projects" 下面的 "Branches"，這裡會列出所有你看得到的 branches，現在應該只有 "master" 這一個 branch，其他的就先不管他... :
![gerrit_branches.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/d1s91VXdSe2tc8k4pBy8_gerrit_branches.PNG)

6. 來看看剛剛的 commit 吧，點前一步圖中 "master" 那一排最右邊的 "(gitweb)" 就會出現 gitweb 的界面，這裡可以看到某個 branch 的歷史還有 commit 內容之類的，可以探索一下，不多贅述，總之我們剛剛的 commit 出現啦! (灑花): 
![gitweb.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/yYPlQ8kRfKomnoAy5law_gitweb.PNG)

如果 Conflict...
================
理論上，寫到這裡簡介就結束了.... 但是，出來 commit，總有一天會遇到 conflict... 所以這邊簡單講一下遇到 conflict 的狀況

1. 如果在 `git push` 之後看到如下的訊息，表示有 conflict 發生:
```
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'http://ryan.chang@172.18.51.25:8082/Sandbox'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

2. Hint 裡其實帶了不少資訊，總之就是有其他人先 commit 了，但現在 gerrit 設定只收 'fast-forward' 的 commit ([P.S.2](#ps2))，所以先把其他人改的版本抓下來:
```sh
    $ git fetch
```

3. 接下來要想辦法修改 local repository 的歷史，好讓 gerrit server 願意收 commit ... 用 `git rebase`

4. 其實通常來說 git 會很聰明的作完調整，所以如果在 `git rebase` 之後沒有看到什麼可怕的訊息，應該就可以回到 `git push` 的路線，再次把 commit 推上去了。但這裡還是講一下真的 conflict 的狀況，你應該會看到類似下面的訊息:
```
$ git rebase
First, rewinding head to replay your work on top of it...
Applying: Hello again!
Using index info to reconstruct a base tree...
M       foo
Falling back to patching base and 3-way merge...
Auto-merging foo
CONFLICT (content): Merge conflict in foo
Failed to merge in the changes.
Patch failed at 0001 Hello again!
The copy of the patch that failed is found in:
   .../Sandbox/.git/rebase-apply/patch
When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```

5. 那麼來 merge 吧，敲 `git mergetool`，`[Enter]`，會叫出你設定的工具，以我來說，隨處可見的 vim 就夠用了:
![mergetool.PNG](http://user-image.logdown.io/user/3659/blog/3715/post/250087/Bdxdl79mSi6JUodhxMPg_mergetool.PNG)

6. 改成你最後希望的樣子之後，存檔，離開編輯器，告訴 git 可以繼續 rebase 了:
```
$ git rebase --continue
Applying: Hello again!
```

7. rebase 完成後，就可以回到 `git push` 路線去了

後記
====
相比於 Subverion，Git 真的強大許多，也複雜許多，整串下寫來，其實有很多概念沒有說清楚，還請有志之士在網路上尋找資源補完... 例如 "[Git - SVN Crash Course][git-svn-course]"，還有 "[Git book][git-book]"，等等。

[git-svn-course]: http://git.or.cz/course/svn.html
[git-book]: http://git-scm.com/book/en/v2/Getting-Started-About-Version-Control

P.S.
====
1. <a name="ps1"></a>這裡的作法我是抄 [Qt wiki][qt-gerrit] 的作法，但是根據 [這篇][git-credential] 的內容來看，應該有更好的辦法，日後再來修正好了。

2. <a name="ps2"></a>總之就是你想推到 server 上的 commit 的歷史，跟 server 上已經有的最新歷史必須是相符的，所以要想辦法把我們造的 commit "搬到新的歷史上"。舉例來說就是如果你想推的 commit 是 c->d，而 server 上的 log 是 a->b (舊到新)，你要推的當下，你的歷史必須是 a->b->c->d，不能是 *a*->c->d 或 a->**b'**->c->d... 好吧，我真的不太會解釋就當作是進階話題，改天再解釋 (掩面)

[qt-gerrit]: http://qt-project.org/wiki/Setting-up-Gerrit "Setting Up Gerrit"
[git-credential]: http://pempek.net/articles/2014/10/16/farewell-netrc-welcome-git-credentials/ "Farewell .netrc, Welcome Git Credentials"


Originally published on [logdown](http://redeyes2015-blog.logdown.com/posts/250087-git-and-gerrit-preliminary).
