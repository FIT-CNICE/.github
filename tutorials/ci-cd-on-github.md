# Best Practices of CI/CD on Github

## Branch protection

Something you should always check:

- Require a pull request before merging
  - Dismiss stale pull request approvals when new commits are pushed
- Require status checks to pass(only push to the branch if tests are passed)
  - Require branches to be up to date before merging
  - Require passing workflows(checks) before merging

Of course, none of these rules will be enforced if you organization is not upgraded to "Githu Team", and you are working with private repo.
