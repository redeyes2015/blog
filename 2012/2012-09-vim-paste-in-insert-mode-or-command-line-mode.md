vim: paste in insert mode or command mode
-----------------------------------------

前幾天在 coderwall 上面看到驚為天人的 protip,

簡單的說就是在插入模式或是命令模式可以用 `<c-r> + {register}` 的組合在不離開當前模式的狀況下貼上對應的文字!

例如原本想複製某一段文字然後在搜尋的時候貼上使用, 可能需要用滑鼠選取之後在用中鍵貼上 (如果你是在用 linux 的話), 現在可以先用 `yw` 複製起來, 然後 , `/<c-r>"<cr>`, 馬上就可以搜尋剛剛複製的字串了! 或是想要在插入模式插入剛剛搜尋的字串, 用 `<c-r>/` 就可以直接貼上, 真是太好用了!

試用了幾天以後, 雖然現在還有點不習慣,但是兩隻手需要離開鍵盤的時間又更少了…! (開心~

Originally published on [tumblr](http://rein-notes.tumblr.com/post/31107746616/vim-paste-in-insert-mode-or-command-mode)
Date: SEPTEMBER 8, 2012 (1:35 PM)
Tags: #VIM
