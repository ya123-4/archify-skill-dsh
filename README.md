# `archify-skill-dsh`

> English | [中文](README.zh.md)

A [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) bundle that exposes the
**Archify 3.0.1** filesystem Skill
([upstream](https://github.com/tt-a1i/archify), MIT, author tt-a1i) as an installable plugin.

Archify creates polished, validated **architecture / workflow / sequence / data-flow /
lifecycle** diagrams as explorable standalone HTML with inline SVG, dark/light themes, optional
trace motion, and PNG/JPEG/WebP/SVG/WebM export. It accepts plain-language requirements or
pasted Mermaid input and can inspect repository evidence when the diagram must reflect real
code.

## What this bundle is

Skill-only: it inserts one filesystem Skill provider (`archify-plugin`) whose single root is the
packaged `skills/` directory. It registers no native render/validate/deliver tools, no Web
client, no telemetry, no credentials handling, no background services, and no
`prepare` `install` `postinstall`hooks.

| Path | Purpose |
|---|---|
| `cordis.patch.yml` | Bundle layer: inserts the `@deepseek-ai/dsh-skill-filesystem` provider with `includeDefaultRoots: false` and `bundledSkillDir` pointing at this package's `skills/` |
| `lib/index.js` | Documented package-resolution helper for the same skill root |
| `skills/archify/` | The vendored, unmodified Archify 3.0.1 release payload (106 files) |
| `package.json` | Bundle manifest (`dsh.bundle.patch`), npm `files` allow-list |

## Why this instead of `@tt-a1i/archify-dsh`

The published adapter `@tt-a1i/archify-dsh@0.1.0` bundles Skill **2.14.0**; the unpublished
`0.2.0` bundles a **2.17.0-dev.1** development snapshot. This package carries the released,
stable Skill **3.0.1** payload taken from the official `archify.zip` release artifact at tag
`v3.0.1` — and nothing else.

## Install

### From a release tarball

1. Download the `archify-skill-dsh-*.tgz` from
   [Releases](../../releases), or build it yourself with `npm pack` in this repository.
2. In the DeepSeek Harness GUI: Sidebar → **Plugins** → **Add plugin**.
3. Paste the absolute path to the `.tgz` (the dialog accepts a package name, a Git address, a
   tarball, or an absolute local path).
4. **Install**. HMR applies the new bundle to the running composition; no restart is needed.

> **Remove the user-level Skill first.** If `~/.dsh/skills/archify/` still exists, the same
> Skill name `archify` would be served by two providers (rank 400 user root and rank 600
> bundled root). Keep exactly one of the two installations.

### From this repository

`npm pack` at the repository root produces the same tarball; install it via the GUI as above.

## Verified behaviour

Measured against DSH 44.0.0 / Node 24.21.0, desktop profile:

| Check | Result |
|---|---|
| Bundle installs and applies without a restart | pass |
| Provider discovers the packaged Skill | pass — catalog name `archify`, base directory `…/node_modules/archify-skill-dsh/skills/archify` |
| Packaged payload integrity | pass — 106/106 files SHA256-identical to the released 3.0.1 payload |
| `archify doctor` from the installed location | pass — all checks green |
| `archify demo <dir>` | pass |
| `archify finalize architecture … --quality showcase --json` | pass — `validate`, `deliver`, `check`, `browser-check` all `pass` |

### `browser-check` needs a Chromium-based browser

`finalize` and `browser-check` drive a real Chromium browser. When none is discoverable the
`browser-check` gate reports `skipped` (exit code 2) with the `viewer/chrome-unavailable`
warning; the other gates still pass. Chromium-based Microsoft Edge works:

```powershell
$env:ARCHIFY_CHROME = 'C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe'
```

The variable must be set in the environment the DSH host process was started from. A missing
browser is an environment limitation, not a bundle defect.

## Uninstall

Plugins page → the bundle's card → **Uninstall**. The profile dependency and bundle layer are
removed; the base profile remains usable.

## Updating the vendored Skill

`skills/archify/` is a verbatim snapshot of the upstream release. To follow a new upstream
version:

1. Download the official `archify.zip` from the upstream release and replace `skills/archify/`
   with its contents (the zip root already contains the `archify/` skill directory).
2. Bump the package `version` to match the Skill version.
3. Verify: `npm run check` (runs `archify doctor`); optionally re-run the `finalize` evidence
   flow from the table above.
4. `npm pack` and install the new tarball.

## Security posture

`cordis.patch.yml` is configuration consumed by the DSH host. Its `bundledSkillDir` value is a
`!!js` expression evaluated in the host process, outside the agent sandbox — treat it as
host-loaded code. The expression uses only Node's built-in `path` and `module` helpers to
resolve this package's own `skills` directory; it makes no network requests, reads no
credentials, spawns no processes, and registers no additional permission path.

## License and third-party notices

This bundle is MIT licensed. The vendored Skill is © tt-a1i ([Archify](https://github.com/tt-a1i/archify),
MIT), based on Cocoon-AI/architecture-diagram-generator (MIT). Optional brand-mark vector data
follows Simple Icons licensing; see
[`skills/archify/THIRD_PARTY_NOTICES.md`](skills/archify/THIRD_PARTY_NOTICES.md).
