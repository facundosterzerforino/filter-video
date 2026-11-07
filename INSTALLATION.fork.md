# Use the reviewed FilterVideo build

## Install on this Windows PC

The extension is already built at:

```text
C:\Users\fsterzer\Downloads\FilterVideo\filter-video\dist
```

1. Open `chrome://extensions` (or `edge://extensions`).
2. Enable **Developer mode**.
3. Click **Load unpacked**, then select the `dist` folder above — not the repository root and not a ZIP file.
4. Reload any streaming tabs that were already open.
5. Pin FilterVideo if desired. Open its popup to choose blur/opacity and intensity.
6. On a detected video page, press **`,`** to toggle the filter. **`r`** is a detection/reset shortcut, not a reset of all saved preferences.
7. In the browser's extension details, prefer **Site access → On specific sites** for the streaming sites you actually use. Do not enable incognito/file access unnecessarily.

Keep the `dist` folder in place: the browser loads the extension directly from it. If you rebuild it, click **Reload** on its extension card and refresh the video page.

Read `AUDIT.md` before granting permissions: no malicious behavior was found, but this is not a guarantee, permissions are broad, build/test dependencies have known vulnerabilities, and the original test suite fails. The separate production-bundle smoke tests are simulated tests, not proof of live Netflix/DRM compatibility. Runtime source and permissions were not hardened or altered.

**Only blur and opacity are supported.** This is not a brightness/contrast/color-filter extension and does not automatically identify scenes to censor.

## ZIP alternative

Ready-to-extract package:

```text
C:\Users\fsterzer\Downloads\FilterVideo-1.2.4-reviewed.zip
```

Extract it to a permanent folder and select the folder containing `manifest.json` with **Load unpacked**. No npm, Node, or build tools are needed just to install the ZIP's contents. Do not run the source archive as if it were the built extension.

Expected SHA-256:

```text
cd0344cf43b9e890582ae866fb0f72d0c54855b08bba238914f4ca16a9518234
```

## Fork and local files

- Maintained fork: https://github.com/facundosterzerforino/filter-video
- Original project: https://github.com/bengmoh/filter-video
- Reviewed runtime source commit: `2adc6f8c3808a778fe8e24b086901709186b746f`
- Local repository: `C:\Users\fsterzer\Downloads\FilterVideo\filter-video`
- Local evidence and reusable smoke-test harness: `C:\Users\fsterzer\Downloads\FilterVideo\audit`
- `origin`: your fork; `upstream`: the original project.
- Ignore the earlier incomplete `forks\facundosterzerforino\filter-video-original` clone and the unrelated empty `filter-video-fork` repository. They are not the finished build.

## Check for future updates without installing them

These commands are for **Git Bash** inside the local repository. The repository is configured to use the authenticated `gh` CLI for GitHub credentials, not a GUI credential prompt.

```bash
cd 'C:/Users/fsterzer/Downloads/FilterVideo/filter-video'
GIT_TERMINAL_PROMPT=0 GCM_INTERACTIVE=never timeout 90 git -c credential.helper= -c 'credential.helper=!gh auth git-credential' fetch upstream

git log --oneline 2adc6f8c3808a778fe8e24b086901709186b746f..upstream/main
git diff 2adc6f8c3808a778fe8e24b086901709186b746f..upstream/main -- src package.json package-lock.json webpack.prod.js webpack.dev.js
```

An empty log means no new upstream commits relative to this reviewed snapshot. Fetching alone does not modify your installed build. Pay particular attention to permissions, new network destinations, executable/dynamic code, dependency provenance, and install/build scripts.

Only after reviewing changes should you merge them into your fork, repeat the security/dependency checks, rebuild, and reload the extension. Do not force-sync away the fork's documentation. The security conclusion and ZIP checksum in `AUDIT.md` apply only to the recorded build, not a later commit or rebuild.

A fork does not automatically notify you about every upstream change. You may optionally configure **Watch** on the original GitHub repository; release notifications only help when upstream actually publishes releases. No subscription or recurring background job was created.

## Rebuild the recorded source

The existing `package-lock.json` pins the original dependencies. They have known advisory findings; review `AUDIT.md`. There is deliberately no `npm audit fix --force` or automatic dependency upgrade in this build.

```bash
npm ci --ignore-scripts --no-audit --no-fund
npm run build

# Run the saved tests against the actual production bundles:
node '../audit/smoke-built.cjs' 'C:/Users/fsterzer/Downloads/FilterVideo/filter-video'

# Original upstream tests: recorded baseline is 1 pass / 14 failures.
npm test -- --runInBand
```

The original `npm run package` uses a `zip` program that is not installed on this PC, so use `npm run build` rather than that packaging command. The delivered ZIP was made with Python's standard-library `zipfile` module and checked against every packaged file.

Webpack cleans `dist` when rebuilding. To redistribute a rebuild, restore the GPL license, corresponding-source provenance, extracted license notices, and CSS/style-loader third-party licenses before packaging. Recalculate the ZIP/file checksums and repeat tests; don't reuse this build's checksum. Keep the reviewed ZIP as a rollback copy.
