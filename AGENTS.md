# Agent Guide

This repository serves as a storage location for binary assets, firmware images, and tools related to the **Loop** Android project (rooting and customization).

## 📂 Project Structure

- **`rooting/`**: The core directory containing binary assets.
  - Firmware images (`.img.xz`, `.img.zst`)
  - Android Packages (`.apk`)
  - Archives (`.tar.zst`) containing collections of APKs.

## 🛠 Workflows & Commands

### Asset Management

- **Git LFS**: This repository uses Git Large File Storage (LFS).
  - All binary files in `rooting/` and `images/` are tracked via `.gitattributes`.
  - **Caution**: When reading files, ensure you are not reading the LFS pointer file if you need the actual content (though agents generally shouldn't inspect binary content).

### Development

- **No Source Code**: There is no editable source code (Java/Kotlin/C++) in this specific repository. It is purely for asset distribution.
- **Builds**: There are no build scripts (`Makefile`, `gradle`, etc.). Assets are assumed to be built externally and committed here.

## ⚠️ Important Context

- **Filenames**: Pay attention to filenames for warnings (e.g., `init_boot_b-patched-with-magisk-you-probably-shouldnt-use-this.img.zst`).
- **Documentation**: Primary documentation is located in the [Project Wiki](https://github.com/daveio/loop/wiki), not in this repo.
