# Git

Git is a distributed version control system (VCS) that allows developers to track
changes in source code, collaborate on software development, and manage
projects of any size efficiently. Created by Linus Torvalds in 2005. A good
source of truth is the documentation on [Git-SCM](https://git-scm.com/doc).

**For a list of files to be pushed, run:**

```bash
git diff --stat --cached [remote/branch]
# example: git diff --stat --cached origin/master
```

**For the code diff of the files to be pushed, run:**

```bash
git diff [remote repo/branch]
```

**To see full file paths of the files that will change, run:**

```bash
git diff --numstat [remote repo/branch]
```

**To stop tracking a file you need to remove it from the index:**

```bash
git rm --cached <file>
git rm -r --cached .
```

**To see list of commits without push:**

```bash
git log origin/master..master
```

**To remove a git commit which has not been pushed:**

> [!WARNING]
>this will revert local changes.

```bash
git reset --hard COMMIT-ID
# example: git reset --hard eb27bf26dd18c5a34e0e82b929e0d74cfcaab316
```

**To clone a repo (with a branch):**

```bash
git clone URL
git add .
git commit -m “message”
git push
```

**To undo ‘git add’ before commit:**

```bash
git reset <file>
git reset
```

**To discard all local uncommitted changes:**

```bash
git checkout .
```

**To discard all unstaged files in current working directory use:**

```bash
git checkout -- .
```

**To remove untracked files:**

```bash
# Remove all untracked files and directories.
# '-f' is force, '-d' is remove directories.
git clean -fd
# preview changes
git clean -nd
```

**To compare local repo with remote repo:**

```bash
git diff master
```

**To list branches:**

```bash
git branch -a
```

**To rename a branch:**

```bash
git branch -m <old name> <new name>
```

**To switch to another branch:**

```bash
git checkout branch-a
```

**To merge other branches with the master branch:**

```bash
git merge branch-a
```

**To create and switch to a new branch:**

```bash
git checkout -b new-branch-name
```

**To undo a pushed commit without any trace:**

```bash
git reset <previous label or sha1>
git commit -am "Some commit message"
git push -f <remote-name> <branch-name>
```

**To merge/squash the last three commits into one:**

```bash
git reset --soft HEAD~3 # [~ is called tilde]
git commit -m "New message for the combined commit"
```

**To start a new feature branch:**

```bash
git flow feature start ch<story number>/<branch name>
```

Now, start committing on your feature. When done, use:

```bash
git flow feature finish ch8376/get-directions
```

**To rebase the current branch with develop branch:**

```bash
git rebase -i origin/develop
:wq
git push -f
```

**To add new changes to a not pushed commit:**

```bash
git add .
git commit --amend --no-edit
```

**To see the difference of the last commit:**

```bash
git diff HEAD^
```

**To sign all the commits automatically:**

```bash
add the following to `.gitconfig`:
  [commit]
    gpgsign = true
```

**Fetch and checkout a remote branch:**

```bash
git fetch
git checkout -b <branch> origin/<branch>
```

**Change the base of the branch:**

```bash
git rebase --onto new-base-branch current-base-branch
```

**Compare local branch with the same branch on remote:**

```bash
git fetch
git diff @{upstream}
```

**Cherry picking:**

```bash
git cherry-pick <commit-hash>
```

**List remote branches:**

```bash
git branch -r
```

**See only files that are having conflicts:**

```bash
git diff --name-only --diff-filter=U
```

**To Delete every branch that starts with bugfix:**

```bash
git branch --list ‘bugfix*’ | xargs -r git branch -d
```

**To stash changes with a name:**

```bash
git stash push -m “some name”
```

**To change remote upstream:**

```bash
git remote rename origin upstream
git remote add origin git@github.com:[new repo]
git config --get remote.origin.url // prints out the new remote upstream
```

**To unset upstream after renaming a branch that has been pushed to the remote:**

```bash
git branch --unset-upstream
```

**Copy diff into clipboard:**

```bash
git diff | pbcopy
```

**Apply the diff inside the clipboard:**

```bash
pbpaste | git apply
```