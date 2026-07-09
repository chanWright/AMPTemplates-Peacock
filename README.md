# Peacock (HITMAN WoA) — CubeCoders AMP Generic Module Template

An unofficial [AMP](https://cubecoders.com/AMP) "Generic Module" deployment template for
[Peacock](https://thepeacockproject.org/) — the open-source, self-hosted replacement for the
HITMAN World of Assassination trilogy's online services (progression, contracts, escalations,
Ghost Mode, etc.), built from the `Peacock-v8.8.1` release you uploaded.

This is **not** an official CubeCoders or Peacock Project release — it's a community-style
config you drop into AMP yourself, in the same format as the templates in
[CubeCoders/AMPTemplates](https://github.com/CubeCoders/AMPTemplates).

## Files in this template

| File | Purpose |
|---|---|
| `peacock.kvp` | The main Generic Module config (Application / Console / Meta sections) |
| `peacockports.json` | Port definition (`App.Ports`), included into the kvp |
| `peacockupdates.json` | Update stages (`App.UpdateSources`) that download Peacock from GitHub Releases |
| `peacockconfig.json` | The Settings page shown in AMP — maps to `options.ini` |
| `peacockmetaconfig.json` | Tells AMP how to write those settings into `options.ini` |
| `manifest-entry-example.json` | Example entry if you host this in your own template repo |

## Important things to understand about how this is wired up

- **Windows** instances run the Node.js binary that Peacock itself ships in
  `nodedist\node.exe` — no extra dependencies needed on the host.
- **Linux** instances need Node.js 22+ on the host, because the Linux release of Peacock
  does **not** bundle its own Node.js runtime (this matches Peacock's own
  [Linux setup guide](https://thepeacockproject.org/wiki/guides/linux-setup/)). The template
  points `App.ExecutableLinux` at `/usr/bin/node` and sets
  `Meta.SpecificDockerImage=cubecoders/ampbase:node`, which is CubeCoders' own base image
  with Node.js + npm preinstalled — if you deploy this instance into a Docker/Podman
  container (recommended on Linux), you get a working Node.js for free. If you run this
  instance directly on a bare host instead, install Node.js 22+ yourself first (check
  `/usr/bin/node` exists, or edit `App.ExecutableLinux` to match `which node` on your system).
- **Port**: Peacock's own built-in default port is `80`, but binding to ports below 1024
  normally requires root, which AMP instances usually don't run as. This template defaults
  the port to `8080` instead. Whatever port you configure in AMP's Ports tab is what you and
  your players need to enter into **PeacockPatcher** (Windows) or the **OnlineTools**
  ZHMModSDK plugin (Linux/Steam Deck) as `<server-ip>:<port>`.
- **No in-game join system**: Peacock isn't a traditional multiplayer server with connect/
  disconnect events, so there's no player list, RCON, or interactive console — `AdminMethod`
  is set to `PinballWizard` (console is read-only from AMP's point of view) and
  `HasWriteableConsole` is `False`.
- **Settings**: the config page exposes the most commonly changed options from
  `options.ini` (gameplay/progression toggles, Discord Rich Presence, leaderboard
  submission, logging, etc.). Anything not exposed can still be hand-edited directly in
  `Peacock-v8.8.1/options.ini` via AMP's File Manager — Peacock reads it fresh on every
  connect.

## Installing this template

### Option A — Local install (quickest, no GitHub required)
1. On the machine running AMP's ADS (Application Deployment Service), create this folder:
   `/__VDS__ADS01/Plugins/ADSModule/DeploymentTemplates/LOCAL-peacock-main/`
   (the folder **must** start with `LOCAL` and end with `-main`).
2. Copy `peacock.kvp`, `peacockports.json`, `peacockupdates.json`, `peacockconfig.json` and
   `peacockmetaconfig.json` into that folder.
3. Restart AMP's ADS instance (or just refresh the browser per CubeCoders' instructions).
4. When creating a new instance, you should see "Peacock HITMAN Server" show up as a
   deployment option.

### Option B — Your own GitHub repository
1. Fork [CubeCoders/AMPTemplates](https://github.com/CubeCoders/AMPTemplates) (or make your
   own repo with the same layout).
2. Drop the five template files in the repo root, and add an entry like
   `manifest-entry-example.json` to that repo's `manifest.json` (give it a unique `id`,
   `origin`, `url`, and `prefix`).
3. In AMP, go to **Configuration → Instance Deployment → Add Configuration Repository**, and
   enter `your-username/your-repo:main`, then click **Fetch** and refresh the browser.

This template hasn't been submitted to (and doesn't meet all of) the official
CubeCoders/AMPTemplates contribution bar, so use Option A or your own fork rather than
expecting to find it in the stock AMP template list.

## Updating Peacock to a newer version

Because Peacock's release zips extract into a version-named folder (e.g.
`Peacock-v8.8.1/`), this template currently pins that exact version in a few places. When a
new Peacock release comes out, edit **four** things and then hit **Update** in AMP:

1. `peacockupdates.json` — change both download URLs from `v8.8.1` to the new tag, e.g.
   `.../releases/download/v8.9.0/Peacock-v8.9.0.zip` and `...-linux.zip`.
2. `peacock.kvp` — update these three lines to the new folder name:
   - `App.BaseDirectory=./peacock/Peacock-v8.9.0/`
   - `App.WorkingDir=Peacock-v8.9.0`
   - `App.ExecutableWin=Peacock-v8.9.0/nodedist/node.exe`
3. `peacockmetaconfig.json` — update `"ConfigFile": "Peacock-v8.8.1/options.ini"` to the new
   folder name too.
4. Click **Check for updates / Update** on the instance in AMP.

(If you'd rather this happened automatically, AMP's Generic module also supports running an
arbitrary script as an update stage — e.g. a `bash -c` one-liner using `curl`/`jq` against
the GitHub Releases API to always grab the latest asset and flatten the version folder. That
approach is used by some of CubeCoders' own templates (see `hytale.kvp` in
CubeCoders/AMPTemplates for an example), but it's more fragile to hand-write correctly than a
one-line version bump, so it's left out of this template.)

## Verifying / troubleshooting

- Peacock logs `Listening on <host>:<port>...` followed by `Server started.` once it's ready
  — this template's `Console.AppReadyRegex` watches for that second line to flip the instance
  to "Running".
- If the instance won't start on Linux, check that `/usr/bin/node` exists in the container/
  host (`which node`), and that Node.js is v22 or newer (Peacock's `.nvmrc` currently
  specifies `v24.14.1`).
- If players can't connect, double check the port they're patching to in PeacockPatcher/
  OnlineTools matches the port shown in AMP's Ports tab for this instance, and that the port
  is actually reachable (firewall/NAT).
