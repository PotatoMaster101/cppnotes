# Merge Conflict

- [Gist](#gist)
- [Basic Routine](#basic-routine)

## Gist
Resolve merge conflict using `git mergetool`, once done use `git commit`.

## Basic Routine
1. Make sure main branch is up to date (commonly named `main` or `master` or `develop`)
```
$ git checkout main
$ git pull origin main
```
2. Merge feature branch into main branch
```
$ git merge new_feature
```
3. Resolve merge conflicts using merge tool
```
$ git mergetool
```
4. Once all resolved, commit again to save merge and update commit message
```
$ git commit
```
