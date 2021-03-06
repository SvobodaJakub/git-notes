# git-notes

Personal goal-driven notes about git that I write as I learn

**more info**

* http://ohshitgit.com/
* http://justinhileman.info/article/git-pretty/
* https://github.com/k88hudson/git-flight-rules
* http://think-like-a-git.net/sections/graphs-and-git.html - a git repo is a graph (also, the learning resources about git are also a graph since they list additional resources and so on : )
* https://stackoverflow.com/questions/5097456/throw-away-local-commits-in-git


**detailed git log**

`git log -v --source  --log-size  --notes --graph --name-status`


**detailed git log including other branches**

`git log -v --source --full-history --log-size --all --notes --graph --name-status`


**how to throw away local commits in master and get what is on origin master (get what is on the server) - ver 1**

(use branch name instead of "master" for use with another branch)
```
git clean -d -x --force
git reset --hard
git checkout master
git reset --hard origin/master
git pull
```

**how to throw away local commits in master and get what is on origin master (get what is on the server) - ver 2**

(use branch name instead of "master" for use with another branch)
```
git checkout [any commit ID, doesn't matter, but not master]
git reset --hard origin/master
git branch -D master
git checkout origin/master -b master
git checkout master
```

**how to throw away local commits in master and get what is on origin master (get what is on the server) - ver 3 - useful when history has been rewritten on the server**

(use branch name instead of "master" for use with another branch)
```
git clean -d -x --force
git reset --hard
git checkout master
git fetch
git reset --hard origin/master
git checkout master
```

**how to revert specific files/folders from a specific commit**

`git checkout 465f051f04d7dd4d400abc6f16c63e64e1d54003 some/folder/foobar_*/*/metadata.txt`

it is then added to both the working copy and to the staging area


**how to apply an arbitrary commit on top of the current branch/commit**

`git cherry-pick 465f051f04d7dd4d400abc6f16c63e64e1d54003`

it applies the whole commit (as a diff) on top of the current branch/commit
the staging area is moved to the next commit, so from the user's point of view, nothing is added to the staging area and if git status showed no edited files before the cherry-pick, it will show no edited files after the cherry-pick
the commit is then ready to be pushed


**how to move a commit to a different branch (e.g. after committing to the wrong branch)**

only applicable if not pushed or if it is possible to force push
if more than one commit, make sure to move the commits in the right order
* `git log` and note the ID
* `git reset --hard HEAD~`  # revert the current branch to the state before the last commit; alternatively use a commit ID instead of "HEAD~"
* `git checkout` to the correct branch (or `git checkout -b branchname` to create a new branch)
* `git cherry-pick COMMIT_ID`  # re-apply the commit that you wanted to move


**what is git head**

it is the "current branch"

You are currently in a specific branch X, and the branch remembers which commit C is the last commit in the branch X. The head is just the information that C is the head of X.

https://stackoverflow.com/questions/2304087/what-is-head-in-git#2304106


**git checkout vs git reset**

`git checkout [branch or tag or commit]`
* sets the current branch to what is specified, also changes the local copy of the files to that

`git reset --soft [branch or tag or commit]`
* doesn't set the current branch, but changes what the current (local copy of the) branch thinks it ends with
* this also changes the whole history of commits in the current (local copy of the) branch because the last commit has a parent and that has a parent, etc. and this lineage can be completely different from what it was before git reset
* doesn't change anything else - the local copy of the files is untouched, the Index (stage, proposed next commit snapshot) is untouched, so git status shows a bunch of staged changes as if the current branch has always been the [branch or tag or commit we just reset to] and we've made such changes to the local files on disk so that they represent the state that was there before we did the git reset (because the local files on disk are still untouched by git reset!) and staged them

`git reset --mixed [branch or tag or commit]`
* same as `git reset --soft` with the difference that the Index is also changed to the state of [branch or tag or commit], so the difference between [branch or tag or commit] and the local files on disk is completely unstaged
* `git reset --mixed` is the default git reset

`git reset --hard [branch or tag or commit]`
* same as `git reset --mixed` with the difference that the local files on disk are also brought to reflect the state of [branch or tag or commit]
* those local files on disk that are not tracked by git are untouched (to clean them, you can use `git clean -d -x --force`)



**reset demystified**

https://git-scm.com/blog/2011/07/11/reset.html

* there are three trees
  * The HEAD - last commit snapshot, next parent
  * The Index - proposed next commit snapshot
  * The Working Directory - sandbox
* git reset --soft - changes what HEAD points to
  * undoes the last commit (locally)
  * With git reset --soft HEAD~ "you are moving the branch back to where it was without changing the Index (staging area) or Working Directory. You could now do a bit more work and commit again to accomplish basically what git commit --amend would have done."
* git reset / git reset --mixed - changes what HEAD points to, "update the Index with the contents of whatever tree HEAD now points to so they're the same"
  * this "undid your last commit, but also unstaged everything. You rolled back to before you ran all your git adds AND git commit."
* git reset --hard - also rewrites the working directory
* git reset --hard doesn't change what branch HEAD points to, but changes the branch, that is HEAD set to point to, to a different commit (that's invasive); git checkout sets HEAD to point to a different branch (or commit), but doesn't change what any branch points to.


**how to force garbage collection**

https://stackoverflow.com/questions/14991916/garbage-collect-commits-in-git#14995269
https://rtyley.github.io/bfg-repo-cleaner/

```
rm -rf .git/refs/original/*
git reflog expire --all --expire=all --expire-unreachable=all
git repack -A -d
git prune
git gc --prune=all  --aggressive
```

if git gc consumes all memory and crashes
https://stackoverflow.com/questions/3095737/is-there-a-way-to-limit-the-amount-of-memory-that-git-gc-uses

```
git config --global pack.windowMemory "50m"
git config --global pack.packSizeLimit "100m"
git config --global pack.threads "1"
```


**when git refuses to checkout a remote branch**

use `checkout --track`

```

$ git fetch -a
$ git checkout custom_remote/a_branch_that_is_on_remote_but_not_locally
Note: checking out 'custom_remote/a_branch_that_is_on_remote_but_not_locally'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at b6c5b74... 
$ git checkout master
Previous HEAD position was b6c5b74... 
Switched to branch 'master'
Your branch is up-to-date with 'custom_remote/master'.
$ git checkout --track custom_remote/a_branch_that_is_on_remote_but_not_locally
Branch a_branch_that_is_on_remote_but_not_locally set up to track remote branch a_branch_that_is_on_remote_but_not_locally from custom_remote.
Switched to a new branch 'a_branch_that_is_on_remote_but_not_locally'

```


**sparse checkout**

```
$ git init ; git config core.sparseCheckout true
$ echo "path_you_want" >> .git/info/sparse-checkout
$ git remote add origin git@example.com:XXX/YYY
$ git pull origin master
```


**limited-depth clone**

Clones only the specified number of commits.

```
$ git clone --depth 1 gitrepo
```


