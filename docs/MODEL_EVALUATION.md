# ToothCare AI · Model Evaluation

*Revised evaluation of ToothCare AI v1.0.0, October 14, 2026. [← Back to README](../README.md)*

> [!IMPORTANT]
> ToothCare AI is a research and educational prototype. It is not a medical device and has not been clinically validated. Every figure on this page measures **agreement with the labels of public datasets**, not clinical accuracy.

## Contents

1. [Summary](#1-summary)
2. [Models](#2-models)
3. [Method](#3-method)
4. [Tooth model (YOLOv11-seg)](#4-tooth-model-yolov11-seg)
5. [Findings model (Faster R-CNN)](#5-findings-model-faster-r-cnn)
6. [Non-dental images](#6-non-dental-images)
7. [Known issues and fixes](#7-known-issues-and-fixes)
8. [Limitations](#8-limitations)
9. [Appendix A: findings per class](#appendix-a-findings-model-per-class)
10. [Appendix B: teeth per class](#appendix-b-tooth-model-per-class)

---

## 1. Summary

**The tooth model is good. The findings model is reliable for 4 classes and weak for the rest.** Its confidence scores overstate how often it is right.

| Metric | YOLOv11-seg (teeth) | Faster R-CNN (findings) |
| :--- | :---: | :---: |
| Verdict | 🟢 Good | 🟡 Mixed: 4 classes Good |
| Test data | dt3 validation split: 114 X-rays, 3,387 teeth | Dental X-Ray Panoramic Dataset v2 test split: 1,380 X-rays, 13,500 annotations |
| Operating threshold in the app | 0.25 (Ultralytics default) | 0.50 |
| Precision | 92.6% | 81.2% (micro-average) |
| Recall | 94.7% | 76.5% (micro-average) |
| F1 | 0.936 | 0.788 |
| mAP50 / mAP50-95 | 0.937 / 0.761 (box) · 0.937 / 0.651 (mask) | 0.309 / 0.183 (mean over the 24 classes present) |
| False detections per image | 2.2 | 1.7 |
| Hallucinations (nothing annotated nearby) | 51 (1.5% of reported teeth) | 664 (5.2% of reported findings) |
| Mean confidence: correct vs. wrong | 0.82 vs. 0.65 | 0.97 vs. 0.83 |
| Calibration error (ECE) | 0.122 (scores slightly too low) | 0.128 (scores too high) |

**Key points**

- **Tooth detection is reliable.** 92.6% of reported teeth are correct and 94.7% of annotated teeth are found.
- **Numbering is not the main source of tooth errors.** Ignoring the tooth number removes only about 30 of roughly 256 false detections. The rest are duplicates, loose boxes and 51 hallucinations.
- **The findings headline is flattered.** Filling and Impacted tooth make up 55% of annotations, so the micro-averaged 81% precision and 77% recall mostly describe the two easiest classes. 13 of 24 classes score AP50 of about 0.
- **Four findings classes are reliable:** Implant (AP50 0.985), Impacted tooth (0.960), Crown (0.868) and Filling (0.815).
- **Diagnostic findings are weaker.** Caries is usually right when reported (87% precision) but about 1 in 3 is missed. Root canal treatment alone is 880 of 2,392 false findings (37%).
- **Bone loss does not work.** 246 Bone loss findings were reported on the test set and 4 were correct.
- **Findings confidence is inflated.** Findings scored 0.5 to 0.6 are correct 29.5% of the time; 0.9 or higher, 89.1%.
- **Non-dental images produce output.** A street photo produced 11 teeth and an Implant finding at 0.83.

### Verdict criteria (per class, at the app's threshold)

| Verdict | Criterion |
| :--- | :--- |
| 🟢 Good | AP50 ≥ 0.80 and precision ≥ 80% |
| 🟡 Mid | AP50 0.50 to 0.80 and precision ≥ 60% |
| 🟠 Bad | AP50 < 0.50, or precision < 60% |
| 🔴 Broken | Never reported, or almost always wrong |

---

## 2. Models

The analysis endpoint runs two models on each uploaded panoramic X-ray and combines their output:

1. **Findings.** Faster R-CNN runs on the original image and keeps detections scored 0.50 or higher.
2. **Teeth.** The image is contrast-enhanced (×1.2) and sharpened (unsharp mask), then YOLOv11 segments and numbers each tooth.
3. **Assignment.** Each finding is attached to the single tooth whose box contains the finding's center and whose center is closest. Findings outside every tooth box are not shown.
4. **Output.** The result is saved to the user's history and returned to the interface.

| | YOLOv11-seg (teeth) | Faster R-CNN (findings) |
| :--- | :--- | :--- |
| File | `yolov11_seg.pt` · 6.0 MB | `fasterrcnn_epoch_148.pth` · 347.8 MB |
| Architecture | YOLO11 instance segmentation (Ultralytics 8.3.233) | torchvision `fasterrcnn_resnet50_fpn_v2` |
| Parameters | 2.85 million | 43.4 million |
| Classes | 32 tooth numbers (Universal numbering 1 to 32) | 32 finding classes + background |
| Training data | Roboflow "manmax" v3 (dt3): 1,074 train / 114 validation images | Roboflow "Dental X-Ray Panoramic Dataset" v2 (COCO): 9,674 train images |
| Training | 100 epochs, image size 1024, batch 5, AdamW, lr0 0.01, pretrained start | Checkpoint from epoch 148, 640 × 640 training images |
| Inference input | 1024 px | Resized internally to 800 px short side (max 1333) |

---

## 3. Method

No model, threshold or code was changed for this evaluation. Both models were scored on held-out data with the application's preprocessing and thresholds reproduced.

- **Tooth model:** dt3 validation split, 114 full-size panoramic X-rays (1991 × 1127 px) with 3,387 annotated teeth. Not used for training.
- **Findings model:** test split of its source dataset, 1,380 images (640 × 640 px) with 13,500 annotations. The training configuration points only at the train split, so this split is held out.
- **Non-dental images:** blank gray, black, random noise, blurred noise, and two ordinary photos.

Predictions were collected down to very low scores and then cut at the app's thresholds, so results match what users see. The tooth model's official mAP came from the Ultralytics validator; the custom metric code reproduced it (0.929 vs. 0.937 mAP50), which supports the custom precision, hallucination and calibration figures.

<details>
<summary><b>Definitions</b></summary>

| Term | Meaning |
| :--- | :--- |
| Correct detection (TP) | Same class as an annotation and box overlap (IoU) of at least 0.5. Each annotation can be matched once. |
| Precision | Share of reported detections that are correct. |
| Recall | Share of annotated objects that were found. |
| AP50 / mAP50 | Average precision over all score cutoffs at IoU 0.5, per class / averaged over classes (COCO 101-point). |
| mAP50-95 | Same, averaged over IoU 0.50 to 0.95 in 0.05 steps. Rewards tighter boxes. |
| False detection (FP) | A reported detection that matches no annotation. |
| Wrong class or duplicate | A false detection that overlaps a real object (IoU ≥ 0.5) but has the wrong label or repeats one already matched. |
| Partial overlap | A false detection with IoU between 0.1 and 0.5 against an annotated object. |
| Hallucination | A false detection with IoU below 0.1 against every annotated object. |
| Calibration / ECE | Detections are grouped by confidence in 0.1 bins and each bin's average confidence is compared to its share correct. 0 means perfectly honest. |
| Micro vs. class average | Micro-averaging pools all detections, so frequent classes dominate. mAP averages per class, so every class counts equally. |

</details>

**Environment:** Apple Silicon Mac (MPS, verified equal to the CPU path the app uses), Python 3.14, PyTorch 2.14.1, torchvision 0.29.1, Ultralytics 8.4.174. Findings model: about 3 s per image. Tooth model: about 13 ms per image.

---

## 4. Tooth model (YOLOv11-seg)

92.6% precision and 94.7% recall at the app's effective cutoff, with 1.5% of reported teeth hallucinated. Its scores are honest or slightly cautious.

### Official validation (Ultralytics)

| | Precision | Recall | mAP50 | mAP50-95 |
| :--- | :---: | :---: | :---: | :---: |
| Boxes | 93.6% | 93.5% | 0.937 | 0.761 |
| Masks (tooth outlines) | 93.6% | 93.5% | 0.937 | 0.651 |

### Results at different confidence cutoffs

| Cutoff | Precision | Recall | F1 | FP / image | Wrong number or duplicate | Hallucinated | ECE |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0.05 (value in code) | 91.0% | 95.0% | 0.930 | 2.78 | 139 | 64 (1.8%) | 0.121 |
| **0.25 (effective in app)** | **92.6%** | **94.7%** | **0.936** | **2.25** | **103** | **51 (1.5%)** | **0.122** |
| 0.50 | 94.0% | 93.6% | 0.938 | 1.76 | 59 | 47 (1.4%) | 0.125 |

### Where the errors come from (cutoff 0.25)

| Error type | Count | Share of false detections |
| :--- | :---: | :---: |
| Wrong number or duplicate (IoU ≥ 0.5) | 103 | ~40% |
| Partial overlap (IoU 0.1 to 0.5) | ~102 | ~40% |
| Hallucinated (IoU < 0.1) | 51 | ~20% |

Ignoring the tooth number, precision is 93.5% and recall 95.6%. That removes only about 30 of the roughly 256 false detections, so wrong numbering on a correctly found tooth is a minority of the errors.

<p align="center"><img src="assets/charts/teeth-by-class.svg" alt="Tooth model AP50 per tooth class" width="100%" /></p>

All tooth classes score AP50 of 0.82 or higher except tooth 27 (0.67) and tooth 11 (0.73). Each class has about 100 validation instances, so per-class figures carry wide uncertainty. Full table in [Appendix B](#appendix-b-tooth-model-per-class).

---

## 5. Findings model (Faster R-CNN)

Reliable for 4 classes, mid for 4, weak for 3 and not working for the rest. The class-averaged mAP50 is 0.309.

### Results at different score cutoffs

At the app's cutoff the model reported 12,715 findings: 10,323 correct and 2,392 false. 3,177 annotated findings were missed.

| Cutoff | Precision | Recall | F1 | FP / image | Wrong class or duplicate | Hallucinated | ECE |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 0.30 | 78.2% | 78.2% | 0.782 | 2.13 | 267 | 799 (5.9%) | 0.126 |
| **0.50 (app)** | **81.2%** | **76.5%** | **0.788** | **1.73** | **189** | **664 (5.2%)** | **0.128** |
| 0.70 | 84.9% | 74.4% | 0.793 | 1.30 | 141 | 455 (3.8%) | 0.117 |

<p align="center"><img src="assets/charts/findings-by-class.svg" alt="Findings model AP50 per class, colored by verdict" width="100%" /></p>

The Mid anatomical landmarks (sinus, canal) are precise but miss almost half of the annotated instances. Eight of the 32 model classes had no annotations in the test split and are not scored.

### Where the false findings come from

| Class | False findings | Share of 2,392 |
| :--- | :---: | :---: |
| Root canal treatment | 880 | 37% |
| Filling | 524 | 22% |
| Bone loss | 242 | 10% |
| Crown | 167 | 7% |
| Periapical lesion | 166 | 7% |
| Missing teeth | 152 | 6% |
| Caries | 112 | 5% |
| Impacted tooth | 83 | 3% |
| Root piece | 57 | 2% |
| All other classes | 9 | < 1% |

**Bone loss check.** To rule out a label-mapping error, each annotated box was compared with the label the model gave it. Other classes come back with their own names (for example, 1,101 Root canal treatment annotations were labeled Root canal treatment), so the mapping is correct. For Bone loss, 312 of 319 annotations had no matching prediction even though the model reported Bone loss 246 times with boxes of similar size: the model places Bone loss in the wrong locations.

### Confidence calibration

<p align="center"><img src="assets/charts/findings-calibration.svg" alt="Findings model score versus actual accuracy per score range" width="100%" /></p>

| Findings score | Findings in bin | Mean score | Actually correct | Overstated by |
| :--- | :---: | :---: | :---: | :---: |
| 0.5 to 0.6 | 421 | 0.55 | 29.5% | ~25 pts |
| 0.6 to 0.7 | 464 | 0.65 | 34.7% | ~30 pts |
| 0.7 to 0.8 | 530 | 0.75 | 41.1% | ~34 pts |
| 0.8 to 0.9 | 777 | 0.85 | 57.7% | ~27 pts |
| 0.9 to 1.0 | 10,523 | 0.98 | 89.1% | ~9 pts |

A cutoff of 0.9 would keep about 9,380 correct findings at 89.1% correct, or about 69% recall (derived from the bins, not run as a separate evaluation).

---

## 6. Non-dental images

Six images with no teeth were sent through both models exactly as the app runs them. A correct result is zero teeth and zero findings.

| Image | Teeth reported | Highest tooth confidence | Findings reported (score ≥ 0.50) |
| :--- | :---: | :---: | :--- |
| Blank gray | 0 | n/a | none |
| Black | 0 | n/a | none |
| Random noise | 4 | 0.34 | none |
| Blurred noise | 0 | n/a | none |
| Photo of a bus | 11 | 0.59 | Implant (0.83) |
| Photo of people | 1 | 0.54 | Crown (0.84), Filling (0.53) |

Blank images are handled correctly, but ordinary photos produce teeth and findings. This test is qualitative (six images) and does not estimate a false-positive rate on real non-dental uploads such as bitewings, periapical films or CBCT slices.

---

## 7. Known issues and fixes

None of these fixes is applied in v1.0.0; every figure on this page describes the build as shipped. Issues 1 to 4 come first because users see their effect directly.

### Fixable with existing data and code

| # | Issue | Severity | Planned fix |
| :---: | :--- | :---: | :--- |
| 1 | Three tooth classes carry dataset-export labels instead of tooth numbers | High | Remap them (likely teeth 1, 2 and 3), verify third-molar numbering visually, re-run the evaluation |
| 2 | Bone loss is shown although it is almost always wrong; 12 classes never fire | High | Show an allowlist: Good classes, and Mid classes with a note. Hide Bone loss and never-reported classes |
| 3 | Findings scores are shown as percentages but overstate accuracy | High | Stop showing raw scores; threshold at 0.90 or calibrate per class on a separate split |
| 4 | No check that an upload is a dental X-ray | High | Require a plausible tooth count before running the findings model, plus aspect-ratio and image-type checks |
| 5 | Root canal treatment is the largest source of false findings | Medium | Test a higher per-class threshold; hide or mark low reliability if precision stays under ~80% |
| 6 | Possible duplicate tooth boxes with different numbers | Medium | Run with class-agnostic NMS and compare false detections per image |
| 7 | Tooth threshold is 0.05 in code but 0.25 is effective | Medium | Pass the threshold explicitly |
| 8 | Unknown whether contrast and sharpening match training | Medium | Check training config; score with and without the step |
| 9 | Impacted tooth AP50 (0.96) is high for a pathology class | Low | Inspect ~50 samples and describe the label accurately in the app |
| 10 | No confidence intervals | Low | Bootstrap over images |

### Blocked by missing data

| # | Issue | What is needed |
| :---: | :--- | :--- |
| 11 | Findings model trained on 640 × 640 images; the app sends full-size X-rays | Labeled full-size panoramic X-rays. Until then, treat findings figures as upper bounds |
| 12 | False-positive rate on healthy X-rays is unknown | Panoramic X-rays labeled as having no findings |
| 13 | End-to-end result (right finding on the right tooth) is unmeasured | Images annotated with both tooth numbers and findings |
| 14 | No external validation and no clinical label review | X-rays from other clinics and machines, reviewed by a dental professional |

---

## 8. Limitations

- The tooth model's validation split may have been used to choose the saved checkpoint, which would make its scores slightly optimistic.
- The findings test images are 640 × 640 and come from the same source as the training data. Performance on full-size X-rays from other clinics or machines was not measured.
- Every findings test image contains annotations, so hallucination is measured as false findings on annotated images plus the non-dental check, not on clean X-rays.
- Per-class tooth figures rest on about 100 instances each, and rare findings classes on a handful. No confidence intervals were computed.
- Box-overlap scoring is strict for spread-out findings such as Bone loss; a looser overlap rule would score them somewhat higher.
- The two models were evaluated on different datasets, so the combined pipeline was not scored.
- Annotation quality and clinical correctness of the labels were not reviewed.

---

## Appendix A. Findings model per class

Score ≥ 0.50. Classes not listed had no annotations and no predictions in the test split.

| Class | Annotations | AP50 | AP50-95 | Precision | Recall | False findings | Verdict |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | :--- |
| Filling | 4,726 | 0.815 | 0.402 | 88% | 84% | 524 | 🟢 Good |
| Impacted tooth | 2,709 | 0.960 | 0.545 | 97% | 96% | 83 | 🟢 Good |
| Root canal treatment | 1,885 | 0.528 | 0.261 | 55% | 58% | 880 | 🟠 Bad |
| Crown | 1,138 | 0.868 | 0.521 | 86% | 90% | 167 | 🟢 Good |
| Caries | 1,047 | 0.667 | 0.365 | 87% | 69% | 112 | 🟡 Mid |
| Periapical lesion | 542 | 0.361 | 0.187 | 61% | 48% | 166 | 🟠 Bad |
| Missing teeth | 390 | 0.407 | 0.275 | 57% | 52% | 152 | 🟠 Bad |
| Root piece | 349 | 0.648 | 0.321 | 81% | 70% | 57 | 🟡 Mid |
| Bone loss | 319 | 0.005 | 0.002 | 2% | 1% | 242 | 🔴 Broken |
| Implant | 125 | 0.985 | 0.608 | 98% | 95% | 3 | 🟢 Good |
| Mandibular canal | 87 | 0.562 | 0.438 | 92% | 54% | 4 | 🟡 Mid |
| Maxillary sinus | 54 | 0.610 | 0.454 | 94% | 57% | 2 | 🟡 Mid |
| Post-core | 31 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Wire | 30 | 0.008 | 0.003 | n/a | 0% | 0 | 🔴 Broken |
| Attrition | 19 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Primary teeth | 14 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Metal band | 9 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Retained root | 8 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Orthodontic brackets | 7 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Abutment | 5 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Supra eruption | 3 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Malaligned | 1 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| TAD | 1 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |
| Bone defect | 1 | 0.000 | 0.000 | n/a | 0% | 0 | 🔴 Broken |

## Appendix B. Tooth model per class

dt3 validation split, precision and recall at the app's effective cutoff of 0.25. Class names as stored in the model file. Each class has about 100 instances, so differences of a few points are within noise.

<details>
<summary><b>Show all 32 classes</b></summary>

| Class | Annotations | AP50 | AP50-95 | Precision | Recall |
| :--- | ---: | ---: | ---: | ---: | ---: |
| 4 | 111 | 0.940 | 0.741 | 96% | 95% |
| 5 | 109 | 0.978 | 0.805 | 97% | 98% |
| 6 | 108 | 0.990 | 0.834 | 96% | 99% |
| 7 | 107 | 0.969 | 0.720 | 97% | 96% |
| 8 | 105 | 0.931 | 0.750 | 92% | 93% |
| 9 | 104 | 0.883 | 0.719 | 90% | 91% |
| 10 | 95 | 0.971 | 0.778 | 90% | 98% |
| 11 | 100 | 0.729 | 0.651 | 78% | 80% |
| 12 | 91 | 0.968 | 0.879 | 94% | 98% |
| 13 | 110 | 0.847 | 0.713 | 85% | 88% |
| 14 | 111 | 0.945 | 0.749 | 92% | 95% |
| 15 | 111 | 0.975 | 0.808 | 96% | 98% |
| 16 | 114 | 0.989 | 0.810 | 98% | 99% |
| 17 | 112 | 0.999 | 0.785 | 97% | 100% |
| 18 | 112 | 0.980 | 0.721 | 96% | 98% |
| 19 | 111 | 0.936 | 0.706 | 94% | 95% |
| 20 | 111 | 0.968 | 0.759 | 96% | 97% |
| 21 | 108 | 0.937 | 0.761 | 96% | 94% |
| 22 | 114 | 0.949 | 0.754 | 96% | 96% |
| 23 | 110 | 0.969 | 0.789 | 96% | 96% |
| 24 | 108 | 0.989 | 0.817 | 95% | 98% |
| 25 | 104 | 0.824 | 0.676 | 88% | 88% |
| 26 | 93 | 0.967 | 0.880 | 96% | 97% |
| 27 | 105 | 0.670 | 0.598 | 73% | 76% |
| 28 | 96 | 0.871 | 0.745 | 87% | 93% |
| 29 | 103 | 0.974 | 0.767 | 96% | 97% |
| 30 | 108 | 0.941 | 0.655 | 95% | 96% |
| 31 | 108 | 0.962 | 0.753 | 94% | 97% |
| 32 | 110 | 0.977 | 0.769 | 96% | 98% |
| A · "License- CC BY 4-0" | 108 | 0.829 | 0.689 | 85% | 89% |
| B · "Provided by a Roboflow user" | 97 | 0.979 | 0.781 | 96% | 98% |
| C · Roboflow project URL | 93 | 0.895 | 0.744 | 88% | 91% |

Classes A–C are dataset-export labels stored in place of tooth numbers (see issue 1).

</details>
