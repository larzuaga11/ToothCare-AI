# 🦷 ToothCare AI
## Offline-first desktop application for automated dental panoramic X-ray analysis
___
ToothCare AI combines a YOLOv11 segmentation model for tooth segmentation with a custom-trained Faster R-CNN for lesion and pathology identification — packaged into a single installable desktop app. No cloud. No internet. No data leaves the machine.
___

## 💾 Download

Go to the [Releases](../../releases) page and download the installer for your platform:

| Platform | File |
| :--- | :--- |
| **🪟 Windows** | `ToothCareAI Setup x.x.x.exe` |
| **🍏 macOS (Apple Silicon)** | `ToothCareAI-x.x.x-arm64.dmg` |
| **🐧 Linux** | `ToothCareAI-x.x.x.AppImage` |

---

## ⚙️ Requirements

| Component / Platform | Minimum Requirement |
| :--- | :--- |
| **🪟 Windows** | Windows 10 64-bit |
| **🍏 macOS** | macOS 12 Monterey (Apple Silicon) |
| **🐧 Linux** | Ubuntu 20.04+ or equivalent |
| **🧠 RAM** | 4 GB (8 GB recommended) |
| **💽 Disk Space** | ~2 GB free (models are bundled) |

> 💡 **Note:** No Python, Node.js, or any other runtime required — everything is bundled in the installer.

---

## 🚀 What It Does

Upload a dental panoramic X-ray and **ToothCare AI** will:

*   **🦴 Segment every visible tooth** — bounding box, polygon mask, and anatomical class.
*   **🔬 Detect pathologies** — 32 condition classes including caries, bone loss, cysts, fractures, implants, and more.
*   **🗺️ Map findings to teeth** — each lesion is assigned to the tooth it spatially overlaps.
*   **💾 Save the analysis** — all results are stored locally and browsable in your history.
*   **📄 Export a PDF report** — one-click generates a structured findings report with confidence scores.

---

## 🔍 Detectable Conditions (32)

Here are the 32 condition classes currently supported by the system:

`Attrition` · `Abutment` · `Bone Defect` · `Bone Loss` · `Caries` · `Crown` · `Cyst` · `Dentals` · `Filling` · `Fracture` · `Gingival Former` · `Impacted Tooth` · `Implant` · `Malaligned` · `Mandibular Canal` · `Maxillary Sinus` · `Metal Band` · `Missing Teeth` · `Orthodontic Brackets` · `Periapical Lesion` · `Permanent Retainer` · `Permanent Teeth` · `Plating` · `Post-core` · `Primary Teeth` · `Retained Root` · `Root Canal Treatment` · `Root Piece` · `Root Resorption` · `Supra Eruption` · `TAD` · `Wire`

---

## 🔒 Privacy

All processing happens on your device. **No images, results, or personal data are ever transmitted to any server.**



