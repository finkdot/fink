<div align="center">

<img src="assets/fink-app.png" width="128" alt="fink." />

# fink.

**a modern privatized ecosystem.**

your assistant, your browser, your notes — everything runs on your machine,
and nothing you do leaves it.

[![website](https://img.shields.io/badge/website-fink.fyi-0a0a0a?labelColor=1a1a1e)](https://fink.fyi)
[![downloads](https://img.shields.io/endpoint?url=https%3A%2F%2Ffink.fyi%2Fapi%2Fdownloads%3Fformat%3Dshields)](https://fink.fyi/download)

</div>

---

## the apps

|  | app | what it is |
|---|---|---|
| <img src="assets/fink-app.png" width="44" /> | **fink.app** | the full ecosystem — an assistant that answers to you, and only you, running on a local model on your own machine; the browser; encrypted notes; and a deck of live widgets. talk to him, or type. he does it, or explains it. |
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
| windows | `fink-setup-<version>.exe` / `fink-browser-setup-…` / `fink-notes-setup-…` | one-click install; asks before it updates |
| macos | `…-mac.dmg` | right-click → open the first time |
| linux | `…-linux.AppImage` | `chmod +x`, then run |

the downloads badge above also counts the closed beta, which was handed out
before these releases existed — so it is higher than the per-file counts on
the [releases page](https://github.com/finkdot/fink/releases).

## verify your download

every release lists **sha256** checksums. compare before you run:

```powershell
Get-FileHash .\fink-setup-<version>.exe -Algorithm SHA256
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

## the code

the source is not published yet — this repository carries the **installers,
checksums and release notes** while it is prepared for the open. watch
[fink.fyi](https://fink.fyi) for news. what you can know about it now:

- **local by design.** the assistant's model, his voice, your notes, history
  and downloads all live on your disk. there is no account, and no server of
  ours in the loop.
- **encrypted at rest.** notes are sealed with a key derived from your master
  password. history, downloads, bookmarks and the assistant's memory are
  encrypted with keys bound to your os user account.
- **isolated where it counts.** incognito tabs live in their own session and
  leave nothing behind. onion routing fails closed — if tor is not up,
  nothing goes out direct. the voice backend runs in its own process, walled
  off from whatever else is installed on the machine, and only the app holds
  the key to talk to it.
- **a hardened shell.** the app binary ships with its electron fuses set: no
  node debugging flags, no loading code from outside the app archive, and
  the archive's integrity is checked at launch. cookies are encrypted with
  the os keystore.
- **nothing runs unchecked.** every third-party binary the app fetches — the
  model runtime, tor, the proxy, the hardware sensor tool — is pinned to an
  exact version and sha256, and refused if it does not match. updates are
  pgp-signed, hash-verified, and only ever fetched from fink.fyi or this
  repository (see [verify your download](#verify-your-download)).
- **checked, then checked again.** the codebase carries its own verification
  suite — dozens of checks that drive the real functions against hostile
  input, each written to fail the moment the guard it covers is removed —
  and every release goes through it before it is signed.

---

<div align="center">

© Sacua Engo — all rights reserved.<br/>
<a href="https://fink.fyi">fink.fyi</a>

</div>
