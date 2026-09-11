# Overlay Forge

> **⚠️ Very early phase project.** Expect rough edges, missing features, and
> breaking changes between commits. Test on a spare device if possible.

A client‑side web app that turns any single-APK Android app into a **systemless
Magisk / KernelSU / APatch module**. The module overlays your modded APK on top
of the installed app via a bind mount — so it runs with a different signature
without ever touching the Package Manager.

---

## How It Works

1. Upload a modded APK in the browser.
2. Overlay Forge generates a flashable module ZIP.
3. Flash it. The module bind-mounts your APK over the installed one.
4. On every boot, `service.sh` re-applies the mount.

Because PMS never sees the modded APK, it keeps reporting the original
signature and version — so signature checks pass and app integrity is preserved.

---

## What Works

- **User apps** installed to `/data/app/` — full support
- **System apps that have already been updated** by a store — supported
  (the mount targets the `/data` overlay, not the `/system` copy)
- **Sideloaded and F-Droid apps** — supported
- **Apps with native libraries** — libs are extracted to the app's lib dir

## Known Limitations

- **Single-APK only.** Split APKs (App Bundles) are not merged — only
  `base.apk` is mounted. Other splits stay original and may crash the app.
- **Pure system apps abort.** The target must exist in `/data/app/`. If it's
  shipped only in `/system/`, install it from a store first so it gets a
  `/data` overlay.
- **No downgrades.** The modded APK's version must be ≥ the installed version,
  or the app will crash on launch from a data schema mismatch.
- **Data is shared with the original.** If you uninstall the module and the
  original app crashes, run `su -c "pm clear <pkg>"` to reset it.
- **Store updates invalidate the mount.** Reboot so `service.sh` re-resolves
  the new path, or re-flash the module.
- **Play Integrity device verdict still fails** on rooted devices — this is
  unrelated to the module and needs a separate spoofing solution.


---

## Usage

1. Clone or download the repo. Keep all files in one folder:

   ```
   index.html
   fflate.min.js
   app-info-parser.min.js
   gsap.min.js
   Draggable.min.js
   assets/bin.zip          (optional)
   ```

2. Open `index.html` — double-click, or serve it:

   ```
   python -m http.server
   # open http://localhost:8000
   ```

3. Select an APK, review the auto-filled fields, click **Generate Zip**.

4. Flash the ZIP in Magisk / KernelSU / APatch.

Hosted demo: **https://anemia004.github.io/overlay-forge/**

---

## Requirements

- Root: Magisk v20.4+, KernelSU, or APatch
- Android 8.0+ recommended
- Target app installed to `/data/app/`, single-APK, version ≤ modded APK

---

## Recovery

**App crashes after removing the module:**

```
su -c "pm clear <package-name>"
```

**Mount not applied after reboot:**

```
su -c "grep rvhc /proc/mounts"
```

If nothing shows, reboot once more (service.sh may race PMS) or re-flash the
module.

---

## Roadmap

Roughly in order of impact:

1. **Split-APK support** — mount every split, extract libs from all of them.
   This unlocks most Play Store apps.
2. **Version mismatch warning** in the web UI.
3. **Optional stock APK upload** for pure system-app targets.
4. **Magisk Action button** to run `pm clear` on demand.
5. **Per-module integrity spoofing profile** (opt-in).
6. **Automatic stock-version fetch** for known targets.

**Not planned:** anything that uses `pm install` to replace the target. That
approach fails with `INSTALL_FAILED_UPDATE_INCOMPATIBLE` for any APK signed
with a different key.

---

## Credits & License

GPL-3.0. See [LICENSE](LICENSE).

Web UI, APK parsing, and module generation: original work.

Shell layer (`utils.sh`, `service.sh`, `customize.sh` scaffolding): adapted
from [j-hc/revanced-magisk-module](https://github.com/j-hc/revanced-magisk-module),
also GPL-3.0.

Bundled: `fflate` (MIT), `app-info-parser` (MIT), `GSAP`/`Draggable` (GreenSock Standard License).
