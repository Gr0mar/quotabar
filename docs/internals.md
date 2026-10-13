# How QuotaBar works

## Where the numbers come from

**Spend** is read from the session logs Claude Code already writes to `~/.claude/projects/**/*.jsonl`. Each assistant response carries a `usage` block; those tokens are priced against a table of published per-million rates — list prices, which is why the totals dwarf a subscription fee. A model with no published price still has its tokens counted and is shown as `—` rather than guessed at. Nothing leaves your machine to compute this.

**Quota windows** come from one of two places, whichever answers first:

1. `https://api.anthropic.com/api/oauth/usage`, called with the OAuth token Claude Code stores in your login keychain. The token is read for that single request; it is never written to disk or logged, the request refuses redirects so the header cannot be replayed to another host, and it goes to Anthropic and nowhere else.
2. `~/.claude/usage-bar/limits.json`, written by the statusline hook that ships inside the app at `QuotaBar.app/Contents/Resources/statusline-limits.sh`. No credentials at all, but it only refreshes while a Claude Code session is running.

If neither answers, the header falls back to a local count of the tokens billed in the last five hours and says so. The endpoint rate-limits, so it is polled every five minutes and doubles the interval after each failure, up to half-hourly; a reading older than five minutes is shown with its timestamp rather than as current.

## Optional: the statusline hook

Claude Code pipes a JSON blob into your statusline command on every turn, and for subscribers it contains the quota windows. `scripts/statusline-limits.sh` saves them and then hands the untouched input to whatever statusline command you already use:

```json
"statusLine": {
  "type": "command",
  "command": "/Applications/QuotaBar.app/Contents/Resources/statusline-limits.sh 'your existing command'"
}
```

With no argument it prints a small `5h 42% · 7d 8%` line of its own.

## Notifications

The app notifies you once the session window passes 80% and again at 95%, so the cap does not arrive mid-thought. Turn that off in the menu. Those notifications go out through `osascript`, so macOS attributes them to Script Editor — allow its notifications in System Settings if they never appear.

## Accuracy

Claude Code writes one log line per content block, and every line of the same response repeats the same message id and the same complete `usage` object — often seconds apart, so a response routinely straddles two scans. The scanner therefore remembers every event id it has already counted; without that, roughly a third of all responses would be counted twice. The same set makes re-reading a file idempotent, so a `/rewind`, a rotation, or a full rescan cannot inflate the totals.

## Projecting the cap

The reset time comes from the API — it is a fact, not a guess. The *arrival* time is the guess: the API says how much of a window is used, never how fast, so the app measures the slope itself from successive readings of the same window.

The span is measured to the present moment, not to the last reading, which is what makes an idle machine stop predicting. A window only reports a new percentage when it moves, so a burst of work followed by an hour of nothing would otherwise keep the last steep rate on screen forever; measured against the clock, that same burst dilutes from 30%/h to 10%/h over the following hour and eventually falls under the floor, taking the prediction with it.

A line appears only when the readings span at least eight minutes, the window is actually moving, and the rate would empty it *before* it resets — otherwise there is nothing to warn about.

## Performance

The first launch parses every log once and caches the day-level rollup in `~/Library/Caches/com.github.gr0mar.QuotaBar`. After that each pass reads only the bytes a log has grown by, tracked per file, and a half-written trailing line is left for the next pass rather than dropped. Idle cost is one `stat` per log every five seconds; the directory tree itself is re-walked at most once a minute.

## What gets installed where

`setup.sh` calls `install.sh`, which puts the runtime under `~/Library/Application Support/QuotaBar`. That split from the checkout is deliberate: an app launched from Finder cannot read `~/Desktop` or `~/Documents`, so a bundle pointing back at a checkout there would die on startup. The bundle carries its own copy of PyObjC and runs on the system `/usr/bin/python3` (1.4 MB download, about 5 MB on disk).

"Launch at login" in the menu writes a LaunchAgent to `~/Library/LaunchAgents` — no admin rights, no installer.
