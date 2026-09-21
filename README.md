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

## what *doesn't* leave your machine

- **his thinking** — a local model via ollama. never a cloud api.
- **his memory** — what you tell him to remember is encrypted on disk, bound
  to your os user. it is never sent anywhere to be "improved".
- **your voice** — wake word, dictation and speech run locally. the one
  online voice option is off until you turn it on, asks first, and tells
  you exactly what it would send.
- **your notes** — encrypted on disk. we could not read them if we wanted to.
- **your history, downloads & bookmarks** — encrypted with keys bound to
  your os user.
- **what you browse** — the sites you open and the searches you run go to
  those sites and that search engine. that is the web, and no browser can
  promise otherwise. what fink promises is everything around it: no
  account, no sync, no telemetry to us. address-bar completions go only to
  the engine you picked, and nowhere at all while onion routing is on.
  trackers and fingerprinting are blocked. onion routing hides your address
  from the sites themselves. incognito leaves nothing behind. the only
  thing the app ever asks our server is whether there is an update.

## the code

the source is not published yet — this repository carries the **installers,
checksums and release notes** while it is prepared for the open. watch
[fink.fyi](https://fink.fyi) for news. until then, here is how it is built,
straight from the shipping code.

**onion routing fails closed.** the proxy follows the *setting*, not tor:
while onion routing is on, every browsing session points at tor before it
has started, while it restarts, and after it exits. requests fail instead of
going out from your real address.

```js
// Fail closed: the proxy follows the setting, not tor. While Onion Routing is
// on, every session points at tor's SOCKS port -- before tor is spawned, while
// it bootstraps or restarts, and after it exits -- so requests fail instead of
// going out direct from the real IP. Only turning the setting off clears it.
function syncTorProxy(settingsManager) {
  if (settingsManager) settings = settingsManager;
  const config = onionRoutingOn() ? TOR_PROXY : { proxyRules: '' };
  for (const s of browsingSessions()) {
    s.setProxy(config).catch((e) => console.error('[TorDaemon] Failed to set proxy:', e));
  }
}
```

**incognito is its own session.** one function decides which session a tab
lives in, and it only ever looks at the tab.

```ts
// The one place a webview partition is chosen: always from the tab it belongs to, so an
// incognito tab can never be handed the persistent, logged-in profile by a navigation.
const partitionFor = (tab?: Pick<Tab, 'isIncognito'>) => tab?.isIncognito ? 'incognito' : `persist:${activeWorkspace}`;
```

**app windows are pinned to their own page.** a window that carries the
app's bridge can only ever show the app. anything else it is asked to open
goes to a browser tab instead.

```js
app.on('web-contents-created', (e, wc) => {
  // Pin every app window to its own document. The bridge follows the window,
  // so external links belong in a browser tab, which is where they go.
  if (wc.getType() === 'window') {
    wc.on('will-navigate', (ev, url) => {
      const here = wc.getURL();
      if (url === here || isAppDocument(url)) return; // a reload, or our own page
      if (UNBRIDGED_UI.test(here) && UNBRIDGED_UI.test(url)) return;
      ev.preventDefault();
      const link = asExternalLink(url);
      if (link) openBrowserWindow(link);
    });
    // Same rule for target=_blank/window.open: never a second bridged window,
    // and never this one navigated away.
    wc.setWindowOpenHandler(({ url }) => {
      if (UNBRIDGED_UI.test(wc.getURL()) && UNBRIDGED_UI.test(url)) return { action: 'allow' };
      const link = asExternalLink(url);
      if (link) openBrowserWindow(link);
      return { action: 'deny' };
    });
  }
});
```

**the sensitive doors ask what is loaded, not what it is.** decrypted
stores and the voice secret are only handed to a window whose top frame is
the app's own document — two independent things have to fail before a page
could ask.

```js
// These hand out decrypted stores and a live localhost secret, so they check
// the frame's real URL, not just that the sender is a window: two
// independent things have to fail before a page can ask.
function isAppFrame(e) {
  try {
    const frame = e.senderFrame;
    return !!(e.sender.getType() === 'window' && frame && !frame.parent && isAppDocument(frame.url));
  } catch (err) { return false; }
}
```

**the voice backend answers only the app.** it listens on localhost, where
anything on the machine could otherwise connect. a secret minted at launch
goes to the backend and to the app's own windows, never to a web page — and
the backend compares it in constant time.

```js
// The voice backend's WebSocket listens on localhost, where any website or
// local process could otherwise connect to it. This per-launch token goes to
// the backend in its env and to app windows over IPC, never to webviews.
const VOICE_WS_TOKEN = require('crypto').randomBytes(24).toString('hex');
ipcMain.on('voice-ws-token', (e) => {
  e.returnValue = isAppFrame(e) ? VOICE_WS_TOKEN : '';
});
```

```python
def token_ok(websocket):
    if not WS_TOKEN:
        return True
    request = getattr(websocket, "request", None)
    path = getattr(request, "path", None) or getattr(websocket, "path", None) or ""
    given = parse_qs(urlsplit(path).query).get("token", [""])[0]
    return hmac.compare_digest(given.encode(), WS_TOKEN.encode())

async def process_audio_stream(websocket):
    if not token_ok(websocket):
        await websocket.close(1008, "unauthorized")
```

**what you type goes to the engine you picked, and never around tor.**
address-bar completions come from your search engine and nobody else, and
the request is skipped entirely while a proxy is on rather than leave from
your real address.

```js
// One host gets what you type: the engine you picked in Settings, and nobody
// else.
const SUGGEST_ENDPOINTS = {
  google: (q) => `https://suggestqueries.google.com/complete/search?client=chrome&ie=UTF-8&oe=UTF-8&q=${q}`,
  duckduckgo: (q) => `https://duckduckgo.com/ac/?type=list&q=${q}`,
  bing: (q) => `https://api.bing.com/osjson.aspx?query=${q}`,
  yahoo: (q) => `https://search.yahoo.com/sugg/gossip/gossip-us-ura/?output=fxjson&command=${q}`,
};
// ...
// Ask the tab session whether it is proxied at all: that covers Onion Routing
// and V2Ray obfuscation alike, and anything else a proxy is set for later.
const proxy = await session?.fromPartition('persist:default').resolveProxy(target);
if (proxy && !/^DIRECT/i.test(proxy)) return null;
```

**web pages get an allowlist, not a denylist.** a page gets fullscreen,
pointer lock, drm playback and a copy button. anything not on the list —
reading your clipboard, watching whether you are idle, a future permission
nobody has thought about yet — is refused by default.

```js
const WEB_ALLOWED = ['fullscreen', 'pointerLock', 'mediaKeySystem', 'clipboard-sanitized-write'];
// The frontend is the assistant's mic and camera, and the address bar's paste-and-go.
const FRONTEND_ALLOWED = WEB_ALLOWED.concat([
  'media', 'camera', 'microphone', 'clipboard-read', 'deprecated-sync-clipboard-read',
]);
// ...
const allowed = isFrontend(webContents.getURL()) ? FRONTEND_ALLOWED : WEB_ALLOWED;
```

**the notes vault file reveals nothing about your key.** what is stored on
disk is an hmac *under* the derived key, which cannot be turned back into it
— and it is written temp-then-rename, so a crash can never leave the vault
without its verifier.

```js
// v2 stores an HMAC under the derived key, which reveals nothing about it.
// Written temp-then-rename so a crash can never leave the vault without a
// verifier.
const masterVerifier = (keys) => crypto
  .createHmac('sha256', Buffer.concat([keys.layer1, keys.layer2, keys.layer3]))
  .update('fink-notes-verifier')
  .digest();
const writeMaster = (salt, keys) => {
  const tmp = masterFile + '.tmp';
  const data = { v: 2, salt: salt.toString('hex'), verifier: masterVerifier(keys).toString('hex') };
  fs.writeFileSync(tmp, JSON.stringify(data), { encoding: 'utf-8', mode: 0o600 });
  fs.renameSync(tmp, masterFile);
};
```

**nothing runs unchecked.** every third-party binary the app fetches — the
model runtime, tor, the proxy, the hardware sensor tool — is pinned to an
exact version and sha256. a download that does not match is deleted, not
run.

```js
const OLLAMA_PIN = {
  version: 'v0.34.2',
  assets: {
    'ollama-windows-amd64.zip': '8f3fd071a2a2f9497b562f43502c77c2b701a99d1ee5dfda28da8c786373063b',
    'ollama-darwin.tgz': 'f33b2a5aa59bc6c961ed3ec23ba9dc646ca6d99ced8d2a0d46eb3a522167dd3f',
  },
};
// ...
if (String(got).toLowerCase() !== want.toLowerCase()) {
  try { fs.unlinkSync(archivePath); } catch (e) {}
  throw new Error(`Ollama download does not match its pinned fingerprint (expected ${want}, got ${got}).`);
}
```

**updates come from home, or not at all.** on top of the pgp signature and
the sha256 (see [verify your download](#verify-your-download)), an update
may only be fetched from fink.fyi or this repository. a stolen signing key
alone is not enough: the installer would also have to be hosted where our
releases live.

```js
// A stolen signing key alone is not enough: the attacker would also have to
// host the installer where our releases live.
const RELEASE_PREFIXES = [
  `${downloadsBase.replace(/\/$/, '')}/`,
  `https://github.com/${ghRepo}/releases/download/`,
];

function isReleaseUrl(url) {
  return typeof url === 'string' && RELEASE_PREFIXES.some((p) => url.startsWith(p));
}
```

**a hardened shell.** the binary ships with its electron fuses set at pack
time: cookies encrypted with the os keystore, no node debugging flags, code
only from the app archive, and the archive's integrity checked at launch.

```js
// Electron fuses, flipped in the packaged binary at pack time.
//  - cookies are encrypted with the profile's OS-keystore key, so a backup
//    or copied profile holds no replayable session cookies;
//  - NODE_OPTIONS and --inspect can't turn the shipped exe into a Node
//    debugger/loader;
//  - only resources/app.asar is loaded, and its header is checked against
//    the hash electron-builder already embeds in the exe / Info.plist.
electronFuses: {
  enableCookieEncryption: true,
  enableNodeOptionsEnvironmentVariable: false,
  enableNodeCliInspectArguments: false,
  onlyLoadAppFromAsar: true,
  enableEmbeddedAsarIntegrityValidation: true,
},
```

**checked, then checked again.** the codebase carries its own verification
suite — dozens of checks that lift the real functions out of the source and
drive them against hostile input, each written to fail the moment the guard
it covers is removed. every release goes through it before it is signed.

---

<div align="center">

© Sacua Engo — all rights reserved.<br/>
<a href="https://fink.fyi">fink.fyi</a>

</div>
