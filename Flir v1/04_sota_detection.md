# FLIR ADAS v1: state-of-the-art detection results (Q6)

**Research date:** 2026-11-02
**Scope:** Detection results on the Teledyne FLIR ADAS thermal dataset **v1** (v1.3) and its derivative **FLIR-aligned** (Zhang et al., ICIP 2020; 4,129 train / 1,013 test pairs). The file is organised by the protocol IDs P1–P14 defined in [`03_tasks_and_benchmarks.md`](03_tasks_and_benchmarks.md) §9.2. Known pitfalls come from [`05_known_problems_workarounds.md`](05_known_problems_workarounds.md).

**Method:**
- I downloaded about 45 paper PDFs (arXiv, CVF open access, ECCV 2026 proceedings) and extracted the FLIR tables with `pdftotext -layout`.
- Every number marked **[paper]** was read from the cited table of the method's own paper.
- **[cited in X]** means the number comes from a comparison table in another paper X; I did not read the original paper.
- **(unverified)** means I found the number only in a search snippet or on a leaderboard mirror.
- No number in this file was estimated or interpolated. "–" means the paper does not report the value.

**Notation:**
- mAP50 = AP at IoU 0.5, averaged over classes. mAP75 = AP at IoU 0.75. mAP = COCO mAP@[.5:.95].
- Per-class AP50 is given as P / C / B (person / car / bicycle).
- Resolution codes: **N** = native 640×512; **LB** = YOLOv5-style training at 640 with mosaic, letterbox/rect test; **640²** = resized to 640×640; **UP** = upscaled (RSDet: 1024×1280).
- Evaluator codes: **YOLO** = Ultralytics/YOLOv5 internal 101-point AP; **COCO** = pycocotools / mmdet `CocoMetric` / Detectron2 / PaddleDetection (framework-inferred unless stated).
- **Venue tier** (my labels, for reading the tables only): **A** = CVPR/ICCV/ECCV/TPAMI; **A−** = top journals and conferences (TIP, TNNLS, TMM, T-ITS, TCSVT, TGRS, PR, Information Fusion, ACM MM, AAAI, IJCAI, WACV); **B** = workshops (CVPRW), ICIP, TIV, TIM, KBS, PRL, DSP, IROS; **arXiv** = preprint, not peer-reviewed as far as I could tell.

---

## TL;DR (as of 2026-11-02)

| Protocol | Best published number (and caveats) | Best "clean" number (peer-reviewed, ImageNet/COCO init only, standard 3-class P1) |
|---|---|---|
| **P1** FLIR-aligned RGB-T, 3 classes | **mAP50 88.6 / mAP 50.4**: TGCD (arXiv 2604.11234, 2026). Its appendix says FLIR-aligned has "4 classes (… dog)" and it uses a frozen CLIP text encoder, so it is **not cleanly comparable**. **88.4 mAP50 / 48.1 mAP**: WaveMamba (ICCV 2025, YOLOv8, 640²). **88.3 / 48.0 / 50.7**: IRDFusion (PR 2026, Co-DETR with ViT backbone, probably large-scale pretrained). | **WaveMamba 88.4 mAP50 / 48.1 mAP** (A, YOLO evaluator, no mAP75). **ZPA-MDETR 87.05 / 48.32 / 49.82** (A, ECCV 2026, RT-DETR R50, 640²; the highest complete 3-metric row I could fully verify under A-tier with standard pretraining). |
| P1 highest mAP@[.5:.95] | **50.7**: IRDFusion (Co-DETR ViT). **50.5**: CAFF-DINO (CVPRW'24, Swin-L DINO) and MS-GDINO-L (arXiv, Grounding-DINO Swin-L). | **ZPA-MDETR 49.82**, then DAMSDet 49.3 (ECCV'24). |
| P1-w (±10 px synthetic IR shift) | CF-Deformable DETR (IJCAI'24): 77.2 → 77.0 mAP50 | same |
| P1-dn (day/night on the aligned test) | **No per-scene numbers found.** RGB-X publishes only its manual split (test 702 day / 311 night). | – |
| **P2** thermal-only on FLIR-aligned | **PFGF (CVPR'25) 84.8 / 44.6 / 47.1**. It uses a PearlGAN prior trained on FLIR v1. CGDet (arXiv'25): 82.8 / 47.2 / 47.1. | PFGF 84.8 |
| P3 RGB→thermal UDA (D3T protocol) | **D3T (CVPR'24) 69.30 mAP** (IoU 0.5 assumed) | same |
| **P4** official v1 thermal-only (val 1,366) | **SSTN101 (IROS'21) 77.57 AP50**; ThermalDet 74.60 [cited in ProbEn]; FLIR README baseline 58.7 | same |
| **P5** ProbEn (unaligned v1, val 1,258) | **ProbEn3 (ECCV'22) 83.76 AP50** (day 82.21 / night 85.56) | same |
| P6 custom v1 alignment (Vadidar) | EME (TIV'25) 78.17 mAP50 / 44.00 mAP (GFL-R50); Vadidar (IV'22) 82.9 "mAP" (= AP50) | – |

**Main warnings:**
1. Above about 84 mAP50, the P1 differences between 2024–2026 methods (about 84–88.6) are of the same size as protocol noise: evaluator, resolution, seed, checkpoint selection on the test set (no val split) and bicycle variance (only 360 test boxes). See §6.
2. **The CVPR 2026 "Distribution-Aligned" paper reports 79.5 ± 0.7 mAP50 on "FLIR (10,228 pairs)", not about 94.** I checked its Table 1. The about-94 figure in the brief appears to be a mistake (LLVIP is the dataset where it reports 98.1). Its protocol is in any case not P1.

---

## 1. P1: FLIR-aligned RGB-T detection (main table)

**Protocol:** train 4,129 / test 1,013 pairs; person, car, bicycle; RGB + thermal at test time. Rows are sorted by mAP50.

**Pretrain column ("Extra pretrain"):**
- **N** = ImageNet/COCO initialisation only.
- **Y** = extra thermal or multispectral data, a large detection or grounding corpus, or a foundation model beyond COCO.
- **Y?** = probable but not confirmed.

Rows whose protocol is not clean P1 are listed separately in §1.2.

### 1.1 Main P1 table

| Method | Venue / year (tier) | Backbone / detector | Res. | Eval. | Extra pretrain | mAP50 | mAP75 | mAP | Per-class AP50 P / C / B | Code | Source |
|---|---|---|---|---|---|---|---|---|---|---|---|
| WaveMamba | ICCV 2025 (A) | YOLOv8 two-stream + wavelet Mamba | 640² | YOLO | N | **88.4** | – | 48.1 | – | [EhanDong/WaveMamba](https://github.com/EhanDong/WaveMamba) | [paper] Tab. 4 (also R50: 86.5/47.9; YOLOv5: 85.8/47.5) |
| IRDFusion | Pattern Recognition 2026 (A−) | Co-DETR, ViT backbone ("Double-Co-DETR", Zhou et al. arXiv 2411.18288) | 640² | COCO (mmdet, inferred) | **Y?** (the Co-DETR ViT-L line is normally Objects365-pretrained; the paper does not say) | 88.3 | 48.0 | **50.7** | – | [61s61min/IRDFusion](https://github.com/61s61min/IRDFusion) | [paper] Tab. 1, Tab. 7 (YOLOv5 variant: 84.8/38.1/43.4) |
| MS2Fusion ‡ (Co-DETR) | Information Fusion 2026 (A−) | Co-DETR (same framework as IRDFusion) | 640² | COCO (inferred) | Y? | 87.8 | – | 49.7 | 90.2 / 93.4 / 79.6 | [61s61min/MS2Fusion](https://github.com/61s61min/MS2Fusion) | [paper] Tab. 2 ("‡ … resized to 640×640") |
| ZPA-MDETR | ECCV 2026 (A) | RT-DETR R50 | 640² | COCO (inferred, RT-DETR) | N | 87.05 | 48.32 | 49.82 | 90.03 / 92.93 / 78.19 | [UserXiangYang/ZPA-MDETR](https://github.com/UserXiangYang/ZPA-MDETR) | [paper] Tab. 1 |
| PNAFusion (Co-DETR) | arXiv 2606.24092, 2026 (arXiv) | Co-DETR | n/a | n/a | Y? | 86.8 | – | 50.3 | 90.4 / 93.4 / 76.5 | "coming" | [paper] Tab. 1 (YOLOv5 variant: 84.2/–/43.8; 87.2/91.1/74.3) |
| DAMSDet | ECCV 2024 (A) | DETR-like, R50 ×2 | 640² (train+test) | COCO (PaddleDetection) | N (COCO) | 86.6 | 48.1 | 49.3 | 89.74 / 92.56 / 77.64 [cited in ZPA-MDETR; DAMSDet itself gives no per-class values] | [gjj45/DAMSDet](https://github.com/gjj45/DAMSDet) | [paper] Tab. 2 |
| TFDet | TNNLS 2024 (A−) | Faster R-CNN family (Swin-T per ZPA-MDETR's table) | 640² (resize + pad) | COCO (inferred) | N | 86.6 | – | 46.6 | – | [XueZ-phd/TFDet](https://github.com/XueZ-phd/TFDet) | [paper] Tab. V |
| DeformCAT (Hu et al., "Deformable cross-attention transformer for weakly aligned RGB-T pedestrian detection") | TMM 2025 (A−) | CSPDarknet53 | n/a | n/a | N? | **86.48** [cited in ZPA-MDETR] **or 84.7** [cited in DOD-SA] | 43.65 / 39.3 | 46.93 / 43.2 | 89.89 / 91.86 / 77.69 [cited in ZPA] | n/a | **Conflicting cited values; original not read (unverified)** |
| CAFF-DINO | CVPRW 2024 (B) | DINO, Swin-L ×2 (frozen) + CAFF fusion | n/a | **pycocotools** (stated) | N? (COCO init; Swin-L is IN-22k) | 85.5 | **51.6** | 50.5 | – | none | [paper] Tab. 3. A later paper from the same lab cites "CAFF-DINO*" at 83.1/42.3/45.6 (arXiv 2512.15971, Tab. 5), so this is probably a different configuration. |
| DOD-SA | TGRS 2025 (A−) | DETR-style, ResNet ×2; **trained with IR labels only** | n/a | COCO (inferred) | N | 85.2 | 44.5 | 46.8 | – | n/a | [paper] Tab. II |
| IGT | Knowledge-Based Systems 2023 (B) | Swin-T | n/a | n/a | N? | 85.0 | 36.9 | 43.6 | – | n/a | [cited in TFDet, PFGF, CGDet] |
| Fusion-Mamba (YOLOv8) | TMM 2025 (A−) | YOLOv8 + Mamba fusion | 640² | YOLO | N | 84.9 | 45.9 [cited in IRDFusion; not in FM's table] | 47.0 | – | none | [paper] Tab. 3 (YOLOv5: 84.3/–/44.4) |
| IRDFusion (YOLOv5) | PR 2026 (A−) | YOLOv5 | 640² | YOLO | N | 84.8 | 38.1 | 43.4 | – | (as above) | [paper] Tab. 7 |
| PNAFusion (YOLOv5) | arXiv 2026 | YOLOv5 | n/a | YOLO | N | 84.2 | – | 43.8 | 87.2 / 91.1 / 74.3 | – | [paper] Tab. 1 |
| RSDet | T-ITS 2025 (A−) | Faster R-CNN R50 (mmdet) | **UP** 1024×1280 | COCO (mmdet `CocoMetric`) | N | 83.9 | 40.1 [cited in UniRGB-IR] | 43.8 | – | [Zhao-Tian-yi/RSDet](https://github.com/Zhao-Tian-yi/RSDet) | [paper] Tab. V. The arXiv v1 numbers **81.1 / 41.4** are still cited by Fusion-Mamba, MiPa, MS2Fusion and IRDFusion. |
| GM-DETR | CVPRW 2024 (B) | DETR, R50 | n/a | COCO (inferred) | N | 83.9 | 42.6 | 45.8 | – | n/a | [cited in IRDFusion, IC-Fusion, ZPA-MDETR] (original not read) |
| COFNet | 2025 (venue not checked) | CSPDarknet | n/a | n/a | N? | 83.6 | 41.7 | 44.6 | – | n/a | [cited in SLGNet, TGCD] |
| MS2Fusion (YOLOv5) | Information Fusion 2026 (A−) | YOLOv5 two-stream + SSM | LB | YOLO | N | 83.3 (10-seed mean 82.99, var 0.15) | – | 40.3 (10-seed mean 40.60) | 85.1 / 89.8 / 74.9 | (as above) | [paper] Tab. 2, Tab. 16 |
| IC-Fusion | arXiv 2505.15137, 2025 | RT-DETR-style; RGB R18 / IR R50 | n/a | COCO (inferred) | N | 83.3 | 42.6 | 46.1 | – | n/a | [paper] Tab. II |
| MDAO / "MDACL" | arXiv 2601.00598, 2026 | YOLO, CSPDarknet53 | n/a | YOLO | N | 83.2 | – | 44.6 | – | n/a | [paper] Tab. III. TGCD cites a "MDACL" at the same 83.2 / 44.6, probably the same work (inferred). |
| CDC-YOLOFusion | IEEE TIV 2025 (B) | CSPDarknet53 | n/a | YOLO | N | 83.1 | – | 44.7 | – | n/a | [cited in ZPA-MDETR] |
| FD²-Net | AAAI 2025 (A−) | YOLOv5 | n/a | YOLO | N | 82.9 | 42.5 (table) / 41.9 (text) | – | 85.3 / 89.9 / 73.2 | n/a | [paper] Tab. 2 (table and text disagree on mAP75) |
| ICAFusion (README, revised) | PR 2024 (A−) | YOLOv5 CSPDarknet53 | LB | YOLO | N (needs COCO YOLOv5l weights) | **82.84** | 33.88 | 40.68 | 84.93 / 89.81 / 73.78 | [chanchanchan97/ICAFusion](https://github.com/chanchanchan97/ICAFusion) | [cited in ZPA-MDETR Tab. 1]. Matches the README's "79.2 → 82.8 after bug fixes" (file 05 §P8). |
| SCFR | T-ITS 2024 (A−) | n/a | n/a | n/a | n/a | 82.3 | 35.7 | – | – | n/a | [cited in IRDFusion] |
| CMX (re-run) | T-ITS 2023 (A−), re-run by MMPedestron | Segformer-style, Swin-T / MiT | n/a | n/a | N | 82.2 | 37.1 | 42.3 | – | [huaaaliu/RGBX_Semantic_Segmentation](https://github.com/huaaaliu/RGBX_Semantic_Segmentation) | 82.2 [MMPedestron paper Tab. 4b]; 37.1 / 42.3 [cited in PFGF, CGDet] |
| YOLOX-CPCF | T-ITS 2024 (A−) | YOLOX | n/a | n/a | N | 82.1 | 41.2 | 44.6 | – | n/a | [cited in IRDFusion, arXiv 2512.15971] |
| MCAFNet | Digital Signal Processing 2025 (B) | CSPDarknet53 | n/a | n/a | N | 81.5 | 40.2 | 43.9 | – | n/a | [cited in ZPA-MDETR] |
| UniRGB-IR | ACM MM 2025 (A−) | ViT-B adapter, Cascade R-CNN-style | n/a | COCO (inferred) | N? (ViT pretrained on COCO, per M-SpecGene) | 81.4 | 40.2 | 44.1 | – | [PoTsui99/UniRGB-IR](https://github.com/PoTsui99/UniRGB-IR) | [paper] Tab. 1 (arXiv v4) |
| CrossModalNet | 2025 (venue not checked) | CSPDarknet | n/a | n/a | N? | 81.7 | 39.1 | 43.3 | – | n/a | [cited in SLGNet, TGCD] |
| "Bag of tricks" best model | arXiv 2411.18288 (header says TPAMI special issue; acceptance not verified) | Co-DETR ViT-L, ICFE+NIN fusion, **LoFTR alignment applied to the test set** | n/a | n/a | Y? | 81.17 ("mAP" = mean of class AP50s) | – | – | 85.30 / 89.51 / 68.71 | [cited by IRDFusion as framework] | [paper] Tab. 5. The FLIR variant is not explicitly named; its baseline rows (CFR 72.44, CFT 78.42) look re-computed. |
| RDGNet | n/a | YOLOv8 | n/a | n/a | n/a | 80.6 | – | – | 85.6 / 91.0 / 65.3 | n/a | [cited in ZPA-MDETR] |
| MMFN | TCSVT 2024 (A−) | n/a | n/a | n/a | n/a | 80.8 | – | 41.7 | 85.7 / 91.2 / 65.5 | n/a | [cited in MS2Fusion, PNAFusion] |
| LRAF-Net | TNNLS 2023 (A−) | CSPDarknet53 | n/a | YOLO | N | 80.5 | – | 42.8 | – | none | [cited in DAMSDet, RGB-X, IRDFusion] |
| SeaDATE | arXiv 2410.11358 (2024) | YOLOv7 ELAN-Net | n/a | YOLO | N | 80.3 | 35.8 | 41.3 | – | n/a | [paper] Tab. VI |
| EI²Det | TCSVT 2025 (A−) | CSPDarknet53 | n/a | n/a | N | 80.2 | – | – | 84.9 / 89.4 / 66.3 | n/a | [cited in ZPA-MDETR] |
| MFPT (Faster R-CNN) | T-ITS 2023 (A−) | Faster R-CNN R50 | n/a | n/a | N | 80.0 | – | 41.9 [cited in WaveMamba] | 83.2 / 89.0 / 67.7 | n/a | [cited in RGB-X Tab. 5, MS2Fusion] |
| MMI-Det | TCSVT 2024 (A−) | CSPDarknet53 | n/a | n/a | N | 79.8 | 35.8 | 40.5 | – | n/a | [cited in M-SpecGene, ZPA-MDETR] |
| CrossFormer | PRL 2024 (B) | YOLOv5 | n/a | YOLO | N | 79.3 | 38.5 | 42.1 | – | n/a | [cited in Fusion-Mamba, MiPa, M-SpecGene] |
| CSSA | CVPRW 2023 (B) | Faster R-CNN R50 ×2 (Detectron2) | 640 resize (stated) | **COCO** (stated) | N | 79.2 | 37.4 | 41.3 | – | unofficial re-implementation only | [paper] Tab. 1 |
| ICAFusion (paper) | PR 2024 (A−) | YOLOv5 CSPDarknet53 | LB | YOLO | N | 79.2 | 36.9 | 41.4 | 81.6 / 89.0 / 66.9 | (as above) | [paper] Tab. 10 |
| CFT | arXiv 2021 (arXiv; widely cited) | YOLOv5 + CFT | LB | YOLO (verified in repo) | N | 78.7 | 35.5 | 40.2 | 84.1 / 89.5 / 61.4 at 78.3 [cited in ICAFusion]; 84.09 / 89.50 / 61.48 at 78.35 [cited in ZPA] | [DocF/multispectral-object-detection](https://github.com/DocF/multispectral-object-detection) | [paper] Tab. 1–3 (PwC / OpenCodePapers list 77.7) |
| UA-CMDet | n/a (a DroneVehicle method) | n/a | n/a | n/a | n/a | 78.6 | – | – | 83.2 / 88.4 / 64.3 | n/a | [cited in RGB-X, MS2Fusion] (provenance unclear) |
| YOLOFusion | PR 2022 (A−) | CSPDarknet53 | n/a | YOLO | N | 76.6 | – | 39.8 | – | n/a | [cited in RSDet, PFGF] |
| TINet | TIM 2023 (B) | Faster R-CNN R50 | n/a | n/a | N | 76.1 | – | 36.5 | – | [NNNNerd/Triple-I-Net-TINet](https://github.com/NNNNerd/Triple-I-Net-TINet) | [cited in RSDet]. **The TINet README says the paper used a different train/test split.** |
| ProbEn3 (CSSA re-run on P1) | ECCV 2022 method, re-run in CVPRW 2023 | Faster R-CNN R50 | 640 | COCO | N | 75.5 | 31.8 | 37.9 | – | – | [CSSA paper] Tab. 1 |
| GAFF (CSSA re-run) | WACV 2021 method, re-run | R50 | 640 | COCO | N | 74.6 | 31.3 | 37.4 | – | – | [CSSA paper] Tab. 1 |
| GAFF (ResNet18) | WACV 2021 (A−) | RetinaNet-style, R18 | N | COCO | N | 72.9 | 32.9 | 37.5 | – | none (official) | [paper] Tab. 8. DAMSDet and IRDFusion cite mAP 36.6, which is GAFF's *without-GAFF* row. |
| GAFF (VGG16) | WACV 2021 | VGG16 | N | COCO | N | 72.7 | 30.9 | 37.3 | – | – | [paper] Tab. 8 |
| CFR³ | ICIP 2020 (B) | VGG16 SSD-style + CFR blocks | N | n/a ("mAP" = AP50) | N | 72.39 | – | – | 74.49 / 84.91 / 57.77 | [zhanghengdev/CFR](https://github.com/zhanghengdev/CFR) | [paper] Tab. 2 (baseline 71.17). ProbEn and EME tabulate bicycle as 55.77 (transcription error). |
| Halfway fusion | (baseline in CFR) | VGG16 | N | n/a | N | 71.2 (CFR) / 71.5 (CSSA re-run: 71.5 / 31.1 / 35.8) | – | – | – | – | [CFT Tab. 3], [CSSA Tab. 1] |

**Leaderboard-only rows (original paper not read, unverified):**
- DSAFormer (2026): 83.5 / – / 42.0 (OpenCodePapers).
- MM-DETR (arXiv 2512.00363): 83.59 mAP50 (search snippet).
- FCI-DETR (Springer journal, 2026): 81.5 mAP50 (search snippet).

### 1.2 P1-adjacent rows: high numbers that are **not clean P1** (do not rank with §1.1)

| Method | Venue (tier) | mAP50 / mAP75 / mAP | Why not comparable | Source |
|---|---|---|---|---|
| TGCD, "Bridging the RGB-IR Gap: Consensus and Discrepancy Modeling for Text-Guided Multispectral Detection" | arXiv 2604.11234, 2026 | **88.6** / 45.7 / 50.4 | Appendix: "FLIR-Aligned: Contains **4 classes** (car, person, bicycle, dog) … fixed train/val splits". YOLO-World protocol with a frozen CLIP ViT-B/32 text encoder (vision-language pretraining). Its CSPDarknet backbone may be YOLO-World-pretrained (not stated). | [paper] Tab. 1, App. A |
| MS-GDINO-L | arXiv 2512.15971, 2025 | 87.8 / 51.3 / 50.5 | Grounding-DINO Swin-L (large grounding pretraining). Full-supervision "upper bound" of a few-shot paper. | [paper] Tab. 5 (also MS-GDINO-B 85.9/49.2/49.4; MS-YOLOW-L 85.3/47.7/48.9) |
| MMPedestron | ECCV 2024 (A) | 86.4 / – / – | **Pre-trained on MMPD (260k images), which includes "FLIR [22], 26K images"**, a Roboflow FLIR mirror whose 26K size suggests v2 or a mix. Overlap with the aligned test images is possible and **unverified**. Paper says "4,192 training pairs". The 86.4 is fine-tuning on P1. | [paper] Tab. 1, Tab. 4b |
| RGB-X scene-adaptive CBAM | WACV 2024 (A−) | 86.16 / 43.0 / 47.1 (P 88.92 / C 90.94 / B 78.61); scene-agnostic 85.45 / 43.3 / 46.8 | The thermal branch was pretrained on the **full unaligned v1 thermal train set** (more data). Scene classifier trained on its own day/night labels. EfficientDet. | [paper] Tab. 5 |
| SLGNet | arXiv 2601.02249, 2026 | 85.8 / 42.3 / 45.1 | Frozen ViT foundation model plus VLM-generated scene captions (extra pretrained models). Otherwise standard P1 (4,129/1,013). | [paper] Tab. I |
| M-SpecGene (ViT-B) | ICCV 2025 (A) | 84.8 / 40.1 / 44.7 (ViT-S: 82.4 / 39.4 / 43.7) | Self-supervised pretraining on RGBT550K. Whether RGBT550K contains FLIR frames was not checked. | [paper] Tab. 1b |
| MiPa | WACV 2025 (A−) | 81.3 / 41.8 / 44.8 | **4 classes (dog kept)**. Uses **IR only at inference** (P2c). Its own Tab. 3/4 report FLIR IR AP50 of only about 70, which is internally hard to reconcile with 81.3. | [paper] Tab. 5 |
| CVPR 2026 "Distribution-Aligned Multimodal Fusion" | CVPR 2026 (A) | **79.5 ± 0.7** (mAP50 only) | "FLIR (10,228 pairs, vehicle)" is the raw v1 count with unclear alignment and split. All baselines were re-implemented on frozen ViT-B + DETR (CFT 76.1, ICAFusion 77.3). **Not about 94** (the brief's figure). | [paper] Tab. 1 |
| ProbEn3 + GAFF (ZPA table) | ECCV 2022 method | 83.68 / 42.86 / 45.48 (P 86.87 / C 89.16 / B 75.02) | Listed in ZPA-MDETR with ResNet101. It matches neither ProbEn's own P5 value (83.76) nor CSSA's P1 re-run (75.5), so its protocol is unknown. | [cited in ZPA-MDETR] (unverified) |

### 1.3 Progression of the best credible P1 mAP50 over time

Rows from §1.1 only: peer-reviewed or widely-cited arXiv, standard 3-class P1. "Best mAP" is the best COCO mAP published that year in §1.1.

| Year | Best mAP50 (method, venue) | Best mAP (method) |
|---|---|---|
| 2020 | 72.39 (CFR³, ICIP) | – |
| 2021 | 72.9 (GAFF, WACV); 78.7 (CFT, arXiv) | 40.2 (CFT) |
| 2022 | 76.6 (YOLOFusion, PR) | 39.8 (YOLOFusion) |
| 2023 | 85.0 (IGT, KBS; cited only); 80.5 (LRAF-Net, TNNLS); 79.2 (CSSA, CVPRW) | 43.6 (IGT) |
| 2024 | 86.6 (DAMSDet, ECCV; TFDet, TNNLS); 85.5 (CAFF-DINO, CVPRW) | 50.5 (CAFF-DINO, Swin-L); 49.3 (DAMSDet) |
| 2025 | 88.4 (WaveMamba, ICCV) | 48.1 (WaveMamba) |
| 2026 | 88.3 (IRDFusion, PR; Co-DETR ViT); 87.05 (ZPA-MDETR, ECCV) | 50.7 (IRDFusion); 49.82 (ZPA-MDETR) |

**Reading the progression:**
- About +16 mAP50 in six years (72 → 88). The mAP@[.5:.95] went from about 37 to about 50.
- Since 2024 the frontier has been flat at about 86–88.4. Gains now come mainly from bigger detectors and pretraining (DETR family with Swin-L / ViT-L / Co-DETR) rather than from fusion modules.
- With YOLOv5-class detectors the ceiling is about 83–85 (Fusion-Mamba v5 84.3, IRDFusion v5 84.8, PNAFusion v5 84.2, MS2Fusion 83.3).

---

## 2. P1 sub-variants

### 2.1 P1-w: weakly aligned FLIR ("FLIR_w", random IR shift ±10 px)

Source: CF-Deformable DETR (IJCAI 2024, A−), mAP50.

| Method | FLIR → FLIR_w | Source |
|---|---|---|
| CF-Deformable DETR | V: 77.2 → 77.0; IR: 77.1 → 77.1 | [paper] Tab. 6 |
| ProbEn3 | 75.5 → 72.8 | [paper] Tab. 6 (re-run with public code) |
| LRAF-Net | 80.5 → 69.9 | [paper] Tab. 6 |
| CFT | 77.7 → 68.3 | [paper] Tab. 6 |
| YOLO-FS (YOLOFusion) | 76.6 → 67.8 | [paper] Tab. 6 |
| MBNet | 72.4 → 69.3 | [paper] Tab. 6 |
| Halfway-FS | 71.2 → 67.1 | [paper] Tab. 6 |

CF-Deformable DETR across offsets (V branch, mAP50; [paper] Tab. 2):

| Offset (px) | 0 | 5 | 10 | 20 | 30 | 50 |
|---|---|---|---|---|---|---|
| mAP50 | 77.2 | 77.1 | 77.0 | 73.2 | 67.5 | 61.7 |

- "V" and "IR" mean the model's two modality-specific outputs.
- DOD-SA (TGRS'25) measures centre offsets of 1–20 px in FLIR-aligned (Fig. 4) but reports no shifted-test numbers.
- arXiv 2512.15971 has a misalignment study in the few-shot setting only.

### 2.2 P1-dn: day/night breakdown on the aligned test set

- **No paper I read reports day/night mAP on the 1,013 aligned test pairs.**
- RGB-X (WACV'24) publishes a manual split (train 3,476 day / 653 night; test 702 day / 311 night; [paper] Tab. 2b) but gives only full-set FLIR numbers.
- TINet hard-codes day/night index ranges in its repo; I did not extract numbers (unverified).
- ProbEn's day/night numbers belong to P5 (§6).
- This is a gap for anyone who wants per-illumination SOTA.

### 2.3 P1-ft: large-pretraining fine-tuning

See §1.2: MMPedestron 86.4, M-SpecGene 84.8, MS-GDINO-L 87.8, SLGNet 85.8. UniRGB-IR (81.4) uses a COCO-pretrained ViT and is kept in §1.1.

---

## 3. P2: thermal-only detection on FLIR-aligned (IR at test time)

Same 4,129/1,013 split, 3 classes. Methods marked † use RGB or a translation prior during training only.

| Method | Venue (tier) | Backbone / detector | Res. | Extra pretrain | mAP50 | mAP75 | mAP | Source |
|---|---|---|---|---|---|---|---|---|
| PFGF † | CVPR 2025 (A) | CSPD53, VMamba + Graph-Mamba, MMDetection | 640² | **Y**: PearlGAN T→V prior trained on FLIR v1 (NTIR2DC split) | **84.8** | 44.6 | **47.1** | [paper] Tab. 1 |
| CGDet † | arXiv 2511.01435 (2025) | YOLOX, MobileNetV2; PearlGAN pseudo-visible | n/a | Y (PearlGAN) | 82.8 | **47.2** | 47.1 | [paper] Tab. I |
| DINO (IR) | baseline in ZPA-MDETR (ECCV'26) | DINO ConvNeXt | 640² | N | 82.07 | 46.56 | 46.48 | [ZPA-MDETR] Tab. 1 (P 86.37 / C 90.70 / B 69.15) |
| DEIM-RT-DETR (IR) | baseline in ZPA-MDETR | R50 | 640² | N | 81.73 | 42.83 | 45.75 | [ZPA-MDETR] Tab. 1 |
| RT-DETR (IR) | baseline in ZPA-MDETR | R50 | 640² | N | 81.71 | 41.84 | 45.19 | [ZPA-MDETR] Tab. 1 (IC-Fusion's own run: 80.5 / 39.6 / 43.6) |
| TIRDet † | ACM MM 2023 (A−) | CSPD53 + PearlGAN prior | n/a | Y (PearlGAN) | 81.4 | 41.1 | 44.3 | [cited in PFGF, TFDet, CGDet] |
| IDA † | n/a | CSPD53 | n/a | n/a | 81.1 | 43.4 | 46.3 | [cited in PFGF] |
| YOLOX-L (IR) | baseline | CSPD53 | n/a | N | 80.9 / 78.1 | 37.5 / 39.4 | 42.0 / 42.6 | [CGDet] / [PFGF] (two runs disagree) |
| DEIMv2 (IR) | baseline in DOD-SA | n/a | n/a | N | 80.8 | 39.5 | 43.8 | [DOD-SA] Tab. II |
| DINO (IR) | baseline in DAMSDet | R50 | 640² | N | 80.6 | 42.7 | 44.8 | [DAMSDet] Tab. 2 |
| YOLOv5 (IR) | baselines | CSPD53 | LB | N | 80.1 (DAMSDet) / 78.4 (TFDet) / **73.9** (CFT) | – / – / 35.7 | 42.4 / 42.2 / 39.5 | [DAMSDet], [TFDet], [CFT] |
| YOLOv11 (IR) | baseline in DOD-SA | n/a | n/a | N | 78.8 | – | 42.8 | [DOD-SA] Tab. II |
| Cascade R-CNN ViT-B (IR) | baseline in UniRGB-IR | ViT-B | n/a | N | 78.6 | 37.3 | 41.9 | [UniRGB-IR] Tab. 1 |
| DINO Swin-L (IR) | baseline in CAFF-DINO | Swin-L | n/a | N | 77.2 | 41.3 | 43.6 | [CAFF-DINO] Tab. 3 |
| Faster R-CNN (IR) | baselines | R50 | n/a | N | 74.4 (CFT) / 75.8 (UniRGB-IR) / 73.4 (CSSA) | 32.5 / 31.6 / 34.2 | 37.6 / 37.6 / 37.9 | [CFT], [UniRGB-IR], [CSSA] |
| YOLOv8l (IR) | baseline in Fusion-Mamba | YOLOv8 | 640² | N | 72.9 | – | 38.3 | [Fusion-Mamba] Tab. 3 |

**Related thermal-only protocols (not P2):**
- **ModTr** (ECCV 2024, IR→RGB-like translation into a frozen detector; COCO-style AP only, 5 seeds), FLIR IR test: FCOS 35.49 ± 0.94, RetinaNet 34.27 ± 0.27, Faster R-CNN 37.21 ± 0.46. Fine-tuning baselines: 27.97 / 28.46 / 30.93 ([paper] Tab. 1–2). These AP values are far below other P2 papers (about 42–47), which suggests a different training regime or evaluation (unexplained).
- **P2b, HalluciDet** (WACV 2024, person only, 80/20 train/val), FLIR IR test person AP50: Faster R-CNN 70.90 ± 1.35 (fine-tuning 61.48, no adaptation 55.85); FCOS 49.18; RetinaNet 49.01 ([paper] Tab. 2).
- **P2c, MiPa** (4 classes, IR test): see §1.2.

**Note on baselines:** The YOLOv5 IR baseline varies from 73.9 to 80.1 mAP50 between papers. A large part of the "fusion gain" claimed in papers that use the weaker 73.9 baseline (e.g. CFT's +5.7) disappears against a well-trained thermal-only detector. DINO ConvNeXt IR alone reaches 82.07, above many 2023–2024 fusion methods.

---

## 4. P3: RGB→thermal unsupervised domain adaptation (D3T protocol)

- Source: the first 2,064 RGB images of aligned train. Target: a disjoint 2,064 thermal images. Test: 1,013 thermal images.
- FCOS-VGG16. AP is assumed to be at IoU 0.5 (not stated).

| Method | Person | Bicycle | Car | mAP | Source |
|---|---|---|---|---|---|
| **D3T** (CVPR 2024, A) | 70.77 | 57.44 | 79.68 | **69.30** | [paper] Tab. 1 |
| HT | 70.87 | 48.11 | 78.45 | 65.81 | [D3T] Tab. 1 |
| EPM | 40.97 | 38.95 | 53.83 | 44.60 | [D3T] Tab. 1 |
| SWDA | 30.91 | 36.03 | 47.94 | 38.29 | [D3T] Tab. 1 |
| DANN | 32.02 | 30.52 | 48.88 | 37.14 | [D3T] Tab. 1 |
| Source only | 28.54 | 28.28 | 47.22 | 34.68 | [D3T] Tab. 1 |

Code: [EdwardDo69/D3T](https://github.com/EdwardDo69/D3T). I found no later paper on this exact protocol (absence not proven).

---

## 5. P4: official v1 thermal-only (train 8,862 / val 1,366, AP@0.5)

**Not comparable with P1/P2.** It uses different test images (a superset), JSON labels and AP@0.5. Values attributed to ProbEn Tab. 4 are as tabulated there; I did not verify the originals.

| Method | Venue (tier) | Detector | AP50 (mean) | P / C / B | Source |
|---|---|---|---|---|---|
| **SSTN101** | IROS 2021 (B) | Self-supervised contrastive + DETR-style, R101 | **77.57** | – | [paper] Tab. II–III (split 8,862 / 1,366 in Tab. I); SSTN50: 75.19 |
| tY | n/a | n/a | 75.00 | – | [cited in SSTN] |
| ThermalDet | ICCC 2019 | multi-layer feature fusion | 74.60 | 78.24 / 85.52 / 60.04 | [cited in ProbEn] Tab. 4 |
| BU (LT,T) | TOMM 2020 (Kieu) | n/a | 73.20 | 75.60 / 86.50 / 57.40 | [cited in ProbEn] |
| BU (AT,T) | TOMM 2020 | n/a | 73.10 | 76.10 / 87.00 / 56.10 | [cited in ProbEn] |
| ODSC (Munir) | Appl. Soft Comput. 2022 | Faster R-CNN family | 69.62 | 71.01 / 82.33 / 55.53 | [cited in ProbEn] (the ODSC PDF table was unreadable after extraction) |
| MMTOD-UNIT (COCO init) | CVPRW 2019 (B) | Faster R-CNN + UNIT pseudo-RGB | 61.54 | 64.47 / 70.72 / 49.43 | [paper] Tab. 1 |
| MMTOD-CG (COCO init) | CVPRW 2019 | + CycleGAN | 61.40 | 63.31 / 70.63 / 50.26 | [paper] Tab. 1 |
| FLIR README baseline | Teledyne FLIR | RefineDet512 (COCO-pretrained) | 58.7 | 79.4 / 85.6 / 58.0 | v1.3 README (file 01/03) |
| MMTOD baseline | CVPRW 2019 | Faster R-CNN | 53.97 | 54.69 / 67.57 / 39.66 | [paper] Tab. 1 |
| DDSSD300 | arXiv 2108.11101 | SSD300 + dilation/deconv | AP50 49.9; COCO AP 22.8 | – | [paper] Tab. 2 (SSD300: 45.3 / 20.3). Train/val as provided; "AP50" here is COCO-style. |

**P4-sub (data efficiency):** MMTOD defines FLIR(1/2) and FLIR(1/4) ([paper] Tab. 3 counts). Values were not extracted.

**Gap:** No thermal-only paper since about 2022 reports on P4 with modern detectors. Work moved to P2 or to FLIR v2.

---

## 6. P5: ProbEn protocol (official v1, unaligned RGB-T, val 1,258, AP@0.5)

| Method | Day | Night | All | P / C / B | Source |
|---|---|---|---|---|---|
| **ProbEn3 (v-avg)** | 82.21 | 85.56 | **83.76** | 87.65 / 90.14 / 73.49 | [paper] Tab. 3–4 (ECCV 2022, A) |
| ProbEn3 (avg) | 82.19 | 84.91 | 83.63 | – | [paper] Tab. 3 |
| MidFusion | 79.37 | 81.64 | 80.53 | 84.16 / 87.63 / 69.80 | [paper] Tab. 3–4 |
| Thermal-only Faster R-CNN | 75.35 | 82.90 | 79.24 | 84.04 / 87.11 / 62.63 | [paper] |
| EarlyFusion | 77.37 | 79.56 | 78.80 | 85.27 / 87.69 / 63.43 | [paper] |

**Caveats:**
- ProbEn's text says "increases AP from prior art 74.6% to 84.4%", but its table says 83.76 (internal inconsistency).
- Its Tab. 4 mixes P4 rows (MMTOD, ODSC, ThermalDet, BU), P1 rows (CFR, GAFF) and P5 rows in one column.
- Re-run on P1 (by CSSA, with R50 instead of R101), ProbEn gives **75.5 / 31.8 / 37.9**.
- Day/night tags: 768 / 490 frames. Code: [Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling](https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling).

---

## 7. P6: custom v1 re-alignment (Vadidar et al.)

| Method | Venue | Split (train / val / test) | Result | Source |
|---|---|---|---|---|
| EME (ShaPE + WeakSup + CoreKD), GFL-R50 | IEEE TIV 2025 (B) | 7,381 / 1,056 / 2,111 (as stated in the TIV paper) | mAP 44.00 ± 0.00, mAP50 78.17 ± 0.05 (3 runs) | [paper] Tab. II, IV (thermal-only GFL: 42.40 mAP) |
| EME, RetinaNet-R50 | TIV 2025 | same | 38.83 / 72.23 | [paper] Tab. IV |
| Vadidar et al. (Scaled-YOLOv4 + attention) | IEEE IV 2022 (B) | 6,924 / 1,982 / 3,960 (IV'22 text) | 82.9 "mAP" (= AP50) | [paper] abstract |

- EME's Tab. VII compares its P6 result with P4 and P1 numbers (CFR 72.39, GAFF 72.90, ProbEn 83.76, …). That is protocol mixing.
- The two splits for the "same" alignment differ (file 03 §9.4); this is unresolved.
- EME notes that random splitting "causes information leakage".

---

## 8. Flags: suspicious or non-comparable numbers

| # | Issue | Affected numbers | Recommendation |
|---|---|---|---|
| F1 | **Revised numbers.** ICAFusion's README replaces 79.2 with 82.8 after "fixing bugs" (ZPA-MDETR tabulates the revision as 82.84 / 33.88 / 40.68). The README result needs COCO YOLOv5l weights; users without them get about 74–75 (file 05 §P8). | ICAFusion 79.2 vs 82.84 | Cite both and say which. Later papers mostly cite 79.2. |
| F2 | **Stale arXiv numbers.** RSDet v1 gave 81.1 / 41.4; the published T-ITS version gives 83.9 / 43.8. Fusion-Mamba, MiPa, MS2Fusion and IRDFusion still cite 81.1. | RSDet | Use the T-ITS values. |
| F3 | **Same method, different protocol.** ProbEn: 83.76 (P5, official val, R101) vs 75.5 (P1 re-run by CSSA, R50). GAFF: 72.9 (own) vs 74.6 (CSSA re-run). CFT: 78.7 (paper) vs 77.7 (PwC/OpenCodePapers, CF-Deformable DETR) vs 78.3/78.35 (ICAFusion/ZPA per-class rows). | ProbEn, GAFF, CFT | Never put the 83.76 in a P1 table. |
| F4 | **Conflicting citations of the same paper.** DeformCAT (TMM'25): 86.48 / 43.65 / 46.93 (ZPA-MDETR) vs 84.7 / 39.3 / 43.2 (DOD-SA). GAFF mAP: 36.6 (the *without*-GAFF row) vs 37.5. CAFF-DINO: 85.5 / 51.6 / 50.5 (own) vs 83.1 / 42.3 / 45.6 ("CAFF-DINO*", arXiv 2512.15971). | DeformCAT, GAFF, CAFF-DINO | Read the original before citing. |
| F5 | **Large or extra pretraining.** MMPedestron (MMPD includes a 26K-image Roboflow FLIR set, so test overlap is possible); M-SpecGene (RGBT550K); RGB-X (full v1 thermal train); MS-GDINO-L (Grounding DINO); SLGNet (frozen ViT + VLM captions); TGCD (CLIP / YOLO-World); IRDFusion / MS2Fusion‡ / PNAFusion Co-DETR (probably Objects365-pretrained ViT-L, unverified); PFGF / TIRDet / CGDet (PearlGAN trained on FLIR v1 images). | §1.2, §3 | Flag "Extra pretrain = Y". Report a separate "COCO/ImageNet-only" SOTA. |
| F6 | **Class-set changes.** MiPa and TGCD keep **dog** (4 classes). HalluciDet uses person only. | MiPa 81.3, TGCD 88.6 | Exclude from 3-class P1 ranking. |
| F7 | **Not FLIR-aligned at all.** CVPR'26 Distribution-Aligned uses "10,228 pairs": **79.5 ± 0.7 mAP50, not about 94**. The bag-of-tricks paper applies **LoFTR re-alignment to the test set** and does not name the FLIR variant. | 79.5, 81.17 | Exclude from P1. |
| F8 | **Random or Roboflow re-splits** inflate results by about 10 points, up to 0.98 mAP50 (CFT issues #50, #58; file 05). TINet's paper numbers use a different split (README warning). | Any repo or blog result above about 90 on "FLIR" | Treat any FLIR-aligned mAP50 above 90 as suspect unless the split files are given. |
| F9 | **Checkpoint selection on the test set.** YOLOv5-lineage repos (CFT, ICAFusion, MS2Fusion, IRDFusion's YOLOv5 runs) use `val = test` and pick `best.pt` on it (file 05 §P8). Only MS2Fusion reports seed statistics (10 seeds: 82.99 mean vs 83.3 reported). | All YOLO-lineage P1 rows | Expect an optimistic bias of about 0.3–1 point (inference from the MS2Fusion seed spread). |
| F10 | **Evaluator and resolution.** YOLO 101-point AP vs pycocotools; native 640×512 vs 640² vs RSDet's 1024×1280 upscale. MS2Fusion explicitly separates its 640² Co-DETR rows (‡). | Cross-family comparisons | Differences under about 2–3 mAP50 are not meaningful (file 05). |
| F11 | **Thin metrics.** Many 2025–2026 YOLO papers report mAP50 + mAP only (WaveMamba, Fusion-Mamba, MS2Fusion), or mAP50 only (FD²-Net omits mAP; DSAFormer's mAP75 is unknown). FD²-Net's mAP75 differs between table (42.5) and text (41.9). | – | Prefer rows with all three metrics. |
| F12 | **Leaderboard drift.** OpenCodePapers lists CFT 77.7 and MMPedestron 86.4 at the top, and misses DAMSDet, WaveMamba, ZPA-MDETR, IRDFusion. Its "Halfway fusion 35.8" row is mAP placed under mAP50. | – | Do not use OpenCodePapers as the SOTA source. |
| F13 | **Cross-protocol rows inside P1 tables.** ICAFusion Tab. 10, MS2Fusion Tab. 2 and bag-of-tricks Tab. 5 list MMTOD (P4, 61.4/61.5) and BU (P4) next to P1 methods. | – | Ignore those rows. |

---

## 9. Uncertainties

1. **Evaluator per paper** is inferred from the framework (mmdet / Paddle / Detectron2 → COCO; YOLO repos → YOLO) except for CFT (repo verified), CSSA and CAFF-DINO (stated in the text).
2. **IRDFusion / MS2Fusion‡ / PNAFusion Co-DETR pretraining:** The papers say they follow the "Double-Co-DETR" bag-of-tricks framework with a ViT backbone, but they do not state the pretraining corpus. I mark them "Y?".
3. **DeformCAT, GM-DETR, IGT, COFNet, CrossModalNet, CDC-YOLOFusion, MMFN, SCFR, MCAFNet, EI²Det, RDGNet, YOLOX-CPCF, MMI-Det, LRAF-Net, YOLOFusion, TINet, MFPT, CrossFormer:** values are taken from other papers' comparison tables; I did not open the originals. DSAFormer, MM-DETR and FCI-DETR values come only from a leaderboard or search snippets.
4. **TGCD's class count:** the appendix says 4 classes incl. dog, but its baselines are standard 3-class numbers. Whether it *evaluates* on 4 classes is unclear.
5. **MMPedestron's FLIR pretraining data** (Roboflow "flir-data-set", 26K images): whether it overlaps the aligned test images is unverified.
6. **WaveMamba mAP75 and per-class values** are not reported. The Fusion-Mamba mAP75 (45.9) exists only as cited by IRDFusion.
7. **D3T's IoU threshold** and **ModTr's unusually low AP** were not resolved.
8. **No P1-dn numbers** were found. RGB-X provides only the split.
9. Venue tiers are my own labels. Venues for COFNet, CrossModalNet, RDGNet, IDA and tY were not checked.
10. The 2025–2026 search was not exhaustive. More arXiv works probably claim 83–87 mAP50 on FLIR-aligned (e.g. MM-DETR, FCI-DETR, DSAFormer).

---

## 10. References (papers whose tables were read)

**P1 RGB-T**
- CFR: https://arxiv.org/abs/2009.12664
- GAFF: https://openaccess.thecvf.com/content/WACV2021/papers/Zhang_Guided_Attentive_Feature_Fusion_for_Multispectral_Pedestrian_Detection_WACV_2021_paper.pdf
- CFT: https://arxiv.org/abs/2111.00273
- CSSA: https://openaccess.thecvf.com/content/CVPR2023W/PBVS/papers/Cao_Multimodal_Object_Detection_by_Channel_Switching_and_Spatial_Attention_CVPRW_2023_paper.pdf
- ICAFusion: https://arxiv.org/abs/2308.07504
- RGB-X: https://arxiv.org/abs/2310.19372
- DAMSDet: https://arxiv.org/abs/2403.00326
- TFDet: https://arxiv.org/abs/2305.16580
- MMPedestron: https://arxiv.org/abs/2407.10125
- CF-Deformable DETR: https://www.ijcai.org/proceedings/2024/0084.pdf
- CAFF-DINO: https://openaccess.thecvf.com/content/CVPR2024W/PBVS/papers/Helvig_CAFF-DINO_Multi-spectral_Object_Detection_Transformers_with_Cross-attention_Features_Fusion_CVPRW_2024_paper.pdf
- MiPa: https://arxiv.org/abs/2404.18849
- SeaDATE: https://arxiv.org/abs/2410.11358
- Bag of tricks: https://arxiv.org/abs/2411.18288
- RSDet: https://arxiv.org/abs/2401.10731
- Fusion-Mamba: https://arxiv.org/abs/2404.09146
- UniRGB-IR: https://arxiv.org/abs/2404.17360
- FD²-Net (AAAI'25): local PDF (`fd2net.pdf`)
- M2D-LIF (ICCV'25; mAP only: 46.1 on FLIR, Tab. 5): https://arxiv.org/abs/2503.11780
- M-SpecGene: https://arxiv.org/abs/2507.16318
- WaveMamba: https://arxiv.org/abs/2507.18173
- DOD-SA: https://arxiv.org/abs/2508.10445
- MS2Fusion: https://arxiv.org/abs/2507.14643
- IRDFusion: https://arxiv.org/abs/2509.09085
- IC-Fusion: https://arxiv.org/abs/2505.15137
- VLM few-shot / MS-GDINO: https://arxiv.org/abs/2512.15971
- MDAO: https://arxiv.org/abs/2601.00598
- SLGNet: https://arxiv.org/abs/2601.02249
- TGCD: https://arxiv.org/abs/2604.11234
- PNAFusion: https://arxiv.org/abs/2606.24092
- ZPA-MDETR: https://media.eventhosts.cc/Conferences/ECCV2026/pdfs/10379.pdf
- CVPR'26 Distribution-Aligned: https://openaccess.thecvf.com/content/CVPR2026/papers/Hao_Distribution-Aligned_Multimodal_Fusion_for_Robust_Object_Detection_CVPR_2026_paper.pdf

**P2 / P2b / P3**
- PFGF: https://openaccess.thecvf.com/content/CVPR2025/papers/Li_Pseudo_Visible_Feature_Fine-Grained_Fusion_for_Thermal_Object_Detection_CVPR_2025_paper.pdf
- CGDet: https://arxiv.org/abs/2511.01435
- ModTr: https://arxiv.org/abs/2404.01492
- HalluciDet: https://arxiv.org/abs/2310.04662
- D3T: https://arxiv.org/abs/2403.09359

**P4 / P5 / P6**
- MMTOD: https://arxiv.org/abs/1905.08789
- SSTN: https://arxiv.org/abs/2103.03150
- ODSC: https://arxiv.org/abs/2006.00821
- DDSSD: https://arxiv.org/abs/2108.11101
- ProbEn: https://arxiv.org/abs/2104.02904
- Vadidar (IV'22): https://arxiv.org/abs/2206.03943
- EME (TIV'25): https://arxiv.org/abs/2405.16038

**Leaderboards and search**
- OpenCodePapers FLIR: https://opencodepapers-b7572d.gitlab.io/benchmarks/multispectral-object-detection-on-flir.html (checked 2026-11-02)
- CVF CAFF-DINO page: https://openaccess.thecvf.com/content/CVPR2024W/PBVS/html/Helvig_CAFF-DINO_Multi-spectral_Object_Detection_Transformers_with_Cross-attention_Features_Fusion_CVPRW_2024_paper.html
- GM-DETR (CVPRW'24): https://openaccess.thecvf.com/content/CVPR2024W/JRDB/html/Xiao_GM-DETR_Generalized_Muiltispectral_DEtection_TRansformer_with_Efficient_Fusion_Encoder_for_CVPRW_2024_paper.html
- FCI-DETR: https://link.springer.com/article/10.1007/s44443-026-01386-y
- MM-DETR: https://arxiv.org/abs/2512.00363
