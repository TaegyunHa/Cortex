# Git References

[[references]]


## Fork

Add original repo for the forked repo

```bash
git remote add upstream https://github.com/<original-owner>/<original-repo.git>
```

## submodule

Add submodule

```bash
git submodule add <repo_url> <path>
```

Checkout all nested submodules

```bash
git submodule update --init --recursive
```

Discard all submodule changes

```bash
git submodule update --init --recursive --force
```

## worktree

Simple worktree add

```bash
git worktree add <path>
```

- Creates a new branch whose name is the final component of `<path>`

Add worktree branch

```bash
git worktree add -b <branch> <path> <remote>/<branch>
```

Remove worktree

```bash
git worktree remove [-f] <worktree>
```

