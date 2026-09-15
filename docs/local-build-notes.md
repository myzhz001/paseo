# Local Build & OpenRouter Notes (fork: myzhz001/paseo)

> Built from source on macOS (arm64) — replaces the previously installed Paseo 0.8.0 app.

## Repo layout
- Fork: `https://github.com/myzhz001/paseo` · local clone: `~/dev/paseo`
- Remotes: `origin` = fork, `upstream` = `getpaseo/paseo` (pull with `git pull upstream main`)

## Build from source
Prereqs: node 26, npm 11, git. First install may ask to approve postinstall scripts
(`npm install-scripts approve <pkg>` for esbuild / sharp / node-pty / workerd / fsevents).

```bash
cd ~/dev/paseo
npm install
npm run build:server          # server + cli + relay/highlight/plugin
```

### Dev run (fast loop, no package)
```bash
npm run dev:desktop           # Electron app, dev daemon on 127.0.0.1:6768
npm run dev:server            # daemon only (dev home in .dev/paseo-home)
```

### Packaged build (Option B — what is installed now)
The desktop build script uses the *clean* variant and does **not** build the web UI,
so `packages/app/dist` must exist first:

```bash
cd packages/app && PASEO_WEB_PLATFORM=electron npx expo export --platform web && cd ../..
cd packages/desktop
CSC_IDENTITY_AUTO_DISCOVERY=false npx electron-builder \
  --config electron-builder.yml --mac dir -c.mac.notarize=false --publish never
# output: release/mac-arm64/Paseo.app  →  copy to /Applications
```

Notes: not signed/notarized (no Apple Developer ID). If the app crashes with
`net::ERR_FILE_NOT_FOUND loading 'paseo://app/'`, the web bundle was missing — redo the
expo export step above (that is the `app-dist` in the package).

## Installed pieces (after this project)
- App: `/Applications/Paseo.app` (ad-hoc signed, 480M)
- CLI: `paseo` → symlink `/usr/local/bin/paseo` → `…/Paseo.app/Contents/Resources/bin/paseo`
- Config/data: `~/.paseo/` (daemon listens 127.0.0.1:6767, relay to app.paseo.sh)
- Backup of prior config/data: `~/.paseo.bak.0915` (keep until confident; then `rm -rf`)
- Old app in Trash: `~/.Trash/Paseo.app` (previous install)

## OpenRouter provider (custom)
In `~/.paseo/config.json` under `agents.providers`:

```json
"openrouter": {
  "extends": "claude",
  "label": "OpenRouter",
  "env": {
    "ANTHROPIC_BASE_URL": "https://openrouter.ai/api/v1",
    "ANTHROPIC_AUTH_TOKEN": "sk-or-v1-…"
  }
}
```

The token value is the machine's `OPENROUTER_API_KEY` (also in
`~/.config/litellm/openrouter_key`). **Never commit the raw token.** After editing:
`paseo daemon restart` (or relaunch the app).

Run an agent through it:
```bash
paseo agent run "your task" --provider openrouter --model anthropic/claude-sonnet-4 \
  --cwd <workdir> --wait-timeout 10m
```
Verified: agent run completed via provider `openrouter`; model replied over
`https://openrouter.ai/api/v1` (HTTP 200).

## Daily use
- `open /Applications/Paseo.app` — or double-click in Finder
- CLI available as `paseo` in any terminal (restart shell if not on PATH)
- Ports: packaged app 6767 · dev build 6768
