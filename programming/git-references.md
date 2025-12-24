# Git References

[[references]]

## SSH

### Generate SSH key

```bash
# Generate ssh key in ~/.ssh/id_ed25519
ssh-keygen -t ed25519 -C "<email>"
# Generate ssh key in <ssh_key_path>
ssh-keygen -t ed25519 -C "<email>" -f <ssh_key_path>
```

### SSH config for multiple keys

Create `~/.ssh/config`

```
# Default GitHub account
# URL: git@github.come:<user>/<repo>.git
Host github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

# Work override
# URL: git@github-work:<org>/<repo>.git
Host github-work
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519_work
```

### Adding SSH key to the GitHub

1. Go to: https://github.com/settings/keys
2. `SSH keys > New SSH key`
3. Copy the SSH public key
	- `pbcopy < ~/.ssh/id_ed25519.pub`
4. Paste in `Key` field
5. Click `Add SHH key`

### Signing commits

https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits

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

