
```
# start an issue
git checkout main && git fetch upstream && git merge --ff-only upstream/main
git checkout -b <type>/<topic>-<issue>

# work
git add <files> && git commit -s -m "<type>(<scope>): <summary> (#<issue>)"

# before pushing
git fetch upstream && git rebase upstream/main && go test ./... -race -count=1
git push -u origin <branch>
```