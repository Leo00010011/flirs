# FLIR ADAS v1: deep learning tasks (Q5) and established benchmarks and protocols (Q9)

**Research date:** 2026-11-02
**Scope:** The Teledyne FLIR ADAS thermal dataset **v1** (v1.3, `FLIR_ADAS_1_3`) and its two derivatives:
- **FLIR-aligned** (Zhang et al., ICIP 2020): 5,142 RGB-T pairs, 4,129 train / 1,013 test.
- **RoadScene** (Xu et al., FusionDN AAAI 2020): 221 registered pairs.

I keep these three variants apart throughout. v2 (2022) appears only where a paper is easy to confuse with v1.
**Builds on:**
- `01_dataset_overview.md`: splits, classes and registered versions.
- `02_usage_and_publications.md`: the 48-paper seed table, cited below as "[#n]" for row n of its table 3.1.

**Method:** I read the PDFs of about 45 seed and related papers and grepped their dataset and protocol sections. Section 2.2 also checks repo configs and a leaderboard mirror (OpenCodePapers).

I recounted the FLIR-aligned XML annotations from `aligned.zip` myself; these numbers are marked **(counted)**. Claims I could not check against a primary source are marked **(unverified)** or **(inferred)**.

---

## TL;DR

- **Tasks, by how much each is used:**
  1. **RGB-T (multispectral) object detection on FLIR-aligned** is clearly the mainstream task (about 25 of the 48 seed papers).
  2. **Infrared-visible image fusion (IVIF) on RoadScene** is mainstream inside the fusion community. RoadScene is a fixed test set next to TNO and MSRS.
  3. **Thermal-only detection** is the next most used. It comes in two flavours: the official v1 val split (older work from 2019 to 2021) and the thermal half of FLIR-aligned (2023 onward).
  4. **Niche:** RGB→thermal domain adaptation, thermal↔RGB translation (night thermal → day colour; IR→RGB for detection), physical adversarial attacks (person subsets of v1.3), guided super-resolution, and segmentation. v1 has no segmentation labels; the only masks are the small PearlGAN set.
- **No official benchmark or leaderboard exists.**
  - FLIR only shipped a RefineDet512 baseline: val mAP@0.5 = 0.587 over 3 classes, in the v1.3 README.
  - Papers with Code once hosted *"Multispectral Object Detection on FLIR"* (the FLIR-aligned protocol) and *"Image-to-Image Translation on FLIR"*. That site now redirects to Hugging Face. Its data survives in the community mirror **OpenCodePapers**; the top entries are MMPedestron (86.4 mAP50), RGB-X (86.16 / 47.1) and CAFF-DINO (85.5 / 50.5).
- **De facto detection protocol (P1):** FLIR-aligned, train 4,129 / test 1,013 pairs, 3 classes (person, car, bicycle), dog removed.
  - Metrics are COCO-style mAP50, mAP75 and mAP@[.5:.95]. Inputs are RGB + thermal.
  - **Resolution is not standardised:** native 640×512, 640 letterbox (train 640×640, test 640×512), 640×640 resize, or upscaled.
  - The test set has 1,013 images and **8,591 boxes**: car 4,124, person 4,107, bicycle 360. It also has **13 dog boxes in 13 images**, and no empty images (counted).
- **Older, separate thermal-only protocol (P4, "MMTOD-style"):** official v1 train 8,862 / val 1,366, 3 classes, AP@0.5, thermal only. Its numbers (about 58–80 AP50) are **not** comparable with P1.
- **Main inconsistencies:**
  - Tables mix results across protocols. ProbEn compares its own 1,258-image val results with CFR/GAFF results from the 1,013-image aligned test. CSSA re-ran ProbEn on FLIR-aligned and got 75.5, against ProbEn's own 83.76 on official val.
  - "mAP" sometimes means AP50 (CFR's "mAP 72.39%" is AP50, as GAFF points out).
  - Handling of dog varies: most drop it, MiPa keeps it as a 4th class.
  - Input resolution, evaluator (YOLOv5 101-point AP vs pycocotools) and confidence/NMS settings differ.
  - Split sizes get misquoted: MMPedestron writes "4,192 training pairs", and two different splits are reported for the same custom v1 alignment.

---

## Q5. Deep learning tasks that use v1 and its derivatives

### 5.0 Overview

| Task | Status | Main variant | How the data is used | Representative papers |
|---|---|---|---|---|
| RGB-T fusion detection | **Mainstream**, growing | FLIR-aligned | Paired 640×512 RGB+T inputs. Thermal boxes reused for both modalities. 3 classes. Test on the 1,013 pairs. | CFR [#4], GAFF [#6], CFT, CSSA [#21], ICAFusion [#25], DAMSDet [#29], RSDet [#37], Fusion-Mamba [#36], TFDet [#33], WaveMamba [#40], M2D-LIF [#41], MS2Fusion [#44], IRDFusion [#45], ZPA-MDETR [#48] |
| RGB-T detection on *unaligned* or custom v1 | Niche | Official v1 / custom alignment | Unaligned pairs with thermal labels (ProbEn), or the authors' own re-registration (Vadidar IV'22) | ProbEn [#15], Vadidar [#16], "Rethinking Early-Fusion" TIV'25 [#38], CVPR'26 Distribution-Aligned [#47] |
| Thermal-only detection | **Mainstream (older)**, declining on raw v1 | Official v1 (val 1,366); FLIR-aligned thermal half (2023+) | 8-bit thermal JPEGs, 3 classes | FLIR RefineDet baseline, MMTOD [#1], SSTN [#9], ODSC (Munir), ThermalDet, DDSSD, TIRDet [#23], PFGF [#39] |
| IVIF (pixel-level image fusion) | **Mainstream in the fusion community** | RoadScene; sometimes FLIR-aligned | Unsupervised fusion of registered pairs. RoadScene is mostly a test set. Quality metrics, no labels. | FusionDN [#2], U2Fusion [#12], DIDFuse [#3], AUIF [#11], TarDAL [#13], CDDFuse [#19], DDFM [#20], MoE-Fusion [#18] |
| Domain adaptation (RGB→thermal) | Niche–moderate | Official v1 (classification crops); FLIR-aligned (detection) | Source: COCO or FLIR RGB. Target: FLIR thermal. | SGADA [#8], STGADA (CVPRW'23), SSTN [#9], Munir (ODSC), D3T [#26] |
| Image/modality translation | Niche | Official v1 (night thermal → day colour); FLIR-aligned (IR→RGB for detection) | Unpaired day-RGB / night-IR sets, or paired aligned sets | PearlGAN [#10], MornGAN, FoalGAN [#34], MMTOD [#1], HalluciDet [#27], ModTr [#30], IRFormer |
| Modality-agnostic / generalist detection | Niche, recent | FLIR-aligned | Train on mixed RGB and IR, test on each modality | MiPa [#35], MMPedestron [#31], UniRGB-IR [#43], M-SpecGene [#42] |
| Adversarial attacks | Niche (mostly one Beihang group plus Tsinghua) | Person subsets of v1.3; FLIR-aligned backgrounds | Digital and physical patches against YOLOv3 thermal person detectors; RGB-T clothing | Bulbs AAAI'21 [#7], Invisible Clothing CVPR'22 [#14], Learnable patches CVPR'23 [#17], RGBT-Clothing CVPR'26 [#46] |
| Guided super-resolution | Niche | Official v1 (rectified pairs) | RGB-guided ×4/×8 thermal SR | Gupta & Mitra ECCVW'20 [#5] |
| Semantic segmentation | **Very niche** (v1 has no masks) | Official v1 night subset; ad hoc | Pixel masks hand-annotated by translation papers, used only to evaluate translations | PearlGAN [#10] masks, FoalGAN; EPOFusion IVOE (arXiv 2026) |
| Weakly aligned / robustness detection | Niche | FLIR-aligned plus synthetic shifts | Random IR offsets of ±10 px ("FLIR_w") | CF-Deformable DETR [#32] |
| VLM evaluation on thermal | Emerging | Probably **v2** | Modality recognition and person counting | ThermEval (arXiv 2602.14989) cites the current v2 page, so it is likely v2 (inferred) |

### 5.1 RGB-T fusion detection (mainstream)

- **Data use:** The registered FLIR-aligned pairs (RGB and thermal, both 640×512) are fed to a two-stream detector with a fusion module. The thermal VOC boxes serve as labels for both modalities.
  - Train on `align_train.txt` (4,129 pairs) and test on `align_validation.txt` (1,013 pairs).
  - Train pairs come from the original v1 `train` IDs and test pairs from `val` IDs, so there is no leakage [01, §4.2].
- **Classes:** person, car, bicycle. Dog is removed. RSDet, M2D-LIF, WaveMamba, CSSA and ModTr say so explicitly.
- **Detector families:**
  - YOLOv5 two-stream (CFT codebase), used by ICAFusion, Fusion-Mamba, WaveMamba and MS2Fusion.
  - Faster R-CNN (CFR/GAFF use VGG16/ResNet18 RetinaNet; CSSA and RSDet use Faster R-CNN).
  - DETR family: DAMSDet (RT-DETR-like), IRDFusion (Co-DETR), ZPA-MDETR (RT-DETR), CF-Deformable DETR.
  - Foundation or adapter models: UniRGB-IR, M-SpecGene.
- **Since 2024 the usual reporting set is FLIR-aligned + LLVIP + M3FD**, sometimes with KAIST, VEDAI or DroneVehicle [02 §2.3].
- **Typical numbers (P1):**
  - CFR / GAFF (2020–21): about 72–73 AP50.
  - CFT (2021): 78.7 / 35.5 / 40.2 (AP50 / AP75 / mAP).
  - ICAFusion: 79.2 AP50.
  - 2024–2026 methods: about 80–86 AP50 and about 41–50 mAP.

  The exact SOTA list is left to group D (`04_sota_detection.md`).

### 5.2 Thermal-only detection

Two protocols exist and should not be mixed:

1. **Official v1, MMTOD-style (P4).**
   - Thermal 8-bit images, train 8,862 / val 1,366, 3 classes, AP@0.5 per class and their mean.
   - Lineage: FLIR's own README baseline (RefineDet512, 0.587) → MMTOD (Faster R-CNN; baseline about 53–58, MMTOD-UNIT 61.54) → ODSC/Munir (69.62) → ThermalDet (74.60) → BU (73.1–73.2). Values are as tabulated in ProbEn Table 4.
   - SSTN (IROS'21) uses the same 8,862/1,366 split with mAP@0.5.
   - DDSSD uses it with SSD300 and COCO-style APs, including small-object AP.
   - MMTOD also defines data-efficiency subsets **FLIR(1/2)** and **FLIR(1/4)** (half or a quarter of the training images).
   - ProbEn's own thermal-only Faster R-CNN reaches **79.24** on the 1,258-image RGB-paired val (P5).
2. **Thermal half of FLIR-aligned (P2), 2023 onward.**
   - Same 4,129/1,013 split, IR input only at test time.
   - TIRDet and PFGF (CVPR'25) use 640×640 and mAP50 / mAP75 / mAP.
   - The CFT paper's YOLOv5 thermal-only baseline gives 73.9 / 35.7 / 39.5.
   - Many fusion papers report "IR-only" baselines on this protocol.

**Trend:** Thermal-only work on the *raw* release is moving to v2. HOTCOLD (AAAI'23) says so explicitly, and "Infrared Reasoner" (CVPRW) uses v2 with 5 classes [02 §2.3].

### 5.3 Image fusion (IVIF)

- **RoadScene:**
  - 221 cropped, registered pairs from FLIR v1 frames. File names keep the v1 IDs, e.g. `FLIR_05164`. Thermal noise was reduced and the pairs cropped to the registered region, so image sizes vary [RoadScene README].
  - No labels and no official split.
  - Used for unsupervised fusion training and/or as a generalisation test set. Many papers call it simply "FLIR" (DIDFuse, AUIF).
- **Training/test conventions (section 9.3)** range from training on 180 RoadScene pairs to training on MSRS and testing on 50 RoadScene pairs (CDDFuse).
- **FLIR-aligned as a fusion set:** MoE-Fusion (ICCV'23) fuses FLIR-aligned and reports fusion metrics (EN, SF, MI, VIF, …) plus detection mAP on the 1,013 test pairs ("fusion-then-detect").
- **2026 example:** EPOFusion re-registers its own FLIR-ADAS pairs for an over-exposure benchmark, IVOE: 201 training pairs and 277 real-world test pairs with boxes and masks. The source version is not stated, but the footnote links the old v1 URL (unverified).

### 5.4 Domain adaptation

- **Classification DA (SGADA, CVPRW'21; STGADA, CVPRW'23):**
  - MS-COCO RGB crops → FLIR thermal crops of bicycle, car and person.
  - Crops are made with `utils/prepare_dataset.py` from the official train and val JSONs and resized to 224.
  - STGADA's crop counts for FLIR: bicycle 4,137, car 43,734, person 26,294.
  - Metric: per-class and mean classification accuracy on FLIR val crops.
  - This is a *classification* benchmark and has nothing to do with detection mAP.
- **Detection DA, RGB→thermal:**
  - **D3T (CVPR'24)** builds a new protocol on FLIR-aligned: the *first 2,064 RGB images* as source and a *disjoint set of 2,064 thermal images* as target, all from the 4,129 train pairs, so no matched pairs are used. Test on the 1,013 thermal images.
  - Classes person/bicycle/car; FCOS with VGG16; short side resized to ≤800. It reports per-class AP and mAP: Source-only 34.68, HT 65.81, D3T 69.30.
  - The IoU threshold is not stated; probably 0.5, VOC-style, following HT (unverified).
  - SSTN (contrastive self-supervision) and Munir/ODSC (style-transfer DA) evaluate on the official v1 val (P4).
- **Not FLIR:** Meta-UDA (WACV'22) does not use FLIR, despite being cited as if it did [02 §3.3].

### 5.5 Image and modality translation

- **Night thermal → day colour (NTIR2DC), the PearlGAN family (T-ITS'22) → MornGAN → FoalGAN (TCSVT'24):**
  - Built from the *official v1*: 5,447 daytime RGB images (domain A) and 2,899 night thermal images (domain B) for training; **490 night thermal images from val** for testing.
  - The 490 matches ProbEn's 490 "night" val frames, but no shared tagging is documented (inferred).
  - Images are resized to 500×400 and centre-cropped to 360×288. The selection lists are in `PearlGAN/img_list/`.
  - Metrics: mIoU on a hand-annotated subset (segmentation of translated images), mAP of a COCO-pretrained YOLOv4 on translated images (person, bicycle, car), and **APCE**, a new edge-consistency metric.
  - FID is mentioned but rejected as insufficient. TIRDet and PFGF reuse the PearlGAN `FLIR_NTIR2DC` checkpoint as a translation prior.
- **Pseudo-RGB for detection:** MMTOD (CycleGAN/UNIT thermal→RGB) on official v1. HalluciDet (WACV'24) and ModTr (ECCV'24) learn IR→RGB-like translations that are optimised for a frozen RGB detector, on FLIR-aligned (P2b/P2). They are judged by detection AP, not image metrics.
- **RGB→IR, judged by image similarity:**
  - IRFormer (arXiv 2404.07072) reports PSNR/SSIM on 256×256 crops of "5,142 sets of images" that it claims to have aligned automatically. The count is exactly FLIR-aligned's, so it is probably that set (inferred).
  - These numbers make up the "Image-to-Image Translation on FLIR" leaderboard: Pix2Next 23.45 / 0.66, IRFormer 17.74 / 0.48.
  - The split is not documented (unverified).

### 5.6 Adversarial attacks

- **Physical patches against YOLOv3 thermal person detectors, built from person-only subsets of v1.3:**
  - "FLIR person select": 710 train / 301 test (AAAI'21 bulbs).
  - **PEOPLE_FLIR**: the 9,900 v1.3 images containing people, split 7,873 / 2,027 (CVPR'22 invisible clothing; input 416×416; clean test AP 85.01%).
- **Correction to file 02:** The CVPR'23 learnable-shape patches paper [#17] uses the same **7,873 / 2,027** person split, so it is **v1**, not ambiguous. It keeps only test images that the model detects with high confidence ("clean AP is 100%").
- **Metrics:** AP drop and attack success rate (ASR).
- **RGB-T attacks:** RGBT-Clothing (CVPR'26) renders adversarial clothing onto FLIR-aligned backgrounds and attacks ProbEn-style early, mid and late fusion detectors and YOLOv11. ASR uses IoU 0.5 and confidence 0.6.

### 5.7 Super-resolution

- Gupta & Mitra (ECCVW'20 PBVS) rectify the unaligned official v1 RGB-T pairs with homographies estimated on one image. Both modalities end up at 512×640.
- Training data: 43,830 pairs, inflated by blur-kernel augmentation (inferred). Test data: 1,257 pairs, which equals the val RGB count.
- ×4 and ×8 guided SR from 64×80 thermal inputs. Metrics PSNR / SSIM (LPIPS mentioned).
- No other top-venue FLIR v1 SR paper is in the seed list. The PBVS thermal-SR challenges use their own data (unverified).

### 5.8 Segmentation

- v1 has **no** segmentation ground truth. The `segmentation` field in the JSON just repeats the box [01 §1.3].
- The only masks are the PearlGAN pixel-level annotations of a FLIR night-thermal subset, used to score translations (mIoU), and EPOFusion's 277 IVOE masks.
- RGB-T segmentation papers in the seed list (MS2Fusion, UniRGB-IR) segment MFNet / PST900 / FMB and use FLIR only for detection.

---

## Q9. Established benchmarks and protocols

### 9.1 Official benchmark or leaderboard?

| Source | What exists | Status (checked 2026-11-02) |
|---|---|---|
| Teledyne FLIR | **No leaderboard.** The v1.3 README only gives a baseline: RefineDet512 (COCO-pretrained, fine-tuned on train 8-bit thermal), **val mAP@0.5 = 0.587** (person 0.794, bicycle 0.580, car 0.856). The README says the train/val split is a *suggested* split and that "test data was not used" [01 §1.2]. | Static document |
| Papers with Code | Task pages **"Multispectral Object Detection on FLIR"** (`/sota/multispectral-object-detection-on-flir`, the FLIR-aligned protocol; CFT's README still carries the badge) and **"Image-to-Image Translation on FLIR"**. The dataset pages `/dataset/flir-` and `/dataset/flir-aligned` were already 404 in an April 2024 Wayback capture. | paperswithcode.com now redirects to Hugging Face "Trending Papers". The successor `paperswithcode.co` has no FLIR page (404). The Wayback Machine refused my requests (HTTP 429, then "temporarily offline"), so **I could not read archived leaderboard snapshots**. |
| **OpenCodePapers** (community mirror seeded from the PwC dump, then hand- and agent-updated) | `benchmarks/multispectral-object-detection-on-flir.html` with metrics mAP50 and mAP. Rows: MMPedestron 86.4; RGB-X Scene-Adaptive CBAM 86.16 / 47.1; CAFF-DINO 85.5 / 50.5; RSDet 83.9 / 43.8; DSAFormer 83.5 / 42.0; CMX 82.2; UniRGB-IR 81.4 / 44.1; MiPa 81.3 / 44.8; CSSA 79.2 / 41.3; CFT 77.7; ProbEn (CSSA re-run) 75.5 / 37.9; GAFF (CSSA re-run) 74.6 / 37.4; YOLOv5-T 73.9; GAFF-R18 72.9; GAFF-VGG 72.7; CFR 72.4; Halfway 71.2; YOLOv5-RGB 67.8. Also `image-to-image-translation-on-flir.html` (PSNR/SSIM: Pix2Next, IRFormer, BCI, pix2pix). | Live. **Use with care:** rows mix protocols. MiPa keeps dog, MMPedestron uses extra pretraining data, RGB-X uses an extra thermal-pretrained branch, and CFT is listed at 77.7 against 78.7 in its paper. The list is badly incomplete for 2024–2026 (no DAMSDet, Fusion-Mamba, WaveMamba, …). |
| Kaggle / Codabench / EvalAI | No FLIR v1 competition found (absence not proven) | n/a |
| RoadScene | No leaderboard. Fusion papers compare in their own tables. | n/a |

**Bottom line:** the "benchmark" is a **community convention** (FLIR-aligned P1). Each paper re-tabulates earlier numbers, often copied across protocols.

### 9.2 Protocol table (precise, for SOTA compilation)

Notation:
- "Aligned" means FLIR-aligned (Zhang et al. 2020).
- "v1 val" means the official `val` split (IDs 8,863–10,228).
- mAP = COCO mAP@[.5:.95]. AP50 / mAP50 = AP at IoU 0.5 averaged over classes.

| ID | Protocol | Variant | Splits (images or pairs) | Classes | Metric(s) | Input resolution (train / test) | RGB used? | Papers using it |
|---|---|---|---|---|---|---|---|---|
| **P1** | **FLIR-aligned RGB-T detection (standard)** | Aligned | train 4,129 / test 1,013 pairs (`align_train.txt` / `align_validation.txt`). No val split; some papers hold out 20% of train. | person, car, bicycle (**dog removed**) | mAP50, mAP75, mAP (COCO); often per-class AP50. Older papers report AP50 only. | **Varies:** (a) 640×512 native: CFR, GAFF, CSSA. (b) train 640×640 mosaic, test 640×512 letterbox (YOLOv5 rect): CFT, ICAFusion, MS2Fusion. (c) 640×640 resize: DAMSDet, Fusion-Mamba, WaveMamba, TFDet (resize long side + pad), IRDFusion, ZPA-MDETR, MS2Fusion ‡ Co-DETR rows. (d) Upscaled to LLVIP resolution: RSDet (Faster R-CNN). | Yes, RGB + T | CFR [#4], GAFF [#6], CFT, CSSA [#21], TINet [#24], ICAFusion [#25], RGB-X [#28], DAMSDet [#29], TFDet [#33], Fusion-Mamba [#36], RSDet [#37], WaveMamba [#40], M2D-LIF [#41], M-SpecGene [#42], UniRGB-IR [#43], MS2Fusion [#44], IRDFusion [#45], ZPA-MDETR [#48], FD²-Net, SeaDATE, DOD-SA (TGRS'25), LRAF-Net [#22] (inferred) |
| P1-dn | Day/night breakdown of P1 | Aligned | Same, test split by hand into day/night | 3 | mAP50 (and mAP75, mAP) per subset | 640×512 | Yes | RGB-X [#28] (own manual split; counts not checked) |
| P1-w | Weakly aligned P1 ("FLIR_w") | Aligned + synthetic shift | Same; IR randomly shifted by ±10 px | 3 | mAP50 / mAP | n/a | Yes | CF-Deformable DETR [#32] |
| P1-ft | P1 as a fine-tuning target after large-scale pretraining | Aligned | Same (MMPedestron misquotes "4,192 train") | 3 | mAP50 | n/a | Yes | MMPedestron [#31], M-SpecGene [#42], UniRGB-IR [#43] |
| **P2** | FLIR-aligned **thermal-only** detection | Aligned (IR half) | train 4,129 / test 1,013 IR images | 3 (dog removed) | mAP50, mAP75, mAP | 640×640 (PFGF); 640 letterbox (CFT baselines) | No at test. Some use RGB as privileged information in training (ModTr, HalluciDet). | CFT IR baseline, TIRDet [#23] (as compared in PFGF), PFGF [#39], ModTr [#30] (AP/AP50/AP75 with FCOS/RetinaNet/Faster R-CNN) |
| P2b | FLIR-aligned **person-only** IR detection | Aligned | 80% of the 4,129 train for training, 20% for val; test 1,013 | person | AP50 (person) | n/a | RGB detector used as teacher | HalluciDet [#27] |
| P2c | FLIR-aligned **modality-agnostic** detection | Aligned | train 4,129 RGB + 4,129 IR; test 1,013 RGB and 1,013 IR separately | **4 (dog kept)** | AP50 per test modality | 640×512 | Yes (mixed patches) | MiPa [#35] |
| **P3** | FLIR-aligned **RGB→thermal UDA detection** | Aligned | source: first 2,064 RGB of train; target: disjoint 2,064 thermal of train; test 1,013 thermal | person, bicycle, car | per-class AP + mAP (IoU 0.5 assumed; unverified) | short side ≤800 (FCOS VGG16) | RGB as source domain only | D3T [#26] (and HT, EPM, SWDA, DANN baselines re-run there) |
| **P4** | **Official v1 thermal-only ("MMTOD-style")** | Official v1 | train 8,862 / **val 1,366** (all frames, incl. those without RGB) | person, car, bicycle (dog dropped; "other vehicle" has no boxes in v1.3) | AP@0.5 per class + mean (VOC-style). DDSSD adds COCO APs and small/medium/large. | Mostly native 640×512 into Faster R-CNN default rescaling (unverified per paper); RefineDet512; SSD300 (DDSSD) | No | FLIR README baseline, MMTOD [#1], ThermalDet (ICCC'19), ODSC/Munir (Appl. Soft Comput.'22), BU (Kieu, TOMM'20), SSTN [#9], DDSSD |
| P4-sub | MMTOD data-efficiency subsets | Official v1 | FLIR(1/2), FLIR(1/4) of train; test val 1,366 | 3 | AP@0.5 | as P4 | No (pseudo-RGB via translation) | MMTOD [#1] |
| **P5** | **Official v1 unaligned RGB-T ("ProbEn")** | Official v1 | train 8,862 (pairs where RGB exists); **val 1,258** (108 val frames without RGB removed); day 768 / night 490 tags | person, car, bicycle | AP@0.5 (explicitly *not* COCO mAP) | all images resized to 640×512 | Yes (unaligned RGB; thermal labels) | ProbEn [#15] |
| P6 | Custom re-aligned v1 RGB-T (Vadidar) | Custom v1 alignment (edge cross-correlation plus cross-labelling) | IV'22 text: **6,924 / 1,982 / 3,960** (train/val/test). TIV'25 quotes **7,381 / 1,056 / 2,111** for the same source. | person, bicycle, car | mAP50 (IV'22 calls it "mAP", 82.9%); TIV'25: mAP@[.5:.95] + AP50 | 640×512 | Yes | Vadidar IV'22 [#16], "Rethinking Early-Fusion" TIV'25 [#38] |
| P7 | Classification DA (COCO→FLIR crops) | Official v1 | FLIR train crops → test on val crops (STGADA counts: bicycle 4,137, car 43,734, person 26,294) | bicycle, car, person | per-class and mean accuracy | 224×224 crops | No (COCO RGB as source) | SGADA [#8], STGADA (CVPRW'23) |
| P8 | Person-only physical adversarial | Official v1.3 person subset | PEOPLE_FLIR 7,873 / 2,027; "FLIR person select" 710 / 301 | person | AP drop, ASR | 416×416 (YOLOv3) | No | AAAI'21 bulbs [#7], CVPR'22 clothing [#14], CVPR'23 patches [#17] |
| P9 | RGB-T adversarial simulation | Aligned | aligned train/test as backgrounds | person | ASR (IoU 0.5, conf 0.6) | n/a | Yes | RGBT-Clothing [#46] |
| **P10** | **RoadScene IVIF** | RoadScene | No official split. Variants: (a) all 221 as test; (b) 180 train / 40 test, random (DIDFuse, AUIF); (c) train on MSRS, 50 RoadScene pairs val + 50 test (CDDFuse); (d) test-only, no training (DDFM). | none | Subsets of EN, SD, SF, MI, SCD, VIF, Q^AB/F, SSIM (CDDFuse uses all 8; DDFM 6: EN, SD, MI, VIF, Qabf, SSIM; TarDAL 3: MI, EN, SD); also AG, PSNR, CC in some papers | native cropped sizes (vary) | Yes (visible, usually Y channel) | FusionDN [#2], U2Fusion [#12], DIDFuse [#3], AUIF [#11], TarDAL [#13], CDDFuse [#19], DDFM [#20] (and over 100 others) |
| P11 | Fusion + detection on FLIR-aligned | Aligned | train 4,129 / test 1,013 | 3 | fusion metrics (EN, SF, MI, VIF, …) + detection mAP | n/a | Yes | MoE-Fusion [#18] |
| P12 | NTIR2DC translation (PearlGAN family) | Official v1 | train: 5,447 day RGB + 2,899 night IR; test: 490 night IR from val (lists in `PearlGAN/img_list/`) | seg classes (own masks); detection person/bicycle/car | mIoU, mAP (YOLOv4 COCO-pretrained), APCE | resize 500×400 → centre-crop 360×288 | Yes (unpaired day RGB) | PearlGAN [#10], MornGAN, FoalGAN [#34] |
| P13 | RGB→IR translation by image similarity | Aligned (inferred: "5,142 pairs") | split undocumented | none | PSNR, SSIM | 256×256 | Yes (RGB input) | IRFormer, Pix2Next (OpenCodePapers rows) |
| P14 | RGB-guided thermal SR | Official v1, rectified | 43,830 train / 1,257 test pairs | none | PSNR, SSIM (×4, ×8) | LR 64×80 → 256×320 / 512×640 | Yes (guide) | Gupta & Mitra ECCVW'20 [#5] |

**Key facts for comparing P1 numbers (counted from `aligned.zip`):**

| Split | Images | Boxes person / car / bicycle | Dog boxes (images) | Empty images | Box size small / medium / large (COCO, 3 classes) |
|---|---|---|---|---|---|
| align_train | 4,129 | 8,987 / 20,608 / 2,566 | 95 (90); 1 image has *only* a dog | 0 (1 after dog removal) | 19,042 / 11,156 / 1,963 |
| align_validation (test) | 1,013 | 4,107 / 4,124 / 360 | 13 (13) | 0 | 3,649 / 4,070 / 872 |

- All XML files are 640×512, with `difficult=0` throughout.
- Bicycle has only **360 test boxes**, so per-class bicycle AP is noisy. A few missed bicycles move mAP50 by tenths of a point.

### 9.3 Notes per protocol

**P1, FLIR-aligned RGB-T detection (de facto standard).**
- *Splits:* Use the two txt files as shipped. There is no official val split. Most papers report on the test set directly, and HalluciDet-style 80/20 hold-outs are rare.
- *The dog quirk:* The paper and nearly all users say "3 classes". The shipped VOC XMLs nevertheless still contain dog boxes (95 train, 13 test). Conversions differ:
  - CFT's YOLO yaml has `nc: 3`, names `[person, car, bicycle]`.
  - RSDet, M2D-LIF, WaveMamba, CSSA, ModTr and D3T explicitly remove dog.
  - MiPa **keeps** dog as a 4th class.
  - Fusion-Mamba and WaveMamba describe FLIR as having "five categories (people, car, bike, dog, other cars)" but evaluate on 3.

  Dropping the 13 test dog boxes barely changes 3-class AP. Evaluating with a 4th class, however, divides the mean by 4 and includes a class with almost no training data, so P2c numbers are not comparable with P1.
- *Metrics:*
  - Since GAFF (2021), papers use COCO mAP, AP50 and AP75, and since CFT (2021) mAP50, mAP75 and mAP.
  - CFR (2020) reports only "mAP" = **AP50** (72.39%).
  - WaveMamba adds precision, recall and F1. Fusion-Mamba and WaveMamba average over 5 or 15 runs for timing.
  - MS2Fusion reports 10-seed statistics in its appendix.
- *Evaluator differences (partly inferred from codebases):*
  - CFT-lineage YOLOv5 `test.py` uses conf 0.001, NMS IoU 0.6, `rect=True` letterbox at 640, and a **101-point interpolated AP integrated with `np.trapz`** (verified in the CFT repo). This is close to, but not identical with, pycocotools.
  - mmdetection (RSDet), PaddleDetection (DAMSDet), Detectron2 (CSSA, ProbEn) and Co-DETR (IRDFusion) use COCO-API evaluation (inferred from the frameworks).
- *Resolution:* The test images are 640×512. Training at 640×640 with letterbox or resize is common. RSDet upscales to LLVIP resolution. MS2Fusion marks its Co-DETR rows "‡ resized to 640×640" as a different setting.
- *Pretraining:* Mostly COCO or ImageNet. RGB-X uses a thermal branch pretrained on the *full unaligned v1 thermal train set*. That is extra data, though without test leakage, because the aligned test comes from v1 val. MMPedestron and M-SpecGene use large multi-dataset pretraining.

**P4, official v1 thermal-only.**
- *Splits:* train 8,862 / val 1,366, used as the test set.
- *Classes:* person, car, bicycle (MMTOD lists train counts 22,372 / 41,260 / 3,986). Dog is dropped, and "other vehicle" has no boxes in v1.3.
- *Metric:* AP@0.5.
- *Input:* the 8-bit AGC JPEGs. 16-bit TIFF use is rare (unverified).
- *Variants:* Some papers use the video split (4,224 frames) as an extra test set (none in the seed list verified). Some quote mixed-up class counts. DDSSD, for example, quotes "person 22,372, car 14,013, bicycle 1,205" [01 §1.4].

**P5, ProbEn.**
- The only established protocol that uses *unaligned* v1 RGB.
- Splits: train 8,862 and val 1,258 (val minus the 108 frames with no RGB). ProbEn's day/night tags: 768 / 490.
- Metric: AP@0.5, 3 classes, all images resized to 640×512.

**P10, RoadScene.**
- *Metrics:* There are no fixed metrics or split. Metric code also differs across groups: the MATLAB scripts from the Ma / Xu / Zhao toolboxes and the VIFB toolbox are common (unverified per paper).
- *Channels:* Visible images are RGB, but most methods fuse on the Y channel and recombine with CbCr.
- *Typical metric sets:*
  - CDDFuse / DDFM lineage: EN, SD, SF, MI, SCD, VIF, Q^AB/F, SSIM (8) or EN, SD, MI, VIF, Qabf, SSIM (6).
  - TarDAL: MI, EN, SD.
  - DIDFuse and AUIF: EN, SD, SF, MI, VIF, AG, …
- *Test subsets:* CDDFuse uses 50 RoadScene pairs for val and 50 for test, while DIDFuse and AUIF use a random 40-pair test. Values are therefore not comparable across papers unless the exact image list is shared. The CDDFuse/DDFM repos ship their `test_img/RoadScene` folders.

**P12, translation.**
- The PearlGAN-family protocol is fully specified by the authors' image lists and evaluates through downstream tasks (mIoU, mAP, APCE).
- Generic image-translation metrics (FID/KID/LPIPS) are **not** the standard for FLIR night-thermal → day-colour translation.
- PSNR/SSIM appear only where paired aligned data is assumed (IRFormer-style RGB→IR on FLIR-aligned).

### 9.4 Protocol inconsistencies across papers (what to watch for when compiling SOTA)

1. **Cross-protocol tables.**
   - ProbEn's Table 4 puts MMTOD / ODSC / ThermalDet / BU (P4, val 1,366), CFR / GAFF (P1, aligned test 1,013) and its own results (P5, val 1,258) in one column, all as "AP50 on FLIR".
   - Vadidar (IV'22) compares its 82.9% (P6 split) with CFR/GAFF's 72.4–72.9% (P1).
   - Later papers copy these rows.
2. **Same method, different numbers.**
   - ProbEn: 83.76 AP50 on P5, but 75.5 / 37.9 when CSSA re-ran it on P1.
   - GAFF: 72.9 in its own paper (P1, RetinaNet R18) and 74.6 in CSSA's re-run.
   - CFT: 78.7 in its paper, 77.7 on PwC/OpenCodePapers.
   - The ICAFusion repo says it "fixed some bugs and now update[d] the results on the FLIR dataset", so its arXiv and published numbers may differ (unverified which).
3. **"mAP" ambiguity.** CFR's and Vadidar's "mAP" is AP50. ProbEn explicitly calls its metric "AP" and contrasts it with COCO mAP. Always check whether the "mAP" column is AP50 or AP@[.5:.95].
4. **Class sets.** Most papers use 3 classes. MiPa uses 4 (with dog), HalluciDet uses person only, and the P8 adversarial papers use person only. Several papers mis-describe FLIR as "five categories" (Fusion-Mamba, WaveMamba).
5. **Split misquotes and mislabels.**
   - MMPedestron: "4,192 training pairs" (should be 4,129).
   - D3T calls FLIR-aligned "the updated FLIR dataset", which is not v2.
   - IRFormer claims an automatic alignment producing "5,142 sets", which is exactly FLIR-aligned's manual count.
   - The custom v1 alignment is reported with two different splits (6,924/1,982/3,960 in IV'22 vs 7,381/1,056/2,111 in TIV'25).
   - CVPR'26 "Distribution-Aligned" uses "FLIR (10,228 pairs, vehicle)" and gets about 94 mAP50, which is clearly not the P1 scale. Its protocol is unexplained and should be excluded from P1 comparisons.
6. **Resolution and augmentation.** Training ranges from 640×512 native to 640×640 letterbox or resize, plus mosaic. Test ranges from 640×512 rect to 640×640, plus the RSDet upscale. Epochs range from 12 (Co-DETR) to 250 (WaveMamba). None of this is standardised, and it is rarely ablated.
7. **Evaluator.** The YOLOv5 internal evaluator (101-point trapezoid integration, conf 0.001) vs pycocotools vs Detectron2. Small systematic offsets are expected (unverified in size).
8. **Extra data.** Pretraining on full v1 thermal (RGB-X) or multi-dataset corpora (MMPedestron, M-SpecGene, UniRGB-IR) is mixed into P1 tables without being flagged.
9. **Raw v1 val vs aligned test overlap.** The 1,013 aligned test pairs are a subset of v1 val IDs. P4 and P1 numbers therefore come from overlapping but different image sets and different label files (JSON vs VOC XML with the same boxes), so they must never be ranked together.
10. **RoadScene subsets and naming.** It is called "FLIR" in some papers (DIDFuse, AUIF), with random or undocumented test subsets of 40, 50 or 221 images and differing metric implementations.

### 9.5 Recommendation for group D (SOTA compilation)

- **Main table:** P1 only (aligned 4,129 / 1,013, 3 classes, RGB+T). Columns: mAP50 / mAP75 / mAP, with a resolution flag (native / letterbox / 640² / other), evaluator family, and an extra-pretraining flag. Exclude MiPa (P2c), MMPedestron-style fine-tuning rows (or flag them), CVPR'26 Distribution-Aligned, and every P4/P5/P6 number.
- **Secondary tables:** P2 (thermal-only on aligned), P4 (official v1 thermal-only, AP50), P5 (ProbEn).

---

## Uncertainties

1. **PwC archive:** The Wayback Machine was rate-limited or offline for me, so the archived PwC FLIR leaderboard was not read directly. OpenCodePapers is a derived mirror and may differ from the last PwC state.
2. **D3T's IoU threshold** is not stated (assumed 0.5). The resolution and evaluator for UniRGB-IR, M-SpecGene and M2D-LIF were not extracted.
3. **Evaluator per paper** (YOLOv5 vs pycocotools) is inferred from the codebase family, except for CFT, which I verified.
4. **RGB-X day/night split counts**, the **PearlGAN mask count** and the **IRFormer split** were not checked.
5. **Which P1 papers re-ran baselines and which copied numbers** was not audited row by row.
6. The **IV'22 vs TIV'25 split discrepancy** for the Vadidar alignment is unresolved. The two may describe different versions of the alignment.
7. The **v1 vs v2 identity of ThermEval and EPOFusion IVOE** is inferred from their cited URLs only.

---

## References

**Seed and dataset files**
- `Flir v1/01_dataset_overview.md` and `Flir v1/02_usage_and_publications.md` (this project); [#n] = row n of table 3.1 in file 02.
- FLIR-aligned archive: https://drive.google.com/file/d/1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y/view; HF mirror https://huggingface.co/datasets/jsonhash/FLIR_aligned (counted from `align/Annotations/*.xml`)
- RoadScene: https://github.com/hanna-xu/RoadScene

**Leaderboards**
- OpenCodePapers FLIR detection: https://opencodepapers-b7572d.gitlab.io/benchmarks/multispectral-object-detection-on-flir.html ; I2I: https://opencodepapers-b7572d.gitlab.io/benchmarks/image-to-image-translation-on-flir.html ; provenance: https://github.com/OpenCodePapers/OpenCodePapers
- Former PwC page (now redirects): https://paperswithcode.com/sota/multispectral-object-detection-on-flir (badge in https://github.com/DocF/multispectral-object-detection)

**Detection protocols (P1–P6)**
- CFR (ICIP'20): https://arxiv.org/abs/2009.12664 · GAFF (WACV'21): https://openaccess.thecvf.com/content/WACV2021/papers/Zhang_Guided_Attentive_Feature_Fusion_for_Multispectral_Pedestrian_Detection_WACV_2021_paper.pdf
- CFT: https://arxiv.org/abs/2111.00273 ; code and eval: https://github.com/DocF/multispectral-object-detection (`test.py`, `utils/metrics.py`, `data/multispectral/FLIR_aligned.yaml`)
- CSSA (CVPRW'23): https://openaccess.thecvf.com/content/CVPR2023W/PBVS/papers/Cao_Multimodal_Object_Detection_by_Channel_Switching_and_Spatial_Attention_CVPRW_2023_paper.pdf
- ICAFusion: https://arxiv.org/abs/2308.07504 · DAMSDet: https://arxiv.org/abs/2403.00326 · RSDet: https://arxiv.org/abs/2401.10731 · Fusion-Mamba: https://arxiv.org/abs/2404.09146 · TFDet: https://arxiv.org/abs/2305.16580 · M2D-LIF: https://arxiv.org/abs/2503.11780 · WaveMamba: https://arxiv.org/abs/2507.18173 · MS2Fusion: https://arxiv.org/abs/2507.14643 · IRDFusion: https://arxiv.org/abs/2509.09085 · M-SpecGene: https://arxiv.org/abs/2507.16318 · UniRGB-IR: https://arxiv.org/abs/2404.17360 · ZPA-MDETR: https://media.eventhosts.cc/Conferences/ECCV2026/pdfs/10379.pdf · RGB-X: https://arxiv.org/abs/2310.19372 · MMPedestron: https://arxiv.org/abs/2407.10125 · CF-Deformable DETR: https://www.ijcai.org/proceedings/2024/0084.pdf · SeaDATE: https://arxiv.org/abs/2410.11358 · DOD-SA: https://arxiv.org/abs/2508.10445 · CVPR'26 Distribution-Aligned: https://openaccess.thecvf.com/content/CVPR2026/papers/Hao_Distribution-Aligned_Multimodal_Fusion_for_Robust_Object_Detection_CVPR_2026_paper.pdf
- MiPa: https://arxiv.org/abs/2404.18849 · ModTr: https://arxiv.org/abs/2404.01492 · HalluciDet: https://arxiv.org/abs/2310.04662 · PFGF (CVPR'25): https://openaccess.thecvf.com/content/CVPR2025/papers/Li_Pseudo_Visible_Feature_Fine-Grained_Fusion_for_Thermal_Object_Detection_CVPR_2025_paper.pdf · D3T: https://arxiv.org/abs/2403.09359
- MMTOD (CVPRW'19): https://arxiv.org/abs/1905.08789 · SSTN (IROS'21): https://arxiv.org/abs/2103.03150 · ProbEn (ECCV'22): https://arxiv.org/abs/2104.02904 · Munir/ODSC: https://arxiv.org/abs/2006.00821 · DDSSD: https://arxiv.org/abs/2108.11101 · Vadidar (IV'22): https://arxiv.org/abs/2206.03943 · Rethinking Early-Fusion (TIV'25): https://arxiv.org/abs/2405.16038

**Other tasks**
- SGADA: https://arxiv.org/abs/2106.07165 , code https://github.com/avaapm/SGADA · STGADA: https://arxiv.org/abs/2304.07031
- PearlGAN: https://arxiv.org/abs/2104.14374 , code and lists https://github.com/FuyaLuo/PearlGAN · MornGAN: https://arxiv.org/abs/2208.02960 · FoalGAN: https://arxiv.org/abs/2310.15688 · IRFormer: https://arxiv.org/abs/2404.07072
- DIDFuse: https://arxiv.org/abs/2003.09210 · AUIF: https://ieeexplore.ieee.org/document/9416456 · CDDFuse: https://arxiv.org/abs/2211.14461 · DDFM: https://arxiv.org/abs/2303.06840 · TarDAL: https://arxiv.org/abs/2203.16220 · MoE-Fusion: https://arxiv.org/abs/2302.01392 · EPOFusion: https://arxiv.org/abs/2603.16130
- Bulbs (AAAI'21): https://arxiv.org/abs/2101.08154 · Invisible clothing (CVPR'22): https://arxiv.org/abs/2205.05909 · Learnable patches (CVPR'23): https://arxiv.org/abs/2303.13868 · RGBT-Clothing (CVPR'26): https://arxiv.org/abs/2605.04675
- Guided SR (ECCVW'20): https://arxiv.org/abs/2003.06216
- ThermEval: https://arxiv.org/abs/2602.14989 · Infrared Reasoner (CVPRW, uses v2), the CVF open-access paper read from local PDF (not linked here)
