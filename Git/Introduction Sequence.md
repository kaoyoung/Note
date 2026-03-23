## Reference [Learn Git Branching](https://learngitbranching.js.org/)
# Motivation
First think why we need to do version  control ? 
1. Multiple developers need to work on the same project at the same time without stepping on each other's toes or overwriting each other's code.
2. You need to save snapshots of your current work. This gives you a timeline of your progress and a safe "restore point" to go back to if you make a mistake and break your code.

What's the corresponding operation 
1.  You create an isolated copy (a branch) of the main code. This lets you build a new feature safely without breaking the main project.
	- `git branch`
	- `git checkout`
	- `git merge`
	- `git rebase`
2.  You save your current state by making a "commit."
	- `git commit`
---
# Command
1. `git commit -m "message"`
A commit in a git repository records a snapshot of all the (tracked) files in your directory. It's like a giant copy and paste, but even better
- It can (when possible) compress a commit as a set of changes, or a "delta", from one version of the repository to the next.
2. `git branch <name>`
Branches are simply pointers to a specific commit -- nothing more. For now though, just remember that a branch essentially says "I want to include the work of this commit and all parent commits."
- `git branch -d <name>`
3. `git checkout <name>`
Telling git we want to checkout the \<name\> branch 
- `git checkout -b [yourbranchname]` : If you want to create a new branch AND check it out at the same time
4. `git merge`
Merging in Git creates a special commit that has two unique parents. A commit with two parents essentially means "I want to include all the work from this parent over here and this one over here,  and the set of all their parents."
5. `git rebase [branchname]`
Rebasing essentially takes a set of commits, "copies" them, and plops them down somewhere else.
- The advantage of rebasing is that it can be used to make a nice **linear sequence of commits**.
---
# Question

>[!question] Question 1
>What's the key difference between `git merge` and `git rebase`

The short answer is: **We don't need `rebase` to make the code work. We need `rebase` to make the project history readable by humans.**
If we only use `git merge`, git history may look like a bowl of spaghetti. The solution is using `git rebase`. It will takes your new commits and neatly stacks them directly on top of the newest main code.