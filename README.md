<div align="center">

<img src="assets/fink-app.png" width="128" alt="fink." />

# fink.

**a modern privatized ecosystem.**

your assistant, your browser, your notes — everything runs on your machine,
and nothing you do leaves it.

[![website](https://img.shields.io/badge/website-fink.fyi-0a0a0a?labelColor=1a1a1e)](https://fink.fyi)

</div>

---

## the apps

|  | app | what it is |
|---|---|---|
| <img src="assets/fink-app.png" width="44" /> | **fink.app** | the full ecosystem — a J.A.R.V.I.S.-style assistant that runs on a local model, the browser, encrypted notes, and a deck of live widgets. talk to him, or type. he does it, or explains it. |
| <img src="assets/fink-browser.png" width="44" /> | **fink.browser** | the browser on its own. adblock, fingerprint spoofing, onion routing, encrypted history — privacy as the default, not a mode. |
| <img src="assets/fink-notes.png" width="44" /> | **fink.notes** | local, encrypted notes. sealed behind a master password only you know. there is no cloud and no recovery — that's the point. |

start with the browser or the notes if you like — if you install the full
ecosystem later, it adopts your data and offers to retire the standalone
app. the browser stays one click away, as its own app, either way.

## download

grab the installer for your product from
**[releases](https://github.com/finkdot/fink/releases/latest)**, or from
[fink.fyi](https://fink.fyi).

| platform | file | notes |
|---|---|---|
| windows | `fink-setup-<version>.exe` / `fink-browser-setup-…` / `fink-notes-setup-…` | one-click install, updates itself silently |
| macos | `…-mac.dmg` | right-click → open the first time |
| linux | `…-linux.AppImage` | `chmod +x`, then run |

## verify your download

every release lists **sha256** checksums. compare before you run:

```powershell
Get-FileHash .\fink-setup-0.1.0.exe -Algorithm SHA256
```

updates inside the app are verified twice on their own: the update manifest
is **PGP-signed** with the fink release key and checked against the key
pinned inside the app, and the downloaded installer is checked against its
pinned sha256 before it ever runs. the same public key is published here as
[`fink_public.key`](fink_public.key), and the standing
[warrant canary](https://fink.fyi/canary) is signed with it too.

## what leaves your machine

- **his thinking** — a local model via ollama. never a cloud api.
- **your voice** — wake word, dictation and speech run locally by default.
  the one online voice option asks first, and tells you exactly what it
  sends.
- **your notes** — encrypted on disk. we could not read them if we wanted to.
- **your history & downloads** — encrypted with keys bound to your os user.

## where's the source?

not published yet. this repository carries the **installers, checksums and
release notes** while the code is still being prepared for the open. watch
[fink.fyi](https://fink.fyi) for news.

---

<div align="center">

© Sacua Engo — all rights reserved.<br/>
<a href="https://fink.fyi">fink.fyi</a>

</div>
