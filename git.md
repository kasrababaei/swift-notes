# Git

- [Git](#git)
  - [For changing the output to be Terminal instead of pager](#for-changing-the-output-to-be-terminal-instead-of-pager)
  - [For a list of files to be pushed, run](#for-a-list-of-files-to-be-pushed-run)
  - [For a list of changed files that are staged/cached](#for-a-list-of-changed-files-that-are-stagedcached)
  - [For the code diff of the files to be pushed, run](#for-the-code-diff-of-the-files-to-be-pushed-run)
  - [To see full file paths of the files that will change, run](#to-see-full-file-paths-of-the-files-that-will-change-run)
  - [To stop tracking a file you need to remove it from the index](#to-stop-tracking-a-file-you-need-to-remove-it-from-the-index)
  - [To see list of commits without push](#to-see-list-of-commits-without-push)
  - [To remove a git commit which has not been pushed](#to-remove-a-git-commit-which-has-not-been-pushed)
  - [To clone a repo (with a branch)](#to-clone-a-repo-with-a-branch)
  - [To undo ‘git add’ before commit](#to-undo-git-add-before-commit)
  - [To discard all local uncommitted changes](#to-discard-all-local-uncommitted-changes)
  - [To discard all unstaged files in current working directory use](#to-discard-all-unstaged-files-in-current-working-directory-use)
  - [To remove untracked files](#to-remove-untracked-files)
  - [To compare local repo with remote repo](#to-compare-local-repo-with-remote-repo)
  - [To list branches](#to-list-branches)
  - [To rename a branch](#to-rename-a-branch)
  - [To switch to another branch](#to-switch-to-another-branch)
  - [To merge other branches with the master branch](#to-merge-other-branches-with-the-master-branch)
  - [To create and switch to a new branch](#to-create-and-switch-to-a-new-branch)
  - [To undo a pushed commit without any trace](#to-undo-a-pushed-commit-without-any-trace)
  - [To merge/squash the last three commits into one](#to-mergesquash-the-last-three-commits-into-one)
  - [To start a new feature branch](#to-start-a-new-feature-branch)
  - [To rebase the current branch with develop branch](#to-rebase-the-current-branch-with-develop-branch)
  - [To add new changes to a not pushed commit](#to-add-new-changes-to-a-not-pushed-commit)
  - [To see the difference of the last commit](#to-see-the-difference-of-the-last-commit)
  - [To sign all the commits automatically](#to-sign-all-the-commits-automatically)
  - [Fetch and checkout a remote branch](#fetch-and-checkout-a-remote-branch)
  - [Change the base of the branch](#change-the-base-of-the-branch)
  - [Compare local branch with the same branch on remote](#compare-local-branch-with-the-same-branch-on-remote)
  - [Cherry picking](#cherry-picking)
  - [List remote branches](#list-remote-branches)
  - [See only files that are having conflicts](#see-only-files-that-are-having-conflicts)
  - [To Delete every branch that starts with bugfix](#to-delete-every-branch-that-starts-with-bugfix)
  - [To stash changes with a name](#to-stash-changes-with-a-name)
  - [To change remote upstream](#to-change-remote-upstream)
  - [To unset upstream after renaming a branch that has been pushed to the remote](#to-unset-upstream-after-renaming-a-branch-that-has-been-pushed-to-the-remote)
  - [Copy diff into clipboard](#copy-diff-into-clipboard)
  - [Apply the diff inside the clipboard](#apply-the-diff-inside-the-clipboard)
  - [Transferring Keys](#transferring-keys)

Git is a distributed version control system (VCS) that allows developers to track
changes in source code, collaborate on software development, and manage
projects of any size efficiently. Created by Linus Torvalds in 2005. A good
source of truth is the documentation on [Git-SCM](https://git-scm.com/doc).

## For changing the output to be Terminal instead of pager

[As of Git 2.16, the default behavior changed.](https://github.com/git/git/blob/master/Documentation/RelNotes/2.16.0.txt#L85-L88)
It uses [LESS](https://en.wikipedia.org/wiki/Less_(Unix)) to display the output.
In order to change that and print out the output in Terminal, need to disable pager:

```bash
git config --global pager.branch false
```

But that only disables it for `git branch`. If the goal is to disable
it for every command, can simply add the following to shell:

```bash
export LESS="$LESS -F -R -X"
```

- `F` (or `--quit-if-one-screen`): Exits immediately if the output
  fits on one screen instead of waiting for user input.
- `R` (or `--raw-control-chars`): Output "raw" control characters.
  The `-R` option in less is used to interpret raw control characters for ANSI colors.
  When `-R` is enabled, less preserves and displays ANSI colors instead of showing
  raw escape sequences.
- `X`: Disables clearing the screen when less exits, so the content remains visible.

## For a list of files to be pushed, run

```bash
git diff --stat --cached [remote/branch]
# example: git diff --stat --cached origin/master
```

## For a list of changed files that are staged/cached

```bash
diff --cached --name-only
# example output:
# Sources/Modules/UIViewController.swift
# Sources/Modules/ViewModel.swift
```

## For the code diff of the files to be pushed, run

```bash
git diff [remote repo/branch]
```

## To see full file paths of the files that will change, run

```bash
git diff --numstat [remote repo/branch]
```

## To stop tracking a file you need to remove it from the index

```bash
git rm --cached <file>
git rm -r --cached .
```

## To see list of commits without push

```bash
git log origin/master..master
```

## To remove a git commit which has not been pushed

> [!WARNING]
>this will revert local changes.

```bash
git reset --hard COMMIT-ID
# example: git reset --hard eb27bf26dd18c5a34e0e82b929e0d74cfcaab316
```

## To clone a repo (with a branch)

```bash
git clone URL
git add .
git commit -m “message”
git push
```

## To undo ‘git add’ before commit

```bash
git reset <file>
git reset
```

## To discard all local uncommitted changes

```bash
git checkout .
```

## To discard all unstaged files in current working directory use

```bash
git checkout -- .
```

## To remove untracked files

```bash
# Remove all untracked files and directories.
# `-f` stands for force, It is required to actually remove the files
# `-d` stands for remove directories. Useful when have untracked directories that also want to remove.
git clean -fd
# preview changes
git clean -nd
```

## To compare local repo with remote repo

```bash
git diff master
```

## To list branches

```bash
git branch -a
```

## To rename a branch

```bash
git branch -m <old name> <new name>
```

## To switch to another branch

```bash
git checkout branch-a
```

## To merge other branches with the master branch

```bash
git merge branch-a
```

## To create and switch to a new branch

```bash
git checkout -b new-branch-name
```

## To undo a pushed commit without any trace

```bash
git reset <previous label or sha1>
git commit -am "Some commit message"
git push -f <remote-name> <branch-name>
```

## To merge/squash the last three commits into one

```bash
git reset --soft HEAD~3 # [~ is called tilde]
git commit -m "New message for the combined commit"
```

## To start a new feature branch

```bash
git flow feature start ch<story number>/<branch name>
```

Now, start committing on your feature. When done, use:

```bash
git flow feature finish ch8376/get-directions
```

## To rebase the current branch with develop branch

```bash
git rebase -i origin/develop
:wq
git push -f
```

## To add new changes to a not pushed commit

```bash
git add .
git commit --amend --no-edit
```

## To see the difference of the last commit

```bash
git diff HEAD^
```

## To sign all the commits automatically

```bash
add the following to `.gitconfig`:
  [commit]
    gpgsign = true
```

## Fetch and checkout a remote branch

```bash
git fetch
git checkout -b <branch> origin/<branch>
```

## Change the base of the branch

```bash
git rebase --onto new-base-branch current-base-branch
```

## Compare local branch with the same branch on remote

```bash
git fetch
git diff @{upstream}
```

## Cherry picking

```bash
git cherry-pick <commit-hash>
```

## List remote branches

```bash
git branch -r
```

## See only files that are having conflicts

```bash
git diff --name-only --diff-filter=U
```

## To Delete every branch that starts with bugfix

```bash
git branch --list ‘bugfix*’ | xargs -r git branch -d
```

## To stash changes with a name

```bash
git stash push -m “some name”
```

## To change remote upstream

```bash
git remote rename origin upstream
git remote add origin git@github.com:[new repo]
git config --get remote.origin.url // prints out the new remote upstream
```

## To unset upstream after renaming a branch that has been pushed to the remote

```bash
git branch --unset-upstream
```

## Copy diff into clipboard

```bash
git diff | pbcopy
```

## Apply the diff inside the clipboard

```bash
pbpaste | git apply
```

## Transferring Keys

Make sure only the owner of the file has full read and write access to it

```bash
chmod 600 id_rsa
```

To add a passphrase to an existing private key:

```bash
ssh-keygen -p -f id_rsa
```

To add add the key to the SSH Agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

To test the connection

```bash
ssh -T git@github.com
```

To install GNUPG

```bash
brew reinstall gnupg
```

Assuming the GPG private key is exported into a file named `private.key`,
to import the key:

```bash
gpg --import ~/Downloads/private.key
```

If it's not exported then:

```bash
gpg --export-secret-keys --armor KEY_ID > filename.asc
```

The `KEY_ID` can be found by running:

```bash
gpg --list-secret-keys --keyid-format=long
```

Example Output:

```bash
[keyboxd]
---------
sec   ed25519/0000RANDOM5F7543C 2023-02-10 [SC]
      0000000RANDOM0000EEA5A97521B2F1000000000
uid                 [ unknown] FULL NAME <email@gmail.com>
ssb   cv25219/RANDOMC000000000 2023-02-10 [E]
```

where `0000RANDOM5F7543C` is the key.

To set git config to use this key for singing:

```bash
git config --global user.signingkey RANDOMC000000000
git config --global commit.gpgsign true
git config --global user.name "FULL NAME"
git config --global user.email "email@gmail.com"
```

To tell Git exactly what to use for GPG:

```bash
git config --global gpg.program $(which gpg)
```
