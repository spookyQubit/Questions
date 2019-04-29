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
