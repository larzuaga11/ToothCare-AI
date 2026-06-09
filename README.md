# 🦷 ToothCare AI
## Offline-first desktop application for automated dental panoramic X-ray analysis
___
ToothCare AI combines a YOLOv11 segmentation model for tooth segmentation with a custom-trained Faster R-CNN for lesion and pathology identification — packaged into a single installable desktop app. No cloud. No internet. No data leaves the machine.
___

## Features 

| Feature | Description |
| :--- | :--- |
| **🔐 Authentication** | Local signup / login with Argon2 password hashing. Session identity is carried via a user-Email header. |
| **🩻 X-ray Analysis** | Upload a panoramic dental X-ray; the pipeline segments every visible tooth and maps lesions / pathologies to each one. |
| **🦷 Tooth Segmentation** | YOLOv11 draws a polygon mask around each tooth with its anatomical class and confidence score. |
| **🔬 Lesion Detection** | Faster R-CNN scans the full panoramic for 32 pathology classes and assigns each finding to the spatially overlapping tooth. |
| **📋 Diagnosis History** | Every analysis is auto-saved to the local SQLite database, browsable and filterable by date or disease name. |
| **📄 PDF Reports** | One-click PDF export per image — includes the source X-ray thumbnail and a structured findings list with confidence scores. |
| **🌗 Light / Dark Mode** | Theme toggled at any time; persisted in localStorage. |
| **📴 Fully Offline** | The FastAPI backend and both ML models are bundled inside the installer. No internet connection required after installation. |



