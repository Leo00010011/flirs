# FLIR ADAS v1: dataset overview (Q1 samples and classes, Q4 registration, Q8 downloads)

**Research date:** 2026-11-02
**Scope:** The original Teledyne FLIR "FREE ADAS" thermal starter dataset, version 1 (first released July 2018; the last v1 release is **v1.3, 16 Aug 2019**, distributed as folder `FLIR_ADAS_1_3`). v2 (Jan 2022) is mentioned only to tell the two apart.

**How this was verified:**

- **Kaggle mirror (first pass):** I downloaded the v1.3 `ReadMe.txt`, the README PDF, the license PDF and the three `thermal_annotations.json` files from the public Kaggle mirror [K1]. I counted every split and class myself (results are in the tables below). I listed all 57,178 files in that mirror, read the header of one 16-bit TIFF, and measured a sample of 20 RGB frames.
- **Aligned FLIR:** I downloaded `aligned.zip` (1.36 GB) [A3] and counted its pairs, splits and classes.
- **Official download (second pass, added 2026-11-02):** The user downloaded v1 from the official Teledyne FLIR Box link in a browser; FLIR's site blocks scripted clients. The copy is stored locally as `FLIR Santa Barbara Regional Thermal Dataset/` [D1]. I read it without extracting anything:
  - checked both sets of MD5 checksums (all OK);
  - listed every zip entry;
  - hashed the docs and the three JSONs;
  - read TIFF and JPEG metadata.
- **Result:** The official files and the Kaggle mirror are **byte-identical** for all annotation files and docs, and have identical names and sizes for every image (§1.6).
- **Labels used in this file:**
  - **(counted)**: numbers I computed from the files.
  - **(verified on official download)**: facts confirmed against the official copy.

---

## TL;DR

- **Size:** 14,452 annotated thermal frames. 10,228 are sampled frames, split into **train 8,862** and **val 1,366**. The other **4,224** come from one continuous 144 s "video" sequence, which the community often uses as a test set. RGB frames are included but **not annotated**, and some are missing: 8,363 / 1,257 / 4,195 RGB frames for train / val / video (499 / 109 / 29 missing). [R1][K1] (counted)
- **Sensors:** Thermal is a FLIR Tau2, 640×512, LWIR, 13 mm f/1.0, 45°×37° FOV. It is stored as 14-bit data in 16-bit TIFF, plus 8-bit AGC JPEG. RGB is a FLIR BlackFly. Most RGB frames are 1800×1600, but other sizes exist. Recorded at 30 Hz and sampled at 2 fps (1 fps in sparse scenes). Santa Barbara, CA, November to May, about 60% day and 40% night. [R1][O1]
- **Labels:** COCO-style JSON for thermal only. The label map has 81 entries, but only 4 classes have boxes: **person, bicycle, car, dog**. "Other Vehicle" is advertised on the official page (2,228 + 540 boxes) but has **zero annotations and no category entry in the v1.3 JSON**. Dog has only 240 boxes in train+val and 0 in video. Most papers use 3 classes: person, car, bicycle. (counted)
- **Registration:** The official release is **not registered**. The two cameras sit about 1.9 in apart, with different resolutions, lenses and viewing geometry, and no calibration is provided. The README states that the thermal boxes do not apply to RGB. Community versions:
  1. **FLIR-aligned** (Zhang et al., ICIP 2020). 5,142 pairs, made by manual filtering: 4,129 train / 1,013 test. RGB is supplied at 640×512. This is the de facto multispectral benchmark. Venue proof: GAFF (WACV 2021), ICAFusion (Pattern Recognition 2024).
  2. **RoadScene** (Xu et al., AAAI 2020 / TPAMI 2020). 221 registered pairs for image fusion. Venue proof: TarDAL (CVPR 2022).
  3. Ad-hoc, unreleased homographies, for example GEM.
- **Download:** **v1 is still officially downloadable.**
  1. Register through the free form on the Teledyne FLIR ADAS dataset page. The page describes v2 and requires a FLIR account login.
  2. This leads to the v2 download guide (`adas-dataset-v2.flirconservator.com`). Its "Release Notes → ADAS Dataset v1.0.0 (Jun 19, 2018)" entry links the **original v1 download on FLIR's Box: https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d** [O4].
  3. The Box share contains v1.3, repackaged in December 2020:
     - `FLIR_ADAS_1_3_full/`: `_train.zip` 9.05 GB, `_val.zip` 1.58 GB, `_video.zip` 5.75 GB, `_docs.zip` 0.2 MB, plus `md5sums.txt`;
     - `FLIR_ADAS_1_3_parts/`: `FLIR_ADAS_1_3.zip.00`–`.15`, 16.35 GB in 1 GiB parts;
     - a `START-HERE-data-download-guide.pdf`.

  The user saved the Box folder locally under the name "FLIR Santa Barbara Regional Thermal Dataset". That name is not printed inside any file, so it is presumably the Box folder title (unverified by me; Box blocks scripted access). The license is the FLIR ADAS Image License Agreement (26 Jul 2018): non-commercial research and education, **no redistribution**, so mirrors such as Kaggle `deepnewbie` (byte-identical to the official multipart zip) are unofficial.

---

## Q1. Samples and classes

### 1.1 Image and frame counts

| Split (folder) | Thermal frames (8-bit JPEG = 16-bit TIFF) | Thermal frames with ≥1 box | RGB frames present | RGB missing (README) | Boxes |
|---|---|---|---|---|---|
| `train` (IDs 1–8,862) | 8,862 | 7,860 | 8,363 | 499 | 67,844 |
| `val` (IDs 8,863–10,228) | 1,366 | 1,360 | 1,257 | 109 | 11,696 |
| `video` (IDs 1–4,224, one 144 s sequence) | 4,224 | 4,166 | 4,195 | 29 | 37,183 |
| **Total** | **14,452** | 13,386 | 13,815 | 637 | 116,723 |

Sources: counted from the v1.3 JSONs and the file listing [K1]. **Verified on the official download [D1]:** the JSONs are byte-identical and the file lists match (§1.6). The README gives the split sizes and the number of missing RGB frames [R1], and these agree exactly with the files: 8,862 − 499 = 8,363, and so on.

- **Annotated vs unannotated:** Every thermal frame is listed in the JSON, so the set is "fully annotated". Frames without boxes are background frames: 1,002 in train, 6 in val, 58 in video. All RGB frames are unannotated [R1].
- **Conflicting counts of frames with boxes:**
  - Official web page [O1]: "10,228 total frames and **9,214** frames with bounding boxes" and "4,224 total frames and **4,183** frames with bounding boxes".
  - v1.3 JSON (counted): **9,220** (7,860 + 1,360) and **4,166**.
  - I trust the JSON values, because they come from the files actually shipped, and the official download confirms them (verified on official download, §1.6). The web-page figures most likely describe v1.0.0 (§1.5).
- **Test set:** v1 has no official test split. The README offers "train" and "val" as a *suggested* split and says "Test data was not used" for the baseline [R1]. Most papers report on `val` (1,366 frames, sometimes reduced to 1,258 after dropping the 108 val frames without RGB [P1]). Some papers use `video` as an extra test set.
- **Day/night:** The official figure is about 60% day and 40% night [O1]. The JSON has no per-image day/night tag (`extra_info` is empty for images; counted). ProbEn hand-tagged the 1,258 RGB-paired val frames as 768 day and 490 night [P1].

### 1.2 Sensors, resolution, bit depth, frame rate, conditions

| Item | Value | Source |
|---|---|---|
| Thermal camera | FLIR **Tau2**, 640×512, 13 mm f/1.0, HFOV 45°, VFOV 37° (uncooled LWIR) | [O1][R1] |
| Thermal formats | `thermal_16_bit/*.tiff`: raw 14-bit, no AGC, stored as **16-bit TIFF**. Verified header: 640×512, bps=16, LZW. `thermal_8_bit/*.jpeg`: 8-bit with AGC applied. `Annotated_thermal_8_bit/*.jpeg`: 8-bit with boxes drawn, for viewing only. | [R1][O1], verified on file |
| RGB camera | FLIR **BlackFly BFS-U3-51S5C-C**, Computar 4–8 mm f/1.4, "FOV set to match Tau2" | [O1] |
| RGB resolution | Sources disagree. The official page says 1280×1024. The README text says "1280 X 512m" (a typo) and also "commonly 1600 X 1800 … including 480 X 720, 1536 X 2048, and 1024 X 1280". **The files show W×H = 1800×1600 for most frames** (14 of a 20-frame sample), plus 2048×1536, 1280×1024 and 720×480. Trust the files and the README's "commonly 1600×1800" (H×W). | [O1][R1], verified on sample |
| Camera spacing | Single enclosure, centerlines 1.9 ± 0.1 in (about 2 in) apart, "collimated to minimize parallax" | [R1][O1] |
| Frame rate | Recorded at 30 Hz. Sampled at 2 frames/s, or 1 frame/s in less object-rich scenes. The video sequence was annotated at 30 fps (4,224 frames, about 140–144 s). | [R1][O1] |
| Capture | Captured over USB3 with FLIR proprietary software. Both cameras ran in default mode. | [R1] |
| Location / period | Streets and highways around Santa Barbara, California, USA, from November to May | [R1][O1] |
| Conditions | Day (60%) and night (40%). "Clear to overcast", "generally clear-sky". No rain, fog or snow subsets. | [O1][R1] |
| Privacy | License plates are blurred in all images. Faces are also blurred in RGB. | [R1] |
| Baseline | RefineDet512 pretrained on COCO and fine-tuned on train 8-bit thermal. val mAP@0.5 = 0.587 over all classes. Per class AP: person 0.794, bicycle 0.580, car 0.856. | [R1][O1] |

### 1.3 Annotation format

- **Format:** One MSCOCO-style `thermal_annotations.json` per split [R1].
  - Keys: `info`, `licenses` (empty), `categories`, `images`, `annotations`.
  - Each image has `file_name` = `thermal_8_bit/FLIR_xxxxx.jpeg` (video: `FLIR_video_xxxxx.jpeg`), width 640, height 512, and an empty `extra_info`.
  - Each annotation has `bbox` [x, y, w, h], `area`, a 4-point `segmentation` that equals the box, `iscrowd` = 0 throughout, and `extra_info: {"human_annotated": true}`. (counted)
- **Label map vs real classes:** `categories` has **81 entries**. IDs 1–70 follow a COCO-like contiguous list where 17 = dog. IDs 71–80 are custom (bike, hydrant, motor, rider, light, sign, motor vehicle, human face, hair drier, license plate), and 0 = empty. **Only IDs 1 (person), 2 (bicycle), 3 (car) and 17 (dog) have boxes.** (counted)
- **README inconsistency:**
  - `ReadMe.txt` still lists "Category 18: Dogs" and "Category 91: Other Vehicle".
  - The v1.3 **PDF** README lists only 1/2/3/17 [R1][R2].
  - The JSON follows the PDF. Real MSCOCO numbering (dog = 18) does **not** apply.
- **Class definitions** [R1]:
  - "Bicycles" includes **motorcycles**, which is not consistent with COCO.
  - "Cars" covers personal vehicles and some small commercial vehicles.
  - "Other Vehicle" was defined as large trucks, boats and trailers, but it is absent from the v1.3 files.
  - Boxes are tight and cover only the visible (non-occluded) parts. Riders are boxed separately from bicycles.

### 1.4 Instances per class per split (v1.3 JSON, counted)

| Class (JSON id) | train | val | train+val | video | Total |
|---|---|---|---|---|---|
| person (1) | 22,372 | 5,779 | **28,151** | 21,965 | 50,116 |
| bicycle (2) | 3,986 | 471 | **4,457** | 1,205 | 5,662 |
| car (3) | 41,260 | 5,432 | **46,692** | 14,013 | 60,705 |
| dog (17) | 226 | 14 | **240** | 0 | 240 |
| other vehicle | 0 | 0 | 0 (official page: 2,228) | 0 (official page: 540) | 0 |
| **all** | 67,844 | 11,696 | 79,540 | 37,183 | 116,723 |

COCO size buckets by box area, counted from the files:

| Split | Small (<32²) | Medium | Large (≥96²) |
|---|---|---|---|
| train | 39,534 | 24,229 | 4,081 |
| val | 5,349 | 5,287 | 1,060 |
| video | 28,748 | 7,223 | 1,212 |

Small objects dominate.

**Agreement with other sources:**

- The official train+val totals [O1] match my counts exactly for person, car, bicycle and dog. The same holds for the video totals of person (21,965), car (14,013), bicycle (1,205) and dog (0).
- **The only disagreement is "Other Vehicle"** (2,228 / 540 on the web page vs 0 in the files). The official download confirms there are 0 Other Vehicle boxes in v1.3 (verified on official download). The web counts most likely describe v1.0.0, and the class was dropped in a later v1.x release (inference; see §1.5).
- Some papers mix up these numbers. For example, [P2] reports "person 22,372, car 14,013, bicycle 1,205" for train+val, which mixes train-person with video-car and video-bicycle. Trust the per-split table above.

**Classes with little or no data:**

- **Dog**: 240 boxes in train+val (0.3% of boxes) and none in video. Most works drop it, for example GEM notes "0.29% of all the annotations" [P3]. Zhang et al. also drop it for FLIR-aligned [A1].
- **Other Vehicle**: no data at all in v1.3.
- **The 76 other label-map entries** (motorcycle, bus, truck, traffic light, …) never occur. Motorcycles are folded into "bicycle".

### 1.5 Version history and v2 disambiguation

- **v1 release:**
  - FLIR's own v2 release notes date **"ADAS Dataset v1.0.0" to 19 Jun 2018**: 5 label categories, 14,452 frames (10,228 images + 4,224 video frames), Santa Barbara [O4].
  - The 2018 web page said "available July 2018" [O2][N1], and the license is dated 26 Jul 2018 [L1].
  - Most likely v1.0.0 was versioned in June and published around July 2018 (the exact public date is unverified).
- **v1.3:** The last v1 release is **v1.3 (16 Aug 2019)** [R1]. It is the only v1 version FLIR still distributes: the Box share holds only `FLIR_ADAS_1_3` (verified on official download).
- **Timestamps inside the official zips:**
  - README PDF and license PDF: 2019-08-16.
  - `ReadMe.txt`: modified 2020-09-02, yet it still has the stale "Category 18 Dogs / Category 91 Other Vehicle" text.
  - Archives: built 2020-12-22, md5sums dated 2020-12-23.

  The Dec 2020 Box package is therefore a repackaging of v1.3, not a new annotation version.
- **Intermediate versions:** FLIR's release notes list only v1.0.0 and v2.0.0 [O4]. **v1.1 and v1.2 are not documented anywhere official that I found.** The contents of v1.0.0 (for example whether it had "Other Vehicle" boxes, as the 2019–2021 web page counts imply) cannot be checked, because v1.0.0 files are not distributed. "5 label categories" in the v1.0.0 note matches the web page's 5-class list (person, car, bicycle, dog, other vehicle), which supports the view that **v1.0.0 had Other Vehicle and a later v1.x dropped it** (inference, unverified).
- **v2 (19 Jan 2022)** is a different dataset [O3][N2]:
  - 26,442 frames with about 520k boxes over 15+ categories, from the USA, England and France.
  - **RGB is annotated too.**
  - A Blackfly S with a 52.8° HFOV lens.
  - Splits `train` / `val` / `video_test` with an `rgb_to_thermal_vid_map.json` file.
  - Results on v1 and v2 are **not comparable**. If a paper says only "FLIR ADAS", check the image counts (8,862/1,366 means v1) and the class list.

### 1.6 Official download vs Kaggle mirror (verified on official download, 2026-11-02)

Official copy: `/home/leonardo.ulloa/projects/data/FLIR Santa Barbara Regional Thermal Dataset/` [D1]. I read it in place and modified nothing. MD5 checks: `md5sum -c` passes for all 4 `_full` zips and all 16 `_parts` files.

| Aspect | Official (Box) | Kaggle `deepnewbie` [K1] | Same? |
|---|---|---|---|
| Version | v1.3 (README PDF "Version 1.3 August 16, 2019") | v1.3 | yes |
| `README - FLIR ADAS Dataset V1_3.pdf` | md5 `c1857406…` (also copied at the top level of the share) | md5 `c1857406…` | **identical** |
| `ReadMe.txt` | md5 `bba9b4d7…` (stale 18/91 class ids) | same md5 | **identical** |
| License PDF | md5 `905c6efb…` | same md5 | **identical** |
| `train/val/video thermal_annotations.json` | md5 `49b07b84…` / `d04967ed…` / `6bbc69e0…` | same three md5s | **identical**, so all counts in §1.1 and §1.4 are official |
| Image files (all 57,174 non-doc files) | names and byte sizes | names and byte sizes | **identical** (diff of name+size lists is empty) |
| Packaging | `_full`: 4 separate zips per split plus docs, **without** the `FLIR_ADAS_1_3/` prefix. `_parts`: one 16.35 GB zip split into 16 parts, **with** the `FLIR_ADAS_1_3/` prefix. | One 16.38 GB zip whose contents match the `_parts` archive, including its stray `FLIR_ADAS_1_3/.ReadMe.txt.swp` vim swap file. The Kaggle copy was evidently made from the official multipart zip. | layout identical to `_parts` |
| Junk files | macOS `.DS_Store` / `._*` files in each split, plus `.ReadMe.txt.swp` in `_parts` | `.ReadMe.txt.swp` only | trivial |
| Video RGB folder name | **`video/rgb`** (lower case) in both official archives; `train/RGB` and `val/RGB` are upper case | Kaggle's file API lists `video/RGB` (unverified whether a fresh Kaggle download keeps the case) | Watch out on case-sensitive filesystems |
| Uncompressed size | 16,736,886,037 B (`_full`, including junk) | 16,736,872,849 B | about the same |
| Label map | 81 categories; boxes only for 1/2/3/17 | same file | identical |
| "Other Vehicle" boxes | **none** (no id 91, no "other vehicle" category) | none | identical |
| Calibration / timing | No calibration files, extrinsics or timestamps. Thermal TIFFs carry only basic TIFF tags (size, bps, compression, strips; no DateTime). Sampled RGB JPEGs are plain JFIF with no EXIF. | same | n/a |

**Resolved uncertainties:**

- The per-split and per-class counts in §1.4 are the **official v1.3 counts**: the Kaggle JSONs are byte-identical to FLIR's.
- The web page's "Other Vehicle 2,228/540" and "9,214/4,183 frames with boxes" **do not describe the files FLIR distributes today**. They most plausibly describe v1.0.0 (see §1.5). For v1.3 use 9,220 (train+val) and 4,166 (video) frames with boxes, and no Other Vehicle.

---

## Q4. Registration between modalities

### 4.1 Is the official release registered? **No.**

- **Sensor geometry:**
  - Two separate cameras sit in one enclosure, 1.9 ± 0.1 in apart, "collimated to minimize parallax" [R1][O1].
  - Resolution, aspect ratio and optics differ: thermal is 640×512 (5:4) with a 45°×37° FOV. RGB is mostly 1800×1600 (9:8) through a 4–8 mm zoom lens whose "FOV was set to match Tau2" by hand.
  - **No intrinsics, extrinsics or homography are provided** [P3].
- **Official statement** [R1]: "The thermal and RGB camera did not have identical placement on the vehicle and therefore had different viewing geometries, so the thermal annotations do not represent the placement of objects in the RGB image."
- **Annotations:** Only thermal frames are annotated, and the RGB frames are "for reference" [R1][O1].
- **Synchronization:**
  - The page claims "synced" imagery [O1], but the two streams were captured over USB3 by proprietary software, and no hardware trigger is documented [R1].
  - ProbEn describes the pairs as unaligned "both spatially and temporally" [P1].
  - Zhang et al. show visibly misaligned pairs (Fig. 3 in [A1]).
- **Missing RGB frames:**
  - 499 in train, 109 in val and 29 in video have no RGB counterpart (README; my file counts agree).
  - Fusion papers usually drop these frames. A common example is removing the 108–109 val frames, which leaves 1,258 val pairs [P1].
- **Varying RGB size:** RGB resolution changes between frames (see 1.2), so any single fixed homography is fragile.
- **Official download adds nothing for registration** (verified on official download, §1.6):
  - There are no calibration files, intrinsics/extrinsics, homographies or per-frame timestamps.
  - The RGB JPEGs have no EXIF, and the thermal TIFFs have no DateTime tag.
  - The only link between modalities is the shared frame ID in the file name (`FLIR_xxxxx.jpeg` ↔ `RGB/FLIR_xxxxx.jpg`; video `FLIR_video_xxxxx`, with the RGB folder named `video/rgb`).
  - Temporal sync therefore cannot be checked from the release.
  - v2 later added an explicit `rgb_to_thermal_vid_map.json` [O4]; v1 has no equivalent.

### 4.2 Community registered or aligned versions

| Version | Who / paper introducing it | Method | Pairs | Classes kept | Splits | Download |
|---|---|---|---|---|---|---|
| **FLIR-aligned** ("aligned FLIR", "FLIR_aligned") | Heng Zhang, Elisa Fromont, Sébastien Lefèvre, Bruno Avignon (Univ. Rennes/IRISA and ATERMES). "Multispectral Fusion for Object Detection with Cyclic Fuse-and-Refine Blocks", **IEEE ICIP 2020**, arXiv:2009.12664 [A1] | **Manual filtering.** "We manually removed the misaligned visible-thermal image pairs" [A1]. RGB is supplied at 640×512, the same as thermal (verified). The exact crop/resize applied to the raw 1800×1600 RGB is not documented (unverified). No per-pair warping is described. | **5,142** (verified) | person, car, bicycle per the paper. **Note:** the shipped VOC XMLs still contain *dog* boxes (95 in train, 13 in test; counted), and users are expected to ignore them. | **4,129 train / 1,013 test** (`align_train.txt`, `align_validation.txt`). Train pairs come only from original `train` IDs and test pairs only from original `val` IDs (counted). | Google Drive (original, from the paper's short link `shorturl.at/ahAY4`): https://drive.google.com/file/d/1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y/view, alive on 2026-11-02 as `aligned.zip` [A2]. HF backup and COCO conversion: https://huggingface.co/datasets/jsonhash/FLIR_aligned (`aligned.zip` 1.36 GB, `flir_align.7z` 0.92 GB COCO-format) [A3]. Baidu (from the CFT repo): https://pan.baidu.com/s/1z2GHVD3WVlGsVzBR1ajSrQ code `qwer` [A4]. Kaggle: `s111mple/flir-aligned` (1.36 GB) [K2]. |
| **RoadScene** | Han Xu, Jiayi Ma et al. "FusionDN", **AAAI 2020**, and "U2Fusion", **IEEE TPAMI 2020** [B1] | Selected from "the FLIR video" (the repo links the FLIR ADAS page). Thermal background noise was reduced, then pairs were "accurately aligned" and cropped to the exact registered region [B1]. The registration method (manual control points or homography) is not stated (unverified). Because the 2019–2020 timeline predates v2, the source must be v1 (inference). | **221** | none (image fusion, no boxes) | none (used as a fusion test set) | https://github.com/hanna-xu/RoadScene [B1] |
| **Ad-hoc homography versions (not released)** | For example GEM (Mazhar, Babuška, Kober, TU Delft; arXiv:2102.12319) [P3] | One global homography from manually picked RGB↔thermal correspondences. RGB is warped to approximately align with thermal. Dog is dropped. | Full 8,862 / 1,366 | person, bicycle, car | official train/val | Not released |

Notes:

- **FLIR-aligned contents (verified):**
  - Folder `align/` with `JPEGImages/FLIR_xxxxx_PreviewData.jpeg` (thermal 8-bit) and `FLIR_xxxxx_RGB.jpg`, both 640×512.
  - `Annotations/FLIR_xxxxx_PreviewData.xml` (Pascal-VOC boxes in thermal coordinates, reused for RGB) and `FLIR_xxxxx_mask.jpg`.
  - `AnnotatedImages/` (boxes drawn).
  - Instance counts, counted from the XMLs:

    | Split | person | car | bicycle | dog |
    |---|---|---|---|---|
    | train | 8,987 | 20,608 | 2,566 | 95 |
    | test | 4,107 | 4,124 | 360 | 13 |

- **Other FLIR-aligned repackagings:** CFT [A4] and ICAFusion [V2] convert the same pairs to YOLO format. They are repackagings of the same 5,142 pairs, not new registrations.
- **Other derived sets (not registrations):** ProbEn [P1] released day/night tags for val but did not align the images. Many image-to-image translation papers (thermal↔RGB) use FLIR pairs "as is" or FLIR-aligned. I found no other *released* re-registration of v1 with a learned or calibrated warp (absence not proven).

### 4.3 Existence proof: important-venue papers using each registered version

| Version | Paper (venue) | Evidence |
|---|---|---|
| FLIR-aligned | **GAFF**: H. Zhang et al., "Guided Attentive Feature Fusion for Multispectral Pedestrian Detection", **WACV 2021**, pp. 72–80 [V1] | Text: "[19] proposed an 'aligned' version … 5,142 well-aligned multispectral image pairs (4,129 pairs for training and 1,013 pairs for testing). FLIR covers three object categories" |
| FLIR-aligned | **ICAFusion**: J. Shen et al., **Pattern Recognition** vol. 145, 109913 (2024), arXiv:2308.07504 [V2] | "There are 5,142 aligned multispectral image pairs, of which 4,129 are used for training and 1,013 for testing … we select the FLIR-aligned" |
| RoadScene | **TarDAL**: J. Liu et al., "Target-aware Dual Adversarial Learning and a Multi-scenario Multi-modality Benchmark…", **CVPR 2022**, arXiv:2203.16220 [V3] | Evaluates infrared-visible fusion on TNO, RoadScene and M3FD ("Xu et al. released Roadscene, having 221 aligned…"). U2Fusion itself is in **TPAMI** [B1]. |
| Ad-hoc homography (GEM) | GEM, arXiv:2102.12319 [P3]. Venue reported as IEEE RA-L/IROS 2021 (unverified). | Not a released version, so it gets no proof entry beyond its own paper. |

For contrast, **ProbEn** (Chen et al., **ECCV 2022**, arXiv:2104.02904) [P1] works on the **unaligned** official v1. It trains early/mid fusion with thermal-only labels and drops the 108 val frames without RGB.

---

## Q8. Ways to download

### 8.1 Official

**Correction (2026-11-02, second pass):** An earlier version of this section said the official v1 link was dead and that FLIR serves only v2. **That was wrong.** My checks were scripted, and FLIR's sites block bots, so they return 404, 403 or 400 to scripts. The user reached the official site in a browser and downloaded v1 from the official link.

**Official access path** (verified from the saved pages [O3s][O4s] and the official download [D1]):

1. **Dataset page:** `https://oem.flir.com/solutions/automotive/adas-dataset-form/`, with localized variants such as `/es-es/` [O3][O3s].
   - The page text describes **v2** (26,442 frames, 15 categories, Blackfly S).
   - It says you "must be logged in to download this content", so a **free Teledyne FLIR account login** is needed.
   - The form asks for first name, last name, email, company, country and state/territory, plus an optional marketing opt-in. Submitting accepts FLIR's privacy and cookie policies.
   - The old `www.flir.com/oem/adas/adas-dataset-form/` remains the canonical URL inside FLIR's HTML. Older papers cite it.
2. **Download guide:** After the form, the user reaches "Download Free ADAS Dataset v2 – Teledyne FLIR", hosted at `https://adas-dataset-v2.flirconservator.com/` [O4][O4s].
   - It states: "By downloading this data you agree to the following terms of service: https://www.flir.com/oem/adas/adas-dataset-agree/".
   - It lists the v2 files (`FLIR_ADAS_v2.zip`, 11.87 GB, or 12 parts).
   - Its **Release Notes** end with: *"ADAS Dataset v1.0.0 (Jun 19, 2018) … 5 label categories … 14,452 total annotated frames … Santa Barbara … The original download can be found here: https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d"*. The same text is in the live `https://adas-dataset-v2.flirconservator.com/dataset/README.txt`, which I fetched on 2026-11-02.
3. **v1 Box share:** `https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d`.
   - This is the **official v1 download**. It needs no extra login beyond the Box link (unverified; the user downloaded it in a browser).
   - The user saved its contents as **"FLIR Santa Barbara Regional Thermal Dataset"**. That is presumably the Box folder title (unverified by me; Box blocks scripted access, and the string appears in no file). The README calls it the "FLIR Thermal Starter Dataset". Papers call it "FLIR ADAS (v1)".
   - Contents, verified on the official download:

| File | Size (bytes) | Notes |
|---|---|---|
| `START-HERE-data-download-guide.pdf` | 1,186,103 | "DEC 2020 FLIR ADAS DATASET DOWNLOAD GUIDE": choose `_full` or `_parts`, join parts with `cat`, check with `md5sum -c`. Its examples are a generic template that names `FLIR_ADAS_SF_1_0_0.zip`, not v1.3. |
| `README - FLIR ADAS Dataset V1_3.pdf` | 123,172 | Same file as inside the docs zip |
| `FLIR_ADAS_1_3_full/FLIR_ADAS_1_3_train.zip` | 9,053,257,881 | md5 `094e564f325fabb1435945e3285b6f55`; root `train/` |
| `FLIR_ADAS_1_3_full/FLIR_ADAS_1_3_val.zip` | 1,579,866,039 | md5 `385d2bec9428bf2b4277f7b2c0898fb1`; root `val/` |
| `FLIR_ADAS_1_3_full/FLIR_ADAS_1_3_video.zip` | 5,747,874,323 | md5 `e23b407529e43e749cb836d567952ef1`; root `video/` (RGB folder named `rgb`) |
| `FLIR_ADAS_1_3_full/FLIR_ADAS_1_3_docs.zip` | 224,155 | md5 `4a1d4489a5e387e39caa8939673452d4`; `ReadMe.txt`, README PDF, license PDF |
| `FLIR_ADAS_1_3_full/md5sums.txt` | 229 | All 4 entries OK |
| `FLIR_ADAS_1_3_parts/FLIR_ADAS_1_3.zip.00` … `.15` | 15 × 1,073,741,824 + 246,508,361 = 16,352,635,721 | Concatenate the parts to get one zip with root `FLIR_ADAS_1_3/`. All 16 entries in `md5sums.txt` are OK. |

   - Total share size: about 32.7 GB, because the full and parts sets are redundant. Download one of them.
   - Archive dates are 2020-12-22, i.e. a Dec 2020 repackaging of v1.3 (§1.5).
4. **Archive history:** Archived copies of the old page exist for 2018-07-05 (pre-release, "Available July 2018" [O2]), 2019-07-01 and 2021-06-08 [O1]. They give the v1 web specs.
5. **Support contact** (v1 README): `ADAS-Support@flir.com` [R1].

**Unpacked content of v1.3** (verified on official download). Each of `train/`, `val/` and `video/` contains `Annotated_thermal_8_bit/`, `thermal_8_bit/`, `thermal_16_bit/`, `RGB/` (video: `rgb/`) and `thermal_annotations.json`. Uncompressed: **16.74 GB, 57,174 data files** plus 3 docs and some macOS junk files:

| Folder | train | val | video |
|---|---|---|---|
| thermal_16_bit (TIFF) | 8,862 files / 2.75 GB | 1,366 / 0.45 GB | 4,224 / 1.39 GB |
| thermal_8_bit (JPEG) | 8,862 / 1.13 GB | 1,366 / 0.17 GB | 4,224 / 0.63 GB |
| Annotated_thermal_8_bit | 8,862 / 1.20 GB | 1,366 / 0.18 GB | 4,224 / 0.66 GB |
| RGB (JPEG) | 8,363 / 4.15 GB | 1,257 / 0.80 GB | 4,195 / 3.18 GB |
| thermal_annotations.json | 30.4 MB | 5.2 MB | 16.6 MB |

### 8.2 Mirrors (all unofficial; see license)

| Mirror | Content | Size | Status / notes |
|---|---|---|---|
| Kaggle `deepnewbie/flir-thermal-images-dataset` [K1] | **Full v1.3** (`FLIR_ADAS_1_3/`, including license and README). **Byte-identical to the official download** for all annotations, docs and image name/size lists (§1.6). | zip 16.38 GB (16,380,731,235 B); files 16.74 GB | Live. Updated 2020-08-18, about 15k downloads, license field "Unknown". Single files can be fetched through the public Kaggle API (`/api/v1/datasets/download/<owner>/<slug>/<path>`). |
| Kaggle `albertofv/flir-thermal-images-dataset-reduced` [K3] | Reduced v1 subset | 1.33 GB | Live. Subset definition not checked. |
| Roboflow Universe `joey-hieronimy-student-uibk-ac-at/flir_adas_1_3` [K4] | Re-exported v1.3 thermal, **re-split**. Search snippets give 9,511 images and a 70% train split of 6,643. | n/a | Splits differ from the official 8,862/1,366, so do not use it for benchmark comparisons. Page returned 403 to my fetcher. |
| Kaggle `kushagraborse/thermal-images-flir-adas-dataset`, `samdazel/...-v2`, `rajababuadigarla/...-v2` | **v2**, not v1 (the 11.88 GB size matches v2) | 11.9 GB | Easy to confuse with v1 |
| Academic Torrents | none found | n/a | No FLIR ADAS torrent found (search on 2026-11-02) |
| Hugging Face | No full v1.3 mirror found. FLIR-aligned is mirrored (see 4.2). | n/a | n/a |

Registered-version downloads are listed in table 4.2: FLIR-aligned on Google Drive, HF, Baidu and Kaggle, and RoadScene on GitHub.

### 8.3 License and terms of use

- **"FLIR ADAS Image License Agreement" (last modified 26 Jul 2018) plus Terms of Use** [L1], shipped as `ADAS User License Agreement (26.Jul.2018) (Final).pdf` in `FLIR_ADAS_1_3_docs.zip`.
  - The official file (md5 `905c6efb2e23830519b9dc2841452d72`, 7 pages) is identical to the Kaggle copy whose text is quoted below (verified on official download).
  - The v2 download guide additionally points to online terms of service at `https://www.flir.com/oem/adas/adas-dataset-agree/` [O4]. That URL returned 404 to my scripted client, so its content is unverified.
  - The terms of use describe the data as for "ADAS and self-driving vehicle researchers, developers, auto manufacturers, and others as approved by FLIR".
  - §1.1 grants a "limited, revocable, non-exclusive, non-sublicensable, non-transferrable license … in the field of neural network development for automotive and other autonomous vehicle applications, and for general **non-commercial educational and research purposes**." No right "to reproduce, **redistribute** or make derivative works".
  - §3.1 says derivative works "shall belong to" FLIR.
  - §1.3(c) requires deleting any data that counts as personal data under privacy law.
  - The data is provided "AS IS".
- **Consequences:**
  - Kaggle, Roboflow and HF copies, *including FLIR-aligned and RoadScene*, are redistributions **not authorised** by the license text (my reading, not legal advice).
  - Kaggle lists the license as "Unknown". Roboflow pages may show CC BY 4.0, which does not override FLIR's terms.
  - ProbEn explicitly avoided redistributing the data for this reason [P1].
- v2 has its own terms on the current page. Not covered here.

### 8.4 Link status (checked 2026-11-02)

Caveat: FLIR and Box block scripted clients. A 404, 403 or 400 seen by my scripts does **not** prove a link is dead in a browser.

| Link | Status |
|---|---|
| `https://www.flir.com/oem/adas/adas-dataset-form/` (cited by most v1 papers) | 404 to scripts. Still the canonical URL inside FLIR's current page HTML. Browser status untested; use `oem.flir.com` instead. Archive.org snapshots work. |
| `https://www.flir.com/adasdataset`, `https://www.flir.com/oem/adas/` | 404 to scripts |
| `https://oem.flir.com/solutions/automotive/adas-dataset-form/` | Live. Login plus form; the page text describes v2 and leads to the v2 guide. |
| `https://adas-dataset-v2.flirconservator.com/` (v2 guide, with the v1 link in its release notes) | Live (200 to scripts) |
| **`https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d` (official v1.3)** | **Live in a browser** (the user downloaded it on 2026-11-02). Returns 400 to scripts. |
| `https://www.flir.com/oem/adas/adas-dataset-agree/` (terms of service) | 404 to scripts; browser status unknown |
| FLIR-aligned short link `http://shorturl.at/ahAY4` | Redirects to https, then returns 403 to scripted clients. The underlying Google Drive file is live. |
| FLIR-aligned Google Drive `1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y` | Live (`aligned.zip`) |
| HF `UserNae3/FLIR_aligned` (cited in some READMEs) | Redirects to `jsonhash/FLIR_aligned` (renamed) |

---

## Open uncertainties

**Resolved in the second pass:**

- Teledyne still distributes v1. It is v1.3, on Box, linked from the v2 guide's release notes.
- The v1.3 counts are official: the Kaggle copy is byte-identical to the official download.
- The web-page counts (Other Vehicle 2,228/540; 9,214/4,183 frames with boxes) do not describe v1.3 and most likely describe v1.0.0 (19 Jun 2018).

**Still open:**

1. What exactly changed between v1.0.0 and v1.3. v1.1 and v1.2 are undocumented, and v1.0.0 files are not distributed, so the removal of "Other Vehicle" in a later v1.x remains an inference.
2. Whether "FLIR Santa Barbara Regional Thermal Dataset" is the literal Box folder title, and the content of the online terms page `adas-dataset-agree`. Both are blocked to scripts.
3. The exact preprocessing that turned the raw RGB into 640×512 in FLIR-aligned (fixed crop + resize vs per-pair adjustment). The paper says only "manually removed".
4. RoadScene's registration method and confirmation that its frames come from v1 (inferred from dates).
5. GEM's final venue.
6. The Roboflow v1.3 mirror's exact counts (page blocked).

---

## References

- **[O1]** FLIR, "FREE - FLIR Thermal Dataset for Algorithm Training" (dataset spec page), archived 2019-07-01: https://web.archive.org/web/20190701150003/https://www.flir.com/oem/adas/adas-dataset-form/. The 2021-06-08 snapshot has identical specs: https://web.archive.org/web/20210608184912/https://www.flir.com/oem/adas/adas-dataset-form/
- **[O2]** Same page, archived 2018-07-05 ("Available July 2018"): https://web.archive.org/web/20180705012657/https://www.flir.com/oem/adas/adas-dataset-form/
- **[O3]** Current Teledyne FLIR page (v2): https://oem.flir.com/solutions/automotive/adas-dataset-form/
- **[O3s]** The same page in the Spanish locale, saved by the user from a browser on 2026-11-02: `/home/leonardo.ulloa/Downloads/GRATIS - Conjunto de datos térmicos para formación en algoritmos FLIR _ Teledyne FLIR OEM.html`
- **[O4]** "Download Free ADAS Dataset v2 – Teledyne FLIR" (download guide and release notes, including the v1.0.0 note and the Box link): https://adas-dataset-v2.flirconservator.com/ and https://adas-dataset-v2.flirconservator.com/dataset/README.txt
- **[O4s]** The same guide saved by the user: `/home/leonardo.ulloa/Downloads/Download Free ADAS Dataset v2 - Teledyne FLIR.html`
- **[D1]** Official v1.3 download from https://flir.box.com/s/suwst0b3k9rko35homhr3rnyytf3102d, saved by the user at `/home/leonardo.ulloa/projects/data/FLIR Santa Barbara Regional Thermal Dataset/`. It contains `START-HERE-data-download-guide.pdf`, `README - FLIR ADAS Dataset V1_3.pdf`, `FLIR_ADAS_1_3_full/` and `FLIR_ADAS_1_3_parts/`.
- **[R1]** `FLIR_ADAS_1_3/README - FLIR ADAS Dataset V1_3.pdf` and `ReadMe.txt`, "FLIR Thermal Starter Dataset Introduction, Version 1.3, August 16, 2019", read from mirror [K1]
- **[R2]** v1.3 `thermal_annotations.json` (train/val/video), read from mirror [K1]. All "(counted)" numbers come from these files.
- **[L1]** `ADAS User License Agreement (26.Jul.2018) (Final).pdf`, read from mirror [K1]
- **[K1]** Kaggle, deepnewbie, "FLIR Thermal Images Dataset": https://www.kaggle.com/datasets/deepnewbie/flir-thermal-images-dataset (API metadata: https://www.kaggle.com/api/v1/datasets/view/deepnewbie/flir-thermal-images-dataset)
- **[K2]** Kaggle, s111mple, "FLIR_aligned": https://www.kaggle.com/datasets/s111mple/flir-aligned
- **[K3]** Kaggle, albertofv, "FLIR Thermal Images Dataset [reduced]": https://www.kaggle.com/datasets/albertofv/flir-thermal-images-dataset-reduced
- **[K4]** Roboflow Universe, FLIR_ADAS_1_3: https://universe.roboflow.com/joey-hieronimy-student-uibk-ac-at/flir_adas_1_3
- **[N1]** Laser Focus World, "FLIR launches open-source thermal imaging dataset…" (June 2018): https://www.laserfocusworld.com/articles/2018/06/flir-launches-open-source-thermal-imaging-dataset-for-autonomous-vehicle-development.html
- **[N2]** Teledyne FLIR press release on the expanded (v2) dataset, Jan 2022: https://oem.flir.com/about/news/expanded-teledyne-flir-starter-thermal-dataset-for-adas-and-autonomous-vehicle-testing/
- **[A1]** H. Zhang, E. Fromont, S. Lefèvre, B. Avignon, "Multispectral Fusion for Object Detection with Cyclic Fuse-and-Refine Blocks", ICIP 2020. arXiv:2009.12664: https://arxiv.org/abs/2009.12664. HAL: https://hal.archives-ouvertes.fr/hal-02872132
- **[A2]** FLIR-aligned, Google Drive: https://drive.google.com/file/d/1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y/view
- **[A3]** Hugging Face mirror, jsonhash/FLIR_aligned: https://huggingface.co/datasets/jsonhash/FLIR_aligned
- **[A4]** CFT repo (Fang et al., arXiv:2111.00273), with dataset links: https://github.com/DocF/multispectral-object-detection
- **[B1]** RoadScene repo: https://github.com/hanna-xu/RoadScene. Papers: Xu et al., "FusionDN", AAAI 2020; Xu et al., "U2Fusion", IEEE TPAMI 2020.
- **[P1]** Y.-T. Chen et al., "Multimodal Object Detection via Probabilistic Ensembling" (ProbEn), ECCV 2022: https://arxiv.org/abs/2104.02904
- **[P2]** "Detecting Small Objects in Thermal Images Using Single-Shot Detector": https://arxiv.org/abs/2108.11101 (example of mixed-up counts)
- **[P3]** O. Mazhar, R. Babuška, J. Kober, "GEM: Glare or Gloom, I Can Still See You – End-to-End Multi-Modal Object Detection": https://arxiv.org/abs/2102.12319
- **[V1]** H. Zhang et al., "Guided Attentive Feature Fusion for Multispectral Pedestrian Detection", WACV 2021: https://openaccess.thecvf.com/content/WACV2021/html/Zhang_Guided_Attentive_Feature_Fusion_for_Multispectral_Pedestrian_Detection_WACV_2021_paper.html
- **[V2]** J. Shen et al., "ICAFusion: Iterative Cross-Attention Guided Feature Fusion for Multispectral Object Detection", Pattern Recognition 145 (2024) 109913: https://arxiv.org/abs/2308.07504. Code: https://github.com/chanchanchan97/ICAFusion
- **[V3]** J. Liu et al., "Target-aware Dual Adversarial Learning and a Multi-scenario Multi-Modality Benchmark to Fuse Infrared and Visible for Object Detection" (TarDAL), CVPR 2022: https://arxiv.org/abs/2203.16220
- v2 file-level stats, for disambiguation only: https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/00-the-dataset.md
