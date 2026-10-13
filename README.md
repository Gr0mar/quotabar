# QuotaBar

[![tests](https://github.com/Gr0mar/quotabar/actions/workflows/tests.yml/badge.svg)](https://github.com/Gr0mar/quotabar/actions/workflows/tests.yml)
[![licence: MIT](https://img.shields.io/badge/licence-MIT-blue.svg)](LICENSE)
![macOS 13+](https://img.shields.io/badge/macOS-13%2B-lightgrey)

A macOS menu bar tracker for Claude Code: how much of your subscription window is left, when it resets — and, once the burn rate is measurable, when it will run out — plus what your usage would cost at API list prices.

<img src="docs/menubar.png" alt="the menu bar item early, halfway and nearly out of quota" width="330">

One item, three points of the same session: the dial fills as the quota goes, so the shape alone tells you where you stand. Click it for the rest.

![the dropdown](docs/dropdown.gif)

*(Rendered from synthetic data — `scripts/make-menubar.py` and `scripts/make-gif.py`. No real project names or spend.)*

## Install

Requirements: macOS 13 or later, Claude Code with at least one logged session, and the system `/usr/bin/python3` (3.9+) with the Xcode Command Line Tools — `xcode-select --install` if you have never installed them.

Download the latest zip from [Releases](https://github.com/Gr0mar/quotabar/releases), unzip it, drag `QuotaBar.app` into `/Applications`. The first launch takes one extra step, or the app never opens:

```bash
xattr -dr com.apple.quarantine /Applications/QuotaBar.app
```

Without it macOS 15 and later answer the first double-click with *"QuotaBar" Not Opened — Apple could not verify…* and offer to move the app to the bin. **Click Done, not Move to Bin**, then either run the command above, or go to **System Settings → Privacy & Security**, scroll to *Security*, press **Open Anyway** beside QuotaBar and confirm. Right-clicking and choosing Open no longer works: Apple removed that bypass in Sequoia. The app is ad-hoc signed rather than notarised, and notarising needs a paid Apple Developer account; if this project earns one, the step disappears.

**macOS will ask once for keychain access.** That is the OAuth token read — one keychain read per request — described in [docs/internals.md](docs/internals.md#where-the-numbers-come-from). Deny it and everything still works: the app falls back to the [statusline file](docs/internals.md#the-statusline-hook) or to a local token count.

The app notifies you at 80% and 95% of the session window. Those notifications go out through `osascript`, so macOS attributes them to Script Editor — allow its notifications in System Settings if they never appear ([details](docs/internals.md#notifications)).

### From source

```bash
git clone https://github.com/Gr0mar/quotabar.git && cd quotabar
./scripts/setup.sh          # venv, tests, and a bundle in /Applications
```

## What it shows

| Section | What it answers |
|---|---|
| Session (5h) / Weekly | How much of the subscription quota is used, when it resets, and — once the burn rate is measurable — when it will run out |
| Running now | The session writing to a log right now: project, model, cost, burn rate per hour |
| Today / 7 days / 30 days | Cost and tokens for the window, with a 14-day daily-cost sparkline |
| Top projects | Which repos the spend went to |
| Models | Opus / Sonnet / Haiku split, and what prompt caching saved |

<img src="docs/menu.png" alt="the menu: what the menu bar shows, refresh, notifications, launch at login" width="330">

The label shows the 5h quota by default; the menu switches it to the weekly quota, today's cost, or the icon alone, turns the notifications off, colours the icon orange instead of letting macOS tint it, and adds the app to your login items.

Every dollar figure is the **API list price** of the tokens used, not a bill: on a subscription you pay a flat fee, so read it as "what this usage would cost pay-as-you-go".

## How it works

Spend comes from the session logs Claude Code already writes under `~/.claude/projects/`; the quota windows come from Anthropic's usage endpoint (one keychain read per request, no redirects, nothing stored) or from the optional statusline hook. Nothing else leaves your machine. The details — de-duplication of log lines, how the run-out time is projected, the statusline hook, notifications, idle cost, what gets installed where — are in [docs/internals.md](docs/internals.md).

## Uninstall

```bash
rm -rf /Applications/QuotaBar.app
rm -rf ~/Library/Application\ Support/QuotaBar
rm -rf ~/Library/Caches/com.github.gr0mar.QuotaBar
launchctl bootout gui/$(id -u)/com.github.gr0mar.QuotaBar 2>/dev/null
rm -f ~/Library/LaunchAgents/com.github.gr0mar.QuotaBar.plist
defaults delete com.github.gr0mar.QuotaBar 2>/dev/null
rm -rf ~/.claude/usage-bar
```

## Development

Tests, preview renders, the README images and the release build: [docs/development.md](docs/development.md). PRs welcome; run the tests before opening one.

## Licence

The code is MIT — see [LICENSE](LICENSE). Fork it, ship it, sell it.

The artwork is not mine to license. `quotabar/assets/claude-mark.svg` is Anthropic's Claude mark, taken from [simple-icons](https://github.com/simple-icons/simple-icons) (a CC0 icon set, but trademarks stay with their owners) and used here to identify what the app reports on. This project is not affiliated with or endorsed by Anthropic. If you fork it into something of your own, draw your own icon.
