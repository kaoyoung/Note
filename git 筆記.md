## 參考自 [為你自己學 Git](https://gitbook.tw/#toc)

>[!question] 問題一
>`.git` 目錄下有啥重要的檔案?

- config(設定檔):

使用者的 local 設定基本都在這個 config 檔中。例如: user.name、user.email，其設定方法如下
- `git config --local user.name "hsu"`
- `git config --local user.email "hsu@c.com"`

>[!note] git 設定的優先級為何? 與其應用
>看的順序為: local -> global -> system。這有一用處在設定 user.name 跟 user.email 上，我們可以在自己的電腦上先用 `git config --global user.name "kao"` 跟 `git config --global user.email "kao@c.com"` 來設定自己的名字跟信箱 (名字跟信箱是寫在 commit 上的署名)，而在公司的專案下設定 `git config --local user.name "hsu"` 跟 `git config --local user.email "hsu@c.com"`。如此可區分公務跟自用的名字跟信箱。要注意 `git config --global` 的設定在 `~/.gitconfig` 中。

>[!question] 問題二
>git 把儲存空間分成幾個分區? 檔案如何在各分區如何移動?

git 裡總共有三個分區: working directory (工作目錄)、staging area (暫存區)、repository (儲存庫)。可以用 `git add` 把檔案從 working directory 存一份到 staging area；`git commit` 把 staging area 的檔案存成一筆紀錄 (snapshot) 存進 repository ，staging area 內的檔案不會在 `git commit` 後消失 (可以用 `git ls-files` 看目前在 staging area 的檔案)。`git add` 可以用 `*` 、 `.`、`--all` 來一次把多個檔案從 working directory 送到 staging area。

>`git add .` 跟 `git add --all` 在現在的主要差別在於範圍：`git add .` 處理**當前目錄**以下（包含所有子目錄）的檔案，`git add --all` 處理**整個專案**的檔案。兩者都會把新增、修改的檔案的完整內容存一份到 staging area，並把已刪除的檔案從 staging area 移除。`git add *` 由 shell 展開，會漏掉隱藏檔和已刪除的檔案。

取消暫存用 `git restore --staged welcome.html`。`git rm --cached welcome.html` 則是讓 Git 停止追蹤這個檔案（保留工作目錄的檔案）；如果檔案已經 commit 過，下次 commit 後它會從 repository 裡被刪除。

>[!note] 如何看 `git status` 的輸出
>untracked 表示在 working directory ，git 管不到。changes not staged 表示 working directory 的版本和 staging area 不一樣。changes to be committed 表示 staging area 的版本和最後一次 commit 不一樣。如果把 `a.txt` 先 `git add` 後再修改 `a.txt` ，則你會再 changes not staged 跟 changes to be committed 看到 `a.txt`。







>[!question]
>如何更改 commit 後的資料?

如果只是想更改上一筆 commit 的資料可以使用 `git commit --amend -m "Hello"` 來更改上一筆 commit 的 message， `git commit --amend --no-edit` (`--no-edit` 表示沿用上一個 commit 的 message)可以用來更改上一次 commit 沒增加的檔案 (需先用 `git add <file>` 把該檔案加進 staging area)。`git commit --amend` 的本質是「拿目前 index 的完整 snapshot 做新 commit」，正如手冊說的
>Replace the tip of the current branch by creating a new commit.

所以其實可以一次新增檔案跟改 message，一樣先用 `git add <file>` 再用 `git commit --amend -m "message"`，即可完成。如果進行多次 `git commit --amend` 對於相同的項目會以最後一個為主，不同的項目之間彼此不衝突，也就是可以同時用 `git commit --amend -m` 更改 message 和 `git commit --amend --no-edit` 加檔案。利用 `git reflog` 可以看到使用 `git commit --amend` 的訊息為
- 30e359d HEAD@{1}: commit (amend): test 2

而一般的 `git commit` 的訊息為
- fc7e0a0 HEAD@{3}: commitFN: add test.txt

可以看到 `git commit --amend` 確實是一筆新的 commit 取代掉舊的 commit。

>[!question]
>Git 一定會追蹤專案內的所有檔案嗎?

不要把不想追蹤的檔案 `git add` 到 staging area 內，git 就不會追蹤。用前面這方法有一問題，如果我不小心 `git add --all` 該檔案會直接殺進 staging area 內，那 git 便會開始追蹤，當然可以使用 `git rm --cached` 來將該檔案移除追蹤，但如此作不省心。常見方法是用 `.gitignore` 來註明哪些檔案不會被 git 自動追蹤，這方法要注意 `.gitignore` 的設定不會朔及既往，正如手冊中說的

>A `.gitignore` file specifies intentionally untracked files that Git should ignore. Files already tracked by Git are not affected; see the NOTES below for details.

`.gitignore` 只會對 untracked file 生效，而 untracked file 即為在 working directory 但不在 staging area 的檔案，untracked file 的觀念可以從 gitignore 的手冊中的 NOTES 推得，原文如下 (index 為 staging area 的別稱)

> To stop tracking a file that is currently tracked, use `git rm --cached` to remove the file from the index.

如果執意要將 tracked files 受到 `.gitignore` 監管，可以使用 `git rm --cached` 將它們變成 untracked files ，把規則寫進 `.gitignore`，再 commit 一次，之後的 snapshot 就不會包含該檔案。





# 雜項
- `git init`
- `git log`、`git log --oneline`、`git log --online --graph`、`git log --oneline --author="Hsu"`、`git log --oneline --author="Hsu\|Kao"`、`git log --oneline --grep="WTF"`、`git log -G "Ruby"`、`git log --oneline --since="9am" --until="12am"`、`git log --onelin --sinc="9am" --until="12am" --after="2017-01"`、`git log welcome.html`、`git log -p welcome.html`
- `git status`
- `git rm welcome.txt`
- `git mv hello.html world.html`
- `git blame index.html`、`git blame 5,10 index.html`
- `git checkout cinderella.html`、`git checkout .` (前兩指令拿到的是 index 的版本)、`git checkout HEAD~2 welcome.html` (這指令會更改 index，think about why)


