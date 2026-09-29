# git-delete-merged

Delete local branches whose pull requests have been merged on GitHub.

Unlike tools that only look at the local history, this one asks GitHub
which pull requests are merged, so it works no matter how they were
merged: merge commit, squash, or rebase.  It also does not require the
remote branch to have been deleted.

## Requirements

- `git`
- [`gh`](https://cli.github.com/), authenticated for the repository

## Installation

Put `git-delete-merged` somewhere on your `PATH`.  Git then also
accepts it as `git delete-merged`.

## Usage

Run it inside a clone of a GitHub repository, typically after fetching:

```sh
git fetch --prune
git delete-merged
```

It takes no options.  Outside a GitHub repository it prints a notice
and exits without doing anything.

## How it decides

The script fetches the 100 most recently updated merged pull requests
and looks at every local branch that shares a name with one of them.
Such a branch is deleted when any of the following holds:

1. Its commit is exactly the merged head of the pull request.
2. Its commit is an ancestor of the merged head, so the local copy is
   simply behind.
3. Every commit on it has an equivalent in the base branch, as reported
   by `git cherry`.  This covers merge commits and rebase merges.
4. The combined patch since it forked from the base branch matches a
   commit in the base branch.  This covers squash merges.

Checks 2 to 4 only run for a branch without a live upstream.  A branch
that still tracks an existing remote branch is considered in use and
is only deleted on an exact match.

A branch that matches by name but still has unmerged changes is left
alone with a message.  So is a branch checked out in another worktree.
If the current branch is deleted, the script first switches to the base
branch of the pull request and fast-forwards it.

The exit status is nonzero if any deletion or fast-forward failed.

## See also

[git-sync](https://github.com/jacobwgillespie/git-sync) is a
reimplementation of `hub sync` that fast-forwards local branches and
deletes those whose upstream branch is gone, using only local history.
It detects merge commits and squash merges, but not rebase merges, and
it needs the remote branch to have been deleted.  This tool instead
relies on GitHub's record of merged pull requests, so it handles every
merge method and works whether or not the remote branch still exists,
at the cost of requiring `gh`.  The two complement each other: run
git-sync to fetch and fast-forward, then git-delete-merged to clean up
what it left behind.

## License

BSD 2-Clause.  See [LICENSE](LICENSE).
