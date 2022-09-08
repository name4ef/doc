### adding tags
```sh
git tag -a v1.0.2 -m "version 1.0.2"
git push --follow-tags
```

### edit pushed commit
```sh
git commit --amend
git push --force origin hotfix
```

### delete pushed commit
```sh
git reset --hard HEAD^
git push origin hotfix --force
```

### work with submodules
```sh
# Creating
git submodule add git@somehost:somelib lib/somelib
git submodule init
git submodule set-branch --branch gentoo lib/somelib # if it need
git add .gitmodules lib/somelib; git commit; git push

# Cloning
git clone git@somehost:someproject; cd someproject
git submodule init; git submodule update
```
[1]: https://chrisjean.com/git-submodules-adding-using-removing-and-updating/

### edit commit (with author)
```sh
git checkout 03f482d6
git commit --amend --author "New Author Name <New Author Email>"
# now we have a new commit with hash assumed to be 42627abe
git checkout master # checkout the original branch
git replace 03f482d6 42627abe
git filter-branch -- --all # rewrite all future commits based on the replacement
git replace -d 03f482d6 # remove the replacement for cleanliness
git push --force-with-lease
```

### make patch from changes
```sh
git diff path/to/one/or/more/files > 005-name.patch

# for apply patch:
patch -p1 < 005-name.patch

# for revert changes:
patch -p1 -R < 005-name.patch
```

delete remote branch: `git push -d origen sambook`
