# FLIR ADAS v2: dataset overview (Q1 samples and classes, Q4 registration, Q8 download)

**Research date:** 2026-11-02
**Scope:** Teledyne FLIR "Free ADAS Thermal Dataset v2" (release v2.0.0, 19 Jan 2022). v1 (2018) is mentioned only to tell the two apart.

**How this was verified.** Besides web sources, I checked the official archive directly. The zip on the official CDN accepts HTTP range requests, so I read its central directory (the file list) and pulled out the small files: `README.txt`, `rgb_to_thermal_vid_map.json`, the six `coco.json` files and the six `*annotation_counts.txt` files. The full 11.9 GB was not downloaded. Every count marked **[verified from archive]** was computed from those files on 2026-11-02 [R1][R2].

---

## TL;DR

- **Size.** FLIR says 26,442 annotated frames: 9,711 thermal images, 9,233 RGB images and 7,498 video frames (3,749 thermal/RGB pairs) [R2][R3]. **The archive actually holds 30,787 frames**: 11,886 thermal images (10,742 train / 1,144 val), 11,403 RGB images (10,318 / 1,085) and 3,749 + 3,749 video test frames. 817 of those frames have no boxes. **[verified from archive]**
- **Annotations.** The format is COCO JSON (`coco.json`, filtered and recommended for training) plus FLIR's Conservator `index.json` (raw). There are 15 official classes, and thermal also has an undocumented `deer` class with 8 boxes. The thermal and RGB sides are annotated separately, with 254,053 thermal boxes and 270,869 RGB boxes. Many classes are rare or missing from val/test: `train`, `dog`, `deer`, `scooter`, `stroller`, `skateboard`, and `bus` in test.
- **Sensors.** Thermal comes from a Tau 2 at 640×512 (13 mm, 45°×37° FOV). RGB comes from a Blackfly S BFS-U3-51S5C with a 52.8° HFOV lens, at 5 different resolutions (720×480 to 2048×1536). The captures are in the US, England and France, by day and night.
- **Registration.** The official release is **not pixel-registered**. Only the video test split is time-synchronised, through `rgb_to_thermal_vid_map.json` (3,749 pairs). The train/val image splits have no pair map. **I found no published or community registered (aligned) version of v2.** The widely used "FLIR-aligned" set (Zhang et al., 5,142 pairs) is built from **v1**.
- **Download.** There is a free official download at `adas-dataset-v2.flirconservator.com`: `FLIR_ADAS_v2.zip`, 11,865,443,849 bytes (one file or 12 parts). It is governed by FLIR's non-commercial dataset license. Kaggle mirrors exist (about 11.9 GB). The Roboflow copies are partial, and some carry a "CC BY 4.0" label that FLIR's license does not support.

---

## v1 vs v2 (disambiguation only)

| | v1.0.0 (19 Jun 2018) | v2.0.0 (19 Jan 2022) |
|---|---|---|
| Annotated frames | 14,452 (10,228 images + 4,224 video) | 26,442 claimed (30,787 in archive) |
| Classes | 5 (person, bike, car, dog, other vehicle) | 15 (+ `deer` in thermal) |
| RGB annotations | RGB not annotated | RGB annotated independently |
| Locations | Santa Barbara, CA, USA | + England, France; video test from N. California, Idaho, Michigan |
| Folder names | `FLIR_ADAS_1_3/train/thermal_8_bit/...` | `images_thermal_train/`, `video_rgb_test/`, ... |

Source: release notes inside the official v2 `README.txt` [R1]. If a paper reports "3 classes", "5,142 pairs", or uses `FLIR_xxxxx.jpeg` file names, it is using **v1** or the v1-derived aligned subset.

---

## Q1: Samples and classes

### 1.1 Frame counts per split

Official claims come from the product page [R3] and `README.txt` [R1]. The archive counts are **[verified from archive]** [R2] and agree with an independent audit [R5].

| Split (folder) | Modality | Frames in archive | Frames with ≥1 box | Empty frames | Boxes (`coco.json`) | Source videos |
|---|---|---|---|---|---|---|
| `images_thermal_train` | thermal | 10,742 | 10,478 | 264 | 175,040 | 133 |
| `images_thermal_val` | thermal | 1,144 | 1,128 | 16 | 16,696 | 17 |
| `video_thermal_test` | thermal | 3,749 | 3,493 | 256 | 62,317 | 8 |
| `images_rgb_train` | RGB | 10,318 | 10,053 | 265 | 169,174 | 132 |
| `images_rgb_val` | RGB | 1,085 | 1,069 | 16 | 16,909 | 17 |
| `video_rgb_test` | RGB | 3,749 | 3,749 | 0 | 84,786 | 8 |
| **Total** | | **30,787** | 29,970 | 817 | **524,922** | |

Notes:
- The `README.txt` calls the third split "validation video" (`val_video`), but the folders are named `video_*_test`. FLIR's own YOLOX baseline is "tested on the video test data" [R1]. In practice everyone treats it as the **test** split.
- The README says the image splits were drawn from "completely independent video sequences" [R1]. The audit in [R5] found no video ID in more than one split.
- The train/val ratio is about 90/10, as stated in the README: 90.4/9.6 for thermal and 90.5/9.5 for RGB [R1][R5].
- `images_rgb_train/data/` holds 10,319 files but `coco.json` lists 10,318 images. One extra file is unexplained **[verified from archive]**. [R5] also reports 8 duplicate RGB frames (by MD5) in train, and one frame that appears in both RGB train and val (unverified by me).
- "Unannotated" data: v2 has no separate unannotated pool. Every shipped frame is listed in `coco.json`, and the only unannotated frames are the 817 background (empty) frames counted above.

**The sources disagree on the totals:**

| Quantity | Official page / README [R1][R3] | Archive [R2][R5] | Which I trust |
|---|---|---|---|
| Thermal train+val images | 9,711 | 11,886 | Archive |
| RGB train+val images | 9,233 | 11,403 | Archive |
| Video frames | 7,498 (3,749 pairs) | 7,498 (3,749 pairs) | Both agree |
| Total frames | 26,442 | 30,787 | Archive |
| Total boxes | "520,000" (375k image + 145k video) | 524,922 (377,819 + 147,103) | Archive. FLIR's figures are rounded |

I trust the archive counts because they come straight from the shipped `coco.json` files, and the per-class box totals in the archive match FLIR's own README tables exactly. The 9,711/9,233 figures are probably a stale pre-release number. Neither FLIR nor [R5] explains the gap of +2,175 thermal and +2,170 RGB frames. Some third-party papers repeat 9,711/9,233 or report filtered subsets (for example a Roboflow copy with 5,817 images [R9]), so check which count a paper uses.

### 1.2 Sensors, resolution, bit depth, frame rate, conditions

| Item | Value | Source |
|---|---|---|
| Thermal camera | Teledyne FLIR Tau 2, 13 mm f/1.0, HFOV 45°, VFOV 37°, T-linear mode | [R1][R3] |
| Thermal resolution | 640×512 in every split **[verified from archive]** | [R2] |
| Thermal formats | `data/`: 8-bit JPEG after AGC. `analyticsData/`: 16-bit TIFF holding 14-bit raw counts, no AGC | [R1][R3][R5] |
| Raw-data caveat | 1,598 test frames (2 of the 8 test videos) carry 8-bit greyscale TIFFs instead of raw data. Those two videos are tagged as a different camera (ADK, 50° HFOV), so the "single Tau 2" claim does not fully hold for the test split | [R5] (unverified by me) |
| RGB camera | Teledyne FLIR Blackfly S BFS-U3-51S5C (Sony IMX250), Edmund Optics 52.8° HFOV lens (part 68215) | [R1][R3] |
| RGB resolutions (train) | 1800×1600: 5,015 · 1224×1024: 1,992 · 2048×1536: 1,886 · 1280×1024: 1,378 · 720×480: 47 **[verified from archive]** | [R2] |
| RGB resolutions (val) | 1800×1600: 879 · 1224×1024: 183 · 1280×1024: 23 **[verified]** | [R2] |
| RGB resolutions (test) | 1224×1024: 2,151 · 1024×768: 1,598 (the two ADK videos) **[verified]** | [R2] |
| RGB bit depth | 8-bit JPEG. Faces and licence plates are lightly blurred | [R1] |
| Frame rate (video split) | **Disputed.** The product page says "24 Hz" [R3]. `README.txt` says "30 frames per second" twice [R1]. The files have frame indices but no timestamps [R5]. I lean to **30 fps** because the README ships with the data and repeats the figure, but this is unverified | [R1][R3][R5] |
| Image splits sampling | Frames picked manually or with a per-video frame skip. Redundant frames (e.g. stopped at a red light) were removed | [R1] |
| Locations | v1 data from Santa Barbara, CA. v2 adds England and France (image splits). The video test split was recorded in Northern California, Idaho and Michigan, USA | [R1][R4] |
| Conditions | Day, night and dawn/dusk. Scene types: city street, residential, highway, parking lot, tunnel, gas station. Weather: clear, partly cloudy, overcast, fog, rain, snow (in `extra_info`). Marketing examples include glare, fog, smoke and steam | [R2][R3][R6] |
| Lighting tags (coverage) | Thermal train: only 3,144 of 10,742 frames carry `hours` (night 2,110 / day 936 / dawn-dusk 98). RGB train: 10,266 of 10,318 tagged (day 6,191 / night 3,658 / dawn-dusk 417). Test: no per-frame tags **[verified from archive]**. [R5] imputes test tags from the video-level `index.json`: night 1,497, day 428, untagged 226 (on its 2,151-frame subset) | [R2][R5] |
| Rig / sync | One thermal and one visible camera mounted together on a vehicle. Capture was time-synced by FLIR's Guardian software | [R1] |

### 1.3 Annotation format

- **`coco.json`** (MS-COCO). This is the filtered version FLIR recommends; superfluous classes such as `face` and `license plate` are removed or remapped. It declares **80 categories** (a modified COCO label map) but uses only 16 **[verified]** [R1][R2].
- **`index.json`** (Conservator format). The raw, unfiltered version with frame-level and annotation-level attributes: occlusion, truncation, `reflection`, `is_rider`, per-video tags [R1].
- Images carry `extra_info` fields (`hours`, `scene`, `weather`, `video_id`) [R2]. Test annotations carry `track_id` for MOTA/MOTP-style tracking evaluation [R1][R5].
- Annotation rules: tight boxes, only the non-occluded part annotated, riders annotated separately from the bike or motorcycle [R1].
- **Category-ID caveats [verified from archive]:** `scooter` is ID **75** in every `coco.json`, while the README says 77 (in the COCO map, 77 is `scissors`). `deer` is ID 18. It appears only in thermal train (8 boxes) and is not among the 15 advertised classes. Match classes by name, not by README ID.

### 1.4 Instance counts per class per split

All values are **[verified from archive]**: they equal the shipped `*annotation_counts.txt` files and my own counts from `coco.json` [R2]. The train/val columns also match the README tables [R1]. The README does **not** list test-split per-class counts; the test columns below come from the archive.

| ID | Class | Thermal train | Thermal val | Thermal test (video) | RGB train | RGB val | RGB test (video) |
|---|---|---:|---:|---:|---:|---:|---:|
| 1 | person | 50,478 | 4,470 | 12,323 | 35,007 | 3,223 | 11,278 |
| 2 | bike | 7,237 | 170 | 113 | 7,560 | 193 | 446 |
| 3 | car | 73,623 | 7,133 | 30,517 | 71,281 | 7,285 | 30,893 |
| 4 | motor | 1,116 | 55 | 3,314 | 1,837 | 77 | 3,573 |
| 6 | bus | 2,245 | 179 | **0** | 1,879 | 183 | **0** |
| 7 | train | 5 | **0** | **0** | 9 | **0** | **0** |
| 8 | truck | 829 | 46 | 2,634 | 1,251 | 47 | 1,499 |
| 10 | light | 16,198 | 2,005 | 6,758 | 18,640 | 2,143 | 17,818 |
| 11 | hydrant | 1,095 | 94 | 277 | 990 | 126 | 402 |
| 12 | sign | 20,770 | 2,472 | 5,660 | 29,531 | 3,581 | 17,213 |
| 17 | dog | 4 | **0** | 25 | **0** | **0** | 17 |
| 18 | deer (undocumented) | 8 | **0** | **0** | **0** | **0** | **0** |
| 37 | skateboard | 29 | 3 | **0** | 412 | 4 | **0** |
| 73 | stroller | 15 | 6 | **0** | 38 | 7 | **0** |
| 75 (README: 77) | scooter | 15 | **0** | **0** | 41 | **0** | **0** |
| 79 | other vehicle | 1,373 | 63 | 696 | 698 | 40 | 1,647 |
| | **Total** | **175,040** | **16,696** | **62,317** | **169,174** | **16,909** | **84,786** |

**Rare or empty classes:**
- **Effectively unusable:** `train` (5 / 9 train boxes, none in val or test), `dog` (4 thermal train boxes and 0 RGB image boxes, yet 25 / 17 in test), `deer` (8 thermal train boxes only), `scooter` (15 / 41 train boxes, none in val or test), `stroller` (15 / 38 train, none in test), `skateboard` (29 thermal train, none in test).
- **Missing from test:** `bus` (2,245 thermal train boxes but 0 in test), `train`, `deer`, `skateboard`, `stroller`, `scooter`. The test split therefore covers only 10 of the 16 classes.
- **Thermal test `bike` caveat:** all 113 thermal test bikes are in the two ADK videos without raw data. A test set limited to raw-TIFF frames (2,151 frames) has **no thermal bikes** [R5] (unverified by me).
- The **dominant classes** are car, person, sign, light and bike. These are the subsets usually kept: 3 classes (person, bike/bicycle, car), 4 classes or 5 classes (+ light, sign) [R5][R7]. Group E covers the filtering conventions.
- FLIR baseline (YOLOX-m, COCO-pretrained, trained on train, tested on video test, AP@0.5): thermal person 75.33 / car 77.23; RGB person 51.42 / car 55.79 [R1].

---

## Q4: Registration between modalities

### 4.1 Is the official release registered?

**No.** The thermal and RGB images are **not pixel-aligned** in any split [R5][R8].

- **Different sensors and optics.** Thermal is 640×512 with a 45°×37° FOV. RGB comes from a separate Blackfly S camera with a 52.8° HFOV lens at 5 different resolutions and aspect ratios: 1800×1600, 2048×1536, 1224×1024, 1280×1024 and 720×480 [R1][R2]. The two cameras sit side by side on the vehicle, so there is parallax that depends on depth. FLIR publishes no intrinsic or extrinsic calibration [R1][R3].
- **Synchronisation.** Capture was time-synced by FLIR's Guardian software [R1]. The product page claims a "1:1 match between thermal and visible frames" for the **video** set [R3].
- **Pair map (video test only).** `rgb_to_thermal_vid_map.json` (464,876 bytes) is a JSON dict with **3,749 entries**, each mapping an RGB test file name to a thermal test file name with the same frame index. Example: `video-BzZspxAweF8AnKhWK-frame-000745-….jpg` → `video-4FRnNpmSmwktFJKjg-frame-000745-….jpg` **[verified from archive]** [R1][R2]. This is **temporal** pairing only, with no geometric warp.
- **Image splits (train/val) have no map.** Thermal (10,742) and RGB (10,318) train frame counts differ, and the RGB and thermal frames carry different video IDs. The MODIPHY authors state that the RGB and thermal images in FLIR v2 "are not directly correspondent" [R10]. The audit in [R5] recovered pairs for about 39% of thermal train frames (4,217), by matching capture names in `index.json` and frame indices (unverified by me).
- **RGB is annotated independently.** Each modality has its own `coco.json`, and the box counts differ (e.g. person: 50,478 thermal vs 35,007 RGB in train; sign: 20,770 vs 29,531). Even in the time-synced test pairs the counts differ (62,317 thermal vs 84,786 RGB boxes) **[verified]**. This differs from v1, where RGB had no annotations [R1].
- The misalignment is large enough that recent work rejects a single global homography because of depth-dependent parallax. XCalib rates the FLIR v2 focal difference as "med" (day) to "high" (night) [R8].

### 4.2 Registered or aligned versions of v2

**I found no published or community registered version of FLIR ADAS v2** as of 2026-11-02: no released dataset, no paper introducing one, and no download link. I searched arXiv, Google-indexed papers, GitHub, Kaggle and Hugging Face (`FLIR_aligned`, `flir_paired`, `flir_adas_video_pairs`, etc.).

What exists instead:

| Candidate | What it is | Why it does not qualify as a "registered v2" |
|---|---|---|
| **FLIR-aligned** (Zhang, Fromont, Lefèvre, Avignon, ICIP 2020; arXiv 2009.12664) [R11]. Mirrors on HF (`jsonhash/FLIR_aligned`, `bushwang/FLIR_aligned`, `UserNae3/FLIR_aligned`), Kaggle (`s111mple/flir-aligned`, `danialqashqai/flir-aligned-rgb-thermal-dataset`, 1.36 GB) [R12] | 5,142 manually filtered thermal–RGB pairs (4,129 train / 1,013 test), 3 classes (person, bicycle, car), thermal labels only | Built from **v1** and published in 2020, before v2 existed. This belongs to the v1 report. It is the "FLIR" in almost every RGB-T detection paper, e.g. ECCV 2022 *Multimodal Object Detection via Probabilistic Ensembling* (arXiv 2104.02904) and TGRS *DOD-SA* (arXiv 2508.10445) [R13][R14] |
| **HF `Maxim37/flir_adas_video_pairs`** [R15] | The 3,749 official video test pairs repackaged (RGB + thermal image + both COCO annotation sets), about 1.6 GB of parquet | Repackaging only. Uses the official temporal map with no geometric registration, and comes with no paper |
| **XCalib** (Godet, Jobert, Dalla Mura; arXiv 2609.21770, Sep 2026) [R8] | Depth-guided dense registration evaluated on two v2 sequences: "FLIR day" (672 pairs) and "FLIR night" (1,040 pairs) | Evaluation use only. No aligned data released (code "will be made publicly available"). Preprint, no peer-reviewed venue |
| **EPOFusion** (arXiv 2603.16130) [R16] | 277 FLIR-ADAS pairs registered with dense cross-modal registration and cropped to 640×512, inside a 447-pair real-world fusion test set (person/car/bicycle) | Tiny and mixed with other datasets. Source version (v1 or v2) not stated (unverified). No standalone release found |

**What researchers do instead** (the details belong to Group E):
1. Use the **v1-based FLIR-aligned** subset for RGB-T fusion detection, which is the dominant practice [R11][R13][R14].
2. Use v2 as a **thermal-only** (or RGB-only) detection dataset, often with 3 to 5 classes [R5][R7][R10].
3. Use the **v2 video test pairs** through `rgb_to_thermal_vid_map.json` as time-synced but unregistered pairs, with late or decision-level fusion, or with misalignment-robust methods [R5][R15].
4. Build small private registered subsets with homography or dense registration [R8][R16].
5. Mix unpaired RGB and thermal images (e.g. MODIPHY [R10]).

**Venue-paper requirement:** since no registered v2 version exists, there is nothing to prove existence for. For the v1-derived FLIR-aligned, ECCV 2022 [R13] and IEEE TGRS [R14] are solid examples (the v1 report should cover these).

---

## Q8: Ways to download

### 8.1 Official

| Item | Detail | Status (checked 2026-11-02) |
|---|---|---|
| Landing / request form | https://oem.flir.com/solutions/automotive/adas-dataset-form/. Requires signing in to a FLIR account; the form asks for name, email, company and country [R3] | 200 OK |
| Old landing URLs | `https://www.flir.com/oem/adas/adas-dataset-form/` and `https://www.flir.com/adasdataset` return **404** (FLIR moved its OEM pages to `oem.flir.com`). Regional copies (`flir.co.uk/oem/adas/dataset`, `flir.asia/...`) still appear in search results | Dead / moved |
| Data host (Conservator) | https://adas-dataset-v2.flirconservator.com/ ("Start Here: Download Guide") [R6] | 200 OK |
| Full archive | `https://adas-dataset-v2.flirconservator.com/dataset/full/FLIR_ADAS_v2.zip`: **11,865,443,849 bytes** (11.87 GB / 11.05 GiB; the page says "11.8 GB"), last modified 14 Jan 2022, MD5 `3150533d3a712c598aa110d5513a1671` (from `md5sums_full.txt`). The file is served from S3/CloudFront and the direct link responded without a login **[verified]** | 200 OK |
| Split archive | `.../dataset/parts/FLIR_ADAS_v2.zip.00` … `.10` (1.0 GB each) + `.11` (54 MB), plus `md5sums_parts.txt`. The page's "≈11.05 GB" total for the parts is the same size in GiB as the full zip [R6] | 200 OK |
| Loose files | `.../dataset/README.txt`, `.../dataset/rgb_to_thermal_vid_map.json` (464,876 B) [R6] | 200 OK |
| Uncompressed layout | `images_{thermal,rgb}_{train,val}/`, `video_{thermal,rgb}_test/`. Each holds `data/` (JPEG), `analyticsData/` (thermal TIFF only), `coco.json`, `index.json`, `*annotation_counts.{txt,tsv}`. Approximate sizes: thermal train TIFF 3.3 GB + JPEG 1.06 GB, RGB train 4.17 GB, RGB test 1.25 GB, thermal test TIFF 0.99 GB **[verified from archive]** | |
| Catalogue entry | TIB LDM record "Teledyne FLIR Free ADAS Thermal Datasets v2", DOI 10.57702/3yajgd26 [R17] | Metadata only |
| Support | ADAS-Support@flir.com [R1] | |
| v1 (for reference) | `https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d` (listed in the v2 README) | Returned HTTP 400 when checked, likely dead |

### 8.2 Mirrors (unofficial; check the license before use)

| Mirror | Content | Size | License label |
|---|---|---|---|
| Kaggle `samdazel/teledyne-flir-adas-thermal-dataset-v2` | Full `FLIR_ADAS_v2/` tree, incl. `coco.json` and count files (about 12.4k downloads) | 11.88 GB | "Other (specified in description)" |
| Kaggle `rajababuadigarla/teledyne-flir-free-adas-thermal-dataset-v2` | Full tree (no top folder) | 11.88 GB | Unknown |
| Kaggle `kushagraborse/thermal-images-flir-adas-dataset` | Same size as above, apparently full v2 (unverified) | 11.88 GB | Unknown |
| Kaggle `dino1902/flir-adas-v2-3k` | Subset | 2.8 GB | Unknown |
| HF `Maxim37/flir_adas_video_pairs` | 3,749 video test pairs with both annotation sets (parquet) [R15] | 1.6 GB | none stated |
| HF `Q-Qyt/flir_adas` | Empty repository (only `.gitattributes`) | none | none |
| Roboflow `mateusz-dorobek/flir_adas_v2_thermal` | Thermal only, 5,817 images, train split only [R9] | n/a | "CC BY 4.0" (conflicts with FLIR's license) |
| Roboflow `disitlab/flir_adas_2` and others | Partial re-exports (unverified contents) | n/a | various |
| Academic Torrents | None found | none | none |
| Papers with Code | `paperswithcode.com/dataset/flir` now redirects to Hugging Face (Papers with Code was shut down in 2025) | Dead | none |

Kaggle sizes and licenses come from the Kaggle public API (`/api/v1/datasets/list`), queried 2026-11-02. Hugging Face entries come from the HF API.

**Registered versions:** none exist for v2 (see Q4). The v1-based FLIR-aligned downloads (Google Drive `1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y`, HF `UserNae3/FLIR_aligned`) are listed in [R12] for completeness only.

### 8.3 License and terms of use

- `README.txt` states: "By downloading this data you agree to the following terms of service: https://www.flir.com/oem/adas/adas-dataset-agree/" [R1]. **That URL now returns 404**, and so does its `oem.flir.com` equivalent. A regional copy is indexed at `flir.ca/oem/adas/adas-dataset-agree` [R18].
- According to the indexed text of that page [R18] (I could not open the full text; treat the details as **partially unverified**):
  - It is the "Teledyne FLIR **Non-Commercial** Image Dataset License Agreement", last modified 19 Jan 2022.
  - The license is limited, revocable, non-exclusive, non-sublicensable and non-transferable.
  - Permitted use: non-commercial neural-network development for automotive and autonomous-vehicle applications, and non-commercial education and research.
  - The data is provided "as is".
  - The dataset is described as proprietary, not open source. FLIR's 2018 press material called v1 "open-source".
- Implications: commercial use is not permitted without a separate agreement. Re-hosting is not authorised by the license, so the Kaggle and Roboflow mirrors, and especially the "CC BY 4.0" labels, are **not** sanctioned by FLIR (my reading; unverified legal interpretation). Use the official host when possible.
- Privacy: faces and licence plates in the RGB images are lightly blurred [R1].

---

## Open uncertainties

1. Why FLIR states 9,711 / 9,233 / 26,442 when the archive holds 11,886 / 11,403 / 30,787 frames. Not explained anywhere.
2. Video frame rate: 24 Hz (product page) or 30 fps (README). No timestamps in the files.
3. The full license text could not be retrieved (the official URL is 404). The redistribution clauses are unverified.
4. Per-video raw-TIFF and camera (ADK vs Tau 2) issues in the test split come from one community audit [R5]. I did not re-check the TIFF contents.
5. A registered v2 subset could exist in a paper I did not find, especially in non-English venues. Treat "none exist" as "none found after a targeted search".

---

## References

- [R1] Official `README.txt` for FLIR ADAS v2 (release notes, labels, counts, baseline, terms link). https://adas-dataset-v2.flirconservator.com/dataset/README.txt
- [R2] Official archive `FLIR_ADAS_v2.zip` (central directory, `coco.json`, `*annotation_counts.txt`) and `rgb_to_thermal_vid_map.json`, inspected 2026-11-02. https://adas-dataset-v2.flirconservator.com/dataset/full/FLIR_ADAS_v2.zip · https://adas-dataset-v2.flirconservator.com/dataset/rgb_to_thermal_vid_map.json · https://adas-dataset-v2.flirconservator.com/dataset/full/md5sums_full.txt
- [R3] Teledyne FLIR, "FREE Teledyne FLIR Thermal Dataset for Algorithm Training" (product and request-form page). https://oem.flir.com/solutions/automotive/adas-dataset-form/
- [R4] Teledyne FLIR press release, "Teledyne FLIR Releases Free Expanded Starter Thermal Dataset for ADAS and Autonomous Vehicle Testing", 19 Jan 2022. https://oem.flir.com/about/news/expanded-teledyne-flir-starter-thermal-dataset-for-adas-and-autonomous-vehicle-testing/
- [R5] GokulEdakkePuram/ThermalObjectDetection, "00-the-dataset.md" (independent audit of v2 files). https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/00-the-dataset.md
- [R6] FLIR Conservator v2 download page. https://adas-dataset-v2.flirconservator.com/
- [R7] mpolinowski/yolov8-nightshift (reproduces the README count tables; per-class YOLOv8 results). https://github.com/mpolinowski/yolov8-nightshift
- [R8] Godet, Jobert, Dalla Mura, "XCalib: Depth-Guided Geometric Optimization for Dense Thermal–Visible Video Registration", arXiv 2609.21770. https://arxiv.org/abs/2609.21770
- [R9] Roboflow Universe, FLIR_ADAS_v2_THERMAL (mateusz-dorobek). https://universe.roboflow.com/mateusz-dorobek/flir_adas_v2_thermal
- [R10] "MODIPHY: Multimodal Obscured Detection for IoT using PHantom Convolution-Enabled Faster YOLO", arXiv 2402.07894. https://arxiv.org/abs/2402.07894
- [R11] Zhang, Fromont, Lefèvre, Avignon, "Multispectral Fusion for Object Detection with Cyclic Fuse-and-Refine Blocks", ICIP 2020, arXiv 2009.12664 (origin of v1-based FLIR-aligned). https://arxiv.org/abs/2009.12664
- [R12] HF mirrors of FLIR-aligned (v1-based): https://huggingface.co/datasets/bushwang/FLIR_aligned · https://huggingface.co/datasets/jsonhash/FLIR_aligned
- [R13] Chen et al., "Multimodal Object Detection via Probabilistic Ensembling", ECCV 2022, arXiv 2104.02904. https://arxiv.org/abs/2104.02904
- [R14] "DOD-SA: Infrared-Visible Decoupled Object Detection with Single-Modality Annotations", IEEE TGRS, arXiv 2508.10445. https://arxiv.org/abs/2508.10445
- [R15] Hugging Face, Maxim37/flir_adas_video_pairs. https://huggingface.co/datasets/Maxim37/flir_adas_video_pairs
- [R16] Wang et al., "EPOFusion: Exposure-aware Progressive Optimization Method for Infrared and Visible Image Fusion", arXiv 2603.16130. https://arxiv.org/abs/2603.16130
- [R17] TIB LDM catalogue, "Teledyne FLIR Free ADAS Thermal Datasets v2" (DOI 10.57702/3yajgd26). https://service.tib.eu/ldmservice/dataset/bibtex/teledyne-flir-free-adas-thermal-datasets-v2
- [R18] Teledyne FLIR, ADAS dataset agreement (regional copy; full text not retrieved). https://flir.ca/oem/adas/adas-dataset-agree
- Kaggle mirrors: https://www.kaggle.com/datasets/samdazel/teledyne-flir-adas-thermal-dataset-v2 · https://www.kaggle.com/datasets/rajababuadigarla/teledyne-flir-free-adas-thermal-dataset-v2 · https://www.kaggle.com/datasets/kushagraborse/thermal-images-flir-adas-dataset · https://www.kaggle.com/datasets/dino1902/flir-adas-v2-3k
