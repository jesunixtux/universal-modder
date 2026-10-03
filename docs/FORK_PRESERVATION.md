# Independent upstream preservation

This fork is `jesunixtux/universal-modder`. The original is
`rehan-remade/universal-modder`.

## What is backed up

The workflow [preserve-upstream.yml](../.github/workflows/preserve-upstream.yml)
checks the original **every hour** (minute 23, UTC) and can also be
triggered manually.

- `main`: YOUR control/default branch. It is never overwritten by this workflow.
- `upstream/main`: current copy of the original main branch.
- `upstream/<branch>`: current copy of every original branch, including new ones.
- `upstream-history/<branch>/<full-old-commit-SHA>`: immutable saved branch tips
  if the author force-pushes/rebases a branch.

The workflow intentionally **never prunes** branch references in this fork.
If the author removes a branch, your last copied `upstream/<branch>` remains.
If the author deletes the repository, the workflow reports an error and leaves
all existing backup refs untouched.

It only fetches upstream Git objects and copies Git references. It does **not**
execute upstream scripts or automatically merge unreviewed changes into `main`.
It also does not automatically copy upstream tags, GitHub issues, releases,
release binary attachments, or Git LFS blobs.

## Required one-time GitHub activation

GitHub disables scheduled workflows on new public forks. The fork owner must:

1. Open **Actions** in the fork and enable workflows if GitHub asks.
2. Open **Preserve upstream branches** and choose **Enable workflow** if disabled.
3. Select **Run workflow** on branch `main`.
4. Confirm the run succeeds and inspect branches named `upstream/*`.

The workflow asks for `contents: write` for its own GitHub token. If the job
reports push permission denied, check **Settings -> Actions -> General ->
Workflow permissions**. Only increase permissions if needed and available;
never paste a token in source files.

On public repositories GitHub may disable scheduled workflows after 60 days
of inactivity. Review the Actions page periodically to ensure they remain enabled.

## Make a real backup outside GitHub

A fork is NOT a substitute for an offline copy. Run this periodically on your
own computer, ideally saving the bundle on another disk:

```bash
git clone --mirror https://github.com/jesunixtux/universal-modder.git
git -C universal-modder.git bundle create ../universal-modder-archive.bundle --all
```

To verify the bundle, use:

```bash
git bundle verify universal-modder-archive.bundle
```

The Git bundle stores tracked Git history and refs that were fetched into the
mirror, but it does not include GitHub issues, release attachments or external
Git LFS object contents. Preserve these separately if you need them.

## When the original disappears

Your public fork is retained under GitHub's normal public fork deletion rules.
The scheduled run will fail because its upstream is unavailable, but none of
your branches is deleted. Use your `upstream/*` branches for the last saved
versions. If necessary, you may detach the fork using GitHub's Settings ->
General -> Leave fork network (this is permanent) or migrate using your mirror.

## Limitations

- Sync runs *periodically*, not instantly on the upstream's push. GitHub may
  delay scheduled runs.
- Concurrent or protected branch updates can make pushes fail; review run logs.
- If the original disappears **between** checks, changes not yet fetched
  cannot be recovered by this workflow.
- Keeping an external backup is necessary against account loss, platform-wide
  issues and takedowns affecting more than just the upstream.
