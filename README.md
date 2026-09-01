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

### Local Use (Primary)

1. **Download the entire repository**  
   Either clone it with Git or download the ZIP from GitHub.

2. **Ensure all files are in the same folder**  
   The following must be present in the same directory:
   - `index.html`
   - `fflate.min.js`
   - `app-info-parser.min.js`
   - `gsap.min.js`
   - `Draggable.min.js`
   - `assets/bin.zip` (optional, for advanced binaries)

3. **Open `index.html`**  
   You can double‑click `index.html` or serve it via a simple HTTP server for best compatibility.

   (If you have Python installed, run `python -m http.server` in the folder and open `http://localhost:8000`)

4. **Select an APK** and follow the on‑screen steps.

### Hosted Version

A live demo is available at:  
[https://anemia004.github.io/pm-moduler-pro/](https://anemia004.github.io/pm-moduler-pro/)

## Credits & License

This project draws on ideas and code from [j-hc/revanced-magisk-module](https://github.com/j-hc/revanced-magisk-module), which is licensed under the GNU General Public License v3.0.

**Changes made:**
- Generalized the module structure to accept any APK, not just ReVanced builds.
- Added a web interface for APK selection, metadata parsing, and customization.
- Removed ReVanced‑specific patching and update logic.
- Included the original `bin` folder (advanced binaries) as an optional component.
- Replaced the native Android installer with a browser‑based ZIP generator.

This project is also licensed under the **GPL‑3.0**. The complete source code is available in this repository, and the full license text is provided in the [LICENSE](LICENSE) file.

## Related Tools

For Zygisk‑enabled devices, [rvmm-zygisk-mount](https://github.com/j-hc/rvmm-zygisk-mount) offers a mount‑based approach that may integrate better with newer Android setups. This project uses PM install; both can coexist depending on your requirements.

## Third‑Party Libraries

- `fflate` – [MIT License](https://github.com/nodeca/fflate)
- `app-info-parser` – [MIT License](https://github.com/chenquincy/app-info-parser)
- `GSAP` / `Draggable` – [Standard License](https://greensock.com/standard-license/)
