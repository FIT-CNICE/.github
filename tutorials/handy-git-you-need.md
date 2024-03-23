# Handy Things to Know about Git

Notes on the git workshop made by [ThePrimeagen](https://github.com/ThePrimeagen/fem-git).

<!-- mtoc-start -->

* [Init git folder](#init-git-folder)
* [Configuring your git experience](#configuring-your-git-experience)
* [Basics(what you use 80% of the time)](#basicswhat-you-use-80-of-the-time)
  * [View changes/history(`git log`)](#view-changeshistorygit-log)
  * [View and change config](#view-and-change-config)
* [Branch management](#branch-management)
  * [git branch](#git-branch)
  * [git merge](#git-merge)
  * [git rebase](#git-rebase)
  * [git reflog and cherry-pick: bring specific commit back to life](#git-reflog-and-cherry-pick-bring-specific-commit-back-to-life)
* [Work with remote repo](#work-with-remote-repo)
  * [Fetch and update](#fetch-and-update)
  * [git pull](#git-pull)
  * [git push](#git-push)
  * [git stash (hide your changes before pull in remote changes)](#git-stash-hide-your-changes-before-pull-in-remote-changes)
* [Squash commits](#squash-commits)
* [Solving conflicts](#solving-conflicts)
  * [git pull --rebase](#git-pull---rebase)
* [Spot bugs](#spot-bugs)
  * [Bisect bug search](#bisect-bug-search)
* [Revert changes](#revert-changes)
* [Meld commits](#meld-commits)
* [Tags: named locations within git history](#tags-named-locations-within-git-history)

<!-- mtoc-end -->

## Init git folder

```bash
# See all git commands
man git
# things to init your git before doing anything
git config --global --unset init.defaultBranch
git config --global rerere.enabled true
```

## Configuring your git experience

- all git config keys are in the shape of `<section>.<key>`
- `--global` flag will ensure you set this key value fro all future use of git
- `user.name` and `user.email` are the keys used in creating a commit tied to you
- to add a key value, use `git config --add --global <section>.<key> "<value>"`

```bash
# an example
git config --add --global user.nickname "Biblibop"
# an local example
git config --add stuff.name "boo"
```

- you can view any value of git config by `git config --get <key>`

## Basics(what you use 80% of the time)

- `git init` to make a folder git-trackable
- `git add <path-to-file | pattern>` will add zero or more files to the _index_(staging area)
- `git commit -m "<message>"` will commit what changes are present in the _index_
- `git status` will describe the state of your git repo

### View changes/history(`git log`)

- `git log --graph --decorate` to show history of repo in a graph format
- Commits exist in the `.git/objects` directory with the first 2 SHA letters as a directory, and the remaining 38 as a file(content in such a file is compressed and not readable)
- To check info of a commit, use `git cat-file -p <commit-sha>`, which shows

```
tree <sha-of-current-commit-the-git-tree>
parent <sha-of-parent-commit>
author <author-info>
committer <committer-info>
```

- If you `git cat-file -p <sha-of-current-commit-the-git-tree>`, you will find cached files(blobs) under current commit and its ancestors. Each blob is also assigned to a SHA code. Another `git cat-file -p` command will reveal what is inside each blob.

- `git log --graph --online` to see historic acyclic graph

You can view what were changed in the last x commits by 

```bash 
# replace x with any positive integer
git log -p --online -x
```
 where `-p` is a shortname for `--patch`.

### View and change config

All values to defined keys(locally) can be found at `.git/config`. Anything that is defined without specifying `--global` is defined locally in the git repo.

To show all the keys in section `<s>`, use

```bash
git config --list | grep s
```

To change value to a key, simply run

```bash
git config <section>.<key> "newVal"
```

To get a value to a key,

```bash
# get the key defined locally
git config --get <section>.<key>
# if there is a key under the same name but defined globally
git config --get --global <section>.<key>
```

To remove a section

```bash
git config --remove-section <section>
```

## Branch management

### git branch

All the branches' SHA are saved at `.git/refs/heads/<branchName>`.

To create a branch in a git folder:

```bash
git branch <branchName>
```

To switch to a branch:

```bash
git checkout <branchName>
# if the branch does not exist, using the following
git checkout -b <branchName>
```

To delete a branch,

```bash
git branch -d <branchName>
```

To force delete, replace `-d` with `-D`.

### git merge

To merge commit `<branch>` into the branch you are on, run

```bash
git merge <branch>
```

### git rebase

A simple visual explanation of how rebase works is shown below. Each node in the sketch is a commit.

```bash
# BEFORE REBASE
# feature branch          5---6
                          /
# main branch     1---2---3---4

# AFTER REBASE
# main branch     1---2---3---4---5---6
```

Assume your feature branch is named as `foo`, and your main branch is `main`. To rebase `foo` to `main`, simply run

```bash
git checkout foo
git rebase main
```

### git reflog and cherry-pick: bring specific commit back to life

Suppose you want to bring a deleted branch back to life. you can first use

```bash
git reflog -5
```

To print out the last 5 record of your HEAD movements, and find SHA for deleted commit. And then cherry-pick it to your current branch by

```bash
git cherry-pick <sha>
```

## Work with remote repo

A remote is just another git repo that is of the same project and has changes we may need. To add a remote the syntax is:

```bash
git remote add <name> <url>
```

where `url` can be either a https, or a path at your local computer. To check your current remote, use `git remote -v`.

A bit naming convention here: `origin` for the remote parent repo. If you forked your repo from the parent repo, name the forked repo `origin`, and the parent `upstream`.

### Fetch and update

We can fetch all the git state from our remote repository by executing `git fetch`. This **won't update** the current branches checked out, just where the `origin/*` has them set to.

The steps for fetching and updating to the remote HEAD are:

```bash
git remote add origin <url>
git fetch origin
# assume HEAD for remote repo is master
git merge origin/master
```

### git pull

If you just want the changes merged for you into your branch, then use `git pull` like the following:

```bash
git pull <remoteName> <remoteBranchName>
```

For the example above, we can simply do `git pull origin master`. If the current branch and the remote have diverged, the user needs to specify how to reconcile the divergent branches with --rebase or --no-rebase (or the corresponding configuration option in pull.rebase).

### git push

Opposite of `git pull`, and following the same syntax:

```bash
git push <remoteName> <remoteBranchName>
```

### git stash (hide your changes before pull in remote changes)

`git stash` will take every change tracked by git and store that result. Much like a commit, into the "stash". So stash is a STACK of temporary changes. It works like a commit, for example,

```bash
# make temporary changes
# add your changes like normal way
git add .
# stash it
git stash -m "stash xxx"
```

Stashes can be listed out

```bash
git stash list
```

To pop a stash,

```bash
# pop the latest stash
git stash pop
# pop the stash of index x
git stash pop --index x
# afterwards, you need to add/commit again
```

## Squash commits

Squash commits when you want to combine them into one to make history clearer. As an example, suppose you are at the tip of your branch, and you want to squash the past 3 commit into one.

To start the squash, run

```bash
git rebase -i HEAD~3
```

where `-i` means that you rebase interactively. The command above will open a text buffer starting with lines like the following:

```
pick <sha-code-1> <commit-msg-1>
pick <sha-code-2> <commit-msg-2>
pick <sha-code-3> <commit-msg-3>
```

To squash, simple remain the first line unchanged and replace the 2nd and the 3rd `pick` with `squash` or `s`. Save and quit the buffer to open the next buffer for you to setup commit message for the new commit:

```
# This is a combination of 3 commits.
# This is the 1st commit message:
<commit-msg-1>
# This is the 2nd commit message:
<commit-msg-2>
# This is the 3rd commit message:
<commit-msg-3>
```

To give a new commit message, simply delete all of the this lines and add your commit msg at the top. Finally, save and quit the buffer to finish your squash.

## Solving conflicts

It's very common that your pull from remote repo fails due to confilcts in the files. A conflict message generally looks like the following:

```
Auto-merging <filename>
CONFLICT (content): Merge conflict in <filename>
Recorded preimage for '<filename>'
Automatic merge failed; fix conflicts and then commit the result
```

But if you see the following message after `git pull <remote> <remoteBranch>`,

```
fatal: need to specify how to reconcile divergent branches
```

it indicates that your local repo is not up to date. So you should do the following

```bash
git fetch <remote> [ <remoteBranch> ]
git merge <remote>/<remoteBranch>
```

A failed merge results in unmerged paths to files in conflicts, and you can check what they are by using `git status`. Opening any one of them shows where the conflicts are. Each conflict is presented by `>>>>,======,<<<<`. For example,

```
<<<<<< HEAD
something at local
======
something at remote
>>>>>> sha-of-remote-branch
```

To resolve a conflict, simply choose a side and remove all `<`, `>`, and `=`. Afterwards, run `git add .` and `git status` to see "All conflicts fixed but you are still merging". Finally, run `git commit` to conclude merge.

### git pull --rebase

This way, we resolve conflicts and rebase. General steps are:

```bash
# first pull from remote
git pull <remote> <remoteBranch> --rebase
# the step might show filenames who have conflicts
# let's say there are file1 and file2 in conflicts
# we can choose which version we want to keep by
# using --ours or --theirs
git checkout --ours file1 # we use our version of file1
git checkout --theirs file2 # use remote version of file2
git add .
git rebase --continue
```

## Spot bugs

A classic problem is the following: somewhere in the last X commits something has gone wrong. To test if something has gone wrong takes several minutes or longer. So you want to find the relevant commit ASAP.

You can use `git log --graph --oneline` and look at those commit messages to take your best guesses. But, the following things might slow you down:

- poor commit messages
- cannot boil down the bug to a specific keyword
- or you simply don't know any keywords to narrow down your search

You can certainly use

```bash
git log --patch --grep "keyword"
```

to find patches whose commit message contains your "keyword". **But what if the occurences of "keyword" are everywhere?** Well, if you have a vague idea of which files bugs might appear, you can use

```bash
git log -p -- path/to/file1 path/to/file2 --
```

If your code is easy to understand, this way is more efficient. But **what could we do if the code is complex?** We need to **search through the repository for the offending commit that changes the code**.

### Bisect bug search

For people who don't know bisection method, you can read about it [here](https://pythonnumericalmethods.berkeley.edu/notebooks/chapter19.03-Bisection-Method.html), The argument `c` that causes `f(c)=0` in the article is analogous to the commit the bug first introduced in our case.

In the following example, we use the first commit of the repo and the current tip of `master` and find the problematic commit via `git bisect`:

```bash 
git bisect start 
# set current commit as bad 
git bisect bad
# find the SHA of a good commit
git log --oneline
# assume the sha is "benice8"
git bisect good benice8
# the command above tells you 
# how many revisions left to check 
# and jump the center commit
run your test # replace this command with the real one
# depending on the result of your test
# label current commit as good/bad
git bisect bad
```

Repeat the last two steps in the snippet above until you see 

```
<sha-of-a-commit> is the first bad commit
```

If your test is run by a binary, you can automate the bisect process by 

```bash 
git bisect start 
git bisect good <sha-of-good-commit>
git bisect bad
git bisect run path/to/your/binary [--options]
```

For `vitest`, you can replace the last command with 

```bash 
git bisect run ./node_modules/.bin/vitest --run
```

## Revert changes

Reverting is simple, you simply use

```bash 
git revert <sha-of-past-commit>
```

Notice that reverting might cause conflicts as well. If it happens, you should consider resolve the conflicts like what's been shown in the section [Solving conflicts](#solving-conflicts). And use `git revert --continue` to finish the reverting.

## Meld commits

We can meld past commits and alter history of a branch(change SHA of commits) by using 

```bash 
git reset --soft HEAD~x # replace x with any positive integer
```

The basics of soft is to reset the history to the point you want (HEAD~x) and the index and worktree will contain the changes whence you came (Our revert in this example). We could have used soft reset and went back several commits and we would have all their changes.

If you replace `--soft` with `--hard`, git will drop changes to the `index`(SHA) and worktree, meaning that **any work that is being tracked by git will be destroyed**. Use `--hard` with caution as it can destroy your work and it can be impossible to get back that work.


## Tags: named locations within git history

The best way to think about a tag is a branch that cannot be changed. It can only be deleted. Basic tag commands are: 

```bash 
git tag # list all the tags
git tag <name> # create
git tag -d <name> # delete
git checkout <tagname> # to checkout a tag
```

You can push tags to a remote via `git push --tags` and pull via `git pull --tags`. Note that `--tags` will fetch branches as well as tags.
