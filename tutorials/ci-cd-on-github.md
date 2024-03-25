# Best Practices of CI/CD on Github

## Branch protection

Something you should always check:

- Require a pull request before merging
  - Dismiss stale pull request approvals when new commits are pushed
- Require status checks to pass(only push to the branch if tests are passed)
  - Require branches to be up to date before merging
  - Require passing workflows(checks) before merging

Of course, none of these rules will be enforced if you organization is not upgraded to "Githu Team", and you are working with private repo.

## Caching in Github workflow

Including [ `action/cache` ](https://github.com/actions/cache) can generally speed up your workflow. But, by the time of writing this note, it's still unknown to the author the best practice of making `bun` work with `actions/cache`. A working method is:

- generate `yarn.lock` from `bun.lockb` by `bun install --yarn`
- install `synp` by `bun install -g synp`
- convert `yarn.lock` to `package-lock.json` by `synp --source-file yarn.lock`
- add the following at each step of your github workflows:

```yml
- uses: actions/cache@v4
  name: cache assets
  id: npm-cache
  with:
    path: ~/.bun/install/cache
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```
