# Development

```bash
./scripts/setup.sh                                 # venv, tests, and a bundle in /Applications
.venv/bin/python -m unittest discover -s tests     # no network, no display, no keychain
.venv/bin/python scripts/preview.py /tmp 7         # renders the dropdown to PNGs
.venv/bin/python scripts/preview.py docs 7 --demo  # a still of the same, synthetic data
./scripts/run.sh                                   # foreground, prints tracebacks
.venv/bin/python scripts/make-icon.py docs/AppIcon.icns     # rebuild the app icon
.venv/bin/python scripts/make-gif.py docs/dropdown.gif      # rebuild the README animation
.venv/bin/python scripts/make-menubar.py docs/menubar.png   # rebuild the menu bar strip
.venv/bin/python scripts/make-menu.py docs/menu.png         # rebuild the picture of the menu
./scripts/build-release.sh                         # self-contained bundle + release zip
```

The app's identity — bundle id, cache directory, LaunchAgent label — lives in `quotabar/identity.py`, and the install script reads it from there.

Rebuilding the animation needs Pillow (`pip install -r requirements-dev.txt`); nothing else does, and the app itself depends only on PyObjC.

## Layout

`quotabar/` splits into pure logic (`parser`, `pricing`, `aggregate`, `scanner`, `live`, `limits`, `projection`, `alerts`, `formatting`, `tokens`, `store`, `identity`) and the AppKit layer (`ui/`). Only `ui/` imports AppKit; everything else is testable without a display. The store owns one background thread that publishes an immutable snapshot, and the UI reads that snapshot once per layout pass — so a repaint can never mix values from two different scans.

