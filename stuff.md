# git screwery

## begin

```bash
git init --bare <repo>
```

Creates bare repository. No worktree attached by default. `<repo>` will be
referred to as `.git` for brevity.

## worktrees

Worktrees exist in `.git/worktrees/<name>` and have a few files:

- `HEAD`: ref where the worktree is currently "based on"
  - usually contains a commit id or a symbolic ref (such as `ref:
    refs/heads/master`)
- `commondir`: pointing to parent `.git` directory
  - usually just `../..`
- `index`: index file for the worktree, contains git's view of the worktree,
  including staged files
  - after creating `HEAD` and `commondir`, create `index` with `git --git-dir
    .git/worktrees/<name> read-tree HEAD`
- `gitdir`: supposed to point to the "real" `.git` of the worktree
  - must point to an existing file or `git gc` will remove the entire worktree
  - unless `locked` exists, in which case it is optional
  - worktrees without `gitdir` will not show up in `git worktree list`
- `locked`: marks worktree as "locked" to prevent `git gc` from removing it
  - must exist if `gitdir` does not exist or points to a nonexistent file
  - existence is enough, but it can also contain a locked reason

After worktree creation, simply set `GIT_DIR=.git/worktrees/<name>` to use its
index and HEAD for operations.

## misc

The `-z` argument can be given for `\x00` separated output in many cases,
bypassing quoting.

The "stage number" is usually only relevant while merging.

- `0`: normal operation
- `1`: merge-base file
- `2`: "ours" version
- `3`: "theirs" version

```bash
# list all objects and their types
git cat-file --batch-check --batch-all-objects

# manual commit from index
# -p: specifies parent commit
tree="$(git write-tree)"
git commit-tree -c user.name=iczero -c user.email=iczero@hellomouse.net commit-tree -m "message" -p HEAD "$tree"

# diff from rev to index
# -p to generate patch, --cached to use index only
git diff-index -p --cached "$rev"

# remove all unreachable reflog entries
git reflog expire --expire=now

# gc loose objects
git gc --prune=now

# write blob to object store
# -t: object type
# -w: write object to store
cat blob | git hash-object -t blob -w --stdin
# read blob from object store
# "blob" can be another type, but they will be returned raw
git cat-file blob "$id"

# read tree or commit from object store
# --full-tree: do not consider working directory
# -r: recurse children trees
# -t: show trees as they are recursed
# can use -z
git ls-tree --full-tree -r -t "$id"

# list files in index
# can use -z
git ls-files -c --full-name --stage

# copy tree to index
git read-tree "$rev"

# update index from stdin
# --add: add entries that don't already exist (default behavior is ignore)
# --remove: remove entries that are not specified (default behavior is ignore)
# --replace: in certain conflicts (for example file -> dir), replace entry
#   instead of failing
# --index-info: read index information from stdin
# can use -z
# can use same format as ls-files --stage
echo "100755 $id 0"$'\t'"$path" | git update-index --add --replace --index-info

# update ref
# with --stdin, supports transactions; see man page
git update-ref "$ref" "$id"

# update symbolic ref
git symbolic-ref HEAD refs/heads/master
```
