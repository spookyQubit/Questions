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
git checkout -b feature/new-questions upstream/HEAD
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
    * [recommended] on Github
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

## Migrate from SVN

* Based on [Git Matters](https://cms.prod.bloomberg.com/team/display/cl/Git%20Matters#GitMatters-AStudy:lib/metraweatherdevgitPorting)
* steps from Denis Chekhlov
    * create empty repo
    ```
    $ git init boom_cfg
    $ cd boom_cfg
    ```
    * specify repo to migrate
    ```
    # this will drop you into a special shell
    $ devgit-config . refs/config/svnsync
    $ echo 'svn+ssh://devsvn/fiet_base' > repository
    $ echo 'trunk/mmo/mmktserv/cfg' > branches
    $ echo '/**' > files
    $ exit
    ```
    * extract all the commits
    ```
    $ devgit-svnsync
    # check if you are on the master branch
    $ git branch -a
    # if not do
    git checkout -t -b master svn/trunk/mmo/mmktserv/cfg
    ```
    * create a repo on GitHub
        * in the top right hand side, next to your login name, there's a '+'.
        * create a new repository, give 'questions' name
        * DO NOT initialise a README.md
        * copy the clone link which will `https://github.com/spookyQubit/Questions.git`
    * add origin and push
    ```
    $ git remote add origin git@bbgithub.dev.bloomberg.com:ficc/boom_cfg.git
    $ git push -u origin master:master
    ```
    * if large files, follow [Removing sensitive data from a repository](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)
    * rename SVN dir
    ```
    cd mmo/mmktserv
    svn mv cfg cfg.old
    # explicitly specify old and new names of deprecated dirs in commit
    svn commit cfg cfg.old -m "migrated cfg to bbgh:ficc/boom_cfg"
    ```
    * eventually, remove SVN dir
    ```
    cd mmo/mmktserv
    svn up
    svn rm cfg.old
    svn commit cfg.old
    ```

## Troubleshooting

* `hub help`
* check repo cfg: `git config --local --list`
* `fatal: git checkout: updating paths is incompatible with switching branches.`
```
# check if remote branch is tracked or new
git remote show origin
# fetch remote branches
git remote update
git fetch
```
* Fix `arc diff` mess
```
# remove last phab commit
git reset --hard HEAD^
git pull origin develop
git pull origin feature/edit-curve
```
* Remove stale remote branches
```
# run dry-run prune of remote
git remote prune origin -n
# prune
git remote prune origin
```

## Issues

* Uncheck [Automatically watch repositories](https://github.com/spookyQubit/Questions/settings/notifications)
* Select [Unwatch all](https://github.com/watching)
* or [Unwatch](https://github.com/watching)

### Training
```

# get rid of local commits and sync with remote branch
git reset --hard origin/master

# delete a remote branch
git push origin :metadata
# delete local branch
git branch -d metadata

# cherry pick a commit from a branch to master
# (use only if you need only 1 commit to production w/o merging eth from the
# branch)
git cherry-pick 4823d83

# or do a merge of a local branch

# stash files in a local storage
# (avoid, prefer branches and commit to the branch)
git stash

# retrieve stash
git stash apply
```

## OLD

### Tutorial 1

```
git branch feature
git checkout feature
# or do both at the same time
git checkout -b feature
# switch to master
git checkout master
# merge branch with master
git merge feature

# show all branches (even remote ones)
git branch -a

# add Final tag when ready for evaluation
git tag -a Final -m 'Ready for evaluation'
# display tags
git tag -n

# push tags, map local to remote branches (-u)
git push --tags -u origin tutorial_1

# test if submitted repo contains all required files
cd ~/training/tmp/
git clone --branch Final https://github.com/spookyQubit/Questions
```

* Showing files
    * `ls`: working dir
    * staging area only
    ```
    git ls-files
    # show files in a specific branch
    git ls-files --name-only tutorial_1
    ```
    * `git show`: show latest commit (diff)

* Changing tags
    * Force creation of a new tag
    ```
    # devgit forbids changing the meaning of a tag which has been pushed
    # change tags only if not pushed
    # tag a later commit with the same tag by forcing it
    git tag -f -a Final -m "Ready for evaluation"
    ```
    * Correct misspelled tag
    ```
    # copy to new tag
    git tag BugFix BugFxi
    # verify ref to same hash
    git show-ref --tags
    # delete
    git tag -d BugFxi
    ```
* Modify a commit
    * Incorrect message when not pushed
    ```
    git commit --amend -m "Correct message"
    ```
    * Incorrect message when pushed (not recommended, only when no pull by
      others)
    ```
    git commit --amend -m "Correct message - modified"
    git push --force origin tutorial_1
    ```
* Recover a file revision
    * `git checkout -- file.cpp`: if not staged
    * `git checkout <SHA1> file.cpp`: if not staged?
    * `git reset file.cpp`: if staged
* Diff-ing
    * `git diff`: not staged
    * `git diff HEAD`: not staged
    * `git diff --staged`: staged

### Bootcamp 1

```
printf hello | git hash-object -w --stdin
b6fc4c620b67d95f953a5c1c1230aaab5db5a1b0

# ouput contents of obj id-ed by hash
git cat-file -p b6fc
hello

# unstage added (but uncommitted) files
# cache == staging area == index
# blob == file
git rm --cached README
# delete actual file
rm README
# equivalent to
git rm -f README

# difference in all areas: staged, unstaged, untracked
git diff
# difference in staging area only
git diff --cached

# short obj hash and commit msg
git log --oneline
# show branches too
git log --oneline --decorate
# show all (even from other branches which are ahead)
git log --oneline --decorate --all
# show graph
git log --oneline --decorate --graph

# what is HEAD pointing to (which branch)
cat .git/HEAD

git branch --list
# show remote branches too
git branch --list --all
# show only remote branches
git branch --list -r
git branch mywork
# base new branch on HEAD (assumed by default)
git branch mywork HEAD
git checkout mywork
git checkout master

# base new branch on remote master
git branch feature/new-questions origin/master
# push branch to origin
git push origin feature/new-questions

git remote -v
# fetch objs from origin (if there is one)
git fetch origin
```
