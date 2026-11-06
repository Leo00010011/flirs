# FLIR ADAS v2: known problems and established workarounds (Q7)

**Research date:** 2026-11-02
**Group:** E (see `../PLAN.md`). Builds on `01_dataset_overview.md` (Group A) and `02_usage_and_publications.md` (Group B).

**How this was verified.**
- **Official annotations.** I re-analysed the official `coco.json` files that Group A pulled out of the archive by range requests. Running my own scripts over them, I computed degenerate and duplicate boxes, box-size statistics, attribute values and per-video class concentration. Results from this are marked **[verified, own analysis]**.
- **Repo code and configs.** I read the dataset loaders, converters and configs of the v2 repos listed by Group B. I downloaded the one annotation file that ThermalChameleon ships (`FLIR_test_raw_corrected.json`) and compared it box by box with the official test `coco.json`.
- **Papers.** I read the dataset sections of 17 papers in full text (arXiv or CVF PDFs).
- **Not verified.** Claims I could not check myself are marked *(unverified)*. The GitHub REST API rate-limited me, so I read the issue threads through the web pages instead, which is less complete.

---

## TL;DR

1. **There is no canonical v2 benchmark protocol.** FLIR's README calls its splits "a suggestion". Papers evaluate on at least five different test sets: the official val split (1,144 frames), the full video test split (3,749), the raw-only test subset (2,151), the raw-only test subset without empty frames (1,895), and custom random splits. Papers also use at least six different class sets. **mAP numbers from different v2 papers are usually not comparable**, and that includes comparisons against FLIR's own YOLOX baseline.
2. **The rare classes are unusable, and the test split is dominated by a single video.** 6 of 16 classes have fewer than 50 thermal train boxes. The test split has no `bus`, `train`, `deer`, `skateboard`, `stroller` or `scooter`. **[verified, own analysis]** One test video (`ZAtDSNuZZjkZFvMAo`, 1,033 of 3,749 frames) holds 74% of test persons, 51% of cars, 66% of lights, and 100% of bikes, motors and dogs. That video is also one of the two recorded with a different camera that ship no raw data.
3. **Class sets.** The most common choice is the **5-class set (person, bike, car, light, sign)**, used by ECDM (ACM MM'24), IR-Reasoner (CVPRW'23), SS-DC/UDAOD and the community audit. Next is **person + car**, used by the FLIR baseline, ThermalChameleon (RA-L'24), Physical Backdoor (CVPR'24), Uzun et al. and NAGC. Adversarial work uses **person only, height >120 px** (HOTCOLD, AAAI'23). One paper prunes classes formally down to 7 (IEEE eIT'26).
4. **Class merging happens quietly.** ThermalChameleon's "corrected" test file relabels every `truck` (948) and `other vehicle` (696) box in its test videos as `car`. **[verified, own analysis]** The paper does not say so. FLIR's own definitions are inconsistent: `car` includes pickups and vans, while `truck` is semi/freight.
5. **Annotation quirks [verified, own analysis].**
   - 74% of thermal train boxes and 85% of thermal test boxes are COCO-"small" (<32²). 25% have a side shorter than 8 px.
   - 82 thermal boxes are degenerate (≤1 px wide or high) and cause NaN losses in Ultralytics.
   - 11 exact duplicate boxes.
   - 53 reflections are kept as positives.
   - RGB and thermal were annotated independently. On the same moments, the visible frames have about 2.5× more lights and 2.7× more signs [community audit].
   - **I found no published systematic label-noise audit or relabelling of v2.**
6. **Raw 16-bit data is broken in part of the test split.** 1,598 test frames from 2 ADK/Boson videos ship 8-bit AGC copies in `analyticsData/` instead of 14-bit raw. A pipeline that only checks whether the file exists renders them solid black. **The established workaround is to read the TIFF header and drop non-16-bit frames, which leaves a 2,151-frame test split** (ThermalChameleon issue #2, community audit). Whether raw beats the 8-bit AGC JPEGs is contested: one paper reports a gain from a learned tone mapping (NAGC, +15 AP50, but under a non-comparable protocol), while a controlled community ablation found no measurable difference (±1.3%).
7. **RGB–thermal pairing.** There is no pixel registration, and pairs exist only in the test split (time-synced, through `rgb_to_thermal_vid_map.json`). The workarounds:
   - use thermal only (the dominant choice);
   - use the v1-based **FLIR-aligned** set for fusion work;
   - use deliberately **unpaired** RGB→IR domain adaptation (SS-DC / FLIR-ADAS-UDAOD, MODIPHY);
   - use **late fusion** (VRU paper);
   - use **dense registration** of video sequences (XCalib, a preprint).

   Some papers still treat v2 pairs as pixel-aligned, which is a pitfall.
8. **Licensing.** The terms-of-service URL in the README is dead (404). The license is non-commercial and does not authorise re-hosting. Several mirrors and derived protocols re-host the images anyway, under "CC BY 4.0" (Roboflow) or "CC BY-NC 4.0" (the UDAOD Baidu archive). The clean workaround, used by ThermalChameleon and the community audit, is to **share only annotation files, ID lists and scripts**.
9. **Domain shift.**
   - Train/val come from Santa Barbara, England and France. The test videos come from N. California, Idaho and Michigan.
   - 2 of the 8 test videos were recorded with a different thermal camera.
   - Weather tags exist for only about 10% of train frames, and those are mostly clear [verified].
   - The test split is harder than train (85% small boxes vs 74%).
   - Zero-shot detectors score about 4× lower on v2 than on FLIR-aligned: AP 0.096 vs 0.372 for Thermal-Det, CVPR'26.

---

## Problem 1: Annotation errors and inconsistencies

### Problem (evidence)

| Issue | Evidence | Source |
|---|---|---|
| Degenerate boxes | Thermal: 75 train + 6 val boxes with w or h ≤ 1 px. RGB: 1 train, 9 test. No out-of-bounds boxes. | **[verified, own analysis]** of official `coco.json`. The audit [C1] reports 68 + 6 sub-pixel boxes after clipping in its 5-class subset |
| Exact duplicate boxes | Thermal: 2 train, 8 val, 1 test. Near-duplicates (same class, IoU > 0.9): 3 in thermal train | **[verified, own analysis]** |
| Duplicate frames / leak | 8 duplicate RGB train frames (MD5) and 1 frame present in both RGB train and RGB val | [C1] *(unverified by me)* |
| Very small boxes | Thermal train: 73.6% of boxes < 32² px, 24.8% have a side < 8 px. Thermal test: 84.7% < 32², 27.6% have a side < 8 px. RGB train: 40.4% < 32². Median thermal box height: sign 11 px, light 16 px, car 22 px, person 26 px, skateboard **4 px** | **[verified, own analysis]**. [C1] agrees |
| Visible-part-only boxes | README: "only non-occluded parts of the object were annotated", favouring heads and shoulders. 47% of the boxes that carry an occlusion attribute are partly or heavily occluded | [R1] README · [C1] |
| Reflections labelled as objects | `reflection: yes` on 53 thermal train boxes (32 person, 20 car, 1 bike) and 290 RGB boxes. `coco.json` keeps them as normal positives | **[verified, own analysis]** · [C1] |
| Undocumented attribute | `specify_if_human_or_not: "spt"` on 31 thermal `person` boxes. Not explained in the README | **[verified, own analysis]** · [C1] |
| Rider flag missing | The README describes an `is_rider` flag, but it is absent from `coco.json`. The `bike` box excludes the rider, so the warm part sits outside the bike box | [R1] · [C1] |
| Vehicle-class ambiguity | README: `car` "includes pick-up trucks and vans", `truck` = semi/freight, `other vehicle` = construction/trailers. The `type` attribute shows `car` boxes tagged trailer (11), emergency (7) and construction (3) | [R1] · **[verified, own analysis]** |
| Independent RGB / thermal labelling | Same 2,151 test moments: visible/thermal box ratio is person 1.00, car 1.09, **light 2.54, sign 2.70**. On all 3,749 moments: light 2.64, sign 3.04 | [C1] doc 03 *(unverified by me; consistent with the totals in 01)* |
| Undocumented `deer`; scooter ID 75 vs README 77 | See 01 §1.3 | [R2] |
| Missing or incomplete labels | **I found no published missing-label audit or relabelled v2.** A web search turned up nothing specific. The one "corrected" file I found (ThermalChameleon) turns out to be class merging, not added objects (Problem 2) | — |

### Impact
- **Degenerate boxes.** A box narrower than 1 px after normalisation produces **NaN loss** in Ultralytics "an hour into a run" [C1].
- **Small boxes.** A 74–85% share of small objects sets a hard ceiling for stride-8/16 detectors and makes AP_S dominate COCO-style AP. Signs and lights sit at the resolution floor.
- **Asymmetric scoring.** Because each modality has its own ground truth, a thermal detector is never penalised for lights or signs that are invisible in thermal. Per-class AP for light and sign is therefore not comparable between RGB and thermal [C1].
- **Reflections.** These teach the detector to fire on mirror images. The effect is tiny (0.03% of boxes).

### Workarounds

| Workaround | Who | Link | How established |
|---|---|---|---|
| Use `coco.json`, not `index.json` (FLIR's filtered and remapped version, "recommended for training") | FLIR, the audit, most repos | README [R1] | Widely used. Exception: some converters read the raw `index.json`, e.g. [FLIR2YOLO `flir2yolo.py#L31`](https://github.com/ax2mazhr/FLIR2YOLO/blob/main/flir2yolo.py#L31) and [customize-flir-dataset](https://github.com/sarisemih/customize-flir-dataset/blob/main/jsontoyolo/convert.py) |
| Clip boxes to the frame, drop boxes under 1 px | Community audit | [`convert.py` `_yolo_line`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/convert.py#L176) | Ad hoc, but it is standard detector hygiene (Ultralytics also drops tiny boxes in its augmentation candidate filter) |
| Minimum-size filter (person height > 120 px) | HOTCOLD (AAAI'23), Strack et al. (ICIP'24) "following Wei et al." | [`json2yolo.py#L38-L45`](https://github.com/weihui1308/HOTCOLDBlock/blob/main/dataset/json2yolo.py#L38-L45) | A few papers, for adversarial-patch work only. It throws away about 99% of persons (1,255 images remain) |
| Upsample the input for small objects (test size 1280) | IR-Reasoner (CVPRW'23) | [README table](https://github.com/tlgksy/ir-reasoner#performance) | A few papers. Most use native 640 |
| Match categories **by name**, not by README ID | Community audit | [C1] | Ad hoc but essential (scooter = 75, and 77 is `scissors`) |
| Keep reflections and `spt` boxes as positives (default), or drop them using the attribute | Audit keeps them | [C1] | No one is documented dropping them |
| Compare RGB vs thermal only on person and car, or compare annotation counts on paired test moments | Community audit | [C1] doc 03 | Ad hoc |

---

## Problem 2: Class imbalance, rare classes, and merging

### Problem (evidence)
- Thermal train boxes: car 73,623 · person 50,478 · sign 20,770 · light 16,198 · bike 7,237 · bus 2,245 · other vehicle 1,373 · motor 1,116 · hydrant 1,095 · truck 829 · skateboard 29 · stroller 15 · scooter 15 · deer 8 · train 5 · dog 4 [R2].
- The car:dog ratio is about 18,000:1.
- **Test coverage [verified, own analysis].** The thermal test split has 0 boxes for `bus` (despite 2,245 in train), `train`, `deer`, `skateboard`, `stroller` and `scooter`, so 10 of 16 classes are scorable.
- **Single-video concentration in test [verified, own analysis].** Video `ZAtDSNuZZjkZFvMAo` holds:
  - 74% of thermal test persons, 51% of cars, 66% of lights and 60% of signs;
  - 100% of bikes (113), motors (3,314) and dogs (25).

  Video `ePoikf5LyTTfqchga` holds 100% of `other vehicle` boxes (696). In val (17 videos), no video holds more than 56% of any class.
- LYNRED-MDS (2026) notes that "6 out of the 16 classes of FLIR have under 100 samples in the train set and most of these are not present in the test set" [P15].

### Impact
- **mAP averages over the classes present.** Ultralytics and pycocotools skip classes with no GT, so the thermal test mAP of a 5-class model is a mean over 4 classes, because there are no bikes in the raw-only test [C1]. The RGB mAP over the same class set is a mean over 5. Headline mAPs across modalities and class sets are not comparable.
- **Rare classes add noise, not signal.** With 4 dog boxes, `dog` carries the same weight in mAP as `car` [C1]. The YOLOv8 per-class results in [C4] show the rare classes "perform abysmal".
- **Few effective test samples.** The test split amounts to 6–8 videos of consecutive frames, so it is not i.i.d. A detector's score on `motor`, `bike` or `dog` is a score on one video. On the 2,151-frame raw subset, one 428-frame daytime video holds all test lights, 96% of signs and 61% of people [C1].

### Workaround: which classes papers and repos keep

| Paper / repo (venue) | Classes kept or merged | Evidence |
|---|---|---|
| FLIR YOLOX-m baseline (README) | Reports **person, car** AP50 only (training class set unstated) | [R1] |
| HOTCOLD Block (AAAI'23) | **person** only, height > 120 px; 1,255 imgs (878/377) | [P1]; [`json2yolo.py#L38`](https://github.com/weihui1308/HOTCOLDBlock/blob/main/dataset/json2yolo.py#L38), [`custom.yaml`](https://github.com/weihui1308/HOTCOLDBlock/blob/main/victim_detector/data/custom.yaml) (`nc: 1`) |
| Strack et al. (ICIP'24) | **person** ("following Wei et al.") | [P2] |
| IR-Reasoner (CVPRW'23) | **person, car, sign, light, bike** (rare classes such as dog dropped) | 02 §3.1. The public repo ships only COCO-default 80-class configs ([`data/coco.yaml`](https://github.com/tlgksy/ir-reasoner/blob/main/yolor-reasoner/data/coco.yaml)), so the class filter is **not reproducible from code** |
| Physical Backdoor (CVPR'24) | Results on **person, car**; "Flir_v2_T", 13,460 thermal images (where the 13,460 comes from is unexplained) | [P3] |
| ThermalChameleon (RA-L'24) | **person, car**, with **`truck` and `other vehicle` merged into `car`** in the test file | **[verified, own analysis]** of [`FLIR_test_raw_corrected.json`](https://github.com/donkeymouse/ThermalChameleon/blob/main/FLIR_test_raw_corrected.json): 2 categories; all 948 truck + 696 other-vehicle boxes in its videos relabelled as `car`; nothing else added or removed. The paper does not mention this [P4] |
| ECDM (ACM MM'24) | **person, bike, car, light, sign** ("due to the limited labeling") | [P5]. The repo has no FLIR loader (LLVIP/PRW only): [`ecdm/data/dataset.py`](https://github.com/lengmo1996/ECDM/blob/main/ecdm/data/dataset.py) |
| Uzun et al. (CVPRW'22; Sci. Rep.'25) | **car, person** ("insufficient number of samples … in the other classes") | [P6] |
| MODIPHY (ICIP'24) | **person, car, light, sign** ("four most represented classes") | [P7] |
| MS-YOLO (IJCNN'25) | 9: **person, bike, car, motor, bus, truck, light, hydrant, sign** | [P8] |
| NAGC (ICCVW'25) | "fifteen categories", **person and car** AP50 reported | [P9] |
| SS-DC (arXiv'25) / FLIR-ADAS-UDAOD protocol | **person, bike, car, sign, light** ("which have complete labels") | [P10]; [UDAOD README](https://github.com/xi666666/FLIR-ADAS-UDAOD#4-object-categories) |
| VRU detection (arXiv'25) | **Merged** into a KITTI/BDD-style map: person (+stroller?) → pedestrian; bike → cyclist; dog → animal; train + other vehicle → other vehicle; skateboard, light, hydrant, sign → "don't care" (parsed from a garbled PDF table, so approximate) | [P11] |
| Two-stage class pruning (IEEE eIT'26) | Stage 1 drops train, dog, deer, skateboard, stroller, scooter (10 left). Stage 2 drops truck, hydrant, other vehicle based on PR/F1 curves, leaving **person, bike, car, motor, bus, light, sign** | [P12]. Note: stage 2 selects classes from performance curves, which risks circularity |
| LIWL-YOLO (Sci. Rep.'25) | Classes with < 100 samples dropped | 02 §3.3 *(unverified by me)* |
| Thermal-Det (CVPR'26) | Open-vocabulary evaluation on v2 class names (motorcycle and traffic light reported) | [P13] |
| Community audit (ThermalObjectDetection) | **person, bike, car, light, sign**, chosen *before* seeing results; bike kept although absent from the raw test | [C1]; [`convert.py#L42`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/convert.py#L42) |
| FLIR2YOLO converter | 11: person, bike, car, motor, bus, train, truck, light, dog, scooter, other vehicle (**silently drops sign, hydrant, skateboard, stroller, deer**) | [`flir2yolo.py#L28`](https://github.com/ax2mazhr/FLIR2YOLO/blob/main/flir2yolo.py#L28) |
| customize-flir-dataset | 6 by default: person, car, bike, motor, bus, truck | [`labels.py`](https://github.com/sarisemih/customize-flir-dataset/blob/main/jsontoyolo/labels.py) |
| yolov8-nightshift | Trains all 16, then reads only the "representative" classes | [C4] |

**How established each workaround is:**
- **5-class set (person, bike, car, light, sign):** the most common v2 convention (≥ 4 papers and repos). It is effectively the inheritance of the v1 classes plus light and sign. Widely used.
- **Person + car only:** widely used for tone-mapping and security papers, and matches what FLIR reports.
- **Merging truck / other vehicle into car:** done silently by ThermalChameleon. A coarse "vehicle" class is common in other datasets, but on v2 I found it explicitly documented only in the VRU paper (other vehicle). Ad hoc.
- **Loss-side handling instead of dropping** (SlideLoss in MS-YOLO [P8]): a few papers.
- **Picking classes from test performance** (eIT'26): avoid. The audit explicitly warns that "dropping a class after seeing its numbers is how a result gets tuned" [C1].

---

## Problem 3: Split ambiguity and non-comparable protocols

### Problem (evidence)
- **Naming.** The README calls the third split "validation video (val_video)", but the folders are `video_*_test`, and FLIR's own baseline is "tested on the video test data" [R1].
- **Splits are optional.** The README states: "The data organization and splits are a suggestion … the end-user is free to use a different scheme" [R1].
- **Frame counts disagree.** The official counts (9,711 / 9,233 / 26,442) differ from the archive (11,886 / 11,403 / 30,787) [01 §1.1]. Papers quote either.
- **Splits actually used:**

| Protocol | Train | Eval | Who |
|---|---|---|---|
| Official image train → **video test** (3,749) | 10,742 | 3,749 | FLIR baseline [R1]; EIT'26 [P12] |
| Official train → **val as test**; new val carved from train | 90% of train | 1,144 | Uzun et al. [P6] ("similar to previous studies") |
| Raw-only test (drops the 2 ADK videos) | 10,742 (TC: **10,724**) | **2,151** | Community audit [C1]; ThermalChameleon [P4] |
| Raw-only test without empty frames | — | **1,895** | ThermalChameleon test JSON **[verified, own analysis]** |
| Train / val / test with empty frames dropped | 10,474 | 3,493 (+1,127 val) | MS-YOLO [P8]. MODIPHY uses 10,478 / 10,053 train, which equals the counts of frames with ≥ 1 box |
| Custom random split after filtering | 878 | 377 | HOTCOLD [P1]. A user reproducing it got **1,169 / 86** ([issue #8](https://github.com/weihui1308/HOTCOLDBlock/issues/8)) |
| v1 ∪ v2 "aligned union", deduplicated by timestamp | 90% of ~12,000 | "3,742" video-test frames | NAGC [P9] |
| 10,742 train / 2,292 test, 8-bit | | | Zha et al. IET'24 (02 §3.3) *(unverified)* |
| RGB→IR UDA: 5,663 RGB labelled / 4,856 IR unlabelled | | 1,144 IR | SS-DC / UDAOD [P10] |

### Impact
- **Numbers do not compare across papers.** NAGC compares its AP50 (90.1 / 86.3) directly with FLIR's README baseline (75.33 / 77.23), even though the training data (v1 ∪ v2), the test frames (3,742) and the detector all differ [P9].
- **Val and test are different distributions.** In the audit, test mAP50-95 sat about 0.10 below val (0.417 vs 0.322) [C1].
- **Random frame-level splits leak.** The image splits come from videos, so random splits after filtering (HOTCOLD) can place near-duplicate frames of the same video in train and test *(likely, not verified)*.
- **The video test split is correlated.** It consists of consecutive frames (index step 1) [C1], so its effective sample size is far smaller than 3,749.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Use the official folders (train → video test) and report the exact frame count | FLIR, EIT'26, MS-YOLO, audit | Most common, but frame counts still vary |
| Report both val and test, per class | Community audit [C1] | Ad hoc |
| Publish image lists or annotation files for the test split actually used | ThermalChameleon (`FLIR_test_raw_corrected.json`); UDAOD (split files) | A few repos |
| Split by **video_id**, never by frame, for custom splits | Recommended by the README's "independent video sequences" design. The audit verified no video appears in two splits | Implied; rarely stated explicitly |
| Avoid comparing against the README baseline unless the protocol matches | Audit ("it is not a comparable number") | Ad hoc |

---

## Problem 4: No registration, and how RGB–thermal pairs are built

### Problem (evidence)
- **Different sensors, no calibration published.** Thermal: Tau 2 at 640×512, 45° HFOV. RGB: Blackfly S at 52.8° HFOV, in 5 resolutions. FLIR publishes no intrinsics or extrinsics [01 §4.1].
- **Pairing map exists only for test.** `rgb_to_thermal_vid_map.json` gives 3,749 time-synced test pairs, with **temporal** pairing only. The image splits have no map. About 39% of thermal train frames (4,217) can be paired by matching capture names and frame indices in `index.json` [C1] *(unverified by me)*.
- **Parallax.** XCalib reports that a global homography fails on v2 because of depth-dependent parallax, and that frame-to-frame flow is large "due to the low framerate" [P14].
- **The literature confuses v2 with FLIR-aligned.**
  - LYNRED-MDS states that FLIR-aligned was obtained by removing misaligned v2 images [P15]. In fact it was built from **v1** (Zhang et al. 2020).
  - MSCoTDet and ThermEval cite the v2 page but use aligned v1 (02 §1).
  - Le MuMo JEPA (CVPRW'26) samples synchronised RGB and thermal crops on FLIR "to preserve pixel alignment" [P16]. v2 pairs are not pixel-aligned, and the paper does not say how it built the pairs *(how they paired is unknown)*.

### Impact
- Early or feature-level fusion trained on raw v2 pairs learns misregistered features.
- Each modality has its own boxes, so one label set cannot be shared across modalities. "A box drawn on a visible frame has no correct position on its thermal pair" [C1].
- Fusion benchmarks on v2 essentially do not exist: 0 of 11 major-venue v2 papers do RGB-T fusion detection (02 §3.1).

### Workarounds

| Workaround | Who | Link | How established |
|---|---|---|---|
| **Use the v1-based FLIR-aligned set** (5,142 pairs, 3 classes) for fusion and keep v2 for thermal-only work | Nearly all RGB-T fusion papers (CVPR / ECCV / ICCV / AAAI, 02 §3.4); e.g. [RGBT-ObjectDetection loader](https://github.com/Danial-Qashqai/RGBT-ObjectDetection/blob/main/Dataloader.py) | 01 §4.2, 02 §3.4 | **Widely used**, the de facto standard |
| **Thermal only** (ignore RGB) | Most v2 papers (A1–A9 in 02) | — | **Widely used** |
| **Deliberately unpaired** RGB→IR domain adaptation: scene-level matching, then randomly drop one modality per scene to avoid weak alignment | SS-DC [P10] / [FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD#3-weak-alignment-mitigation) | | A few papers (new, 2025–26) |
| **Mixed unpaired multimodal training** (RGB and thermal images as separate samples of one detector) | MODIPHY [P7]: "RGB and thermal images are not directly correspondent … our curated multimodal dataset mixes various thermal and RGB images"; yolov8-nightshift "combined" model [C4] | | A few |
| **Late or decision-level fusion** (two detectors, confidences combined) | VRU paper [P11] | | A few |
| **Time-synced test pairs via the official map** (same moments, separate GT) | Community audit [C1]; HF `Maxim37/flir_adas_video_pairs` | | Ad hoc |
| **Dense depth-guided registration** of video sequences (672 day + 1,040 night pairs) | XCalib [P14] (preprint, code not yet released) | | Single preprint |
| **Recover image-split pairs** through `index.json` capture name + frame index (about 39% of train) | Community audit [C1] | | Ad hoc, unofficial |
| **"Pair" the thermal 8-bit JPEG with the 14-bit TIFF** (same sensor, so pixel-aligned) instead of with RGB | 2610.33118 (arXiv'26) [P17]: "native grayscale channel is strictly pixel-aligned with the 14-bit calibrated TIFF" | | Single preprint, but a sound trick |

---

## Problem 5: Missing RGB frames and unequal modality coverage

### Problem (evidence)
- Train: 10,742 thermal vs 10,318 RGB. Val: 1,144 vs 1,085. So **at least 424 + 59 thermal frames have no RGB frame at all**, and the rest have no official pairing [R2].
- Test: 1:1 by map, but:
  - 256 thermal test frames are empty while all 3,749 RGB test frames have boxes **[verified]**;
  - RGB test resolution is 1224×1024 (2,151 frames) or 1024×768 (1,598 frames, the ADK videos) [R2].
- RGB train resolutions vary (720×480 to 2048×1536).
- The audit found 9 duplicate RGB frames in train/val [C1].

### Impact
- An "RGB-T" training set built from the image splits is either very small (only the recoverable pairs) or unpaired.
- When one modality's test frames are dropped (e.g. the 1,598 non-raw thermal frames), their partners must also be dropped, or the RGB and thermal scores refer to different moments [C1].
- Downscaling RGB to 640 px roughly equalises object pixel heights with thermal for person and car, but sign and light end up about 25% smaller on RGB [C1].

### Workarounds
- **Drop test pairs symmetrically** through the official map, so both modalities are scored on the same 2,151 moments. Used by the community audit ([`paired_rgb_stems`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/convert.py#L229)). Ad hoc.
- **Treat the modalities as independent datasets** (separate detectors, or a mixed detector). MODIPHY [P7], nightshift [C4], VRU [P11]. A few papers.
- **Resize everything to a fixed long side** (640) and accept the resolution asymmetry. Most YOLO pipelines do this, and the audit documents its effect. Widely used, rarely acknowledged.

---

## Problem 6: 8-bit AGC JPEG vs 16-bit raw TIFF

### Problem (evidence)
- **What the files are.** `data/` holds 8-bit JPEGs after FLIR's proprietary per-frame AGC. `analyticsData/` holds 16-bit TIFFs containing 14-bit counts (the maximum observed in 1,700 frames is 11,947), from a Tau 2 in T-linear mode [R1][C1].
- **AGC is proprietary and per-frame.** "The code of the FLIR TMO is not publicly available" [P18]. AGC removes absolute temperature: "the same person … is a different pixel value in the next frame if a hot exhaust enters the scene" [C1].
- **Raw is a narrow band.** The median frame spans about 764 counts (p1–p99), while a dataset-wide window needs about 2,219, so a global linear window uses only about 88 of 255 output levels [C1].
- **The papers disagree on the effect:**

| Study | 8-bit AGC vs raw finding |
|---|---|
| ThermalChameleon (RA-L'24) [P4] | Direct use of RAW (linear /2¹⁴, min-max) is **worse** than 8-bit. Its learned task-adaptive tone mapping beats FLIR AGC by 5–12% |
| NAGC (ICCVW'25) [P9] | Learned 16→8-bit mapping lifts YOLOX-m AP50 to 90.1 / 86.3 vs the README's 75.3 / 77.2. **The protocol is not comparable** (v1 ∪ v2, different test count) |
| Community audit [C1] | AGC vs global window vs per-frame p1–p99: **all within 1.3%** on val (single seed). The global window learns more slowly early on (0.274 vs 0.351 at epoch 10) |
| Joint tone mapping / denoising (SPIE'23) [P18] | FLIR's 8-bit images "perform best across most of the measures" among classical TMOs. (Uses "v1.3" while citing the v2 URL, so the version is ambiguous) |

### Impact
- The choice of input representation can move results by several AP points, and most papers do not state which representation they use. Unless a paper says "raw" or "TIFF", the default is the 8-bit JPEG.
- Pipelines that read raw TIFFs hit the broken test TIFFs (Problem 7).

### Workarounds

| Workaround | Who | Link | How established |
|---|---|---|---|
| **Use the 8-bit AGC JPEGs** (default) | FLIR baseline, almost every detection paper | — | **Widely used** |
| **Per-frame percentile stretch** of raw (p0.5–p99.5 or p1–p99) | NAGC (as pre-normalisation, after centre-crop and resize to 640×512) [P9]; audit `p1p99` arm | [audit `radiometry.py`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/radiometry.py) | A few |
| **Fixed global window** from train-split pooled percentiles (never from val/test) | Community audit (`global` arm) | [C1] doc 02 | Ad hoc |
| **Counts → temperature** (Kelvin = counts × 0.04, i.e. T-linear), then a learned multi-channel embedding | ThermalChameleon | [`thermal_embedding.py#L26-L37`](https://github.com/donkeymouse/ThermalChameleon/blob/main/thermal_embedding.py#L26-L37) | A few (the 0.04 K/count factor is a FLIR T-linear convention, not documented in the v2 README; *unverified for every frame*) |
| **Learned tone mapping** trained end-to-end with the detector | ThermalChameleon (TCNet), NAGC (U-Net gain/offset) | [ThermalChameleon](https://github.com/donkeymouse/ThermalChameleon), [NAGC](https://github.com/hosseinjavidnia/NAGC) (README only, code not in repo as of 2026-11-02) | A few papers, an active topic |
| **CLAHE on 8-bit** | EIT'26 [P12] (modest gains, medium model only) | | Ad hoc |
| Replicate 1 channel to 3 for ImageNet/COCO weights; thermal photometric augmentation off (hsv_h = hsv_s = 0) | ThermalChameleon [P4]; audit [`base.yaml`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/configs/base.yaml); Le MuMo JEPA (no colour jitter on thermal) [P16] | | Widely used |

---

## Problem 7: Test-split quirks

### Problem (evidence)
1. **Fake raw data.** Videos `ZAtDSNuZZjkZFvMAo` (1,033 frames) and `t3f7QC8hZr6zYXpEZ` (565) ship **8-bit greyscale TIFFs** (copies of the AGC frame) in `analyticsData/`, 1,598 of the 3,749 test frames. They are tagged as an ADK/Boson camera (50° HFOV) in `index.json` [C1].
   - The ThermalChameleon author confirmed this in [issue #2](https://github.com/donkeymouse/ThermalChameleon/issues/2), which was opened by a user whose reproduction on the full 3,749-frame test "did not meet expectations". About 1,000 images that should be 14-bit are 8-bit, e.g. `video-ZAtDSNuZZjkZFvMAo-frame-001473`.
   - With a global window, every such frame renders **solid black** (400/400 sampled) without raising any error [C1].
2. **Class coverage.** There are no `bus`, `train`, `deer`, `skateboard`, `stroller` or `scooter` boxes. All 113 thermal bikes, 3,314 motors and 25 dogs are in the excluded ADK video **[verified, own analysis]**. A raw-only test therefore has **no bike, motor or dog**.
3. **Single-video dominance.** See Problem 2. The full test is dominated by `ZAtDSNuZZjkZFvMAo`. On the raw subset, one 428-frame daytime video holds all lights [C1].
4. **Metadata.** There are no per-frame `hours`/`scene`/`weather` tags in test **[verified]**. The video-level tags in `index.json` can be propagated to frames, which is an imputation [C1]. Test boxes carry no occlusion or reflection attributes, only `track_id` **[verified]**.
5. **Harder and sparser than train.** 84.7% small boxes vs 73.6% in train **[verified]**. Raw-subset density is 7.2 boxes per frame vs 15.7 in train [C1].
6. **Frame rate unknown** (24 Hz per the product page, 30 fps per the README; no timestamps) [01].
7. **256 empty thermal test frames**, all inside the 6 raw videos (2,151 − 1,895 = 256) **[verified]**.

### Impact
- **Results differ by test variant.** Results on "the v2 test split" differ by which variant is used (3,749 / 2,151 / 1,895), by whether empty frames are kept (empty frames affect false-positive counts), and by class coverage.
- **Raw-data methods.** Methods that consume raw data evaluated on all 3,749 frames are silently fed AGC images for 43% of the frames.
- **Thin per-class evaluation.** Per-class test numbers for bike, motor and dog rest on one video, and light and sign on one video in the raw subset.

### Workarounds

| Workaround | Who | Link | How established |
|---|---|---|---|
| Read the TIFF header and treat non-16-bit files as missing; drop those frames from every arm (8-bit too) for a 2,151-frame test | Community audit; ThermalChameleon | [`convert.py` `_raw_stems`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/convert.py#L122); [TC issue #2](https://github.com/donkeymouse/ThermalChameleon/issues/2) | **Emerging standard for raw-data work** (2 independent sources). Not documented by FLIR |
| Use the shipped test JSON of a prior paper for exact comparability | ThermalChameleon `FLIR_test_raw_corrected.json` (1,895 frames, person + car, truck/OV merged) | [file](https://github.com/donkeymouse/ThermalChameleon/blob/main/FLIR_test_raw_corrected.json) | Single repo. Its merging must be stated when reused |
| Keep empty frames as negatives (an empty `.txt`, not a missing file) | Community audit | [C1] | Ad hoc but correct. Several papers instead drop empty frames (MS-YOLO / MODIPHY counts) |
| Report val **and** test, per class, and by lighting tag (imputed from `index.json`) | Community audit | [C1] | Ad hoc |
| Use the val split as the test set | Uzun et al. [P6] | | A few |

---

## Problem 8: Licensing and redistribution

### Problem (evidence)
- **Dead terms link.** The README says "By downloading this data you agree to … https://www.flir.com/oem/adas/adas-dataset-agree/". That URL returns **404** (01 §8.3).
- **License terms.** The indexed regional copy is the "Teledyne FLIR Non-Commercial Image Dataset License Agreement": limited, revocable, non-sublicensable and non-transferable. Its redistribution clauses are *unverified* because the full text could not be retrieved [01 §8.3].
- **Re-hosted copies under other labels:**
  - Kaggle full mirrors (license "Other" or unknown);
  - Roboflow `mateusz-dorobek/flir_adas_v2_thermal`, labelled **CC BY 4.0**;
  - HF `Maxim37/flir_adas_video_pairs` (no license stated);
  - the [FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD#download) protocol, which distributes `FLIR_ADAS_UDAOD_coco.zip` (images included) through Baidu Netdisk with a **CC BY-NC 4.0** badge;
  - the community audit's fetch script, which expects users to re-host the archive on their own "Anyone with the link" Google Drive ([`fetch_dataset.sh`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/scripts/fetch_dataset.sh)), because the official form blocks unattended download.
- **No citable paper.** FLIR has no dataset paper, so citation practice varies: the dead URL, the v2 Conservator URL, or "FLIR 2018/2019/2020/2023" (02 §2).

### Impact
- **Legal risk.** Commercial use, and possibly any public re-hosting (including derived subsets with images), is not covered (my reading; *not legal advice*).
- **Provenance confusion.** Mirror labels such as CC BY 4.0 are not sanctioned by FLIR. Partial Roboflow copies (5,817 images; a car-only export) get mistaken for the full dataset.
- **Reproducibility.** Without a citable paper and a fixed URL, readers cannot tell which version or subset was used.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Download from the official Conservator host (the direct zip URL works without login; MD5 is published) | 01 §8.1 | Recommended. Usage unknown |
| **Redistribute only annotations, ID lists and scripts, never images** | ThermalChameleon (test JSON only); community audit (converter + counts); SS-DC protocol defined by counts | A few repos |
| Cite version, URL, access date and catalogue DOI 10.57702/3yajgd26 (TIB LDM) | Some papers (Le MuMo JEPA cites the v2 URL with access date) | Rare |
| State "non-commercial research use under the Teledyne FLIR license" in derived releases | No consistent example found | Not established |

---

## Problem 9: Domain shift (locations, cameras, conditions)

### Problem (evidence)
- **Locations.** v1 data: Santa Barbara. v2 image splits add England and France. The video test split comes from N. California, Idaho and Michigan [R1]. Train/val and test do not share locations.
- **Cameras.** Train/val thermal are Tau 2 (45° HFOV). Of the 8 test videos, 5 are Tau 2, 2 are ADK/Boson (50° HFOV) and 1 is untagged [C1]. RGB uses 5 resolutions in train and 2 in test [R2].
- **Conditions [verified, own analysis].**
  - Thermal train weather tags exist for only 1,125 of 10,742 frames: clear 623, partly cloudy 432, overcast 69, snowy 1.
  - Scene tags: city street 5,581, residential 2,018, highway 1,380.
  - LYNRED-MDS describes FLIR as "mostly in favorable weather conditions" [P15].
  - Lighting (`hours`) tags cover only 29% of thermal train frames, and 10 train videos carry conflicting `hours` values [C1].
- **Difficulty gap.**
  - Test: 84.7% small boxes. Train: 73.6% **[verified]**.
  - Thermal-Det zero-shot AP: **0.096 on FLIR-V2 vs 0.372 on FLIR-aligned** [P13].
  - Zha et al. call v2 "considerably harder than v1" (02 §3.3, *unverified*).
  - The audit's val→test drop: mAP50-95 0.417 → 0.322 [C1].
- **v1/v2 overlap.** The README describes v2 as an *expansion* of v1 ("+83%"), so v2 likely contains v1 imagery. NAGC had to deduplicate "timestamps" when taking the v1 ∪ v2 union [P9]. Training on FLIR-aligned (v1) and testing on v2 (or the reverse) may therefore leak *(unverified: no frame-level overlap check exists)*.

### Impact
- Val scores overestimate test scores.
- Test results partly measure camera and location transfer.
- Lighting-stratified analyses rest on imputed tags and few videos. In the audit, night results come from 1,497 frames and day results from 428 [C1].

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Restrict test to the Tau 2 videos (which coincides with the raw-only 2,151) | Audit; ThermalChameleon | A few |
| Report lighting-stratified results using video-level tags from `index.json`, stated as imputation; drop videos with conflicting tags | Audit [C1] | Ad hoc |
| Cross-dataset evaluation (FLIR ↔ KAIST / LLVIP / LYNRED-MDS / STheReO) | LYNRED-MDS [P15]; ThermalChameleon zero-shot on STheReO [P4]; FreqKD KAIST→FLIR | A few |
| Physics-motivated augmentation (atmospheric turbulence) | Uzun et al. [P6] | A few |
| Pretrain on larger RGB or driving data, then fine-tune (COCO init; Waymo→FLIR) | FLIR baseline (COCO); Le MuMo JEPA [P16]; audit (COCO beats scratch on both splits) | Widely used |
| Check for v1/v2 overlap before mixing them | NAGC deduplicates by timestamp | Single paper |

---

## Recommended practice checklist (starting with v2)

**Data acquisition and provenance**
- [ ] Download `FLIR_ADAS_v2.zip` from the official Conservator host and check the MD5 (`3150533d3a712c598aa110d5513a1671`). Do not use Roboflow or partial mirrors as "v2".
- [ ] Report counts from the files: 10,742 / 1,144 / 3,749 thermal; 10,318 / 1,085 / 3,749 RGB. Do not quote the README's 9,711 / 9,233 / 26,442 without saying so.
- [ ] Cite "Teledyne FLIR Free ADAS Thermal Dataset v2 (v2.0.0, 19 Jan 2022)", the Conservator URL, the access date and DOI 10.57702/3yajgd26. Never call FLIR-aligned (v1) "v2".
- [ ] Do not re-host images. Share annotation subsets, image-ID lists and conversion scripts only. Non-commercial use only.

**Annotations and classes**
- [ ] Use `coco.json` (not `index.json`). Map categories **by name** (scooter = 75; `deer` = 18 is undocumented).
- [ ] Clip boxes to the image and drop boxes under 1 px (82 such thermal boxes). Remove the 11 exact duplicate boxes.
- [ ] Decide the class set **before** seeing results, and state it. The default for comparability is **person, bike, car, light, sign**. For tone-mapping and security comparisons, also report **person and car**. Never include train, dog, deer, skateboard, stroller or scooter in mAP.
- [ ] If you merge `truck` / `other vehicle` into `car` (or into "vehicle"), say so. ThermalChameleon's test file does this.
- [ ] Keep empty frames as negatives (empty label files). Report whether you did.
- [ ] Decide on reflections (53 thermal boxes) and the undocumented `spt` persons. Keeping them is the default.

**Splits and evaluation**
- [ ] Train on `images_thermal_train`, select models on `images_thermal_val`, and report the final numbers on `video_thermal_test`. Never tune on test.
- [ ] Report **both** the full 3,749-frame test and the **raw-only 2,151-frame** test (drop videos `ZAtDSNuZZjkZFvMAo` and `t3f7QC8hZr6zYXpEZ`), with frame and box counts.
- [ ] Report **per-class AP** next to mAP, and say which classes have no GT in the test variant. Bike, motor and dog are absent from the raw test. Bus is absent from every test variant.
- [ ] Any custom split must be **by `video_id`**, not by frame.
- [ ] Do not compare against FLIR's YOLOX baseline (75.33 / 77.23 AP50) or other papers unless the test frames, class set and input representation match.
- [ ] Use several seeds. The test split amounts to 6–8 correlated videos, and single videos dominate several classes.

**Input representation**
- [ ] State whether you use 8-bit AGC JPEGs (`data/`) or 16-bit TIFFs (`analyticsData/`).
- [ ] If you use raw: check `dtype == uint16` for every TIFF, compute any normalisation statistics on train only, and expect 14-bit values (max ≈ 12k).
- [ ] Thermal augmentation: no hue or saturation jitter. Replicate to 3 channels for RGB-pretrained weights, or adapt the first conv.

**Multimodal use**
- [ ] Do **not** treat v2 RGB–thermal pairs as pixel-aligned. For fusion detection, use FLIR-aligned (v1) or a registered dataset (LLVIP, M3FD), or use late fusion or unpaired domain adaptation on v2.
- [ ] For paired evaluation, use `rgb_to_thermal_vid_map.json` (test only) and drop pairs symmetrically.
- [ ] Compare RGB vs thermal per class only for person and car. Light and sign GT differ about 2.5–3× between modalities.

**Domain shift**
- [ ] Expect a val→test gap from location, camera and density differences. Report both.
- [ ] Use lighting tags from `index.json` (video level) for day/night breakdowns, labelled as imputed.
- [ ] Do not mix v1 and v2 (or train on FLIR-aligned and test on v2) without checking for frame overlap.

---

## Open uncertainties

1. **No systematic label-quality study of v2 exists** as far as I found: no relabelling, no missing-object audit, no noise-rate estimate. My error statistics cover structural problems only (degenerate, duplicate and tiny boxes), not semantic mistakes.
2. **The ThermalChameleon "corrected" JSON:** the merging of truck and other vehicle into car is verified from the file, but the authors' intent (a deliberate class merge, or a fix for perceived mislabels) is undocumented. Their train-side preprocessing is not public ("usage will be announced after review").
3. **The audit's findings** on duplicate RGB frames, the 39% image-split pairability and the RGB/thermal label ratios come from one community repository. I verified the test TIFF problem only indirectly (the ThermalChameleon author confirmed it independently).
4. **Physical Backdoor (13,460 images), NAGC (3,742 test frames, ~12,000 v1 ∪ v2 frames), Zha et al. (2,292 test) and ThermalChameleon (10,724 train):** I could not reconcile these split sizes with the archive.
5. **License text:** the full text is unavailable (404), so the redistribution clauses are unverified, and my reading is not legal advice.
6. **v1/v2 frame overlap is plausible** (the README calls v2 an expansion, and NAGC deduplicated), but I did not check it at frame level.
7. **IR-Reasoner's 5-class set** comes from the paper as summarised in 02. The public repo contains only COCO-default configs, so it cannot be confirmed from code.
8. GitHub issue threads were read through HTML pages (API rate limit), so I may have missed issues on repos I did not open individually.

---

## References

**Dataset and prior group reports**
- [R1] Official `README.txt` (v2.0.0). https://adas-dataset-v2.flirconservator.com/dataset/README.txt
- [R2] Official archive `coco.json` files and `rgb_to_thermal_vid_map.json`, analysed 2026-11-02 (see `01_dataset_overview.md`). https://adas-dataset-v2.flirconservator.com/
- `01_dataset_overview.md`, `02_usage_and_publications.md` (this folder).

**Code and community**
- [C1] GokulEdakkePuram/ThermalObjectDetection: audit and ablations. Docs [00-the-dataset](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/00-the-dataset.md), [02-radiometry-and-agc](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/02-radiometry-and-agc.md), [03-thermal-vs-visible](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/03-thermal-vs-visible.md), [04-reading-the-metrics](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/04-reading-the-metrics.md), [05-experiment-log](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/05-experiment-log.md); code [`convert.py`](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/src/thermaldet/convert.py)
- [C2] donkeymouse/ThermalChameleon: [`FLIR_test_raw_corrected.json`](https://github.com/donkeymouse/ThermalChameleon/blob/main/FLIR_test_raw_corrected.json), [`thermal_embedding.py`](https://github.com/donkeymouse/ThermalChameleon/blob/main/thermal_embedding.py), [issue #2](https://github.com/donkeymouse/ThermalChameleon/issues/2)
- [C3] weihui1308/HOTCOLDBlock: [`dataset/json2yolo.py`](https://github.com/weihui1308/HOTCOLDBlock/blob/main/dataset/json2yolo.py), [issue #8](https://github.com/weihui1308/HOTCOLDBlock/issues/8)
- [C4] mpolinowski/yolov8-nightshift. https://github.com/mpolinowski/yolov8-nightshift
- [C5] ax2mazhr/FLIR2YOLO. https://github.com/ax2mazhr/FLIR2YOLO · sarisemih/customize-flir-dataset. https://github.com/sarisemih/customize-flir-dataset
- [C6] xi666666/FLIR-ADAS-UDAOD. https://github.com/xi666666/FLIR-ADAS-UDAOD
- [C7] tlgksy/ir-reasoner. https://github.com/tlgksy/ir-reasoner · lengmo1996/ECDM. https://github.com/lengmo1996/ECDM · hosseinjavidnia/NAGC. https://github.com/hosseinjavidnia/NAGC · Danial-Qashqai/RGBT-ObjectDetection. https://github.com/Danial-Qashqai/RGBT-ObjectDetection

**Papers**
- [P1] Wei et al., HOTCOLD Block, AAAI 2023. https://arxiv.org/abs/2212.05709
- [P2] Strack, Waseda et al., Defending Against Physical Adversarial Patch Attacks on Infrared Human Detection, ICIP 2024. https://arxiv.org/abs/2309.15519
- [P3] Yin et al., Physical Backdoor, CVPR 2024. https://arxiv.org/abs/2404.19417
- [P4] Lee et al., Thermal Chameleon, IEEE RA-L 2024. https://arxiv.org/abs/2410.18340
- [P5] Zhu et al., ECDM, ACM MM 2024. https://arxiv.org/abs/2408.03748
- [P6] Uzun et al., CVPRW 2022 (https://arxiv.org/abs/2204.08745) and Sci. Rep. 2025 (https://arxiv.org/abs/2405.06383)
- [P7] Mukherjee et al., MODIPHY, ICIP 2024. https://arxiv.org/abs/2402.07894
- [P8] Zhang et al., MS-YOLO, IJCNN 2025. https://arxiv.org/abs/2509.21696
- [P9] Javidnia, Task-Driven Neural Adaptive Gain Control (NAGC), ICCV 2025 Workshops. https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html
- [P10] Zhang et al., SS-DC, arXiv 2025. https://arxiv.org/abs/2507.12017
- [P11] Brown et al., Multi-Modal Camera-Based Detection of Vulnerable Road Users, arXiv 2025. https://arxiv.org/abs/2509.06333
- [P12] Chen, Fang, Saniie, Two-Stage Dataset Curation via Class Pruning for Improved AI-Based Thermal Image Object Detection, IEEE eIT 2026. https://ecasp.ece.iit.edu/publications/2012-present/EIT2026_240.pdf
- [P13] Ranasinghe et al., Thermal-Det, CVPR 2026. https://arxiv.org/abs/2605.10130
- [P14] Godet, Jobert, Dalla Mura, XCalib, arXiv 2026. https://arxiv.org/abs/2609.21770
- [P15] Arbez et al., Descriptor: LYNRED Mobility Dataset Multimodal Detection Subset, arXiv 2026. https://arxiv.org/abs/2607.01871
- [P16] Cornelissen, Leroux, Simoens, Le MuMo JEPA, CVPR 2026 Workshop. https://arxiv.org/abs/2603.24327
- [P17] Zhang et al., The Paired Thermal Channel Is Two Orthogonal Axes, arXiv 2026. https://arxiv.org/abs/2610.33118
- [P18] Gödrich et al., Joint tone mapping and denoising of thermal infrared images via multi-scale Retinex and multi-task learning, SPIE 2023. https://arxiv.org/abs/2305.00691
