## golden rule of rebasing
> never use git rebase on public branches


generally good to use rebase only on your local branch to clean up 

```
git checkout feature
git rebase -i HEAD~3
```

do not use rebase after submitting a pull request, this will screw up the history of the PR


if you and another dev are working on a feature branch, and yours both split off
you can `git pull --rebase` so that his changes are on top / after yours, and instead of making a merge commit

## when pushing upstream changes to main
1. rebase your feature onto the tip of the main branch
2. use `git merge` to integrate feature into the main code base

- essentially, perform a rebase before the merge into public branch 
- can perform rebase in temp branch to make sure the commit history is correct

#git #commits #rebase