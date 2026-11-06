# FLIR ADAS v2: deep learning tasks (Q5) and established benchmarks and protocols (Q9)

**Research date:** 2026-11-02
**Group:** C (see `../PLAN.md`). This file builds on `01_dataset_overview.md` (Group A) and `02_usage_and_publications.md` (Group B), and does not repeat their work. Paper IDs such as **A5** and **C8** refer to the tables in the B file.
**Scope:** Teledyne FLIR ADAS **v2** (v2.0.0, Jan 2022). v1 (2018) and the v1-derived **FLIR-aligned** subset (Zhang et al. 2020: 5,142 pairs, 4,129/1,013, 3 classes) are kept separate. They appear here only where papers use them instead of v2, or alongside it.

**How this was verified.** I read the full text of 17 papers: the arXiv/CVF PDFs, extracted to text and checked for split, class, metric and resolution statements. I also queried the 2025-07 **Papers with Code archive** (`pwc-archive/evaluation-tables` on Hugging Face) for every leaderboard whose dataset name contains "FLIR". Statements marked **[verified]** were read in the paper or archive. Statements marked **(unverified)** come from abstracts, the B file, or my inference.

---

## TL;DR

- **No official benchmark or leaderboard exists for v2.** The only "official" protocol is the **baseline in FLIR's own README**. It uses a COCO-pretrained YOLOX-m at 640×640, trained on the image-train split and tested on the **video test split** (3,749 frames), and reports **AP@0.5 for person and car only**: thermal 75.33 / 77.23, RGB 51.42 / 55.79 **[verified]**. The Papers with Code archive does have "FLIR" leaderboards (multispectral detection, object detection, image-to-image translation, MSCOCO→FLIR domain adaptation), but **every entry there is FLIR-aligned (v1)**, not v2 **[verified in archive]**.
- **Tasks on v2 (Q5).** v2 is used mainly as a **thermal-only** dataset.
  - **Mainstream:** (1) supervised thermal object detection. This is the largest group, made of mostly YOLO-style papers in mid- and low-tier venues plus a few CVPRW papers.
  - **Distinctive and growing:** (2) adversarial, physical and backdoor attacks on IR person detectors (AAAI, CVPR, ICIP); (3) 14/16-bit **tone mapping / AGC** learned for detection, which is unique to v2 because it ships 14-bit raw TIFFs (RA-L, ICCVW); (4) **RGB→thermal generation** as data augmentation (ACM MM); (5) **open-vocabulary / zero-shot** thermal detection (CVPR 2026); (6) **RGB→IR unsupervised domain adaptation**, which v2 enables because the RGB side has its own labels (SS-DC, preprint).
  - **Niche:** self-supervised RGB-T pretraining, VLM/MLLM evaluation, multi-object tracking (the video test split has `track_id`s), and RGB-T registration.
  - **Essentially absent:** RGB-T fusion detection (the v2 pairs are not registered, so fusion work uses FLIR-aligned v1) and semantic segmentation (v2 has no masks).
- **Protocols (Q9).** There is no consensus protocol. I identified **14 distinct v2 detection protocols** (table in §Q9.3). They differ in:
  - **test split:** video test 3,749; its non-empty 3,493; its raw-TIFF subset 2,151; or the official val 1,144 used as test;
  - **classes:** 2 (person, car), 4, 5 (person, bike, car, light, sign), 9, all 15/16, or person-only filtered by height;
  - **metric:** AP@0.5 per class, COCO mAP@[.5:.95], or "mAP" that is really mAP50;
  - **input:** 8-bit JPEG vs 14-bit raw; 640×512, 640×640, 800 or 1280;
  - **pretraining:** COCO-pretrained vs trained from scratch.
- **Closest to a de facto standard** for supervised thermal detection: **train on `images_thermal_train`, test on `video_thermal_test`**, with COCO-style AP. People and cars are always reported, and 5-class subsets are common. Comparisons across papers are **only valid within one protocol row**. Group D should not pool numbers across rows.
- **Known inconsistencies:**
  - total frames: 26,442 claimed vs 30,787 on disk;
  - thermal stills: 9,711 claimed vs 11,886;
  - train images: 10,742 on disk vs 10,724 (Thermal Chameleon) vs 10,474/10,478 (non-empty);
  - test frames: 3,749 vs 3,742 vs 3,493 vs 2,151 vs 2,292;
  - val frames: 1,144 vs 1,128/1,127;
  - "13,460 images" in Physical Backdoor is exactly 9,711 + 3,749 (claimed counts, not files);
  - "mAP" means AP50 in some papers and COCO mAP in others;
  - the `scooter` class ID is 75 in the files and 77 in the README.

---

## Q5: Deep learning tasks that use v2

### 5.1 Overview

"Status" is relative to v2 use, not to the field as a whole. Paper IDs are from the B file. **New** marks papers found in this pass.

| # | Task | Status on v2 | How v2 is used | Representative v2 papers (venue) |
|---|---|---|---|---|
| T1 | Supervised **thermal object detection** (incl. lightweight/edge, small objects, robustness augmentation) | **Mainstream** (largest group; mostly YOLO variants in MDPI, PLOS, Sci Rep, IET; a few CVPRW) | Train on thermal image splits, test on video test or val. 8-bit JPEG usually, 14-bit raw sometimes. Rare classes dropped | A1 Uzun CVPRW'22; A3 IR Reasoner CVPRW'23; A8 MS-YOLO IJCNN'25; C2 Zha IET-IP'24; C5 Uzun Sci Rep'25; C6 MDCFVit-YOLO PLOS'25; C7 LIWL-YOLO Sci Rep'25; C10 Fs2PA'26; C12 class-pruning eIT'26; **new:** MODIPHY / YOLO Phantom (ICIP 2024, [arXiv 2402.07894](https://arxiv.org/abs/2402.07894)) |
| T2 | **Adversarial / physical / backdoor attacks and defenses** on IR person detectors | **Distinctive, strong in top venues** (AAAI, CVPR, ICIP) | Person-only subset filtered by box height (>120 px), or thermal set with person+car. Detector fine-tuned, then attacked. Metrics: AP drop, attack success rate (ASR) | A2 HOTCOLD AAAI'23; A4 Physical Backdoor CVPR'24; A7 POD ICIP'24; C4 TID-Attack IPT'24; ambiguous: Wei CVPR'23 (B-table), KGAT IJCV'26 |
| T3 | **Tone mapping / AGC** (14/16-bit → 8-bit, learned for detection) | **Niche but v2-specific** (needs raw TIFFs and labels) | Train detector + tone-mapper on raw `analyticsData/` TIFFs; compare with FLIR's vendor AGC JPEGs | A5 Thermal Chameleon RA-L'24; A9 Neural-AGC ICCVW'25; C1 joint TM+denoise SPIE'23 |
| T4 | **RGB→thermal image translation / generation** (pseudo-thermal data for detection) | Moderate; most work uses aligned-v1 or LLVIP for paired metrics | Unpaired v2 RGB/thermal (no pixel pairs), or generated thermal used to augment detector training. Metrics: FID/KID plus detector mAP gain | A6 ECDM ACM MM'24; possible: Kaili Wang et al. component-aware MUNIT ([arXiv 2509.23243](https://arxiv.org/abs/2509.23243); version not stated, unverified). Contrast: ThermalGen, DiffV2IR, F-ViTA, Pix2Next use aligned or other data |
| T5 | **Open-vocabulary / zero-shot** thermal detection | **Emerging** (CVPR 2026) | v2 used as a **held-out zero-shot test set** ("FLIR-V2") next to FLIR-aligned; no v2 training | A10 Thermal-Det CVPR'26 |
| T6 | **RGB→IR unsupervised domain adaptation (UDAOD)** | Emerging; v2-enabled (RGB has independent labels) | Labelled RGB source subset → unlabelled thermal target subset → evaluation on thermal val | C8 SS-DC (arXiv 2507.12017; code/protocol repo [xi666666/FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD)); A1 Uzun (COCO→thermal fine-tuning, "thermal adapted") |
| T7 | **Multimodal (unpaired) RGB+thermal training** | Niche | Mix RGB and thermal images from both splits in one detector (no correspondence) | **new:** MODIPHY ICIP'24; C9 VRU multi-dataset (arXiv 2509.06333) |
| T8 | **Self-supervised multimodal pretraining** | Niche | RGB-T pairs (synchronized crops) for SSL; downstream CenterNet probe on FLIR detection | A11 Le MuMo JEPA CVPRW'26 |
| T9 | **VLM / MLLM evaluation** on thermal | Niche | Zero-shot prompting of GPT-4V/Gemini to count or classify objects; or VLM QA benchmarks | C3 MLLM thermal (MDPI Automation'24); C13 ThermEval (ambiguous v1/v2) |
| T10 | **Multi-object tracking** | **Niche / underused** despite labels | Video test split has `track_id` (8 videos, 3,749 frames per modality); MOTA/MOTP-style eval suggested by FLIR | C2 Zha et al. IET-IP'24 (detection + MOT; details paywalled, unverified). No other v2 MOT paper found |
| T11 | **RGB-T registration** | Niche | Uses the time-synced video test pairs (`rgb_to_thermal_vid_map.json`) as unregistered pairs | C11 XCalib (arXiv 2609.21770) |
| T12 | **RGB-T fusion detection** | **Absent on v2**; mainstream on aligned-v1 | v2 pairs unregistered → fusion papers use FLIR-aligned (4,129/1,013). See §Q9.2 | none confirmed on v2 (B Table A: 0) |
| T13 | **Semantic / instance segmentation** | **Absent** | v2 has boxes only, no masks. Thermal segmentation DA papers use "FLIR ADAS" only as unlabelled target (version unclear) | none on v2 (e.g., [arXiv 2505.06951](https://arxiv.org/abs/2505.06951) uses "FLIR ADAS", version unverified) |
| T14 | Other: thermal super-resolution, image fusion, class-pruning / dataset curation | Niche / ambiguous | Raw frames as HR targets; small hand-registered subsets for fusion; class pruning | DuaDiff TNNLS'25 (ambiguous); EPOFusion (277 pairs, version unclear); C12 class pruning eIT'26 |

### 5.2 Task notes

**T1 Thermal object detection (mainstream).**
- **Splits.** The common pattern is to train on `images_thermal_train` and test on `video_thermal_test`. Examples: FLIR baseline; MS-YOLO (10,474 / 1,127 / 3,493) **[verified]**; MODIPHY (thermal test 3,749) **[verified]**; Thermal Chameleon (raw subset) **[verified]**. A second group uses the **official val (1,144) as test**: Uzun CVPRW'22 and Sci Rep'25 **[verified]**.
- **Classes.** These are almost always reduced:
  - 2 classes (person, car): Uzun **[verified]**
  - 4 classes (person, car, light, sign): MODIPHY **[verified]**
  - 5 classes (person, bike, car, light, sign): IR Reasoner, ECDM, SS-DC **[verified]**
  - 9 classes: MS-YOLO **[verified]**
  - all 15: Neural-AGC Table 3 **[verified claim]**
- **Inputs.** Most use 8-bit JPEGs. IR Reasoner trains from scratch on **14-bit raw** divided by 2¹⁴−1 at 1280 px **[verified]**.
- **Reported numbers** (to show spread; Group D compiles SOTA):
  - IR Reasoner YOLOv7-W6-Reasoner: 40.5 AP / 58.5 AP50 (5 classes, raw, scratch) **[verified]**
  - MS-YOLO YOLOv8n baseline: 0.517 mAP50 / 0.318 mAP50-95 (9 classes) **[verified]**
  - Uzun VfNet R50 (2 classes, val-as-test): 52.4 AP / 81.9 AP50 **[verified]**
  - FLIR YOLOX-m: AP50 person 75.33, car 77.23 **[verified]**

**T2 Adversarial security.**
- **HOTCOLD** (AAAI'23) defines the reusable subset: person images with body height >120 px, **1,255 images (878 train / 377 test)**, 1,366 / 598 person boxes. Metrics are AP and ASR **[verified]**. Its code is at [weihui1308/HOTCOLDBlock](https://github.com/weihui1308/HOTCOLDBlock).
- **POD** (ICIP'24) says it "follows Wei et al." but gets **1,169 train / 84 test** images (1,810 / 154 persons) **[verified]**. The same filter rule gave a different subset, so the subsets are not interchangeable.
- **Physical Backdoor** (CVPR'24) uses "Flir_v2_T", the thermal images only, **13,460 images**, 640×512. Clean mAP for person/car: YOLOv5 79.1 / 82.5, YOLOv3 84.4 / 85.8, Faster R-CNN 51.65 / 71.56 **[verified]**. 13,460 = 9,711 + 3,749, i.e. FLIR's *claimed* counts (my inference). The train/test split is not clearly stated (unverified).
- All three report person (± car) only and use different subsets, so **no shared benchmark exists** within this task.

**T3 Tone mapping / AGC.**
- **Thermal Chameleon** (RA-L'24):
  - Data: raw 14-bit, **train 10,724 / test 2,151**. The 2,151 test frames are the 6 Tau 2 test videos that have raw TIFFs; the other 2 ADK videos (1,598 frames) have no raw data (see A file).
  - Setup: 640×512 input; RetinaNet, YOLOX and Sparse R-CNN; COCO mAP, AP50, AP75, APs/m/l **[verified]**.
  - Results (YOLOX): FLIR AGC baseline 53.5 mAP / 81.4 AP50; TCNet 56.0 / 83.8 **[verified]**.
  - Classes are not stated in the paper (unverified; probably all classes present in that test subset). The repo ships `FLIR_test_raw_corrected.json`.
- **Neural-AGC** (ICCVW'25):
  - Data: claims an "aligned union of FLIR ADAS v1 and v2", **12,000 frames, 15 classes**, 90/10 split, and a video test of **3,742 frames** **[verified as stated]**.
  - Results: reports AP50 person/car on video test (YOLOX-m 90.1 / 86.3) and COCO AP over 15 classes (65.8) **[verified as stated]**.
  - **Caution:** v1 has only 5 classes, the frame counts match no split, and the gains are unusually large. Treat it as a separate, non-comparable protocol (my assessment).

**T4 RGB→thermal generation.**
- **ECDM** (ACM MM'24) uses v2.0 with 5 classes (person, bike, car, light, sign) at 512×640. It trains detectors (Faster R-CNN, RetinaNet, CenterNet, VFNet, DINO) with and without generated pseudo-thermal data and reports mAP / AP50 / AP75. Faster R-CNN R50 goes from 23.5 to 24.4 mAP **[verified]**. The exact source of its train/test split is not stated in the text I read (unverified).
- Paired translation metrics (PSNR/SSIM) cannot be computed on v2 images because nothing is registered. The PwC "FLIR" image-to-image translation leaderboard (PSNR/SSIM) is on aligned v1 data **[verified in archive]**.

**T5 Open-vocabulary detection.**
- **Thermal-Det** (CVPR'26) treats "FLIR-V2" as a hard zero-shot benchmark, using Grounding DINO Swin-T at 800×800 and reporting AP / AP50 / AP75 **[verified]**. Selected results:

| Model | FLIR-V2 (AP / AP50) | FLIR-aligned (AP / AP50) |
|---|---|---|
| Thermal-Det (zero-shot) | 0.096 / 0.173 | 0.372 / 0.664 |
| G-DINO (zero-shot) | 0.081 / 0.144 | 0.337 / 0.636 |
| G-DINO fine-tuned (supervised reference) | 0.200 / 0.345 | 0.414 / 0.749 |

- The per-class results cover person, car, motorcycle and traffic light. **The split and class list are not stated** (unverified). The low v2 scores suggest an evaluation over many classes, including small ones.

**T6 RGB→IR UDA.**
- **SS-DC** builds a new protocol from v2: **5,663 labelled RGB (source), 4,856 unlabelled IR (target train), 1,144 IR eval** (= official thermal val). It uses 5 classes, DINO-R50 with shortest edge 800, and reports per-class AP and "COCO-style mAP" **[verified]**. Results: SS-DC 48.02, D3T 43.04, source-only 31.81, oracle 64.19.
- The values look closer to AP50 than to COCO mAP@[.5:.95], so the metric definition should be double-checked (my assessment). The same paper also runs the D3T protocol on FLIR-aligned (VOC mAP50, 3 classes).

**T7–T11 niche tasks.** See the table. The video-test `track_id`s make v2 the only FLIR release usable for MOT, but I found **no MOT benchmark protocol** (MOTA/IDF1/HOTA) used on it by more than one paper.

**Where people use v1 or aligned instead of v2.** RGB-T fusion detection, paired translation (PSNR/SSIM), RGB→IR UDA in the D3T lineage (2,064 RGB / 2,064 IR / 1,013 test), modality-agnostic detection (MiPa, ModTr, ModPrompt, HalluciDet) and corruption robustness (WiSE-OD's FLIR-C) all use **FLIR-aligned (v1)**. See B Table D.

---

## Q9: Established benchmarks and protocols

### 9.1 Is there an official benchmark or leaderboard?

| Source | Finding |
|---|---|
| FLIR `README.txt` | **The only official protocol.** YOLOX-m designed for 640×640, MSCOCO-pretrained, trained on the provided training data, **tested on the video test data**, COCO eval. Reports AP@IoU=0.5: thermal person 75.33, car 77.23; RGB person 51.42, car 55.79 **[verified]** ([README](https://adas-dataset-v2.flirconservator.com/dataset/README.txt)). There is no leaderboard and no evaluation server. Whether val was merged into training is not stated |
| Papers with Code (closed 2025) | The 2025-07 archive ([pwc-archive/evaluation-tables](https://huggingface.co/datasets/pwc-archive/evaluation-tables)) has leaderboards on dataset **"FLIR"**: *Multispectral Object Detection* (mAP50, mAP), *Object Detection* (AP 0.5), *Image-to-Image Translation* (PSNR, SSIM), and *(Unsupervised) Domain Adaptation "MSCOCO to FLIR ADAS"* (accuracy, SGADA 91.2). **All are aligned v1 or v1** (e.g., CFT 77.7 mAP50, CAFF-DINO 85.5 / 50.5, MMPedestron 86.4, RSDet 83.9 / 43.8, UniRGB-IR 81.4 / 44.1, MiPa 81.3 / 44.8) **[verified in archive]**. There is **no v2 leaderboard**. The archived PwC dataset pages `dataset/flir-` and `dataset/flir-aligned` return "Not Found" in the Wayback Machine |
| Other leaderboards (Roboflow, Kaggle, HF, alphaXiv, PBVS challenges) | None found for v2. Kaggle and Roboflow host mirrors and notebooks only. PBVS thermal challenges use other datasets (unverified for every year) |
| Benchmark-like reuse | Thermal-Det's "FLIR-V2" zero-shot column and SS-DC's RGB→IR protocol (with a public split repo) are the only v2 protocols designed for reuse by others. Neither has been adopted by a second paper yet (as of 2026-11) |

### 9.2 Which protocols are de facto standard?

- **Splits.** Most often: train on `images_thermal_train`, test on `video_thermal_test`, as in the FLIR baseline, MS-YOLO, MODIPHY and VRU. Others use official val as test (Uzun ×2, SS-DC eval) or custom subsets (adversarial work, Thermal Chameleon's raw subset). The README calls the third split "validation video", so some papers call it "val" or "test-all". **Always map paper terms back to folder names.**
- **Classes.** No standard. The **5-class set (person, bike, car, light, sign)** is the most frequent explicit choice (IR Reasoner, ECDM, SS-DC), and person+car is the minimum everyone reports. "All classes" in practice means the 10 classes that occur in the test split (bus, train, deer, skateboard, stroller and scooter have 0 test boxes; see A file).
- **Metrics.** COCO tooling dominates. FLIR-faithful papers use **AP@0.5 per class** (person, car). Others use COCO **mAP@[.5:.95] + AP50 + AP75 + APs/m/l**. YOLO/ultralytics papers report **mAP50 and mAP50-95**. MR⁻² (miss rate) is **not used** on v2: it belongs to KAIST and pedestrian work.
- **Input.** Native 640×512 (Thermal Chameleon, ECDM at 512×640), letterboxed 640×640 (FLIR baseline, MS-YOLO, MODIPHY, Le MuMo JEPA), 800 short side or 800×800 (SS-DC, Thermal-Det), or 1280 (IR Reasoner). 8-bit JPEG is the default; 14-bit raw is used in T3 papers and in IR Reasoner.
- **Use of RGB.** In the vast majority, thermal only. RGB is used for UDA source (SS-DC), unpaired multimodal training (MODIPHY, VRU), SSL pairs (Le MuMo JEPA), the RGB baseline (FLIR README), and generation (ECDM).
- **Contrast with the aligned-v1 standard.** That standard is 4,129 train / 1,013 test, 3 classes (person, bicycle, car; dog removed), mAP50 and mAP@[.5:.95], 640×512, RGB+thermal input. It is far more standardized than any v2 protocol, which is why fusion papers stay on it.

### 9.3 Protocol variants table (for Group D)

The protocol ID is meant to be quoted by Group D. "VT" = `video_thermal_test`. "Val" = `images_thermal_val`. "Train" = `images_thermal_train`. Counts are as stated by each paper.

| ID | Protocol description | Splits (train / val / test) | Classes | Metric(s) | Input / data | Papers using it |
|---|---|---|---|---|---|---|
| **P0-FLIR** | Official README baseline | Train (10,742 on disk) / – / **VT 3,749** | trained on all; **reported person, car** | **AP@0.5 per class** (COCO eval) | YOLOX-m, 640×640, COCO-pretrained, 8-bit; thermal and RGB separately | FLIR README [verified]; reused as reference by Neural-AGC (A9) and Thermal Chameleon (as "FLIR AGC") |
| **P1-VT-9c** | Thermal train → VT, YOLO-style (empty frames dropped) | **10,474 / 1,127 / 3,493** | 9: person, bike, car, motor, bus, truck, light, hydrant, sign | P, R, **mAP50, mAP50-95** | YOLOv8 family, 640×640, 8-bit | MS-YOLO IJCNN'25 (A8) [verified] |
| **P2-VT-4c-mm** | Mixed thermal+RGB training → VT (thermal) and RGB test | thermal 10,478 + RGB 10,053 / 1,128 + 1,069 / thermal **3,749**, RGB "3,493" (sic) | 4: person, car, light, sign | **mAP50-95** (COCO), FPS | YOLOv8n-based, 640×640, COCO-pretrained | MODIPHY / YOLO Phantom ICIP'24 [verified] |
| **P3-VT-5c-raw** | Thermal train → "test" from scratch on 14-bit raw | official train / val / test (counts not given; test presumably VT, unverified) | 5: person, bike, car, light, sign | COCO **AP, AP50, AP75, APs/m/l** | 14-bit raw ÷ (2¹⁴−1), 1280, **no pretraining** | IR Reasoner CVPRW'23 (A3) [verified] |
| **P4-RAW2151** | Raw-TIFF subset: train → VT restricted to the 6 Tau 2 videos | **10,724 / – / 2,151** | not stated (unverified); bike absent from this test subset (A file) | COCO **mAP, AP50, AP75, APs/m/l** | 14-bit raw (and 8-bit AGC baseline), 640×512 | Thermal Chameleon RA-L'24 (A5) [verified]; Neural-AGC re-trains TCNet as a baseline |
| **P5-VAL-2c** | Thermal train → **official val used as test** | train 90% of "9,711" (Sci Rep: 10% of train held out as val) / – / **val (10%)** | 2: person, car | COCO **AP, AP50, AP75, APs/m/l** | 8-bit, 640×512, COCO-pretrained VfNet/TOOD/YOLOR, ± turbulence | Uzun CVPRW'22 (A1), Uzun Sci Rep'25 (C5) [verified] |
| **P6-ECDM-5c** | Detector trained with/without generated pseudo-thermal | v2.0 (split source not stated, unverified) | 5: person, bike, car, light, sign | **mAP, AP50, AP75** (+ FID/KID for generation) | 512×640, 8-bit | ECDM ACM MM'24 (A6) [verified] |
| **P7-UDA-SSDC** | RGB→IR UDA | source **5,663 RGB (labelled)**; target **4,856 IR (unlabelled)** / – / **1,144 IR (= val)** | 5: person, bicycle, car, sign, light | per-class AP + "COCO-style mAP" (values look like AP50, check) | DINO-R50, short side 800, RGB + IR | SS-DC (C8) [verified]; split at [xi666666/FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD) |
| **P8-OV-ZS** | Zero-shot open-vocabulary eval (no v2 training) | test only (split not stated, unverified) | not stated (includes person, car, motorcycle, traffic light) | COCO **AP, AP50, AP75** (+ per class / scale) | Grounding-DINO Swin-T, 800×800 | Thermal-Det CVPR'26 (A10) [verified] |
| **P9-ADV-HC** | Person-only, height >120 px (HOTCOLD subset) | **878 / – / 377** (1,255 imgs) | person | AP, **ASR** | YOLOv5 etc. fine-tuned | HOTCOLD AAAI'23 (A2) [verified]; POD ICIP'24 "follows" it but has **1,169 / 84** (A7) [verified] |
| **P10-BKD** | Thermal set "Flir_v2_T" | 13,460 images (= 9,711 + 3,749); split unclear (unverified) | person, car | per-class mAP, ASR, BAF | 640×512; YOLOv5/v3, Faster R-CNN | Physical Backdoor CVPR'24 (A4) [verified counts] |
| **P11-NAGC-v1v2** | "Aligned union v1+v2", 90/10 + video test | 12,000 frames 90/10 / **3,742 VT** | person/car AP50; all 15 for COCO AP | **AP50 per class**, COCO AP@[.5:.95] | raw 16-bit, 640×512 | Neural-AGC ICCVW'25 (A9) [verified as stated; internally inconsistent] |
| **P12-SSL-probe** | SSL encoder → CenterNet probe | FLIR RGB-T (subset and pairing not stated, unverified) | all FLIR detection classes; also car | **mAP50, Car mAP50** | 640×640, RGB+thermal synchronized crops | Le MuMo JEPA CVPRW'26 (A11) [verified] |
| **P13-VRU-merge** | v2 merged with KITTI and BDD100K under a VRU class map | RGB 10,319 / thermal 10,742 train; 1,085 / 1,144 val; 3,749 test pairs | custom VRU map (pedestrian, cyclist, motorcyclist, ...) | **mAP50, mAP50:95** | YOLOv8, 640 | VRU detection (C9, arXiv 2509.06333) [verified] |
| *(other)* | Thermal train 10,742 / test 2,292, 8-bit | 10,742 / – / 2,292 | ? | mAP (78.6%) | – | Zha et al. IET-IP'24 (C2), from B file (paywalled, unverified; 2,292 matches no known split) |
| *(other)* | Per-class counting with MLLMs | sample of images | 15 | TPR, FPR, MAE, MAPE | GPT-4V, Gemini | MLLM thermal (C3) [verified] |
| *contrast* **PA-aligned** | **FLIR-aligned (v1)**, not v2 | 4,129 / – / 1,013 pairs | 3: person, bicycle, car | mAP50, mAP@[.5:.95] | 640×512, RGB+T | Nearly all fusion papers; all PwC "FLIR" leaderboards; Thermal-Det's "FLIR-Aligned" column |

**Comparable groupings for Group D:**
- P0 ≈ P1 ≈ P2 only for **person and car AP50 on VT**, and only if P1/P2 report per-class AP. Their mAP averages cover different class sets.
- P3 and P4 are raw-input protocols and cannot be compared with each other: different test sets (VT vs the 2,151 subset) and different pretraining.
- P5 (val-as-test) is not comparable with any VT-based number.
- P7–P13 are task-specific.

### 9.4 Inconsistencies across papers

| Issue | Values seen | Explanation / advice |
|---|---|---|
| Total frames | **26,442** (README, IR Reasoner, MODIPHY, Uzun, MLLM) vs **30,787** (MS-YOLO; archive) | Archive count is correct (A file). Papers quoting 26,442 still use the shipped files |
| Thermal stills | 9,711 (README; Uzun, SS-DC) vs **11,886** on disk (10,742 + 1,144) | Same as above. "90/10 of 9,711" in Uzun = the official train/val folders |
| Train images | **10,742** (disk; VRU) vs **10,724** (Thermal Chameleon) vs **10,478** (non-empty; MODIPHY) vs **10,474** (MS-YOLO) | 10,724 = 18 fewer, likely frames without a usable raw TIFF (unverified). 10,47x = frames with ≥1 box after class filtering |
| Val images | 1,144 (disk) vs 1,128 (non-empty) vs 1,127 (MS-YOLO) | Empty frames dropped (YOLO export) |
| Test frames | **3,749** (VT) vs **3,742** (Neural-AGC, "duplicate timestamps removed") vs **3,493** (non-empty VT; MS-YOLO, and MODIPHY's "RGB test") vs **2,151** (raw subset; Thermal Chameleon) vs **2,292** (Zha, unverified) | Record which one. 3,493 thermal test frames have boxes; RGB VT has 3,749 non-empty, so MODIPHY's "RGB 3,493" is probably a mislabel (my inference) |
| "Validation" naming | README calls VT "validation video" | Some papers' "val" = VT. Check counts |
| Claimed-count arithmetic | Physical Backdoor 13,460 = 9,711 + 3,749 | Count derived from README figures, not from files |
| Class IDs | `scooter` 75 (files) vs 77 (README); `deer` (ID 18) undocumented | Match by name |
| Classes missing from test | bus, train, deer, skateboard, stroller, scooter: 0 VT boxes; bike: 0 in the raw 2,151 subset | "All-class" mAP on VT really averages ≤10 classes; on the raw subset bike drops out |
| Metric naming | "mAP" = AP50 (YOLO / VOC habit) or AP@[.5:.95] (COCO) | Use the column headers. SS-DC's "COCO-style" values look like AP50 |
| Input bit depth / tone mapping | 8-bit vendor-AGC JPEG vs 14-bit raw (÷2¹⁴−1, percentile stretch, min-max, learned) | Raw-input numbers are much lower without learned tone mapping (Thermal Chameleon RAW 37.9 vs AGC 53.5 mAP for YOLOX) |
| Resolution / pretraining | 512×640 to 1280; COCO-pretrained vs scratch | Changes AP by several points. Record it |
| Version mixing | "FLIR" leaderboards (PwC) = aligned v1; Neural-AGC mixes v1+v2; Thermal-Det reports both | Never mix v1/aligned and v2 numbers |

---

## Uncertainties

1. **Exact splits are unstated** for IR Reasoner (which "test"), ECDM, Thermal-Det ("FLIR-V2" split and classes), Physical Backdoor and Le MuMo JEPA. These rows are marked unverified.
2. **Thermal Chameleon's class set** and the cause of its 10,724 train count are not stated in the paper. Its repo contains `FLIR_test_raw_corrected.json`, which I did not open.
3. **Neural-AGC's protocol** ("aligned union of v1 and v2", 12,000 frames, 15 classes, 3,742 test) does not match either release. Treat its numbers with caution.
4. **SS-DC's metric**: labelled COCO-style, but the magnitudes suggest AP50.
5. **Zha et al. (IET-IP 2024)** was paywalled (HTTP 403). Its 2,292-frame test set and its tracking metrics are unverified.
6. **Leaderboard search**: I searched the PwC archive, Wayback and the web. A v2 leaderboard on a platform I did not check (e.g., a Chinese benchmark site) cannot be ruled out.
7. Many lower-tier YOLO papers on v2 (B Table C) were not read in full. They probably add more variants of P1 (custom class subsets).

---

## References

- FLIR ADAS v2 README (baseline, release notes): https://adas-dataset-v2.flirconservator.com/dataset/README.txt
- Papers with Code archive, evaluation tables (2025-07): https://huggingface.co/datasets/pwc-archive/evaluation-tables
- A1 Uzun et al., CVPRW 2022: https://arxiv.org/abs/2204.08745 · C5 Sci Rep 2025: https://arxiv.org/abs/2405.06383
- A2 HOTCOLD Block, AAAI 2023: https://arxiv.org/abs/2212.05709 · code https://github.com/weihui1308/HOTCOLDBlock
- A3 IR Reasoner, CVPRW 2023: https://openaccess.thecvf.com/content/CVPR2023W/PBVS/html/Gundogan_IR_Reasoner_Real-Time_Infrared_Object_Detection_by_Visual_Reasoning_CVPRW_2023_paper.html · code https://github.com/tlgksy/ir-reasoner
- A4 Physical Backdoor, CVPR 2024: https://arxiv.org/abs/2404.19417
- A5 Thermal Chameleon, RA-L 2024: https://arxiv.org/abs/2410.18340 · code https://github.com/donkeymouse/ThermalChameleon
- A6 ECDM, ACM MM 2024: https://arxiv.org/abs/2408.03748 · code https://github.com/lengmo1996/ECDM
- A7 POD, ICIP 2024: https://arxiv.org/abs/2309.15519
- A8 MS-YOLO, IJCNN 2025: https://arxiv.org/abs/2509.21696
- A9 Neural-AGC, ICCVW 2025: https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html · code https://github.com/hosseinjavidnia/NAGC
- A10 Thermal-Det, CVPR 2026: https://arxiv.org/abs/2605.10130
- A11 Le MuMo JEPA, CVPRW 2026: https://arxiv.org/abs/2603.24327
- MODIPHY / YOLO Phantom, ICIP 2024: https://arxiv.org/abs/2402.07894
- C2 Zha et al., IET Image Processing 2024: https://ietresearch.onlinelibrary.wiley.com/doi/full/10.1049/ipr2.13053
- C3 MLLMs on thermal images: https://arxiv.org/abs/2406.13898
- C8 SS-DC: https://arxiv.org/abs/2507.12017 · split repo https://github.com/xi666666/FLIR-ADAS-UDAOD
- C9 VRU multi-modal detection: https://arxiv.org/abs/2509.06333
- C11 XCalib: https://arxiv.org/abs/2609.21770
- MDQF (aligned-v1 example; states "well-registered and relabeled version provided by Zhang"): https://arxiv.org/abs/2601.08458
- RGB→thermal component-aware translation (version unstated): https://arxiv.org/abs/2509.23243
- Thermal segmentation cross-spectral UDA ("FLIR ADAS", version unstated): https://arxiv.org/abs/2505.06951
- v2 audit (raw-TIFF subset, empty frames): https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/00-the-dataset.md
