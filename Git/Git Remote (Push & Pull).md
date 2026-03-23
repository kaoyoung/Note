# Motivation
When collaborating within a Distributed Version Control System, developers must navigate several core synchronization challenges:
1. **State Retrieval :** How do we accurately retrieve the most up-to-date state of the remote repository?
2. **Code Integration :** How do we safely reconcile our local, unshared modifications with the latest upstream changes?
3. **State Publication :** How do we update the shared remote repository with our newly integrated code?

To address these challenges and maintain a consistent codebase, developers must execute a specific sequence of operations :
1. **Fetch the Remote State :** Retrieve the current commit graph (topology) and associated objects from the remote server, updating the local repository's tracking branches without modifying the working directory.
2. **Merge or Rebase Local Commits :** Reconcile the local commit history with the newly fetched remote history. This integrates the divergent commit topologies into a single, cohesive timeline.
3. **Push the Reconciled State :** Transmit the integrated commit graph and newly created objects back to the remote server, updating the shared repository for other collaborators.
---
# Terminology
1. Remote branch
It reflect the _state_ of remote repositories (since you last talked to those remote repositories). They help you understand the difference between your local work and what work is public -- a critical step to take before sharing your work with others.
>[!Note] Concept about remote branch
>**Remote branches have the special property that when you check them out, you are put into detached `HEAD` mode**. Git does this on purpose because you can't work on these branches directly ; you have to work elsewhere and then share your work with the remote (after which your remote branches will be updated).
>To be clear: Remote branches are on your _local_ repository, not on the remote repository.

>[!Important]
>不能在 `origin/main` 做開發。

2. `<remote name>/<branch name>` : `origin/main`
Most developers actually name their main remote `origin`
>[!Important]
> `origin/main`  will only update when the remote updates.

---
# Command
1. `git clone` 
It is the command you'll use to create _local_ copies of remote repositories (from github for example).
2. `git fetch`
`git fetch` performs two main steps, and two main steps only. It:
- **downloads** the commits that the remote has but are missing from our local repository, and...
- **updates** where our remote branches point (for instance, `o/main`)
>[!note] Concept about remote branch
>`git fetch`, however, does not change anything about _your_ local state. It will not update your `main` branch or change anything about how your file system looks right now.
3. `git pull`
`git pull` = `git fetch`+`git merge`
4. `git push`
It is responsible for uploading _your_ changes to a specified remote and updating that remote to incorporate your new commits. Once `git push` completes, all your friends can then download your work from the remote.
5. `git pull --rebase`
`git pull --rebase` = `git fetch`+`git rebase`

---
# Other
## Remote Rejected
>[!question] Remote Rejected
>If you work on a large collaborative team it's likely that **main is locked and requires some Pull Request process to merge changes**. 

If you commit directly to main locally and try pushing you will be greeted with a message similar to this:
```
! [remote rejected] main -> main (TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.)
```

>[!solution] solution of remote rejected
>You meant to follow the process creating a branch then pushing that branch and doing a pull request. We have to create another branch called feature and push that to the remote. Also **reset your main back to be in sync with the remote** otherwise you may have issues next time you do a pull and someone else's commit conflicts with yours.