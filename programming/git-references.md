# Git References

[[references]]

## Basic

Discard all submodule changes
```bash
git submodule update --init --recursive --force
```

Add original repo for the forked repo
```bash
git remote add upstream https://github.com/<original-owner>/<original-repo.git>
```
