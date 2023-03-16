cd C:\dev\prj\training\git\git_mastering\master-git\03\demos\cookbook

### show remote url
```
git remote -v
```

### add file
```
git add menu.txt
git commit -m "add pizza"
```

### Show diff between two branches
```
git diff lisa master
```

### switch branch
```
git checkout lisa
```

### Add file to tracked files
```
atom Copyright.txt
git add Copyright.txt
```

### Remove file from tracked files
```
git rm --cached Copyright.txt
```

### Rename file in hard way
```
mv menu.txt menu.md
git add menu.md
git commit -m "rename file" -m "test rename file"
```

### Rename file simply way
```
git mv menu.md menu.txt
git commit -m "rename back file" -m "test rename file"
```

### Reset (hard <- reset index and workspace from repository data)
```
git reset --hard 500a0ba6892485621ff13bb6955812797fbb8d42
git reset HEAD
git reset --hard HEAD
```

### Put change in a reserved zone to continue dev after a high priority
```
touch recipes\guacamole.txt
git stash --include-untracked
git stash apply
git commit -m "new recipe"
git stash clear
git stash list
```

### Play with merge
```
git branch tomato
git checkout tomato
git commit -m "new ingredient"
git commit -m "add onion"
git merge tomato
git commit
```


atom recipes\README.txt
```
git add .
git reset HEAD menu.txt
git reset --hard HEAD menu.txt
git checkout HEAD menu.txt
git commit -m "to uppercase"
git branch hunks
git checkout hunks
git add --patch menu.txt
```

### show diff between workspace and index
```
git diff
```

### show diff between index and repository
```
git diff --cached
```

### restore ??
```
git restore menu.txt

git branch
git checkout nogood
git log
```

### Show details for commits
```
git show 7160d61
git show a87f2cc
git show HEAD^
git show HEAD~1
git show HEAD~2
git show HEAD~2^2
git show HEAD~2^^2
git show HEAD^^
```

### Show file history
```
git blame recipes\apple_pie.txt
```

### show diff between two commit or two branches
```
git diff HEAD HEAD~2
git diff nogood master
```

### Show history with details
```
git log --patch
```

### Show history that contains apples word
```
git log --grep apples --oneline
git  log -Gapples --patch
```

### Show all commit that are in master but not in nogood
```
git log nogood..master --oneline
```

### add data to the last commit
```
atom menu.txt
git add menu.txt
git commit -m "add salad"
atom recipes\caesar.txt
git statyus
git add recipes\caesar.txt
git commit --amend
git status
```

### rework on history that are not shared (pushed and used by other)
```
atom recipes\chicken.txt
git add recipes\chicken.txt
git commit -m "chicken"
git rebase -i origin/master
atom recipes\guacamole.txt
git add recipes\guacamole.txt
git rebase --continue
git log --graph --oneline --decorate
```

### show all information about all git actions (action that have influence)
```
git reflog
git show HEAD@{15}
git reflog refs/heads/master
```

### Invert a commit content
```
git filter-branch
git checkout lisa
git revert 5720fdf
git log --graph --decorate --oneline
```


more on stash :

- git stash
- git stash apply
- git stash list
- git stash apply + index-number

Manipulation of older stashes:
- git stash push -m 'stash-description'

Deleting stashes: 6:10
- git stash drop + index-number

Choose a stash to implement:
- git stash pop + index-number

When we don't need stashes anymore:
- git stash clear
