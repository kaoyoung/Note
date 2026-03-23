# Motivation
How to move through the commit topology that represents your project.

---
# Terminology
1. `HEAD`
`HEAD` is the symbolic name for the currently checked out commit -- it's essentially what commit you're working on top of.
- `HEAD` always **points to the most recent commit which is reflected in the working tree**.
2. Caret operator : `^`
Each time you append that to a ref name, you are telling Git to find the parent of the specified commit.
- `main^^` : It is the grandparent (second-generation ancestor) of `main`
3. Tilde operator : `~[num]`
It takes in a trailing number that specifies the number of parents you would like to ascend.
- `git branch -f main HEAD~3` : moves (by force) the main branch to three parents behind HEAD.  Note: In a real git environment `git branch -f` command is **not allowed for your current branch**.
---
# Command
1. Detaching `HEAD`
Detaching HEAD just means attaching it to a commit instead of a branch. This is what it looks like beforehand: `git checkout [commit hash value]`
2. `git reset`、`git revert HEAD`
Like committing, reversing changes in Git has both a low-level component (staging individual files or chunks) and a high-level component (how the changes are actually reversed). Our application will focus on the latter.
- `git reset HEAD~1` : It will move a branch backwards as if the commit had never been made in the first place. **It only do on your own machine**.
- `git revert HEAD` : In order to reverse changes and **share** those reversed changes with others, we need to use `git revert`. A new commit plopped down below the commit we wanted to reverse. That's because this new commit `C2'` introduces _changes_


a new commit plopped down below the commit we wanted to reverse. That's because this new commit `C2'` introduces _changes_ -- it just happens to introduce changes that exactly reverses the commit of `C2`.


