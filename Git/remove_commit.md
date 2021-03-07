# Remove Commits

- [Gist](#gist)
- [Using `git reset`](#using-git-reset)
- [Using `git rebase`](#using-git-rebase)
- [Deleting `.git` Folder](#deleting-git-folder)

## Gist
Numerous ways to remove commits, commonly use `git reset` or `git rebase`.

**Removing commit is dangerous. Ask team before doing it.**

**Removing commit is dangerous. Ask team before doing it.**

**Removing commit is dangerous. Ask team before doing it.**

## Using `git reset`
To delete commit 4 and 5:
```
$ git log --oneline
fce6c6c (HEAD -> main) Commit 5
fe31dbf Commit 4
870f02c Commit 3
1a5f686 Commit 2
ce19e75 Commit 1
```

Hard reset git to commit 3 (`870f02c`):
```
$ git reset --hard 870f02c
HEAD is now at 870f02c Commit 3
$ git log --oneline
870f02c (HEAD -> main) Commit 3
1a5f686 Commit 2
ce19e75 Commit 1
```

This works the best if you want to remove the last N commits.

## Using `git rebase`
To delete commit 2 and 3:
```
$ git log --oneline
5d9eca3 (HEAD -> main) Commit 5
a146180 Commit 4
870f02c Commit 3
1a5f686 Commit 2
ce19e75 Commit 1
```

Delete with `git rebase`:
```
$ git rebase -i ce19e75
```

Text editor will show up with commits to pick. Delete the commit 2 and 3 lines and pick the other commits:
```
pick a146180 Commit 4
pick 5d9eca3 Commit 5
```

Close editor and commits 2 and 3 will be removed:
```
$ git log --oneline
3c1b93e (HEAD -> main) Commit 5
ba8f781 Commit 4
ce19e75 Commit 1
```

**Note**: sometimes there might be a merge conflict, in that case resolve the merge conflict and run `git rebase --continue`.

## Deleting `.git` Folder
**Not recommended.** To delete all history, just remove `.git` folder and `init` again:
```
$ rm -rf .git
$ git init
```
