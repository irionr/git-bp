# git-bp

A `git` subcommand to prune local branches whose upstream has been
deleted, and remove stale remote branches on `origin`.

Defaults are safe: the script reports what it would delete; pass
`--delete` to actually do it. A handful of branch names are always
protected, even with `--delete`.

## Installation

Drop `git-bp` anywhere on your `PATH` and make sure it's executable.
`git` discovers any `git-<name>` executable on `PATH` and exposes it as
`git <name>`, so once the file is on `PATH` you can run it as `git bp`:

```sh
git clone https://github.com/irionr/git-bp.git
ln -s "$PWD/git-bp/git-bp" ~/bin/git-bp
```

## Usage

```
git bp [--delete] [-d|--days N] [-p|--pattern GLOB]
```

By default the script runs in dry-run mode and prints lines like:

```
would delete local: dev/old-feature
would delete remote: origin/stale-branch
```

Pass `--delete` to perform the deletions.

### Options

- `--delete` — actually delete instead of dry-running.
- `-d N`, `--days N` — staleness threshold in days (default `90`).
  - A local branch is only deleted if its tip commit is older than this
    *and* its upstream is `[gone]`.
  - A remote branch is only deleted if its last commit is older than this.
- `-p GLOB`, `--pattern GLOB` — restrict deletion to branches matching
  the glob. If the pattern has no glob characters (`*`, `?`, `[`), `*`
  is appended automatically. So `-p dev/fi` is equivalent to
  `-p 'dev/fi*'`.

### Examples

```sh
# Show what would be deleted with the default 90-day threshold.
git bp

# Same, but limit to your own dev/foo/* branches and a 30-day cutoff.
git bp -p dev/foo -d 30

# Actually delete.
git bp -p dev/foo -d 30 --delete
```

### Always-protected branches

Regardless of `--pattern`, the following branches are never deleted:

- `master`
- Names containing `hotfix`
- Names containing `REL_` (e.g. `REL_5_STABLE`)
- Names containing `REL` followed by a digit (e.g. `REL3_7_7_plus`,
  `REL4_0_STABLE`)
- The remote default branch (`origin/HEAD`, typically `main`)

Branches checked out in any worktree are also skipped, so the local
prune never tries to delete a branch you have checked out somewhere.
