## edit config
```
git config --system --edit
git config --global --edit
git config --local --edit
```

## Moving

* to move a repo from one org to another
    * go to Settings of a repo and Transfer to a new org
    * on dev mach. change remote
    ```
    git remote set-url origin <url of new repo>
    ```
* to move a directory in a repo to a brand new repo (preserving commit history) use [git filter-branch](https://help.github.com/articles/splitting-a-subfolder-out-into-a-new-repository/)

## A simple workflow

* based on [a simple git branching model](https://gist.github.com/chalasr/fd195d83a0a01e4291a8)
* avoid
    * `git pull` when synch-ing with `origin/master` before PR, instead do
        * `git fetch origin master`
        * `git rebase origin/master`
    * `git cherry-pick`: instead do `git rebase` or `git rebase -i`
    * forking (manually): if you own the code, just create a feature branch
    * release/bugfix/long-lived branches: use short-lived branches & tag
    * squashing all PR commits into one
* workflow
```
# clone org repo
git clone ficc/ficscila

# create a feature/bugfix/issue branch
git checkout -b feature/new-feature

# edit
$EDITOR file

# commit
git add .
git commit -m "new feature"

# fetch remote orgin
git fetch origin

# rebase on top of remote changes
git rebase origin/master
# or interactively squash commits into logical chunks
# (but don't squash all commits into one)
git rebase -i origin/master

# push feature branch
git push origin feature/new-feature
# if you have rebased due to a merged PR or squashing, force push feature branch
git push origin feature/new-feature --force

# submit a pull request on GitHub
# assign code reviewers
# fix code based on feedback
# merge pull request

# pull remote origin
git checkout master
git pull

# delete the local branch
git branch -d feature/new-feature

# delete the remote branch
git push origin :feature/new-feature
# or
git push origin --delete feature/new-feature

# build a BPKG

# tag commit (annotated) with the BPKG version
git tag -a v1.5 9fceb02 -m "new feature 1.5"

# push tags
git push origin --tags
```
* using tags
```
# see all tags
git tag -n

# show specific tag
git show v1.5

# checkout branch from tagged commit
git checkout -b next-version v1.5
```
## Conflicts

* check [Getting out a of git mess](http://justinhileman.info/article/git-pretty/)
* pick a version
```
# local version
git checkout --ours .
# or remote version
git checkout --theirs .

# mark conflict files as merged
git add -u

# commit
git commit
```

## ~~A fork repo + PR workflow~~

* Fork repo
    * do manually
    ```
    git clone spookyQubit/questions
    git remote add upstream spookyQubit/questions
    git config --local --list
    git config remote.pushdefault origin
    git config branch.master.mergeoptions '--ff-only'
    git config --add remote.upstream.fetch '+refs/notes/*:refs/notes/*'
    git config --local --list
    ```
    * [recommended] verify remote
    ```
    git remote -v
    # should show:
    # origin  https://github.com/spookyQubit/Questions.git (fetch)
    # origin  https://github.com/spookyQubit/Questions.git (push)
    # upstream        https://github.com/spookyQubit/Questions.git (fetch)
    # upstream        https://github.com/spookyQubit/Questions.git (push)

    git pull
    ```
* Create feature/bug branch, check it out, and set to track upstream
```
git checkout -b feature/calcrt-proxy upstream/HEAD
```
* Do some work
```
git add .
git commit -m ""
```
* [recommended] Get latest upstream changes and rebase local commits (before submitting a pull
  request or if you need something committed after your branch was created)
```
git fetch upstream
git rebase upstream/master
```
* [optional] Resolve conflicts if any
```
git add .
git rebase --continue
```
* Push local branch to forked repo (origin)
```
git push -u origin feature/new-questions
```
* Submit a pull request
    * [recommended] on Github
    * or do manually: `git bb-pull-request`
* [optional] Squash all commits in the pull request (optional and only if > 1 commit)
```
git rebase -i HEAD~[# of commits in pull request]
# leave first commit in list as 'pick', change all the rest to 'squash'
# force push *feature branch* to your forked repo
git push origin feature/calcrt-proxy --force
```
* [optional] To undo a rebase gone wrong
    * find where the HEAD was before the rebase (the last commit before rebase)
    ```
    git reflog
    ```
    * reset where `N` is the HEAD commit number found in reflog
    ```
    git reset --hard HEAD@{N}
    ```
* Merge the pull request
    * [recommended] on BBGithub
    * or do manually
* [recommended] Sync changes with local forked repo
```
git checkout master
git pull
git push
```
* [recommended] Delete the local branch to avoid confusion
```
git branch -d feature/new-questions
```
* [recommended] Delete the remote branch
```
git push origin :feature/new-questions
# or
git push origin --delete feature/new-questions
```
