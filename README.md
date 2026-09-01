# PM Moduler Pro

A client‑side web application that generates Magisk modules from any APK.  
The module leverages the Package Manager (`pm install`) with a session‑install fallback, enabling app updates without removing the original system package.

## Features

- **APK to Magisk Module** – upload an APK and produce a flashable module ZIP.
- **Auto‑fill** – extracts package name, label, version, and version code from the APK.
- **Optional Advanced Binaries** – include `bin/` tools for KernelSU support and APK processing; choose between auto‑fetch from this repository or manual upload.
- **Modern UI** – matrix rain, glassmorphism, and an animated liquid toggle.
- **Fully Offline** – all dependencies are local; no server uploads.

## Usage

1. Open `index.html` in a modern browser.
2. Select an APK.
3. Adjust module metadata if desired.
4. Optionally include advanced binaries.
5. Click **Generate Zip** to download the module.

## Credits & License

This project builds upon the Magisk module structure and scripts originally developed in [j-hc/revanced-magisk-module](https://github.com/j-hc/revanced-magisk-module), which is licensed under the GNU General Public License v3.0.

**Modifications made:**
- Generalized the module to accept any APK, removing ReVanced‑specific logic.
- Added a web‑based interface for APK selection and metadata editing.
- Included the original `bin` folder as an optional component.
- Replaced the on‑device installer with a browser‑based ZIP generator.

This project is also licensed under the **GPL‑3.0**. The complete source code is available in this repository, and the full license text is provided in the [LICENSE](LICENSE) file.

## Alternative Approach

For users interested in a Zygisk‑mounting method instead of PM install, see [j-hc/rvmm-zygisk-mount](https://github.com/j-hc/rvmm-zygisk-mount).

## Third‑Party Libraries

- `fflate` – [MIT License](https://github.com/nodeca/fflate)
- `app-info-parser` – [MIT License](https://github.com/chenquincy/app-info-parser)
- `GSAP` / `Draggable` – [Standard License](https://greensock.com/standard-license/)
