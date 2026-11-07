# FilterVideo 1.2.4 — source review and build record

Reviewed on 2026-11-07. This is a source review by an AI assistant, not an independent security certification or a guarantee that no defect exists.

## Scope and conclusion

**No obvious malicious behavior was found in the reviewed runtime source or the emitted JavaScript.** The review covered the exact upstream commit `2adc6f8c3808a778fe8e24b086901709186b746f`, the Chrome/Firefox manifests, every runtime JS file, popup HTML/SCSS, build/test configuration, dependency lock provenance, and the CSS runtime helpers actually included in the production bundle. The upstream `src/` files, package files, and build configuration were not modified.

This review applies to this locally built GitHub source snapshot, **not** to a Chrome Web Store binary or future upstream changes. The extension manifest version is **1.2.4**; the npm package version is still 1.0.0.[1][7]

The extension implements only **blur and opacity**. It does not implement brightness, contrast, saturation, or automatic recognition/censoring of particular scenes.[3][5]

## Runtime findings

- No outbound request APIs, analytics clients, remote-script loaders, dynamic `eval`/`Function` execution, or credential/cookie extraction were found in the extension source. The targeted scan was repeated on all emitted JS, with no matches for network-request or dynamic-code entry points. This is an observation from code review and pattern checks, not a proof against every possible data channel.
- The popup includes ordinary, user-clicked links to GitHub, a Google feedback form, Buy Me a Coffee, and PayPal. These are external sites, not background telemetry.[5]
- Settings are read/written through `chrome.storage.local`; detected-tab/filter state is kept in memory.[2] The popup uses internal extension messages to read/change its settings and video status.[4]
- Page keyboard listeners compare pressed keys against the configured toggle/reset shortcuts. No recording or transmission of arbitrary keystrokes was found. However, they do not exclude text inputs/contenteditable elements, so typing a shortcut in a form can trigger a filter/reset.[3]
- Manifest V3 packages local content, service-worker, and popup scripts. No custom `update_url`, external messaging interface, native messaging permission, cookie permission, or history permission is declared.[1]

## Permissions: the main privacy concern

The extension requests `tabs`, `webNavigation`, `storage`, and `alarms`; its content script matches **`<all_urls>`**.[1]

Those permissions are broader than a Netflix-only filter needs. The code uses them for tab/video coordination, navigation detection, settings, and a keep-alive alarm.[2][3] Broad access remains a risk even though malicious use was not observed. The scope was left unchanged to preserve upstream behavior; this is **not a least-privilege hardened fork**.

Recommended: after loading the extension, set Chrome/Edge site access to only the streaming sites you use. This limits content-script exposure; it does not eliminate the broad metadata access granted by the separate tab/navigation permissions. Do not enable incognito or local-file access unless you actually need them.

## Dependencies and supply-chain limitations

Fresh `npm audit --json` against the original lockfile reported **29 vulnerable dependency entries: 9 low, 6 moderate, 13 high, 1 critical**. These are unremediated in this exact-source build. Audit results change over time; the raw report is saved with the local build evidence.

All declared dependencies are development dependencies.[7] The critical entry is `form-data` in the Jest/jsdom test dependency graph. `npm audit --omit=dev --json` reports zero findings, but that alone is not a browser-bundle security assessment.

Webpack's module inventory shows the browser bundle contains extension code, generated Babel helpers, and popup CSS/style-loader runtime helpers. The vulnerable `form-data`/jsdom/test stack is not shipped in the extension ZIP. The shipped CSS/style runtime helper sources were read separately. The remaining advisory findings were not all reproduced or proven unreachable: **do not describe this as a zero-vulnerability build**.

The lockfile has 550 dependency package entries; each resolved package points to `registry.npmjs.org` and has an integrity value. This was not a line-by-line audit of all 550 package sources, a package-signature validation, or a malware/antivirus certification.

The final rebuild used `npm ci --ignore-scripts --no-audit --no-fund`, which prevents dependency lifecycle/install hooks from running. The earlier interrupted attempt had already performed a normal `npm install`; rebuilding this way does not retroactively sandbox that earlier install. The reviewed webpack build still executes the installed build tools, so this is not equivalent to an isolated-container build.

## Functional verification and known limitations

| Check | Result |
|---|---|
| Locked clean install, dependency lifecycle hooks disabled | Passed |
| Production webpack build | Passed, 0 errors / 0 warnings |
| Original upstream Jest suite, unchanged | **1 passed / 14 failed / 15 total** |
| Separate tests of actual production bundles | **7 passed / 0 failed**, no unexpected jsdom exceptions |
| ZIP CRC check and byte-for-byte comparison with packaged files | Passed |
| Real Chrome/Edge installation and live Netflix/DRM playback | **Not tested** |

The seven production-bundle scenarios use Node + jsdom and mocked Chrome APIs, not real streaming pages: YouTube-shaped and Netflix-shaped video pages; Prime Video's second-player selection; auto-filter; shortcut updates/disable; a non-player page; and popup initialization/settings. They exercise detection, blur/opacity, intensity updates, storage messages, and toggling. No request attempts were observed through the instrumented request APIs in those scenarios. These tests do **not** establish real-site compatibility, browser service-worker lifecycle correctness, DRM compatibility, or coverage of all data channels.

The original Jest suite contains stale expectations and incomplete Chrome message mocks; there are also real robustness concerns in the runtime (nullable shortcut initialization, duplicate reset listeners on reattachment, substring-based URL checks, and in-memory service-worker state).[3] The upstream test failures were **not fixed, hidden, or reclassified as passing**. Long-session worker suspension, every supported platform, iframe/fullscreen edge cases, and keyboard behavior while typing remain unverified.

## Artifact provenance

- Source: `2adc6f8c3808a778fe8e24b086901709186b746f`
- Runtime source / lockfile / build configuration changes: **none**
- Node: `24.14.0`; npm: `11.12.0`
- Webpack: `5.97.1`; build hash: `fb2257cbca078af824e8`
- Production config removes console/debugger statements.[6]
- ZIP: `FilterVideo-1.2.4-reviewed.zip`
- ZIP size: **41,316 bytes**
- ZIP SHA-256: `cd0344cf43b9e890582ae866fb0f72d0c54855b08bba238914f4ca16a9518234`
- 15 packaged files, including GPL license, source-provenance text, and third-party notices. No executables, native libraries, or `node_modules` were packaged.

The project is licensed under **GPLv3**, not MIT; the earlier recommendation incorrectly described its license.[7][8] Corresponding source, build configuration, and the lockfile remain available in the fork and its source archive. GPL and bundled third-party notices are retained.

## Fork and updates

Real GitHub fork: https://github.com/facundosterzerforino/filter-video

Its GitHub parent is `bengmoh/filter-video`. The local `origin` points to the fork and `upstream` points to the original. This fork adds review/install documentation; it does not add new filter features, dependency fixes, or permission hardening.

Unpacked extensions do **not** automatically install later upstream builds. Review a future diff, rebuild, and reload manually. A fork is not an automatic notification service; optional GitHub watch notifications can be configured separately. See `INSTALLATION.fork.md`.

The earlier attempt also created an empty, unrelated `facundosterzerforino/filter-video-fork` repository and a second incomplete local clone. Those are not the maintained fork/build and were not deleted.

## Sources

[1] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/src/manifest.json
[2] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/src/background.js
[3] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/src/content.js
[4] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/src/popup/popup.js
[5] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/src/popup/popup.html
[6] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/webpack.prod.js
[7] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/package.json
[8] https://github.com/bengmoh/filter-video/blob/2adc6f8c3808a778fe8e24b086901709186b746f/LICENSE
