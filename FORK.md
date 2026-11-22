# Fork notes

This is a personal fork of
[truhanen/pebble-another-timer](https://github.com/truhanen/pebble-another-timer)
carrying a small stack of local changes on top of upstream.

## Branches

| Branch   | Contents                                                        | Rules                                   |
|----------|-----------------------------------------------------------------|-----------------------------------------|
| `master` | Exact mirror of `upstream/master`                               | Fast-forward only. Never commit here.   |
| `custom` | `upstream/master` + the fork's commits (this file included)     | Rebased onto upstream; force-pushed.    |

Build and install from `custom`. The fork's changes are always exactly
`git log upstream/master..custom`, one commit per change, so each one can be
dropped, reworded or sent upstream as a PR on its own.

## Local changes

| Commit subject                                         | What / why                                                                                                                         |
|--------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `CHORE: Add fork notes and upstream sync script`       | This file and `scripts/sync-upstream.sh`.                                                                                          |
| `FIX: Alarm Stop honours AutoReturnStop again`         | Stop on the alarm screen returns to the watchface when "Return to watchface -> After stopping a timer" is on (behaviour before upstream a00478f). +1 Min / Overtime keep upstream behaviour. |
| `FEAT: Wrap-around UP/DOWN in the main timer list`     | UP on "+ New timer" jumps to the last timer; DOWN on the last timer jumps to "+ New timer". Fresh presses only; held repeat stops at the edge. |
| `FIX: First UP/DOWN press after launch is a no-op`      | On launch the MenuLayer cursor sat on row 0 ("+ New timer") while the highlight showed the first timer, so the first UP/DOWN did nothing when no timer was running (a running timer's 1 s tick re-synced it). Upstream bug. |

Keep this table up to date when adding or dropping a fork commit.

## Updating from upstream

```bash
git remote add upstream https://github.com/truhanen/pebble-another-timer.git   # once per clone
scripts/sync-upstream.sh            # fetch, fast-forward master, rebase custom
# build + test (see CLAUDE.md), then:
git push origin master
git push --force-with-lease origin custom
```

`scripts/sync-upstream.sh --push` does the two pushes itself. The script
saves the pre-rebase stack as a local `backup/custom-<timestamp>` branch and
enables `git rerere`, so a conflict resolved once is replayed automatically
on later rebases in the same clone.

On a conflict: resolve, `git add`, `git rebase --continue`. If upstream has
adopted or superseded a fork change, `git rebase --skip` that commit and
remove it from the table above.

## Adding a new local change

```bash
git checkout custom
# edit, build, test
git commit          # one logical change per commit, upstream's subject style
git push origin custom
```

To amend an existing fork commit instead of stacking a fix on top, use
`git commit --fixup=<sha>` followed by `git rebase --autosquash upstream/master`.

## Releases

Tag builds you install as `<upstream-version>-custom.<n>`, e.g.
`v2.3.0-custom.1`, so an older patched build can be rebuilt after later
rebases rewrite `custom`.
