# FLIR ADAS v2: state-of-the-art detection results (Q6)

**Research date:** 2026-11-02
**Group:** D (see `../PLAN.md`). This file builds on `03_tasks_and_benchmarks.md` (protocol IDs P0–P13), `01_dataset_overview.md`, `02_usage_and_publications.md` (paper IDs A1–A11, C1–C13) and `05_known_problems_workarounds.md` (known pitfalls). It does not repeat their content.
**Scope:** Teledyne FLIR ADAS **v2** (v2.0.0, Jan 2022) only. FLIR-aligned (v1-derived) results appear only as a short contrast note in §9.

**How the numbers were checked**
- I re-read the result tables of 15 papers from their arXiv or CVF PDFs (`pdftotext -layout`). For 7 MDPI, PLOS and Nature papers I read the full-text XML through the Europe PMC REST API.
- **[verified]** means I read the number in the stated table myself. **(unverified)** means it comes from an abstract, from another file in this folder, or from my inference.
- I copied every number exactly as printed. Values in % and values in 0–1 are kept as each paper prints them.
- Numbers from different protocol rows are **not comparable**. Each table below covers one protocol, and the cross-protocol view in §7 is for orientation only.

**New protocol IDs added in this pass** (they extend the P0–P13 list in 03):

| ID | Description | Papers |
|---|---|---|
| **P14-VT-3c-scratch** | Official split: `images_thermal_train` 10,742 → model selection on val 1,144 → test on **VT 3,749**. 3 classes: person, car, and "bicycle" (bike and motor merged). Trained from scratch, 640×640, 8-bit. | Fs2PA (Sensors 2026) |
| **P15-VAL-multi** | Thermal train → official **val 1,144 used as the reported evaluation set**, multi-class YOLO (6–11 classes) | MDCFVit-YOLO (PLOS ONE'25), CDYL (Sci Rep'24); **in fact also MS-YOLO** (see the P1 correction below) |
| **P16-UNSTATED** | Official split exists, but the paper does not say which split the numbers are on | YOLO-APDM (Sensors'24), class pruning (eIT'26), Perimeter Security (Sensors'25) |
| **P17-CUSTOM** | Author-made split of v2 images (random or study-specific), or v2 mixed with other data | RADFRNet (Sensors'26), TCLNet (PLOS ONE'26), LIWL-YOLO (Sci Rep'25), Zha (IET-IP'24) |

**Correction to P1 in 03.** MS-YOLO's per-class table (Table IV) is headed "all | 1144 pictures | 16455 instances" and includes `bus` (179 instances), a class with 0 boxes in VT. **The reported MS-YOLO numbers are on the official val split (1,144), not on the 3,493-frame test** **[verified]**. Any val-based checkpoint selection then also happens on the reporting set.

---

## TL;DR (2026-11-02)

- **There is no single v2 SOTA.** Every number below belongs to one protocol (see 03 §9.3). No v2 leaderboard exists, and no method has been evaluated under two different v2 protocols by independent groups.
- **Best number per main protocol** (all **[verified]** unless marked):

| Protocol | Best reported result | Method (venue, tier) |
|---|---|---|
| **P0-FLIR** (VT 3,749, person/car AP50) | 75.33 / 77.23 (official baseline). No paper re-reports exactly P0 | FLIR YOLOX-m (README) |
| **P14-VT-3c-scratch** (VT 3,749, official split) | **mAP50 64.06, mAP50:95 37.02; AP50 person 77.16 / car 81.83** | Fs2PA, YOLOv11n-based (MDPI Sensors 2026, lower tier) |
| **P1/P15** (official val 1,144 as evaluation set) | MDCFVit-YOLO **mAP50 0.67 / mAP50-95 0.405**, person 77.3 / car 85.1 (6 classes shown); MS-YOLO paper's best is its YOLOv8n* baseline, 0.517 / 0.318 (9 classes) | PLOS ONE 2025 / IJCNN 2025 |
| **P2-VT-4c-mm** (VT 3,749, 4 classes, mAP50-95) | 25.21 (thermal-only YOLOv8n); YOLO Phantom 24.82 | MODIPHY (ICIP 2024) |
| **P3-VT-5c-raw** (14-bit raw, scratch, 1280) | **AP 40.5 / AP50 58.5** | YOLOv7-W6-Reasoner (CVPRW 2023) |
| **P4-RAW2151** (raw subset, person+car with truck/OV merged) | **mAP 56.0 / AP50 83.8** | TCNet + YOLOX (IEEE RA-L 2024) |
| **P5-VAL-2c** (val as test, person+car) | **AP 58.4** (YOLOv8-x + turbulence aug.); **AP50 87.7** (DINO-4scale + aug.) | Uzun et al. (Sci Rep 2025) |
| **P6-ECDM-5c** | mAP 27.3 / AP50 52.1 (CenterNet + ECDM pseudo data) | ECDM (ACM MM 2024) |
| **P7-UDA-SSDC** (RGB→IR UDA, val 1,144) | "COCO-style mAP" **48.02** (values look like AP50) | SS-DC (arXiv 2025) |
| **P8-OV-ZS** (zero-shot open vocabulary) | **AP 0.096 / AP50 0.173** (supervised G-DINO FT reference: 0.200 / 0.345) | Thermal-Det (CVPR 2026) |
| **P11-NAGC** (v1∪v2, 3,742 "video-test") | AP50 person/car **92.3 / 89.1** (Faster R-CNN), **90.1 / 86.3** (YOLOX-m); COCO AP 65.8 | Neural-AGC (ICCVW 2025). **Not comparable and internally inconsistent** |

- **The most trustworthy current SOTA on the official v2 test split is Fs2PA (P14).** On VT 3,749 it reaches person AP50 77.16 and car AP50 81.83, above FLIR's YOLOX-m reference (75.33 / 77.23). Fs2PA is a nano-scale model trained from scratch, published in a lower-tier venue, with no code. The comparison with P0 is only approximate: Fs2PA trains on 3 merged classes from scratch, while P0 trains a COCO-pretrained YOLOX-m on all classes.
- **The highest raw numbers are untrustworthy.** NAGC (90–92 AP50) uses a non-standard v1∪v2 training set and a 3,742-frame test. Its "vendor-AGC" rows copy the README baseline exactly for two different detectors. HOTCOLD and POD (clean AP 94.8 / 0.94) are evaluated on tiny person-only subsets of large pedestrians.
- **Many "v2" numbers are really val numbers.** This applies to MS-YOLO, MDCFVit-YOLO, CDYL and Uzun et al., and probably to YOLO-APDM and the eIT paper. Val numbers run higher than test numbers (the community audit found mAP50-95 0.417 on val vs 0.322 on test; see 05 Problem 3).
- **RGB, RGB-T and open-vocabulary results are sparse.** RGB: only the FLIR README (51.42 / 55.79) and MODIPHY (mAP50-95 ≈ 19.7). RGB-T fusion: **no v2 result exists**. Open-vocabulary: only Thermal-Det (CVPR 2026).

---

## 1. Thermal-only, official video test split (VT): P0, P14, P2, P3

### 1a. P0-FLIR: official reference (VT 3,749; person/car AP@0.5)

| Method | Venue/year (tier) | Backbone/detector | Input | AP50 person | AP50 car | Other | Code | Source |
|---|---|---|---|---|---|---|---|---|
| FLIR baseline (thermal) | FLIR README 2022 (official) | YOLOX-m, COCO-pretrained | 8-bit, 640×640 | **75.33** | **77.23** | — | none | README.txt [verified in 03] |

No paper re-runs exactly P0 (YOLOX-m, all classes trained, VT 3,749). The closest protocols are P14 (same test set, different class set and pretraining) and P11 (claims to be "video-test", but see §6).

### 1b. P14-VT-3c-scratch: Fs2PA protocol (official train / val for selection / **VT 3,749 test**; person, car, bicycle = bike+motor merged; from scratch; 640×640; 300 epochs; seed 42)

Sorted by mAP50. All rows are **[verified]**, from Fs2PA Tables 2, 4 and 5 (MDPI *Sensors* 26(7):2257, 2026). The authors trained every baseline themselves under the same recipe.

| Method | Venue/year (tier) | Backbone/detector | Input | mAP50 | mAP50:95 | AP50 person | AP50 bicycle* | AP50 car | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|
| **Fs2PA** | Sensors 2026 (MDPI, lower tier) | YOLOv11n + GIA + P2 + SAS-Head, 11.3 GFLOPs | 8-bit, 640, scratch | **64.06** | **37.02** | **77.16** | **33.18** | **81.83** | none | T2, T4 |
| YOLOv8s | (baseline) | YOLOv8s | same | 62.27 | 35.94 | – | – | – | – | T5 |
| YOLOv9s | (baseline) | YOLOv9s | same | 61.89 | 36.24 | – | – | – | – | T5 |
| YOLOv10s | (baseline) | YOLOv10s | same | 61.80 | 36.32 | – | – | – | – | T5 |
| YOLOv5su | (baseline) | YOLOv5su | same | 61.77 | 36.18 | – | – | – | – | T5 |
| YOLOv11s | (baseline) | YOLOv11s | same | 61.59 | 36.32 | – | – | – | – | T5 |
| YOLOv8-IRD | (baseline, IR-specific) | YOLOv8n variant | same | 60.60 | 34.60 | – | – | – | – | T4 |
| YOLOv9t | (baseline) | YOLOv9t | same | 58.36 | 33.03 | – | – | – | – | T4 |
| YOLOv11n | (baseline) | YOLOv11n | same | 57.55 | 32.55 | 72.43 | 25.35 | 74.88 | – | T2, T4 |
| D-FINE-N | (baseline) | D-FINE-N | same | 57.50 | 32.00 | – | – | – | – | T4 |
| YOLOv8n | (baseline) | YOLOv8n | same | 56.75 | 32.44 | – | – | – | – | T4 |
| RT-DETR-N | (baseline) | RT-DETR-N | same | 51.45 | 26.58 | – | – | – | – | T4 |
| YOLOX-nano | (baseline) | YOLOX-nano | same | 45.90 | 23.70 | – | – | – | – | T4 |
| CenterNet-R18 | (baseline) | CenterNet R18 | same | 17.00 | 5.90 | – | – | – | – | T4 |

\*"Bicycle" = FLIR `bike` + `motor` merged by the authors. In VT, all bikes and motors come from one video (05 Problem 2), so this column measures one sequence.
Flags:
- The abstract says "6.51% relative gain vs. YOLOv11". The actual gain is 6.51 **percentage points** (57.55 → 64.06).
- The weak trained-from-scratch baselines (CenterNet 17.0, YOLOX-nano 45.9) make the margins look larger than they would be against COCO-pretrained detectors.
- Single seed. No code or split files are released.

### 1c. P2-VT-4c-mm: MODIPHY (thermal VT 3,749; person, car, light, sign; COCO mAP50-95; YOLOv8n family, COCO-pretrained, 640)

| Method | Venue/year (tier) | Backbone/detector | Input | mAP50-95 (thermal, PyTorch) | Code | Source |
|---|---|---|---|---|---|---|
| YOLOv8n trained on thermal | ICIP 2024 (baseline) | YOLOv8n | 8-bit, 640 | **25.21** | – | Table III [verified] |
| YOLO Phantom (MODIPHY), mixed RGB+thermal training | ICIP 2024 (major conf.) | YOLOv8n + Phantom conv | 8-bit, 640 | 24.82 | none found | Table III [verified] |
| YOLOv8n trained on RGB (cross-modal) | ICIP 2024 | YOLOv8n | – | 7.21 | – | Table III [verified] |

Flag: the numbers come from the Ultralytics `benchmark` function (Sec. V-B), which evaluates on the dataset YAML's `val` entry. Whether that entry pointed to VT is not stated (unverified). Values in other export formats (ONNX, OpenVINO, ncnn) differ slightly.

### 1d. P3-VT-5c-raw: IR Reasoner (14-bit raw ÷ (2¹⁴−1); person, car, sign, light, bike; trained from scratch; 1280)

Sorted by AP (COCO AP@[.5:.95]); all **[verified]**, Table 1 (CVPRW 2023, PBVS).

| Method | Venue/year (tier) | Detector | Input | AP | AP50 | AP75 | APS | APM | APL | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **YOLOv7-W6-Reasoner** | CVPRW 2023 (workshop) | YOLOv7-W6 + self-attention reasoner | raw 14-bit, 1280 | **40.5** | **58.5** | 44.8 | 37.2 | 47.8 | 94.9 | [tlgksy/ir-reasoner](https://github.com/tlgksy/ir-reasoner) | T1 |
| YOLOv7-W6 | (baseline) | YOLOv7-W6 | same | 39.5 | 57.5 | 44.0 | 35.9 | 49.1 | 94.7 | – | T1 |
| YOLOR-P6-Reasoner | CVPRW 2023 | YOLOR-P6 + reasoner | same | 39.4 | 57.5 | 44.2 | 35.2 | 50.8 | 93.6 | same | T1 |
| YOLOR-P6 | (baseline) | YOLOR-P6 | same | 36.4 | 53.1 | 39.9 | 32.2 | 49.1 | 92.9 | – | T1 |
| YOLOv4-P6-Reasoner | CVPRW 2023 | YOLOv4-P6 + reasoner | same | 29.1 | 44.9 | 31.0 | 24.5 | 43.8 | 92.9 | same | T1 |
| YOLOv4-P6 | (baseline) | YOLOv4-P6 | same | 26.3 | 40.4 | 28.2 | 21.5 | 41.9 | 91.8 | – | T1 |

Flags:
- The paper says only "pre-classified train, validation and test splits". That the test is VT is my inference (unverified).
- If it is VT, then 1,598 of 3,749 test "raw" frames are 8-bit AGC copies (05 Problem 7). Divided by 16,383, those frames are nearly black, and nothing in the paper suggests the authors handled them.
- APL ≈ 92–95% for every model is unusually high for this dataset.
- No per-class AP is reported.

---

## 2. Thermal-only, raw-TIFF subset and tone mapping: P4, P11

### 2a. P4-RAW2151: Thermal Chameleon (train 10,724 / test 2,151 raw-TIFF frames from the 6 Tau 2 videos; COCO metrics; 640×512)

Sorted by mAP within each detector. All **[verified]**, Table II (IEEE RA-L 2024). The class set is person + car, with `truck` and `other vehicle` relabelled as `car` in the shipped test JSON (05 Problem 2, own analysis).

| Method (tone map) | Venue/year (tier) | Detector | Input | mAP | AP50 | AP75 | APs | APm | APl | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **TCNet** | RA-L 2024 (journal, robotics top) | YOLOX | raw 14-bit → learned TM | **56.0** | **83.8** | 60.5 | 47.8 | 73.4 | 77.0 | [donkeymouse/ThermalChameleon](https://github.com/donkeymouse/ThermalChameleon) | T-II |
| FieldScale | (baseline) | YOLOX | raw → 8-bit | 54.0 | 82.2 | 59.8 | 47.8 | 73.0 | 75.8 | – | T-II |
| FLIR AGC (vendor 8-bit JPEG) | (baseline) | YOLOX | 8-bit AGC | 53.5 | 81.4 | 57.6 | 45.8 | 70.0 | 71.5 | – | T-II |
| TE(3) | (ablation) | YOLOX | raw | 53.3 | 80.9 | 58.2 | 44.8 | 72.0 | 73.9 | – | T-II |
| Min-max | (baseline) | YOLOX | raw | 50.8 | 81.6 | 53.9 | 43.1 | 67.5 | 71.7 | – | T-II |
| RAW (÷2¹⁴) | (baseline) | YOLOX | raw | 37.9 | 63.3 | 39.5 | 27.1 | 62.2 | 62.1 | – | T-II |
| **TCNet** | RA-L 2024 | Sparse R-CNN | raw | **45.7** | 70.5 | 49.8 | 36.1 | 67.5 | 69.1 | same | T-II |
| FLIR AGC | (baseline) | Sparse R-CNN | 8-bit | 37.5 | 60.9 | 41.0 | 29.2 | 55.0 | 63.4 | – | T-II |
| **TCNet** | RA-L 2024 | RetinaNet | raw | **34.7** | 53.3 | 37.4 | 22.7 | 64.1 | 67.7 | same | T-II |
| FLIR AGC | (baseline) | RetinaNet | 8-bit | 25.2 | 49.8 | 22.5 | 14.7 | 49.4 | 58.4 | – | T-II |

Flags:
- The merging of truck and other vehicle into car is not stated in the paper.
- The test JSON has 1,895 non-empty frames out of 2,151.
- No validation split is described, so checkpoint selection on test cannot be ruled out (unverified).
- No per-class AP is reported.
- The raw subset has no bike, motor or dog.

### 2b. P11-NAGC: Neural-AGC ("aligned union of v1+v2", ~12,000 frames 90/10; "video–test" 3,742 frames; detectors frozen after vendor-AGC pretraining)

All **[verified as printed]**, Tables 2, 3, 5 and 6 (ICCV 2025 Workshops, MIRA).

| Method (tone map) | Venue/year (tier) | Detector | Input | AP50 person | AP50 car | COCO AP (all "15 classes") | Code | Source |
|---|---|---|---|---|---|---|---|---|
| **Neural-AGC** | ICCVW 2025 (workshop) | Faster R-CNN R50-FPN (frozen) | 16-bit → learned AGC, 640×512 | **92.3** | **89.1** | – | [hosseinjavidnia/NAGC](https://github.com/hosseinjavidnia/NAGC) (README only, no code as of 2026-11) | T2 |
| Neural-AGC | ICCVW 2025 | RTMDet-s | same | 90.2 | 86.7 | – | same | T2 |
| Neural-AGC | ICCVW 2025 | YOLOX-m | same | 90.1 | 86.3 | **65.8** | same | T2, T3 |
| TCNet (re-implemented) | (baseline) | Faster R-CNN / YOLOX-m / RTMDet-s | same | 86.5/84.0 · 85.4/83.8 · 83.1/80.5 | (pairs = person/car) | 57.4 | – | T2, T3 |
| CLAHE-Adaptive | (baseline) | Faster R-CNN / YOLOX-m | same | 81.7/83.6 · 80.9/82.4 | | 52.9 | – | T2, T3 |
| Vendor-AGC | (baseline) | Faster R-CNN / YOLOX-m | 8-bit | 75.3/77.2 · 75.3/77.2 | | 46.3 | – | T2, T3 |
| RAW (percentile stretch) | (baseline) | Faster R-CNN / YOLOX-m | raw | 63.4/66.2 · 65.1/67.8 | | – | – | T2 |

**Strong caution, not comparable with anything else:**
1. The training data mixes v1 and v2.
2. The 3,742-frame test matches no official split.
3. "COCO AP over all 15 classes" is reported on a test split that contains at most 10 classes.
4. The **vendor-AGC row is identical (75.3 / 77.2) for Faster R-CNN and YOLOX-m and matches the FLIR README**, yet Sec. 4.4 says D_YOLO reaches **75.9** / 77.2.
5. 1,598 VT frames have no true raw data (05 Problem 7), which a raw-input method cannot benefit from. The paper does not mention this.
6. The +15 AP50 gain contradicts the community ablation (±1.3%) and Thermal Chameleon's +2.4 AP50.

---

## 3. Thermal-only, official val as the evaluation set: P5, P1/P15

### 3a. P5-VAL-2c: Uzun et al. (train = 90% of official train; **test = official val**; person + car; COCO-pretrained; 640×512)

Sorted by AP. All **[verified]**.

| Method | Venue/year (tier) | Detector | Input | AP | AP50 | AP75 | APS | APM | APL | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| **YOLOv8-x + turbulence aug.** | Sci Rep 2025 (journal, mid tier) | YOLOv8-x (backbone frozen) | 8-bit | **58.4** | 85.6 | 62.9 | 47.6 | 79.2 | 81.5 | none | T5 (clean test) |
| RTMDet-x + aug. | Sci Rep 2025 | RTMDet-x | 8-bit | 58.2 | 85.2 | 63.1 | 47.2 | 79.1 | 82.3 | none | T5 |
| **DINO-4scale + aug.** | Sci Rep 2025 | DINO R50 | 8-bit | 57.8 | **87.7** | 61.8 | 47.5 | 77.1 | 81.8 | none | T5 |
| YOLOv8-x (no aug.) | Sci Rep 2025 | YOLOv8-x | 8-bit | 57.8 | 85.0 | 62.4 | 47.0 | 78.9 | 80.0 | none | T5 |
| VfNet + aug. | CVPRW 2022 (workshop) | VfNet R50 | 8-bit | 54.7 | 83.8 | 57.1 | 43.7 | 75.2 | 79.9 | none | T1 (no turbulence) |
| YOLOR-P6 + aug. | CVPRW 2022 | YOLOR-P6 | 8-bit | 53.8 | 81.0 | 57.2 | 41.4 | 77.5 | 83.3 | none | T1 |
| VfNet (no aug.) | CVPRW 2022 | VfNet R50 | 8-bit | 52.4 | 81.9 | 54.8 | 41.1 | 73.4 | 76.8 | none | T1 |

Flag: AP here is the mean over 2 classes. No per-class AP is reported.

### 3b. P1 / P15-VAL-multi: multi-class YOLO evaluated on the official val (1,144 frames)

Sorted by mAP50. Class sets differ, so the rows are **not comparable with each other**.

| Method | Venue/year (tier) | Detector | Classes | Input | mAP50 | mAP50-95 | AP50 person | AP50 car | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|
| **MDCFVit-YOLO** | PLOS ONE 2025 (lower tier) | YOLOv8 + RepViT + MIA | "nine" claimed, 6 in table (person, bike, car, motor, bus, light) | 8-bit, YOLO default | **0.67** | **0.405** | **0.773** | **0.851** | none | Table 4 [verified] |
| CDYL | Sci Rep 2024 (mid tier) | YOLOv8l + CFC-DC | all present in val (11) | 8-bit | 56.4 | – | – | – | none | Table 4 [verified] |
| YOLOv8n* (retrained baseline in MS-YOLO) | IJCNN 2025 (major conf.) | YOLOv8n | 9 | 8-bit, 640 | **0.517** | **0.318** | – | – | – | Table III [verified] |
| MS-YOLO | IJCNN 2025 | YOLOv8n + MobileNetV4 + SlideLoss | 9 | 8-bit, 640 | 0.484 | 0.282 | 0.714 | 0.814 | none (Ultralytics) | Tables III, IV [verified] |

Flags:
- **MS-YOLO's proposed model is worse than its own retrained baseline** (0.484 vs 0.517).
- Evaluation is on val (Table IV header), although the paper's split table lists a 3,493-frame test.
- MDCFVit quotes **v1-like split counts** (train 8,862 / val 1,366 / test 3,674) but evaluates on 1,144 v2 val images. The paper is internally inconsistent.
- In Ultralytics pipelines, val is usually also used to select the checkpoint (unverified per paper).

---

## 4. Thermal-only, other protocols: P6, P7, P9, P10, P12, P13, P16, P17

### 4a. P6-ECDM-5c (5 classes; split source not stated; 512×640; COCO metrics)

All **[verified]**, Table 5 and Supplementary Table 3 (ACM MM 2024, top-tier conference).

| Method | Detector | mAP | AP50 | AP75 | AP50 person | AP50 car | Code | Source |
|---|---|---|---|---|---|---|---|---|
| **CenterNet + ECDM pseudo data** | CenterNet R50 | **27.3** | **52.1** | 24.5 | 60.3 | 74.6 | [lengmo1996/ECDM](https://github.com/lengmo1996/ECDM) (no FLIR loader) | T5, Supp. T3 |
| CenterNet (no pseudo data) | CenterNet R50 | 25.5 | 48.9 | 22.8 | 57.5 | 72.7 | – | same |
| Faster R-CNN + ECDM | R50 | 24.4 | 44.0 | 23.1 | 51.7 | 69.9 | – | same |
| Faster R-CNN | R50 | 23.5 | 42.7 | 22.6 | 50.4 | 66.9 | – | same |
| RetinaNet + ECDM | R50 | 15.2 | 30.8 | 13.2 | 39.6 | 58.8 | – | same |
| VFNet | R50 | 15.1 | 32.0 | 12.2 | 40.2 | 61.1 | – | same |
| DINO + ECDM | Swin-L | 9.2 | 20.8 | 6.8 | 39.3 | 40.3 | – | same |

Flag: the absolute values are far below the other protocols. DINO Swin-L at 7.6–9.2 mAP is clearly under-trained (batch size 2). The evaluation split is not stated. Bikes and medium/large objects are present, which rules out the raw-2151 subset.

### 4b. P7-UDA-SSDC (RGB→IR unsupervised domain adaptation; labelled RGB 5,663 → unlabelled IR 4,856 → **IR val 1,144**; 5 classes; DINO R50; short side 800)

Sorted by mAP. All **[verified]**, Table 1 (arXiv 2507.12017, preprint).

| Method | Venue (tier) | AP person | AP bicycle | AP car | AP sign | AP light | "mAP (COCO style)" |
|---|---|---|---|---|---|---|---|
| Oracle (DINO, IR labels) | – | 77.05 | 45.65 | 82.51 | 56.17 | 59.57 | 64.19 |
| **SS-DC** | arXiv 2025 (preprint) | 56.36 | 41.58 | 75.82 | 39.83 | 26.51 | **48.02** |
| D3T | CVPR 2024 (reproduced) | 52.75 | 41.87 | 71.78 | 30.99 | 17.80 | 43.04 |
| HT | CVPR 2023 (reproduced) | 54.19 | 41.97 | 70.89 | 29.05 | 16.48 | 42.52 |
| DATR | TIP 2024 (reproduced) | 52.38 | 38.13 | 69.59 | 32.84 | 16.94 | 41.98 |
| Semi-DETR | CVPR 2023 (reproduced) | 51.40 | 36.83 | 70.68 | 32.12 | 15.24 | 41.25 |
| Source only (DINO, RGB labels) | – | 39.32 | 32.53 | 57.97 | 20.05 | 9.20 | 31.81 |

Code and split: [xi666666/FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD).
Flag: the label says "COCO style", but the oracle car AP of 82.51 is close to the val-split car **AP50** of MS-YOLO (81.4) and MDCFVit (85.1) and far above their car AP50-95 (≈0.57–0.61). The metric is **probably AP50, mislabelled** (my assessment).

### 4c. P9 / P10: adversarial and backdoor work (clean-model accuracy only)

| Method | Venue (tier) | Protocol | Detector | Clean metric | Source |
|---|---|---|---|---|---|
| POD / Adv-POD (defence) | ICIP 2024 (major conf.) | P9 variant: person, height >120 px, **84 test images** | YOLOv5 | AP@0.5 0.9360 / **0.9395** (std. training 0.9199) | Table 1 [verified] |
| HOTCOLD victim model | AAAI 2023 (top) | P9: person >120 px, 377 test | YOLOv5, COCO-pretrained | AP **94.8%** (IoU not stated) | Sec. 4 text [verified] |
| Physical Backdoor clean models | CVPR 2024 (top) | P10: "Flir_v2_T" 13,460 images, split unclear | YOLOv3 / YOLOv5 / Faster R-CNN | "mAP" person/car **84.40/85.80** · 79.10/82.50 · 51.65/71.56 (IoU not stated) | Table 1 [verified] |

These are not detection SOTA. They are easy subsets (P9) or metrics without a stated definition (P10), and they must not be compared with P0 or P14.

### 4d. P12 / P13: representation-learning and VRU papers

| Method | Venue (tier) | Protocol | Result | Source |
|---|---|---|---|---|
| Le MuMo JEPA (fine-tune) | CVPRW 2026 (workshop) | P12: frozen-encoder CenterNet probe | mAP50 **2.39**, car mAP50 12.88 (units as printed) | Table 5 [verified] |
| YOLOv8m thermal, class-weighted loss | arXiv 2025 (preprint) | P13: custom VRU class map, evaluated on val | mAP50 **0.724**, mAP50:90 0.442 | Tables 9, 11 [verified] |

Flags:
- Le MuMo JEPA is a linear-probe-style evaluation and is not a detector SOTA.
- In VRU Table 11, the "Thermal" row repeats the "RGB Aug" values in 3 of its 4 columns (0.442 / 0.780 / 0.638), which looks like a copy error. Its Table 8 columns also appear swapped relative to Table 7.

### 4e. P16-UNSTATED: official split exists, evaluation split not stated

| Method | Venue/year (tier) | Detector | Classes | Input | mAP50 | mAP50-95 | Notes | Source |
|---|---|---|---|---|---|---|---|---|
| Class-pruning curation, YOLOv11-L | IEEE eIT 2026 (low tier) | YOLOv11-L | 7 (pruned using **performance curves**) | 8-bit | **72.754%** | 47.536% | The 10-class YOLOv11-L gets 59.53 / 33.82. Class choice is circular (05 Problem 2) | Table III [verified] |
| Perimeter Security, YOLOv8m @896 | Sensors 2025 (MDPI, lower tier) | YOLOv8m | 6 (car, person, truck, motorcycle, bus, bicycle) | 8-bit, 896 | 0.692 | 0.475 | Trained jointly on FLIR + HIT-UAV + fire datasets; "official split" used, but which one is not said | Table 3 [verified] |
| YOLO-APDM | Sensors 2024 (MDPI, lower tier) | YOLOv8n + ASF-P2 + DCNv3 + MSCA | 8 (person, bike, car, motor, bus, light, hydrant, sign) | 8-bit, 640 | 0.639 | 0.391 | Includes bus (0 VT boxes), so the evaluation is probably on val (unverified) | Tables 6, 8 [verified] |

### 4f. P17-CUSTOM: author-made splits or mixed data (not comparable; listed for completeness)

| Method | Venue/year (tier) | Split | Classes | mAP50 | mAP50-95 | Source |
|---|---|---|---|---|---|---|
| Zha et al. | IET-IP 2024 (lower tier) | 10,742 / 2,292 test (matches no known split) | ? | "78.6% mAP" | – | from 02 (paywalled, **unverified**) |
| LIWL-YOLO | Sci Rep 2025 (mid tier) | "FLIR-WSL" = v2 + the authors' water-surface images | <100-sample classes dropped | "69%" | – | abstract (unverified detail) |
| RADFRNet | Sensors 2026 (MDPI) | study-specific 7,334 / 1,048 / 2,096 subset | 8 (sign, hydrant dropped) | 0.653 | 0.394 | ablation table [verified] |
| TCLNet | PLOS ONE 2026 (lower tier) | random 7:2:1 re-split of 11,886 train+val images (8,321 / 2,377 / 1,188) | 10 | 45.4 | 23.91 | Table 4 [verified] |

Random frame-level re-splits of video-derived frames almost certainly leak near-duplicate frames between train and test (05 Problem 3).

---

## 5. RGB-only, RGB-T and open-vocabulary

### 5a. RGB-only on v2

| Method | Venue (tier) | Protocol | Result | Source |
|---|---|---|---|---|
| FLIR baseline (RGB) | README (official) | P0, RGB VT 3,749 | AP50 person **51.42**, car **55.79** | README [verified in 03] |
| YOLO Phantom (mixed training) / YOLOv8n RGB-only | ICIP 2024 | P2, RGB "test 3,493", 4 classes | mAP50-95 **19.72** / 19.65 | MODIPHY Table I [verified] |
| YOLOv8m RGB | arXiv 2025 | P13 VRU map, val | mAP50 0.620 | VRU Table 11 [verified] |

No other RGB-only v2 result was found. Thermal beats RGB on every shared protocol: P0 75.33/77.23 vs 51.42/55.79, and P2 25.21 vs 19.65.

### 5b. RGB-T fusion on v2

**None found.** The v2 pairs are not registered (05 Problem 4), and fusion papers use FLIR-aligned (v1). The only RGB-T uses of v2 are the SSL probe (P12), late fusion without a fused metric (VRU), and registration (XCalib).

### 5c. Open-vocabulary / zero-shot: P8 (Thermal-Det, CVPR 2026; Grounding-DINO Swin-T; 800×800; split and class list not stated)

Sorted by AP on FLIR-V2. All **[verified]**, Table 1 and Table S2.

| Method | Venue (tier) | FLIR-V2 AP | AP50 | AP75 | (FLIR-aligned AP / AP50) | Code |
|---|---|---|---|---|---|---|
| G-DINO fine-tuned (supervised reference, not zero-shot) | – | 0.200 | 0.345 | 0.203 | 0.414 / 0.749 | – |
| **Thermal-Det (zero-shot)** | CVPR 2026 (top) | **0.096** | **0.173** | 0.091 | 0.372 / 0.664 | not released |
| G-DINO (zero-shot) | ECCV 2024 | 0.081 | 0.144 | 0.078 | 0.337 / 0.636 | – |
| LLMDet | CVPR 2025 | 0.048 | 0.075 | 0.051 | 0.359 / 0.628 | – |
| MM-GDINO | – | 0.042 | 0.060 | 0.048 | 0.354 / 0.619 | – |
| T-Rex2 | ECCV 2024 | 0.033 | 0.048 | 0.036 | 0.276 / 0.514 | – |
| YOLO-World | CVPR 2024 | 0.029 | 0.044 | 0.032 | 0.266 / 0.487 | – |
| GLIP | NeurIPS 2022 | 0.025 | 0.041 | 0.028 | 0.251 / 0.471 | – |

Per class, Thermal-Det zero-shot on FLIR-V2 reaches person AP 0.366 (AP50 0.56) vs 0.512 for G-DINO FT, car AP 0.35, and motorcycle and traffic light 0.006 each (Sec. 4.2 [verified]).

---

## 6. Flags: non-comparable or suspicious numbers

| Paper | Issue | Severity |
|---|---|---|
| **Neural-AGC (P11)** | v1∪v2 training data; 3,742-frame "video-test"; "15-class COCO AP" on a test split with ≤10 classes; identical vendor-AGC numbers for two detectors that copy the README (Sec. 4.4 says 75.9); raw-input gains on frames that have no raw data; compares directly with the README baseline (**unmatched baseline**) | **High: do not cite as SOTA** |
| **MS-YOLO (P1)** | Reported on **val 1,144**, not test (Table IV header); proposed model is below its own baseline; val likely also used for checkpoint selection | High (protocol mislabelled) |
| **MDCFVit-YOLO (P15)** | Quotes v1 split counts while evaluating on 1,144 v2 val images; says 9 classes, table shows 6 | Medium |
| **Thermal Chameleon (P4)** | Silent truck/other-vehicle → car merge in the test JSON; 1,895 non-empty frames; no val split described | Medium |
| **IR Reasoner (P3)** | Test split not named; if VT, 43% of the "raw" test frames are fake 8-bit; APL ≈ 95% for all models | Medium |
| **SS-DC (P7)** | "COCO-style mAP" whose magnitude matches AP50 (**likely a mislabelled metric**) | Medium |
| **Physical Backdoor (P10), HOTCOLD (P9)** | "mAP" / "AP" with unstated IoU; split unclear (P10); easy subset of large persons (P9) | Medium |
| **Fs2PA (P14)** | "Relative gain" is actually absolute; bike+motor merged; from scratch (weak baselines); single seed; no code; lower-tier venue | Low–medium (protocol itself is clean) |
| **eIT'26 (P16)** | Classes pruned using PR/F1 curves (on the reporting split?); split unstated | Medium (circular class selection) |
| **YOLO-APDM, Perimeter (P16)** | Evaluation split unstated; bus included (0 VT boxes), which suggests val; Perimeter trains on mixed datasets | Medium |
| **VRU (P13)** | Duplicated cells in Table 11; columns appear swapped in Table 8; val-only | Medium |
| **ECDM (P6)** | Split unstated; DINO under-trained | Low (relative gains only) |
| **TCLNet, RADFRNet, LIWL, Zha (P17)** | Custom or random splits, mixed data, paywalled | Not comparable |
| **Le MuMo JEPA (P12)** | Probe metric with unclear units; not a detector SOTA | n/a |
| General | "mAP" means AP50 in YOLO papers and AP@[.5:.95] in COCO papers. Class sets range from 2 to 11. 8-bit vs raw input. VT vs val vs raw-2151 test sets | — |

---

## 7. Cross-protocol summary on the one shared metric: AP50 for person and car

For orientation only. **Rows with different test sets or class sets are not comparable**, even though they share the metric name.

| Method | Protocol (test set) | AP50 person | AP50 car | Comparable with P0? | Source |
|---|---|---|---|---|---|
| **Fs2PA** (YOLOv11n-based, scratch) | P14 (**VT 3,749**) | **77.16** | **81.83** | Nearly (same test frames; different class set and pretraining) | Fs2PA T2 [verified] |
| FLIR YOLOX-m | **P0 (VT 3,749)** | 75.33 | 77.23 | reference | README [verified] |
| YOLOv11n (Fs2PA baseline) | P14 (VT 3,749) | 72.43 | 74.88 | Nearly | Fs2PA T2 [verified] |
| Neural-AGC + Faster R-CNN | P11 (3,742, v1∪v2) | 92.3 | 89.1 | **No** | NAGC T2 [verified as printed] |
| Neural-AGC + YOLOX-m | P11 | 90.1 | 86.3 | **No** | NAGC T2 |
| MDCFVit-YOLO | P15 (val 1,144) | 77.3 | 85.1 | No (val) | T4 [verified] |
| MS-YOLO | P1 (val 1,144) | 71.4 | 81.4 | No (val) | T IV [verified] |
| CenterNet + ECDM | P6 (split unstated) | 60.3 | 74.6 | No | Supp. T3 [verified] |
| Physical Backdoor YOLOv3 (clean) | P10 ("mAP", split unclear) | 84.40 | 85.80 | No (metric unclear) | T1 [verified] |
| Thermal-Det zero-shot | P8 | 0.56 (0–1 scale) | n/r (AP 0.35) | No (zero-shot, all classes) | Sec. 4.2 [verified] |
| FLIR YOLOX-m (RGB) | P0-RGB (RGB VT) | 51.42 | 55.79 | RGB reference | README |

**Reading:**
- On the official VT test, the best verified person/car AP50 is about **77 / 82** (Fs2PA), roughly +2 / +5 points over FLIR's 2022 YOLOX-m reference.
- Numbers in the high 80s and low 90s exist only under non-standard protocols (NAGC), on val, or with an unclear metric.
- Car AP50 is consistently higher than person AP50.

---

## 8. Practical SOTA guidance for this project

- **To compare against SOTA on the official v2 test,** report person and car AP50 on VT 3,749 (P0 metric). Also report mAP50 and mAP50:95 for the 5-class set (person, bike, car, light, sign) on VT, and on the raw-2151 subset if raw data is used. The values to beat are FLIR YOLOX-m (75.33 / 77.23) and Fs2PA (77.16 / 81.83, different class set).
- **For raw / tone-mapping work,** P4 (Thermal Chameleon's test JSON) is the only reusable raw protocol: YOLOX + TCNet 56.0 mAP / 83.8 AP50. If you reuse the JSON, state the truck/OV→car merge.
- **For zero-shot / open-vocabulary work,** the target is Thermal-Det (0.096 AP / 0.173 AP50), and the supervised G-DINO FT reference is 0.200 / 0.345. The v2 split and class list must be guessed, because the paper does not state them.

---

## 9. Contrast only: FLIR-aligned (v1) SOTA range (not researched in depth; another agent covers v1)

- From the PwC archive numbers already collected in 03 §9.1 (aligned v1, 4,129/1,013, 3 classes), **RGB-T fusion** reaches roughly **77.7–86.4 mAP50 and ~44–50.5 mAP@[.5:.95]**. Examples: CFT 77.7, RSDet 83.9 / 43.8, UniRGB-IR 81.4 / 44.1, MiPa 81.3 / 44.8, CAFF-DINO 85.5 / 50.5, MMPedestron 86.4.
- **Thermal-only** aligned results seen in passing: CGDet (arXiv 2511.01435) 82.8 mAP50 / 47.1 mAP **[seen in a search snippet; aligned confirmed from its text, number not re-checked]**.
- Thermal-Det's zero-shot AP is about 4× higher on aligned than on v2 (0.372 vs 0.096). **Aligned-v1 numbers are much higher than v2 numbers and must never be mixed with them.**

---

## 10. Uncertainties

1. **Evaluation splits are often unstated.** This holds for IR Reasoner, ECDM, Thermal-Det, Physical Backdoor, YOLO-APDM, eIT'26 and Perimeter Security. I assigned them to protocols by inference (class counts, presence of bus or bike).
2. **Fs2PA** was read in the Europe PMC full-text XML; the MDPI site returned 403. The numbers are verified there, but no code or split files exist to reproduce them.
3. **MODIPHY's evaluation set:** the Ultralytics `benchmark` may have run on the YAML `val` entry rather than VT (unverified).
4. **SS-DC's metric** (AP50 vs AP@[.5:.95]) is my inference.
5. **NAGC:** I report the numbers as printed. Their internal consistency is doubtful (see §6).
6. **Zha et al. (IET-IP'24)** and **LIWL-YOLO** were not read in full (paywall, mixed data). Their numbers are unverified.
7. **Search coverage.** Searches covered the web, arXiv and Europe PMC (MDPI, PLOS, Sci Rep). IEEE Xplore, Elsevier and Chinese-language journals were not searched systematically, so further lower-tier P16/P17-style YOLO papers almost certainly exist. Every arXiv 2025–26 candidate I checked (2601.09228, 2505.10825, 2508.10445, 2309.16592, 2511.01435) turned out to use FLIR-aligned, not v2.
8. **No paper reports results on more than one v2 test variant** (VT vs raw-2151 vs val), so the size of the gaps between protocols on v2 is known only from the community audit (05).

---

## References

- FLIR ADAS v2 README (baseline): https://adas-dataset-v2.flirconservator.com/dataset/README.txt
- Fs2PA, *Sensors* 26(7):2257, 2026, doi:10.3390/s26072257: https://www.mdpi.com/1424-8220/26/7/2257 (full text via Europe PMC PMC13075273)
- IR Reasoner, CVPRW 2023: https://openaccess.thecvf.com/content/CVPR2023W/PBVS/html/Gundogan_IR_Reasoner_Real-Time_Infrared_Object_Detection_by_Visual_Reasoning_CVPRW_2023_paper.html · https://github.com/tlgksy/ir-reasoner
- Thermal Chameleon, IEEE RA-L 2024: https://arxiv.org/abs/2410.18340 · https://github.com/donkeymouse/ThermalChameleon
- Neural-AGC, ICCVW 2025: https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html · https://github.com/hosseinjavidnia/NAGC
- Uzun et al., CVPRW 2022: https://arxiv.org/abs/2204.08745 · Sci Rep 2025: https://arxiv.org/abs/2405.06383
- MS-YOLO, IJCNN 2025: https://arxiv.org/abs/2509.21696
- MODIPHY / YOLO Phantom, ICIP 2024: https://arxiv.org/abs/2402.07894
- ECDM, ACM MM 2024: https://arxiv.org/abs/2408.03748 · https://github.com/lengmo1996/ECDM
- SS-DC, arXiv 2025: https://arxiv.org/abs/2507.12017 · https://github.com/xi666666/FLIR-ADAS-UDAOD
- Thermal-Det, CVPR 2026: https://arxiv.org/abs/2605.10130
- HOTCOLD Block, AAAI 2023: https://arxiv.org/abs/2212.05709 · https://github.com/weihui1308/HOTCOLDBlock
- POD, ICIP 2024: https://arxiv.org/abs/2309.15519
- Physical Backdoor, CVPR 2024: https://arxiv.org/abs/2404.19417
- Le MuMo JEPA, CVPRW 2026: https://arxiv.org/abs/2603.24327
- VRU multi-modal detection, arXiv 2025: https://arxiv.org/abs/2509.06333
- MDCFVit-YOLO, PLOS ONE 2025: https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0324700 (PMC12173352)
- CDYL, Sci Rep 2024, doi:10.1038/s41598-024-54146-1 (PMC10861438)
- YOLO-APDM, *Sensors* 24:7197, 2024, doi:10.3390/s24227197 (PMC11598205)
- Perimeter Security Utilizing Thermal Object Detection, *Sensors* 2025 (PMC12610692)
- RADFRNet, *Sensors* 26:4850, 2026, doi:10.3390/s26154850 (PMC13468968)
- TCLNet, PLOS ONE 2026, doi:10.1371/journal.pone.0361114 (PMC13690208)
- LIWL-YOLO, Sci Rep 2025 (PMC12572292)
- Two-stage class pruning, IEEE eIT 2026: https://ecasp.ece.iit.edu/publications/2012-present/EIT2026_240.pdf
- Zha et al., IET Image Processing 2024: https://ietresearch.onlinelibrary.wiley.com/doi/full/10.1049/ipr2.13053
- CGDet (aligned v1, contrast): https://arxiv.org/abs/2511.01435
- Papers with Code archive (aligned-v1 leaderboards): https://huggingface.co/datasets/pwc-archive/evaluation-tables
