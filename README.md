# Overlay Forge

A client‑side web application that generates **systemless Magisk / KernelSU / APatch modules** from any single-APK Android app. The generated module overlays a modded APK on top of the installed app via a **bind mount**, letting your custom-signed build run without ever touching the Package Manager — which means no signature mismatch, no data wipe, and no Play Integrity break at the app level.

> **Status:** Working prototype. Proven on user-installed single-APK apps (F-Droid, sideloaded). Split APKs and pure system apps are **not yet supported**.

---

## How It Works

```
                        ┌──────────────────────────────┐
                        │  Your modded APK (new sig)   │
                        └───────────────┬──────────────┘
                                        │
                            uploaded via the web UI
                                        │
                                        ▼
                        ┌──────────────────────────────┐
                        │  Overlay Forge generates a   │
                        │  flashable Magisk module     │
                        └───────────────┬──────────────┘
                                        │
                                    flashed
                                        │
                                        ▼
        ┌───────────────────────────────────────────────────────────┐
        │ customize.sh                                              │
        │   1. Resolve target base path via `pm path`               │
        │   2. Abort if target is a pure system app (/system/...)   │
        │   3. Extract native libs from the modded APK              │
        │   4. Stage APK to /data/adb/rvhc/<pkg>.apk                │
        │   5. `mount -o bind` over /data/app/.../base.apk          │
        └───────────────────────────────────────────────────────────┘
                                        │
                                every boot
                                        │
                                        ▼
        ┌───────────────────────────────────────────────────────────┐
        │ service.sh                                                │
        │   Reapplies the bind mount after sys.boot_completed       │
        └───────────────────────────────────────────────────────────┘
```

**The PackageManagerService (PMS) never sees the modded APK.** It keeps its original record — version, signature, permissions — and reports those to whatever asks. Meanwhile, ART (the Android runtime) loads the modded code from the bind-mounted path. This is why:

- **App Info shows the original version** — PMS is reading its cache, not the file
- **Signature-based checks pass** — PMS reports the original signature
- **The modded code runs** — ART reads the modded bytes at the mounted path

This is the same technique used by `j-hc/revanced-magisk-module`, minus the `pm install` step.

---

## Features

- **APK → Magisk Module** — upload a single APK, get a flashable module ZIP
- **Auto-fill** — extracts package name, label, version, and version code
- **Pure bind-mount overlay** — no PMS interaction, no signature mismatch
- **Boot persistence** — `service.sh` re-mounts on every boot
- **Data preservation** — the app's `/data/data/<pkg>/` is untouched on install
- **Modern UI** — matrix rain, glassmorphism, animated liquid toggle
- **Fully offline** — all dependencies are local; no server uploads
- **Optional advanced binaries** — auto-fetch from repo or manual upload, for KernelSU profile registration and future tooling

---

## Current State

### ✅ Works

| Target type | Notes |
|---|---|
| **User apps** (installed to `/data/app/`) | Full support |
| **System apps that have been updated** (system APK + `/data/app` overlay) | Full support — the mount targets the `/data` path |
| **F-Droid apps** | Tested and confirmed working (OpenCalculator 3.2.1) |
| **Sideloaded APKs** | Supported, as long as the target is in `/data/app/` |
| **Apps with no native libs** | Supported |
| **Apps with native libs** | Supported — extracted to `${BASEPATH}/lib/${ARCH}/` |

### ⚠️ Limitations

| Limitation | Impact | Workaround |
|---|---|---|
| **Single-APK only** | Split APKs (App Bundles) are not merged; only `base.apk` is mounted | None yet — see [Future Scope](#future-scope) |
| **User apps only** | Pure system apps abort with a clear error | Install the app from a store first so it gets a `/data` overlay |
| **Modded version must be ≥ installed version** | Downgrades crash on launch (data schema mismatch) | Uninstall the app first, or clear its data |
| **Data is not sandboxed** | The modded app shares `/data/data/<pkg>/` with the original | If you uninstall and the original crashes, run `su -c "pm clear <pkg>"` |
| **Store updates invalidate the mount** | Updating from Play Store / F-Droid replaces the target APK | Reboot to let `service.sh` re-resolve, or re-flash the module |
| **Play Integrity device verdict still fails** | Rooted devices fail `deviceIntegrity` regardless | Use a separate integrity-spoofing solution |
| **Android 7.1 support is untested** | Some Magisk mount behaviors differ on older ROMs | Test on a modern device for reliable results |

### 🔬 Proven in Practice

- Bind mount succeeds on first flash without reboot
- Modded code executes at runtime (verified via in-app About screen)
- PMS version display stays original (verified via system App Info)
- `service.sh` persists the mount across reboots
- Module uninstall cleans up the staged APK and mount
- Recovery from a bad downgrade confirmed via `pm clear`

---

## Usage

### Local Use (Primary)

1. **Download the entire repository** — clone with Git or download the ZIP.

2. **Ensure all files are in the same folder:**

   ```
   index.html
   fflate.min.js
   app-info-parser.min.js
   gsap.min.js
   Draggable.min.js
   assets/bin.zip          (optional — for advanced binaries)
   ```

3. **Open `index.html`** — double-click, or serve via HTTP for best results:

   ```
   python -m http.server
   # then open http://localhost:8000
   ```

4. **Select an APK**, review the auto-filled metadata, and click **Generate Zip**.

5. **Flash the ZIP** in Magisk / KernelSU / APatch. No reboot required for a user app install, though a reboot is recommended the first time to confirm `service.sh` persistence.

### Hosted Version

Live demo: **https://anemia004.github.io/overlay-forge/**

---

## Requirements

- **Root:** Magisk v20.4+, KernelSU, or APatch
- **Android:** 8.0+ recommended (7.1 untested)
- **Target app:** installed to `/data/app/`, single-APK, version ≤ modded APK

---

## Recovery Procedures

### If the original app crashes after removing the module

The modded version wrote data at a schema the original doesn't understand.

```
su -c "pm clear <package-name>"
```

Then reinstall the app from its original source.

### If the mount is not applied after boot

Check whether the mount line exists:

```
su -c "grep rvhc /proc/mounts"
```

If nothing shows:

1. Reboot once and check again (`service.sh` may have raced PMS).
2. Re-flash the module.
3. Check Magisk logs for `service.sh` output.

### If the target app was updated from a store after flashing

The install path changed, so the mount is now orphaned. Either:

- Reboot (so `service.sh` re-resolves the path), or
- Re-flash the module.

---

## Future Scope

The current build targets the simplest useful case. The following features are on the roadmap, in rough order of impact:

### 1. Split-APK support _(highest impact)_

- Multi-file upload in the web UI (base + config splits)
- Mount **every split**, not just `base.apk`
- Extract native libs from all splits
- Version-lock warning if base and splits disagree

This unlocks Chrome, YouTube, most Play Store apps — essentially anything shipped as an Android App Bundle.

### 2. Version mismatch guard in the web UI

- Warn at build time if the uploaded APK's `versionCode` looks like a downgrade
- Cannot verify against the device from a browser, but a clear warning helps

### 3. Stock APK upload for pure system apps

- Optional second file: the original stock APK
- Enables the j-hc flow: install stock → bind mount modded over it
- Required for YouTube on fresh ROMs where the app has no `/data` overlay

### 4. Magisk "Action" button

- Adds an `action.sh` that runs `pm clear` for the target app
- Lets users reset app data with one tap from the Magisk app

### 5. Per-module integrity profile

- Ship a `system.prop` with common spoof values for apps that check device integrity
- Off by default; opt-in during module build

### 6. Automatic stock-version fetch

- Server-side or jsdelivr-hosted index of known-good stock APKs
- Removes the need for users to hunt down the exact base version

### Explicitly **not** planned

- Any approach that uses `pm install` to replace the target — this is fundamentally incompatible with a differently-signed APK and was the original source of `INSTALL_FAILED_UPDATE_INCOMPATIBLE`
- Automatic patching of APKs in the browser — that belongs to ReVanced and similar projects, not here

---

## Credits & License

This project derives its Magisk module structure and overlay approach from [j-hc/revanced-magisk-module](https://github.com/j-hc/revanced-magisk-module), licensed under the **GNU General Public License v3.0**.

**Modifications made:**

- Replaced the PMS `install` flow with a pure bind-mount overlay
- Generalized the module to accept any single-APK app
- Added a client-side web UI for metadata parsing and module generation
- Removed ReVanced-specific patching and update logic
- Retained the optional `bin/` tooling for KernelSU profile support
- Replaced the on-device installer with a browser-based ZIP generator

Licensed under **GPL-3.0**. Full text in [LICENSE](LICENSE).

---

## Third-Party Libraries

- [`fflate`](https://github.com/nodeca/fflate) — MIT License
- [`app-info-parser`](https://github.com/chenquincy/app-info-parser) — MIT License
- [`GSAP` / `Draggable`](https://greensock.com/standard-license/) — Standard License

---

## Related Tools

- [`j-hc/revanced-magisk-module`](https://github.com/j-hc/revanced-magisk-module) — the upstream project this work derives from; supports ReVanced builds with stock-APK management and split support
- [`rvmm-zygisk-mount`](https://github.com/j-hc/rvmm-zygisk-mount) — Zygisk-based mount variant for newer Android setups
- [ReVanced](https://github.com/ReVanced) — patching framework for the modded APKs you'd feed into this tool
