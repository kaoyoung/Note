## 參考自 [為你自己學 Git](https://gitbook.tw/#toc)、[Pro Git](https://git-scm.com/book/en/v2)


>[!question]
>Git 的基本的資料結構為何?

Git 基本的資料結構為 blob (binary large object) object、 tree object 、 commit object 和 tag object。 Blob object 是個壓縮過的二進制的檔案用來存檔案的內容，Git 會用 SHA-1 值來命名這個 blob，而這 SHA 值可以用來索引這檔案，也就可以使用 `git cat-file -p <object>` 來看檔案的內容。Tree object 跟 UNIX directory 一樣是用來存檔案根子目錄，也就是存 blob 跟 tree 的。可以4 `git cat-file -t <object>` 來看 object 的型別。Commit object 存了五個訊息: 一個對應整個 snapshot 的 tree object 、parent commit 、author 、committer 和 commit message ，parent commit 讓 commit 構成一串 DAG 狀的歷史。Tag（annotated tag）記錄它指向的物件、tagger 和 tag 訊息。



>[!question] 
>`.git` 目錄下有啥重要的檔案?

### config:
使用者的 local 設定基本都在這個 config 檔中。例如: user.name、user.email，其設定方法如下
- `git config --local user.name "hsu"`
- `git config --local user.email "hsu@c.com"`
### HEAD:
紀錄目前的 branch。
### ORIG_HEAD:
紀錄 ORIG_HEAD 的值。當作 `git merge` 、`git rebase`、 `git reset` 等會改變目前 checked-out commit 的操作時，會把當前 checked-out commit 存在 ORIG_HEAD 再跳到該去的 commit。
### index:
紀錄目前在 staging area (index) 的所有檔案。可以使用 `git ls-files --stage`  來看在 staging area 的檔案，如果要對於這些檔案更詳細的描述，可以用 `git ls-files --debug`。
### objects 目錄:



### logs 目錄:



### refs 目錄

`refs/heads` 儲存 branch 的名字和其對應 commit。




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
>請描述 `git commit` 時 git 究竟做了那些操作，存了那些檔案，存在哪裡? `commit` 的本質是啥?






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

>[!question] 
>如何移動 `branch` 到其他的 commit 上? 移動過去後，經過的檔案去哪兒了? 如何移回來?

直接用 `git reset <commit>` 跳到想要的 `<commit>` 上，或是借助 `^` 、`~`、 `HEAD` 來作跳躍，`^` 表前一個 commit ，`~N` 表向前移動 N 個 commit， `HEAD` 表當前指向的 commit。舉幾個例子
- `git reset 5eac9ec^`: 表從 `5eac9ec` 向前移動一個 commit
- `git reset HEAD~3`: 表從 `HEAD` 向前移動三個 commit，即為從當前 commit 向前移動三個 commit。

`git reset` 常搭配的參數有 `--soft`、`--mixed` 跟 `--hard`，預設的搭配是 `--mixed`。這三個參數對應的是 working directory 跟 staging area 在 `git reset` 後改變的樣子

|                   | `--soft` | `--mixed`    | `--hard`     |
| ----------------- | -------- | ------------ | ------------ |
| working directory | 不變       | 不變           | 重設成目標 commit |
| staging area      | 不變       | 重設成目標 commit | 重設成目標 commit |

Commit 過的檔案不會因為 `git reset` 而消失，正如手冊所說的
>This form resets the current branch head to `<commit>` and possibly updates the index (resetting it to the tree of `<commit>`) and the working tree depending on `<mode>`. 

他只移動 current branch 到想要的 `<commit>` 上，並依 mode 決定是否重設 index 和 working tree。被跳過的 commit 不會被刪除，只是沒有 branch 指向它們，可以透過 reflog 找回。。可以使用 `git reflog` 看 "reflogs" 的紀錄，"reflogs" 會儲存本地 HEAD 和各 branch 指標的移動歷史 (對於當前 commit 可以 reach 的紀錄預設只有 90 天內的紀錄，對於當前 commit 不可以 reach 的紀錄預設只有 30 天內的紀錄)，所以用 "reflogs" 來找要跳回去的 commit 再用 `git reset` 就好，舉例
- `git reset --hard e12d8ef`: 跳回 sha 值為 e12d8ef 那個 commit。

`--hard` 的意思如上所敘，會洗掉未  commit 的修改。

`
>[!question]
>如何創造、合併、刪除和剪裁 `branch`? `branch` 的本質是啥?

使用 `git branch` 變能看目前專案內所有的 branch。`git branch <branchname>` 可以創建出一個新的 branch。 `git branch -m [<oldbranch>] <newbranch>` 可以改 branch 的名字。`git branch -d <branchname>` 可以刪除一個 branch。`git checkout <branchname>` 可以跳到想去的 branch，`git checkout -b <branchname>` 可以創造一個新 branch 並把 HEAD 移過去，`git checkout -b bird 657fce7` 能夠在 SHA 值為 657fce7 開頭的 commit 開出一個分支叫做 bird 並把 HEAD 移到該 commit。`git merge <branchname>` 會先造一個 commit 把當前 branch 跟新增的 branch 合在一起 (或是更精確的說是這個 commit 指向當前 branch 的 commit 和新增 branch 的 commit)，並把當前的 head 和 branch 移到該 commit。`git branch -d <branchname>` 會刪除 branch。`git branch <branchname> [<start_point>]` 可以直接在 `<start_point>` 上建立 branch，`<start_point>` 在手冊的描述如下
>The new branch head will point to this commit. It may be given as a branch name, a commit-id, or a tag. If this option is omitted, the current HEAD will be used instead.

`git rebase` 提供了把兩個 branch 合併成一個 branch 的可能。假設目前的 HEAD 在 cat branch 且 HEAD 指向 b174a5 做 `git rebase dog` 整個過程和結果如下圖
![[git rebase picture.png]]
參考自: [為你自己學 Git 第六篇第八章 (另一種合併方式（使用 rebase）)](https://gitbook.tw/chapters/branch/merge-with-rebase)
一般的 `git merge` 可以用 `git reset --hard HEAD^` 會到 commit 前；而 `git rebase` 可以用 `git reset --hard ORIG_HEAD` 回到 rebase 前，會是去翻 `git reflog` 找到 `git rebase` 前一個 commit，依據手冊
>If `<branch>` is specified, git rebase will perform an automatic git switch `<branch>` before doing anything else.

`git rebase` 前會跳掉要當新 base 的那個 commit。`git checkout --ours <pathspec>` 跟 `git checkout --theirs <pathspec>` 可以在 merge 或是 rebase 衝突在同一檔案時，決定要用誰後直接把 `<pathspec>` 改成衝突的檔案名。`git rebase -i <commit>` 可以修改從當前 HEAD 到該 commit 的所有 commit message (reword)；合併這之間的多個 commit (squash)；把一個 commit 拆成多個 commit (edit)，在到 edit 的那個 commit 可以使用 `git reset HEAD^` 來把這 commit 拆了再修改；刪除 commit 或是修改 commit 順序，直接去 `git rebase -i <commit>` 頁面改。一樣 `git rebase -i <commit>` 的操作可以用 `git reset --hard ORIG_HEAD` 來還原。`git revert HEAD --no-edit` 可以用來取消做後一次 commit，其原理是透過一個新 commit 來取消原來的 commit，可以再使用一次 `git revert HEAD --no-edit` 來取消該才那次取消的 commit，或是直接用 `git reset --hard HEAD^`。






在 "Pro git" 3.1 節的擷取原文如下
>What happens when you create a new branch? Well, doing so creates a new pointer for you to move around.

從這可以看出 branch 就只是個指向 commit 的指標。




`git log --oneline --graph --all`


>[!important]
>不要在推出去的 commit 用 `git rebase` 跟 `git reset`，因為他們會修改歷史紀錄。




>[!question]
>`git rebase` 似乎修改 commit 的歷史，我是真的在 `git rebase` 後無法得知那些 commit 是被誰改的又改了啥嗎? 為何 `git rebase -i`  後改到有 branch 指向的 commit，該 branch 在改前顯示在 `git log` 上在改後變不在 `git log` 上?




>[!question]
>為何建議少用 `git checkout` ，多用 `git switch` 跟 `git restore` ?





>[!question]
>Git 想要儲存什麼東西，提供甚麼服務? 儲存東西的結構為何?



# 雜項
- `git init`
- `git log`、`git log --oneline`、`git log --online --graph`、`git log --oneline --author="Hsu"`、`git log --oneline --author="Hsu\|Kao"`、`git log --oneline --grep="WTF"`、`git log -G "Ruby"`、`git log --oneline --since="9am" --until="12am"`、`git log --onelin --sinc="9am" --until="12am" --after="2017-01"`、`git log welcome.html`、`git log -p welcome.html`
- `git status`
- `git rm welcome.txt`
- `git mv hello.html world.html`
- `git blame index.html`、`git blame 5,10 index.html`
- `git checkout cinderella.html`、`git checkout .` (前兩指令拿到的是 index 的版本)、`git checkout HEAD~2 welcome.html` (這指令會更改 index，think about why)


