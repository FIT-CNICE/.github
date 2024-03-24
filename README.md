# FIT C-NICE

Center for Networked Intelligent Components and Environments at Foxconn Interconnect Tech. Inc.

## Tutorials

Notes on public workshops can be found at [tutorials](./tutorials).

## How to Contribute

You can only contribute to this repo if you are one of the collaborators. Because this is a public repo, please follow the steps below before you make any contributions:

1. clone this repo `git clone https://github.com/FIT-CNICE/.github.git`
2. create a **PRIVATE** repo on Github website under your username

- your repo should named as `fit-github-downstream`
- url to your repo should look like `https://github.com/<your-username>/fit-github-downstream.git`

3. in your local cloned folder:

- create a new private branch `git checkout -b private`
- set up **2 remote repo** by

```bash
git remote add origin https://github.com/FIT-CNICE/.github.git
git remote add private https://github.com/<your-username>/cnice-github-downstream.git
```

- at `private` branch, get up-to-date with the main branch `git pull origin main`
- you should now make changes **ONLY** on your `private` branch
- push all your commits to your own private repo `git push private main`

4. before pushing to `origin main`

- at your `private` branch, get up-to-date with the public main by `git pull origin main`
- resolve conflicts after `git pull` and make a new commit
- at your `private` branch, create a new branch called `pre-release`
  - if you already have `pre-release`, `git merge private` at `pre-release`
- at `pre-release` delete everything that's not supposed to be public
- create a new commit at `pre-release` with msg starting with **"release to main:"**

5. create pull request to the public main

The public main branch is protected, and you can only create pull request to it. At `pre-release`, 

```bash 
gh pr create -a "@fit-sizhe" --title "release to main: description of ur pr"
```

Because you have set up 2 remote repositories, the command above will ask you to choose one of them to push. You can skip adding message body.
