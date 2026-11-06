# FLIR ADAS v1: known problems and established workarounds (Q7)

**Research date:** 2026-11-02
**Scope:** This file covers three things:
- the Teledyne FLIR ADAS thermal starter dataset **v1** (last release v1.3, Aug 2019);
- its two main derivatives, **FLIR-aligned** (Zhang et al., ICIP 2020; 5,142 RGB-T pairs) and **RoadScene** (Xu et al., AAAI/TPAMI 2020; 221 pairs);
- a few custom v1 subsets.

**Builds on:**
- [`01_dataset_overview.md`](01_dataset_overview.md): counts, classes, registration, downloads, license.
- [`02_usage_and_publications.md`](02_usage_and_publications.md): the 48 top-venue papers and their repos.

**Method:**
- I read the dataset loaders and configs of about 20 public repos, plus GitHub issues on the two most-forked FLIR-aligned codebases (CFT and ICAFusion).
- I extracted the dataset paragraphs from about 15 paper PDFs.
- I ran a box-level audit of the three v1.3 `thermal_annotations.json` files and read pixel statistics from five 16-bit TIFFs. Both came from the Kaggle mirror `deepnewbie/flir-thermal-images-dataset`.
- Numbers marked **(counted)** are my own counts. Claims I could not verify are marked **(unverified)**.

---

## TL;DR

1. **Nobody has re-annotated FLIR-aligned in public.** I found no corrected or "sanitized" FLIR-aligned or v1 label set. Some papers call FLIR-aligned "sanitized" or "cleaned", but that set only *removed misaligned pairs*. The thermal boxes were not relabelled. The only partial re-labelling I found is Vadidar et al. (IEEE IV 2022): they cross-labelled RGB from the thermal boxes, manually added missing objects, and did not release the result. Missing labels on small or distant pedestrians are reported (Vadidar Fig. 7). D3T (CVPR 2024) says the raw v1 has "many labeling errors" but gives no evidence.
2. **Class handling is the most uniform practice.** About 90% of works keep **person, car, bicycle**. Dog (240 boxes in train+val; 108 boxes in FLIR-aligned) and the empty "Other Vehicle" class are dropped. There are exceptions:
   - MiPa keeps dog.
   - HalluciDet, the person-only adversarial subsets and an ICAFusion config keep **person only**.
   - Class order differs between repos (CFT: person, car, bicycle; RSDet: car, person, bicycle; v1 JSON IDs: 1 person, 2 bicycle, 3 car, 17 dog; MMTOD's pre-v1.3 converter: dog = 18). This causes real bugs (CFT issue #34).
3. **Misalignment is handled at three levels:**
   - (a) **Data level.** Use FLIR-aligned (almost everyone), or build your own global homography or offset (GEM, Agrawal et al., Vadidar et al.; all unreleased).
   - (b) **Model level.** Use misalignment-tolerant fusion: deformable cross-attention (DAMSDet, ECCV 2024), cross-modal deformable attention with synthetic offsets "FLIR_w" (CF-Deformable DETR, IJCAI 2024), or adaptive affine calibration (ADCNet, Remote Sensing 2023).
   - (c) **Decision level.** Use late fusion on unaligned v1 (ProbEn, ECCV 2022).

   FLIR-aligned still shows "obvious misalignment" (DAMSDet), and no paper reports a measured residual offset.
4. **There is no official test split, and the community split is used inconsistently.** The two usual protocols are:
   - **FLIR-aligned 4,129 / 1,013**, the de facto standard;
   - **official val 1,366**, or 1,258 after dropping frames without RGB.

   Problems seen in practice:
   - Some repos re-split at random: 9:1 or Roboflow splits. CFT issues #50 and #58 report about 10 points too high, and even mAP50 = 0.98. Frames sampled at 2 fps from continuous drives leak across random splits.
   - Model selection is done on the test set, because there is no val set (seen in ICAFusion issue #56).
   - TINet's paper and repo use different splits. ICAFusion's README revised its FLIR mAP50 from 79.2 to 82.8 after "fixing bugs".

   Comparisons across papers within about 2–3 mAP50 points are therefore not reliable.
5. **Bit depth: nearly everyone uses the 8-bit AGC JPEGs.** FLIR-aligned ships only 8-bit. The 16-bit TIFFs hold 14-bit counts in a narrow, drifting band: about 6,000–8,300 out of 65,535, with only 500–1,750 distinct levels per frame (counted on 5 frames). Naive `/65535` normalization therefore gives an almost flat image. The few 16-bit users apply per-frame percentile stretching or learned tone mapping (N-AGC, ICCVW 2025). There is no matched 8-bit vs 16-bit study on v1 (gap).
6. **Other problems:**
   - **Missing RGB frames** (637). Fusion works drop them, e.g. ProbEn's 1,258 val frames.
   - **Licensing and dead links.** The official v1 link returns 404, the license forbids redistribution, and the CFT README's Drive link at one point pointed to an eBay listing.
   - **Domain limits.** One region (Santa Barbara), mostly clear weather, about 10k frames, and val boxes larger than train boxes (median person height 52 px in val vs 34 px in train; counted). The workarounds are multi-dataset protocols (FLIR-aligned + LLVIP + M3FD), multi-dataset pretraining (MMPedestron, M-SpecGene), and the move to v2 for thermal-only work.

---

## How to read each section

Each section has the same four parts:
- **Problem:** what is wrong, with evidence and source.
- **Impact:** what it does to training and evaluation.
- **Workarounds:** who uses each one, with a link.
- **How established:** **widely used** (most FLIR papers or repos do it), **a few papers** (2–5 published works), or **ad-hoc** (one paper, one repo or forum advice).

---

## P1. Annotation errors and inconsistencies (including re-annotated versions)

### Problem

**Evidence from the files (v1.3 JSON, counted).** The JSON is mechanically clean:
- 0 boxes out of image bounds;
- 0 boxes with non-positive width or height;
- `iscrowd = 0` everywhere;
- every box is flagged `human_annotated: true`.

Duplicates are rare: 4 / 3 / 2 exact duplicate boxes (same class) in train / val / video, and 1 / 0 / 4 same-class pairs with IoU > 0.9. The real problems are about **consistency**, not syntax:

| Split | boxes | area < 16² px | min side < 8 px | median person height | median car height |
|---|---|---|---|---|---|
| train | 67,844 | 11,049 (16.3%) | 1,088 | 34 px | 30 px |
| val | 11,696 | 553 (**4.7%**) | 4 | **52 px** | 35 px |
| video | 37,183 | 14,423 (**38.8%**) | 7,552 (185 under 4 px) | 27 px | 24 px |

- **Different minimum-size conventions between splits** (counted). Val has almost no boxes with a side under 8 px (4 boxes vs 1,088 in train). Its objects are much larger, and video is much smaller. The likely causes are different drives, different annotators or a size cut-off, but which one is **unverified**. In any case, the effective "how small do we label" rule is not the same across the official splits.
- **No ignore or crowd regions.** Distant or occluded people that annotators skipped look exactly like background, and nothing in the data marks them as ambiguous.
- **Missing labels are reported in papers.** Vadidar et al. (IV 2022) show detections of sidewalk pedestrians that "were not labeled", and a night target that "was neither labeled nor provided a sufficient amount of pixel information" ([arXiv 2206.03943](https://arxiv.org/abs/2206.03943), Sec. V, Fig. 7). D3T (CVPR 2024) chose FLIR-aligned "over the older one because it has many labeling errors" ([arXiv 2403.09359](https://arxiv.org/abs/2403.09359), Sec. 4.1). D3T gives no audit, so this claim is **unverified**.
- **Class-definition noise.**
  - "Bicycle" includes **motorcycles** (README v1.3).
  - Riders are boxed separately from the bicycle.
  - "Car" includes "some small commercial vehicles".
  - "Other Vehicle" (large trucks, boats, trailers) is advertised (2,228 + 540 boxes on the 2019–2021 web page; "cars, other vehicles, people, bicycles, and dogs" in the [2018 press release](https://www.flir.com/news-center/press-releases/flir-releases-starter-thermal-imaging-dataset-for-machine-learning-advanced-driver-assistance-development/)) but has **0 boxes and no category** in v1.3 (see 01 §1.4). Trucks and buses are therefore either unlabelled background or folded into car (**unverified** which).
- **Label-map and ID drift between releases.**
  - The v1.3 JSON uses dog = 17. `ReadMe.txt` says "Category 18: Dogs" and "Category 91: Other Vehicle".
  - MMTOD's converter for the older per-frame JSON layout (`FLIR_ADAS/training/Annotations/*.json`) maps `{'3': 'car', '1': 'person', '2': 'bicycle', '18': 'dog'}` ([FLIR_to_PVOC.ipynb](https://github.com/tdchaitanya/MMTOD/blob/master/generate_annotations/FLIR_to_PVOC.ipynb)). So pre-v1.3 releases used dog = 18 and a different file layout (inferred from code).
  - The JSON `categories` list has 81 entries, of which only 4 are used.
- **FLIR-aligned inherits all of the above** and adds its own quirks:
  - VOC XMLs still contain 108 dog boxes (95 train / 13 test; counted in 01).
  - Thermal boxes are reused unchanged for RGB.
  - Converted label sets differ slightly. ICAFusion's test log shows 4,106 person / 4,123 car / 359 bicycle ([issue #18](https://github.com/chanchanchan97/ICAFusion/issues/18)), against 4,107 / 4,124 / 360 in the original XMLs (01). The cause, perhaps one image dropped or a degenerate box filtered, is **unverified**.
- **Re-annotated versions: none released.**
  - The "sanitized" label exists only for **KAIST**. TINet, for example, uses "the sanitized annotation provided by Li et al." for KAIST but the plain FLIR-aligned for FLIR ([TINet README](https://github.com/NNNNerd/Triple-I-Net-TINet)).
  - HalluciDet and MiPa call FLIR-aligned "the sanitized and aligned paired sets provided by Zhang et al." ([arXiv 2310.04662](https://arxiv.org/abs/2310.04662), [arXiv 2404.18849](https://arxiv.org/abs/2404.18849)). CF-Deformable DETR calls it "a cleaned version" ([IJCAI 2024](https://www.ijcai.org/proceedings/2024/0084.pdf)). Zhang et al. only "manually removed the misaligned visible-thermal image pairs" ([arXiv 2009.12664](https://arxiv.org/abs/2009.12664)), so the word is a misnomer.
  - Two re-labelling efforts exist, and neither is public. Vadidar et al. cross-labelled RGB and added "additional manual labeling of the missing objects, and checking the data one by one". Agrawal et al. used an 8-point homography to make "FLIR RGB" labels ([arXiv 1909.13551](https://arxiv.org/abs/1909.13551)).

### Impact

- Missing labels on small or far objects count as false positives, which caps precision and penalises detectors that are good at small objects.
- Because val has few tiny boxes, a model tuned on val will look better than it really is on small objects (video and train have far more). Numbers on val, FLIR-aligned test and video are not interchangeable.
- The motorcycle-in-bicycle merge makes the bicycle class visually heterogeneous. Bicycle is consistently the weakest class: ICAFusion's README gives AP50 73.8 for bicycle vs 89.8 for car.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Use the labels as they are, keep 3 classes, accept the noise | Practically every FLIR-aligned paper (CFT, ICAFusion, DAMSDet, RSDet, ...) | **widely used** |
| Use FLIR-aligned *because* it is assumed cleaner (it is not relabelled) | D3T, HalluciDet, MiPa, CF-Deformable DETR | **a few papers** (based on a misunderstanding) |
| Re-label or cross-label RGB, add missing objects manually (not released) | Vadidar et al. IV 2022 (later reused by "Rethinking Early-Fusion", TIV 2025); Agrawal et al. 2019 | **ad-hoc** |
| Filter tiny or empty boxes and images in the loader: mmdet `filter_cfg=dict(filter_empty_gt=True, min_size=32)` | RSDet [`configs/_base_/datasets/FLIR.py`](https://github.com/Zhao-Tian-yi/RSDet/blob/master/configs/_base_/datasets/FLIR.py). Note: `min_size` here filters *images* smaller than 32 px, not boxes. Only empty images are effectively removed. | **a few repos** (the mmdet default) |
| Drop images without boxes | MMTOD trains on "9,214 images with bounding box annotations" ([CVPRW 2019](https://openaccess.thecvf.com/content_CVPRW_2019/html/PBVS/Devaguptapu_Borrow_From_Anywhere_Pseudo_Multi-Modal_Object_Detection_in_Thermal_Imagery_CVPRW_2019_paper.html)); D3T removes unlabeled KAIST images; mmdet `filter_empty_gt` | **a few papers** |
| Visually check the converted labels before training | DocF's (CFT author's) advice in [CFT #23](https://github.com/DocF/multispectral-object-detection/issues/23): "可视化一下你的数据集标注" ("visualize your dataset's annotations") | **ad-hoc** |

**Gap:** No public label audit or corrected FLIR-aligned annotation file exists. Size-stratified reporting (APs, APm, APl, which some recent papers give) partly controls for the size-convention issue.

---

## P2. Class imbalance and rare classes

### Problem

Counted in 01 and above:

| Set | person | car | bicycle | dog | other vehicle |
|---|---|---|---|---|---|
| v1 train+val | 28,151 | 46,692 | 4,457 | 240 (0.3%) | 0 |
| v1 video | 21,965 | 14,013 | 1,205 | 0 | 0 |
| FLIR-aligned train | 8,987 | 20,608 | 2,566 | 95 | – |
| FLIR-aligned test | 4,107 | 4,124 | **360** | 13 | – |

- Bicycle is 5–8% of boxes.
- FLIR-aligned test has **only 360 bicycles**, so bicycle AP, which is one third of mAP, has high variance. GEM notes that bicycle performance "is not stable", given that the class "only constitutes 8.23%" ([arXiv 2102.12319](https://arxiv.org/abs/2102.12319)).
- Dog is unusable. MiPa and ModTr: "the 'dog' objects are inadequate for training". GEM: "0.29% of all the annotations".
- LYNRED-MDS (2026) on v2: "class imbalance makes most of those [classes] useless for training" ([arXiv 2607.01871](https://arxiv.org/abs/2607.01871)).

### Impact

- mAP is dominated by bicycle variance: one third of the weight sits on 360 test instances.
- A stray dog class in the label files silently changes `nc` and the mAP denominator. Users of the CFT/FLIR-aligned zip hit this in [CFT #34](https://github.com/DocF/multispectral-object-detection/issues/34) (2023–2024 comments): "数据集本身还包含一个'dog'分类" ("the dataset itself also contains a 'dog' class"), and a reply guessing that the author removed dog because it has "仅有108个" ("only 108") samples.
- If class IDs are reordered between conversion and inference, every car is predicted as person (same issue, resolved as "标签顺序的问题", "a label-order problem").

### Classes kept: paper / repo → classes

| Paper / repo (venue) | Variant | Classes kept | Evidence |
|---|---|---|---|
| FLIR README baseline (RefineDet) | v1 | person, bicycle, car (+ "all") | README v1.3 |
| CFR / FLIR-aligned (ICIP 2020) | creates aligned | "only kept the 3 more frequent classes": bicycle, car, person. Dog boxes remain in the XML. | [arXiv 2009.12664](https://arxiv.org/abs/2009.12664) §4.1 |
| MMTOD (CVPRW 2019) | v1 | converter maps 4 (incl. dog); reports Bicycle / Person / Car | [notebook](https://github.com/tdchaitanya/MMTOD/blob/master/generate_annotations/FLIR_to_PVOC.ipynb), paper Table |
| GEM (arXiv 2021) | v1 + own homography | person, bicycle, car (dog dropped, 0.29%) | [arXiv 2102.12319](https://arxiv.org/abs/2102.12319) |
| SGADA (CVPRW 2021) | v1 crops | bicycle, car, person | [README](https://github.com/avaapm/SGADA) "for bicycle, car and person classes only" |
| SSTN (IROS 2021) | v1 | car, person, bicycle | 02 §3.1 |
| ProbEn (ECCV 2022) | v1 RGB-paired | person, car, bicycle (`NUM_CLASSES = 3`) | [demo_train_FLIR.py](https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling/blob/main/demo/FLIR/demo_train_FLIR.py) L56 |
| Vadidar et al. (IV 2022) | own v1 alignment | car, pedestrian, bicycle | [arXiv 2206.03943](https://arxiv.org/abs/2206.03943) |
| CFT (arXiv 2021, base codebase) | FLIR-aligned | `nc: 3`, `['person','car','bicycle']` | [data/multispectral/FLIR_aligned.yaml](https://github.com/DocF/multispectral-object-detection/blob/main/data/multispectral/FLIR_aligned.yaml) |
| ICAFusion (PR 2024) | FLIR-aligned | 3-class config `['person','car','bicycle']`, **plus a person-only config** (`nc: 1`) | [FLIR-align-3class.yaml](https://github.com/chanchanchan97/ICAFusion/blob/main/data/multispectral/FLIR-align-3class.yaml), [FLIR-align.yaml](https://github.com/chanchanchan97/ICAFusion/blob/main/data/multispectral/FLIR-align.yaml) |
| RSDet (T-ITS 2025) | FLIR-aligned | `('car', 'person', 'bicycle')`, **different order** | [configs/_base_/datasets/FLIR.py](https://github.com/Zhao-Tian-yi/RSDet/blob/master/configs/_base_/datasets/FLIR.py) |
| YOLOv11-RGBT (2025) | FLIR-aligned "FLIR3C" | `["person", "car", "bicycle"]` | [README](https://github.com/wandahangFY/YOLOv11-RGBT) |
| MMPedestron (ECCV 2024) | FLIR-aligned | `num_classes=3` (a `classes=('person',)` option is commented out) | [mmpedestron_flir_2x.py](https://github.com/BubblyYi/MMPedestron/blob/main/configs/mmpedestron/flir_exp/mmpedestron_flir_2x.py) |
| D3T (CVPR 2024) | FLIR-aligned | "people", "cars", "bicycle" | [arXiv 2403.09359](https://arxiv.org/abs/2403.09359) §4.1 |
| ModTr (ECCV 2024) | FLIR-aligned | dog removed → 3 | [arXiv 2404.01492](https://arxiv.org/abs/2404.01492) |
| **MiPa (WACV 2025)** | FLIR-aligned | **keeps dog → 4** ("we have decided to keep it") | [arXiv 2404.18849](https://arxiv.org/abs/2404.18849) |
| **HalluciDet (WACV 2024)** | FLIR-aligned | **person only** | [arXiv 2310.04662](https://arxiv.org/abs/2310.04662) |
| Adversarial attacks (AAAI 2021, CVPR 2022) | v1.3 subsets | **person only** ("FLIR person select" 710/301; PEOPLE_FLIR 7,873/2,027) | 02 §3.1 |
| CGDet (arXiv 2025) | FLIR-aligned | person, bicycle, car | [arXiv 2511.01435](https://arxiv.org/abs/2511.01435) |
| RGBXFusion (WACV 2024) | FLIR-aligned | effdet with `--num-classes 90` (COCO label space; FLIR classes mapped in) | [README](https://github.com/dsriaditya999/RGBXFusion) |
| Fusion-Mamba (TMM 2025) | FLIR-aligned | text says "five categories"; evaluates 3 | 02 §3.4 |

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Keep person, car, bicycle; drop dog and other vehicle | All rows above except MiPa, HalluciDet and the person-only works | **widely used**, the de facto protocol |
| Person-only subset | HalluciDet, the ICAFusion `nc: 1` config, the adversarial-attack papers | **a few papers** |
| Keep dog | MiPa (states it is not for benchmark comparison) | **ad-hoc** |
| Report per-class AP next to mAP50 so bicycle variance is visible | CFT, ICAFusion, MS2Fusion, RSDet READMEs (person / car / bicycle columns) | **widely used** |
| Use a COCO-pretrained detector, so rare classes inherit COCO features | FLIR baseline (RefineDet COCO), MMTOD, ProbEn, HalluciDet; ICAFusion's improved result needs YOLOv5l COCO weights ([ICAFusion #44](https://github.com/chanchanchan97/ICAFusion/issues/44)) | **widely used** |
| Re-weighting or focal loss for imbalance | I found no FLIR-specific re-weighting. Generic focal loss (RetinaNet) appears in baselines. | not established (gap) |

---

## P3. Small objects and input resolution

### Problem

- Small objects dominate (COCO "small" = 58% of train boxes, 77% of video boxes; 01 §1.4).
- Median heights are 24–52 px depending on the split (counted, P1).
- FLIR-aligned is 640×512, and many YOLO repos letterbox it to 640×640.

### Impact

- Results depend strongly on input size. Vadidar et al.: "10.5% raise in the mAP due to a higher input resolution", from 320 to 640 ([arXiv 2206.03943](https://arxiv.org/abs/2206.03943)).
- CFT trains at 640×640 but validates with rectangular 544×672 batches ([CFT #45](https://github.com/DocF/multispectral-object-detection/issues/45)), so train and test geometry differ silently.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Native 640×512, or letterbox 640 (YOLO default) | CFT, ICAFusion, YOLOv11-RGBT, ProbEn ("We resize all images to resolution 640x512") | **widely used** |
| Upsample 2× (1280×1024 / 1024×1280) | RSDet `PairedImagesResize(scale=(1024, 1280))` ([FLIR.py](https://github.com/Zhao-Tian-yi/RSDet/blob/master/configs/_base_/datasets/FLIR.py)) | **a few repos** (mmdet-based fusion repos) |
| Report APs / APm / APl | CGDet, DAMSDet-style DETR papers | **a few papers** |

When comparing numbers, always record the input size.

---

## P4. No registration in the official release

### Problem

- RGB (mostly 1800×1600, other sizes too) and thermal (640×512) come from separate cameras about 1.9 in apart.
- They have different FOVs, and no intrinsics, extrinsics or homography are provided.
- The README says the thermal boxes "do not represent the placement of objects in the RGB image" (01 §4.1).
- ProbEn: the pairs are "unaligned both spatially and temporally" ([arXiv 2104.02904](https://arxiv.org/abs/2104.02904), Fig. 4).
- Vadidar et al. list "camera resolutions ... ranging from 0.3 to 3.1 MP, misalignment of resolution/aspect ratio, a vast difference between the field of views ... and no possibility to align the RGB and IR frames without having a correlation matrix".
- Agrawal et al.: images are uncorrected, and "the points radially distant from the center do not align well" under a homography ([arXiv 1909.13551](https://arxiv.org/abs/1909.13551)).

### Impact

- There are no RGB labels at all.
- Pixel-level early fusion is meaningless on raw pairs.
- A single global warp is imperfect: there is lens distortion, parallax that depends on depth, and RGB resolutions that vary between frames.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| **Use FLIR-aligned**: manual removal of misaligned pairs, RGB delivered at 640×512 (the exact crop/resize is undocumented) | CFR/GAFF and essentially all RGB-T detection papers since 2021 (26 of the 48 papers in 02) | **widely used** |
| Ignore RGB and do thermal-only detection | FLIR baseline, MMTOD, SSTN, SGADA, the adversarial-attack papers | **widely used** (thermal tasks) |
| Train fusion on unaligned pairs with thermal labels only; let mid-fusion "act as an implicit alignment network"; late-fuse detectors (ProbEn) | ProbEn ECCV 2022; CVPR 2026 "Distribution-Aligned" uses 10,228 pairs (handling unclear, 02) | **a few papers** |
| Global homography from hand-picked correspondences; warp RGB to thermal | GEM ([arXiv 2102.12319](https://arxiv.org/abs/2102.12319)); Agrawal et al. ("8 point homography", about 8,000 train / 1,247 val) | **a few papers** (unreleased) |
| Edge-based search for scale and offset (Canny + sliding cross-correlation) per pair, then cross-label RGB; 6,924 / 1,982 / 3,960 pairs | Vadidar et al. IV 2022, Algorithm 1; reused by TIV 2025. The 02 file quotes 7,381 / 1,056 / 2,111 for the same work; the split differs between arXiv versions (**unverified** which is final). | **ad-hoc** (unreleased) |
| Unpaired use (CycleGAN-style day-RGB → night-IR domains), so registration is irrelevant | PearlGAN / MornGAN / FoalGAN (resize 500×400, centre crop 360×288; [PearlGAN README](https://github.com/FuyaLuo/PearlGAN)), MMTOD's translation branch | **widely used** in translation work |
| Fusion-only pairs (RoadScene: noise-reduced, aligned, cropped) | Image-fusion community | **widely used** (fusion) |

---

## P5. Residual misalignment in FLIR-aligned (and how methods handle it)

### Problem

FLIR-aligned was made by *selecting* pairs, not by warping them. Residual offsets therefore remain:
- DAMSDet (ECCV 2024): the aligned version "contains both day and night scenes, with **obvious misalignment of image pairs**" ([arXiv 2403.00326](https://arxiv.org/abs/2403.00326), §4.1 and Fig. 1b).
- CFR's own conclusion: "Several visible/thermal image pairs have a misalignment problem in FLIR dataset" ([arXiv 2009.12664](https://arxiv.org/abs/2009.12664)).
- Vadidar et al. show "well-aligned" pairs with visibly shifted boxes (Fig. 3a). They state that "the preprocessing part, could not tackle all the issues", and report "3.6% mAP improvement ... solely achieved by better preprocessing".
- **No paper reports a measured residual offset distribution** for FLIR-aligned (gap). The size of the offsets is **unverified**.
- Boxes are in thermal coordinates and are reused for RGB.

### Impact

- An RGB branch trained on thermal boxes learns slightly wrong localisation.
- Pixel-wise early fusion and "pixel-aligned" assumptions (for example the YOLOv11-RGBT framework, which states it "is applicable to all pixel-aligned images") are violated.
- mAP@0.75 and mAP@[.5:.95] suffer more than mAP50. This is plausibly why FLIR-aligned mAP sits around 0.4 while mAP50 is about 0.8 (inference).

### Workarounds

| Workaround | Who / link | How established |
|---|---|---|
| Ignore it; mid-level feature fusion with stride ≥ 8 tolerates small shifts | Most YOLOv5 two-stream repos (CFT, ICAFusion, MS2Fusion, IRDFusion) | **widely used** (implicit) |
| Use thermal GT only for evaluation (the standard). RGB-only baselines are scored against thermal boxes. | All FLIR-aligned papers | **widely used** (the convention, not a fix) |
| Deformable or offset-aware cross-modal attention, sampling each modality at learned offsets | **DAMSDet** (ECCV 2024, "Multispectral Deformable Cross-attention ... adapt to misalignment situations"; [code](https://github.com/gjj45/DAMSDet)); **CF-Deformable DETR** (IJCAI 2024, cross-modal deformable attention plus a feature-consistency loss); WCCNet ("mitigate misalignment", [arXiv 2308.01042](https://arxiv.org/abs/2308.01042)) | **a few papers** (growing) |
| Explicit spatial calibration (adaptive affine transform of features) | **ADCNet**, Remote Sensing 15(19):4887, 2023: +3.3 mAP50 on FLIR over its baseline ([MDPI](https://www.mdpi.com/2072-4292/15/19/4887); code `Hiram1026/misaligned_RGB-IR_detection`, not checked). Which FLIR variant it uses is **unverified**. | **ad-hoc / a few** |
| Robustness test with **synthetic offsets** ("FLIR_w"): random horizontal and vertical shifts of the IR image in [−10, 10] px, RGB fixed. Performance "remains stable when the offset is less than 10 pixels" for their model, while strictly-aligned fusion methods "suffer from a significant performance drop". | CF-Deformable DETR ([IJCAI 2024 PDF](https://www.ijcai.org/proceedings/2024/0084.pdf); [repo](https://github.com/116508/CF-Deformable-DETR), README empty). The protocol follows AR-CNN's KAIST shifts (Zhang et al. ICCV 2019). | **ad-hoc** for FLIR (one paper); common on KAIST |
| Late fusion (box-level), immune to pixel offsets | ProbEn (on raw v1) | **a few papers** |
| Re-align better (cross-correlation scale and offset search) instead of filtering | Vadidar et al. | **ad-hoc** |

Related methods designed for weak misalignment but evaluated on *other* datasets (DroneVehicle/KAIST: AR-CNN, TSFADet, OAFA CVPR 2024, C²Former) are sometimes transferred to FLIR-aligned. I did not verify FLIR results for them.

---

## P6. Missing RGB frames

### Problem

499 train, 109 val and 29 video thermal frames have no RGB file (README; counted in 01).

### Impact

- Two-stream loaders crash, or worse, silently pair the wrong files.
- The val count changes from 1,366 to 1,258 (or 1,257), which shifts the metrics.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Drop thermal frames without RGB: "we remove 108 thermal images in the val-set that do not have the RGB counterparts", giving 1,258 val, 768 day / 490 night | ProbEn ([arXiv 2104.02904](https://arxiv.org/abs/2104.02904)), following [60] (= GAFF/CFR line); ProbEn ships `FLIR_thermal_RGBT_pairs_{train,val}.json` ([code](https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling/blob/main/demo/FLIR/demo_train_FLIR.py) L26–30) | **widely used** among raw-v1 fusion works |
| Use FLIR-aligned, where every pair exists by construction | All FLIR-aligned users | **widely used** |
| Count usable pairs: Vadidar list FLIR as 13,813 frames (≈ the 13,815 RGB files) | Vadidar et al. | **ad-hoc** |
| Ignore it in thermal-only work (no effect) | Thermal-only papers | n/a |

Note: the 108 vs 109 discrepancy (ProbEn says 108; README and file count say 109) is unresolved. One frame may also lack something else (**unverified**).

---

## P7. 8-bit vs 16-bit thermal

### Problem

- v1 ships `thermal_8_bit` (vendor AGC JPEG) and `thermal_16_bit` (raw 14-bit counts in 16-bit TIFF).
- **FLIR-aligned contains only the 8-bit frames** (`FLIR_xxxxx_PreviewData.jpeg`; 01 §4.2).
- Five 16-bit train frames I read (counted):

  | Frame | min | p1 | p50 | p99 | max | distinct levels |
  |---|---|---|---|---|---|---|
  | FLIR_00001 | 5,991 | 6,055 | 7,118 | 7,479 | 7,745 | 1,659 |
  | FLIR_00500 | 6,174 | 6,228 | 7,278 | 7,896 | 7,969 | 1,747 |
  | FLIR_02000 | 6,559 | 6,618 | 7,596 | 8,043 | 8,313 | 1,552 |
  | FLIR_05000 | 6,100 | 6,157 | 7,226 | 7,424 | 8,045 | 1,669 |
  | FLIR_08000 | 6,989 | 7,014 | 7,169 | 7,292 | 7,536 | 516 |

  The useful band is 278–1,668 counts wide and its offset drifts between frames. N-AGC describes the same thing as "low contrast and frame-to-frame intensity drift".
- The 8-bit AGC has the opposite problem. Its global tone curve "compress[es] mid-tone bandwidth", so "small, cool pedestrians or distant vehicles are rendered barely above noise" ([Javidnia, ICCVW 2025](https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html)).
- LYNRED-MDS: "The 16-bit images correspond to the minimally processed output", and tone mapping is "a relevant component of the embedded object detection pipeline".

### Impact

- `x/65535` or `x/255` applied to 16-bit data gives near-constant inputs. COCO-pretrained backbones expect 8-bit-like statistics.
- The AGC is per-frame and not reproducible, so the same object has different intensities across frames.
- Results from 8-bit and 16-bit inputs are not comparable.
- RGB-T fusion users cannot use 16-bit unless they map FLIR-aligned IDs (`FLIR_xxxxx`) back to the v1 TIFFs. That is possible because the IDs are preserved, but it is not done in any repo I checked.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Use the 8-bit AGC JPEG, replicated to 3 channels, with ImageNet/COCO normalization or `/255` | FLIR baseline ("8-bit thermal"), MMTOD, ProbEn (`train/thermal_8_bit/`, [L25](https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling/blob/main/demo/FLIR/demo_train_FLIR.py)), all FLIR-aligned repos | **widely used** (more than 95% of works, by inspection) |
| Dataset-specific thermal mean/std: `--thermal_mean 0.519 ×3 --thermal_std 0.225 ×3` | RGBXFusion ([README](https://github.com/dsriaditya999/RGBXFusion)); ProbEn uses an extra thermal `PIXEL_MEAN` channel of 135.438 for early fusion (L78–85) | **a few repos** |
| Invert the IR polarity and expand to 3 channels before an RGB detector ("inverting pixels for the IR and expanding it to three channels significantly enhance the initial performance") | HalluciDet / ModTr line | **a few papers** |
| 16-bit with per-frame percentile stretch, or a learned tone mapping (N-AGC U-Net trained to mimic vendor AGC, then fine-tuned with detection loss) | Javidnia ICCVW 2025 on "FLIR ADAS v1+v2" ([code](https://github.com/hosseinjavidnia/NAGC), not checked). Its gain (person AP50 75.3 → 90.1) is measured against the *official YOLOX-m baseline*, not a matched 8-bit run, so the effect of bit depth is not isolated. | **ad-hoc** |
| Framework support for 16-bit TIFF input (`Gray16bit`, `Multispectral_16bit`) | YOLOv11-RGBT (added 2025-11-22) | **ad-hoc** (tooling only) |

**Gap:** I found no controlled 8-bit vs 16-bit comparison on v1 with the same detector (**unverified absence**).

---

## P8. No official test split; inconsistent splits, leakage and evaluation drift

### Problem

- The README offers train/val as a *suggested* split and says "Test data was not used" (01 §1.1).
- FLIR-aligned defines train (4,129) and "test" (1,013, taken from v1 val), but gives **no val split**.
- Frames are sampled at 2 fps (1 fps in sparse scenes) from continuous drives, so neighbouring frames are near-duplicates.

**Observed problems:**
- **Random re-splits inflate results.**
  - [CFT #58](https://github.com/DocF/multispectral-object-detection/issues/58): reproduction "比论文高10%" ("10% higher than the paper"). A commenter guesses the cause: "FLIR数据集划分作者没指定，我自己按照9:1划分的" ("the authors did not specify the split; I split 9:1 myself").
  - [CFT #50](https://github.com/DocF/multispectral-object-detection/issues/50): Roboflow train/valid/test re-split gives **test mAP50 = 0.98**.
  - The Roboflow v1.3 mirror is itself re-split (01 §8.2).
  - That near-duplicates are the leak mechanism is my inference, but a strong one.
- **Repos disagree with their own papers.**
  - TINet README: "in our paper the results are given by a different train/test data distribution. If you intend to include our results, please make sure that the data distribution is aligned" ([TINet](https://github.com/NNNNerd/Triple-I-Net-TINet)).
  - ICAFusion's README strikes out the paper's FLIR mAP50 79.2 and replaces it with 82.8 ("We fixed some bugs"). The released weights give 0.828 ([#18](https://github.com/chanchanchan97/ICAFusion/issues/18)). Users without COCO-pretrained weights get about 0.74–0.75 ([#44](https://github.com/chanchanchan97/ICAFusion/issues/44), [#56](https://github.com/chanchanchan97/ICAFusion/issues/56)).
- **Model selection on the test set.** ICAFusion #56: "我是直接把测试集作为验证集，因为有些数据集没有测试集" ("I just use the test set as the validation set, because some datasets have no test set"). In YOLOv5 repos `val` = FLIR-aligned test, and `best.pt` is chosen on it by default (**inference** from the CFT/ICAFusion yaml: `val_rgb/val_ir` = test).
- **Evaluation-setting drift.** DocF (CFT) attributes reproduction gaps to the `conf-thres` / `iou-thres` used at test time ([CFT #23](https://github.com/DocF/multispectral-object-detection/issues/23)). There is also rectangular vs square inference ([CFT #45](https://github.com/DocF/multispectral-object-detection/issues/45)) and YOLOv5's mAP implementation vs pycocotools (RSDet uses `CocoMetric`).
- **Protocol variants in top venues:**
  - ProbEn val 1,258 (RGB-paired);
  - MMTOD's FLIR, FLIR-1/2 and FLIR-1/4 train subsets, with one shared `test.txt`;
  - HalluciDet and ModTr: "80% of the training set for training and the rest for validation", test = FLIR-aligned test;
  - D3T: disjoint 2,064 RGB + 2,064 thermal training images (RGB→thermal domain adaptation), with "NEW ImageSets";
  - RGBXFusion and TINet: their own day/night splits (TINet hard-codes day/night index ranges);
  - Vadidar et al.: their own 3-way split;
  - PearlGAN: `img_list` txt splits (day-RGB vs night-IR).
- **Cross-derivative overlap.** RoadScene's 221 frames are **176 from v1 train, 28 from v1 val and 17 from v1 video** (counted from file names in [hanna-xu/RoadScene](https://github.com/hanna-xu/RoadScene) `cropinfrared/`). Fusion-then-detect pipelines that train a fusion net on RoadScene and test detection on FLIR-aligned test (drawn from v1 val) may overlap. The exact overlap with `align_validation.txt` was **not checked**.

### Impact

Differences of 2–3 mAP50 points between papers are within protocol noise:
- split choice;
- test-set model selection;
- COCO pretraining;
- conf/iou thresholds;
- YOLO vs COCO metric code.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Use FLIR-aligned train/test exactly as shipped (`align_train.txt` / `align_validation.txt`) | CFT, ICAFusion, DAMSDet, RSDet, MS2Fusion, TINet's README results, UniRGB-IR (4,129 / 1,013) ... | **widely used** |
| Use official v1 train/val (8,862 / 1,366) for thermal-only work; `video` as an extra test | MMTOD, SSTN, DDSSD, the FLIR baseline | **widely used** (raw v1) |
| Carve a val split out of train (random 80/20) and keep test untouched | HalluciDet, ModTr (5 seeds) | **a few papers** (random within train, so val is still optimistic) |
| Report mean ± std over seeds | HalluciDet / ModTr / MiPa (5 seeds) | **a few papers** |
| Publish split files and converted annotations | MMTOD (Drive `trainval.txt` / `test.txt`), TINet (COCO json), UniRGB-IR (`Annotation_train/test.json`), D3T (ImageSets), ProbEn (RGBT-pair json), M-SpecGene (`COCO_FLIR`) | **widely used** (but they differ from each other) |
| Release day/night tags to enable breakdowns | ProbEn (768 / 490 val), TINet (index ranges), RGBXFusion | **a few papers** |
| COCO-style evaluation with pycocotools (mAP, mAP50, mAP75) rather than the YOLO internal metric | RSDet (mmdet `CocoMetric`), DAMSDet, CGDet | **a few / growing** |
| Sequence-aware (drive-level) splitting to avoid near-duplicates | None found for v1; DAMSDet splits M3FD "by different scenes" for this reason | **not established for FLIR** (gap) |

---

## P9. Licensing and dead links

### Problem

Detailed in 01 §8.3–8.4:
- The FLIR ADAS Image License (26 Jul 2018) allows non-commercial research and education only. It gives **no right to "redistribute or make derivative works"**, and derivative works "shall belong to" FLIR.
- The official v1 form URL returns 404, and the current page serves v2 only.

Community evidence:
- [CFT #63](https://github.com/DocF/multispectral-object-detection/issues/63) (2023): "FLIR Google Drive Link points to random eBay listing". Users fell back to the original Drive file `1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y` and CalayZhou's resource list.
- The CFR short link `shorturl.at/ahAY4` returns 403 to scripts.
- The `zonaqiu/FLIR-align` repo no longer resolves (02).
- Many repos host FLIR-aligned on Baidu Pan (ICAFusion, MS2Fusion, CFT), with passwords that change between README revisions.

### Impact

- Reproducibility depends on unofficial mirrors.
- There is no checksum, so different users may train on different copies. Converted label counts already differ slightly (P1).
- Commercial use is excluded.

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Download from Kaggle `deepnewbie/flir-thermal-images-dataset` (full v1.3, includes the license) | Most raw-v1 users since 2020 (about 15k downloads) | **widely used** (unofficial) |
| FLIR-aligned from the original Drive / HF `jsonhash/FLIR_aligned` / Baidu (CFT, ICAFusion) / Kaggle `s111mple/flir-aligned` | All FLIR-aligned users | **widely used** (unofficial) |
| Do not redistribute images; release only annotations or tags | ProbEn ("We will release our annotations"; images not redistributed, 01 §8.3) | **ad-hoc** |
| Cite the Wayback snapshot of the official page | 01 [O1] | **ad-hoc** |

Practical advice: record the SHA-256 of the archive you used, plus the image and box counts per split, in your paper or repo (my recommendation, not observed practice).

---

## P10. Small size and domain limits (one region)

### Problem

- About 10k annotated train+val frames (FLIR-aligned: about 5k pairs).
- Captured in **one region**: streets and highways around Santa Barbara, CA, from November to May.
- About 60/40 day/night, "generally clear-sky", with no rain, fog or snow subsets (01 §1.2).
- LYNRED-MDS: "images acquired mostly in favorable weather conditions, where neither the visible nor the infrared sensors are significantly challenged" ([arXiv 2607.01871](https://arxiv.org/abs/2607.01871)).
- One camera pair (Tau2 640×512, 45° HFOV).

### Impact

- Over-fitting: mAP50 saturates around 80–85 on FLIR-aligned.
- Weak transfer to other thermal sensors and regions. I found no measured zero-shot FLIR→KAIST/CVC drop (**unverified**). The domain-adaptation literature treats FLIR, KAIST and CVC-14 as separate domains.
- Night and day results depend on a small night subset (490 val frames in ProbEn's tags).

### Workarounds

| Workaround | Who | How established |
|---|---|---|
| Report on several datasets: **FLIR-aligned + LLVIP + M3FD** (+ KAIST / DroneVehicle / VEDAI) | DAMSDet, Fusion-Mamba, RSDet, WaveMamba, MS2Fusion, IRDFusion, ICAFusion ... (02 §2.3) | **widely used** |
| COCO pretraining, or modality translation to reuse RGB detectors (pseudo-RGB) | MMTOD, HalluciDet, ModTr, TIRDet, PFGF | **widely used** |
| Multi-dataset or foundation pretraining (MMPD mix; RGBT550K) | MMPedestron (ECCV 2024), M-SpecGene (ICCV 2025) | **a few papers** |
| Domain adaptation RGB→thermal or COCO→FLIR | SGADA, SSTN, D3T | **a few papers** |
| Data-efficiency subsets to study the small-data regime | MMTOD FLIR-1/2, FLIR-1/4 | **ad-hoc** |
| Move to v2 (26k frames; USA, England, France; 15+ classes) or combine v1+v2 | HOTCOLD (AAAI 2023) "we use v2.0 instead of v1.0"; N-AGC uses v1+v2 | **a few papers** (thermal-only); RGB-T stays on v1-aligned |
| Day/night scene-specific models | RGBXFusion (WACV 2024), TINet | **a few papers** |

---

## P11. Version and derivative confusion (v1 vs v2, raw vs aligned)

### Problem

Papers often describe the dataset wrongly:
- CGDet (2025) says FLIR "includes 15 object categories" (that is v2) and calls FLIR-aligned "manually corrected to improve registration" ([arXiv 2511.01435](https://arxiv.org/abs/2511.01435)). It was filtered, not corrected.
- Fusion-Mamba says "five categories".
- D3T calls FLIR-aligned "the updated FLIR dataset".
- Several papers mix up per-split counts (01 §1.4, e.g. arXiv 2108.11101).

### Impact

Results get attributed to the wrong dataset version, and comparisons across v1, v2 and aligned become invalid.

### Workarounds

- State the variant, the image counts (8,862 / 1,366; 4,129 / 1,013) and the classes explicitly.
- Cite Zhang et al. ICIP 2020 for FLIR-aligned.

This is done in CFT (§4: "the FLIR that appears later all refer to its 'aligned' version"), DAMSDet and MiPa. **Widely used**, but not universal.

---

## Recommended practice checklist

**Data and version**
- [ ] State exactly which data you use: v1.3 raw (train 8,862 / val 1,366 / video 4,224), FLIR-aligned (4,129 / 1,013), RoadScene (221) or a custom subset. Never just "FLIR".
- [ ] Record the archive source (Kaggle, Drive, HF, Baidu), its SHA-256, and per-split image and box counts after conversion. Check them against: FLIR-aligned test 4,107 person / 4,124 car / 360 bicycle (+13 dog); v1 val 5,779 / 5,432 / 471 (+14 dog).
- [ ] Read and respect the FLIR license: non-commercial use, no redistribution of images. Release annotations, splits and tags, not images.

**Classes and labels**
- [ ] Keep **person, car, bicycle**, drop dog (and the empty "Other Vehicle"), unless you say otherwise and explain why.
- [ ] Fix and publish the class-index order. Assert after conversion that `nc = 3` and that no dog boxes leaked in from the FLIR-aligned XMLs.
- [ ] Remember that "bicycle" includes motorcycles, and report per-class AP. Bicycle has only 360 test instances in FLIR-aligned.
- [ ] Visualise a sample of the converted labels on both modalities before training.
- [ ] Do not call FLIR-aligned "sanitized": it was filtered for alignment, not relabelled.

**Splits and evaluation**
- [ ] Never re-split FLIR at random across train and test: 2 fps sequences leak. Use the shipped splits.
- [ ] Hold out a val split carved from **train** (preferably by contiguous ID blocks, as a proxy for drive-level splitting) and do **not** pick checkpoints on the FLIR-aligned test set. If you do, say so.
- [ ] Report mAP50, mAP75 and mAP[.5:.95] with pycocotools (or state the metric code), plus input size, pretraining (COCO or not), conf/NMS thresholds at evaluation, and mean ± std over at least 3 seeds.
- [ ] For raw-v1 RGB-T work, drop the frames without RGB (val → 1,258) and say so. Day/night tags: reuse ProbEn's when available.
- [ ] If you use RoadScene for fusion training and FLIR-aligned for detection testing, check for frame overlap (RoadScene includes 28 v1-val and 17 video frames).

**Modalities and alignment**
- [ ] Do not use raw v1 RGB with thermal boxes as if they were registered. Either use FLIR-aligned, a documented homography (and release it), or a late or implicit-alignment fusion design.
- [ ] Assume residual misalignment in FLIR-aligned. Prefer feature-level, deformable or offset-aware fusion over pixel-level early fusion. Consider reporting robustness to synthetic IR shifts of ±5 / ±10 px (the CF-Deformable DETR protocol).
- [ ] Evaluate against the thermal-coordinate ground truth and say so.

**Bit depth and preprocessing**
- [ ] Default to 8-bit AGC for comparability, and say so. If you use 16-bit, apply per-frame (or dataset) percentile normalization, never `/65535`, and also report an 8-bit baseline with the same detector.
- [ ] State the thermal normalization (mean/std or `/255`), the channel replication and any polarity inversion.

**Generalisation**
- [ ] Do not draw conclusions from FLIR alone. Add LLVIP and M3FD (RGB-T) or v2/KAIST (thermal), and mention the Santa Barbara, clear-weather limitation.

---

## Open uncertainties

1. Whether any curated or corrected FLIR-aligned annotation exists in a non-English venue or a private repo. None was found on GitHub, HF, Kaggle or arXiv (search on 2026-11-02).
2. The cause of the train/val size-convention difference (P1), and whether train and val come from disjoint drives.
3. The actual residual offset distribution in FLIR-aligned. Nobody has measured it.
4. Whether ADCNet's "FLIR" is FLIR-aligned, and Vadidar et al.'s final split numbers (6,924 / 1,982 / 3,960 in the arXiv text I read vs 7,381 / 1,056 / 2,111 cited in 02).
5. The exact overlap between RoadScene and the FLIR-aligned test list.
6. ProbEn's 108 vs the README's 109 val frames without RGB.
7. My 16-bit statistics come from 5 frames only. They are indicative, not dataset-wide.
8. I could not inspect the DAMSDet, WaveMamba, PFGF or M2D-LIF loaders (README empty or unreachable; GitHub API rate limit).
9. I found no Kaggle or Roboflow discussion threads about v1 label quality. The GitHub issues above are the main community evidence.

---

## References (main)

**Dataset and derivatives**
- FLIR ADAS v1.3 README/JSON/license, via Kaggle `deepnewbie/flir-thermal-images-dataset`: https://www.kaggle.com/datasets/deepnewbie/flir-thermal-images-dataset
- FLIR 2018 press release: https://www.flir.com/news-center/press-releases/flir-releases-starter-thermal-imaging-dataset-for-machine-learning-advanced-driver-assistance-development/
- FLIR-aligned (CFR, ICIP 2020): https://arxiv.org/abs/2009.12664. Drive: https://drive.google.com/file/d/1xHDMGl6HJZwtarNWkEV3T4O9X4ZQYz2Y/view
- RoadScene: https://github.com/hanna-xu/RoadScene

**Papers (dataset handling)**
- ProbEn (ECCV 2022): https://arxiv.org/abs/2104.02904. Code: https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling
- Vadidar et al., Robust Environment Perception (IEEE IV 2022): https://arxiv.org/abs/2206.03943
- Agrawal et al., Enhancing Object Detection in Adverse Conditions (2019): https://arxiv.org/abs/1909.13551
- GEM: https://arxiv.org/abs/2102.12319
- MMTOD (CVPRW 2019): https://openaccess.thecvf.com/content_CVPRW_2019/html/PBVS/Devaguptapu_Borrow_From_Anywhere_Pseudo_Multi-Modal_Object_Detection_in_Thermal_Imagery_CVPRW_2019_paper.html. Code: https://github.com/tdchaitanya/MMTOD
- DAMSDet (ECCV 2024): https://arxiv.org/abs/2403.00326
- CF-Deformable DETR (IJCAI 2024): https://www.ijcai.org/proceedings/2024/0084.pdf
- ADCNet (Remote Sensing 2023): https://www.mdpi.com/2072-4292/15/19/4887
- D3T (CVPR 2024): https://arxiv.org/abs/2403.09359. Code: https://github.com/EdwardDo69/D3T
- HalluciDet (WACV 2024): https://arxiv.org/abs/2310.04662
- ModTr (ECCV 2024): https://arxiv.org/abs/2404.01492
- MiPa (WACV 2025): https://arxiv.org/abs/2404.18849
- CFT: https://arxiv.org/abs/2111.00273
- WCCNet: https://arxiv.org/abs/2308.01042
- CGDet: https://arxiv.org/abs/2511.01435
- LYNRED-MDS: https://arxiv.org/abs/2607.01871
- N-AGC (Javidnia, ICCVW 2025): https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html

**Repos and configs**
- CFT: https://github.com/DocF/multispectral-object-detection (`data/multispectral/FLIR_aligned.yaml`). Issues #23, #34, #45, #50, #58, #63.
- ICAFusion: https://github.com/chanchanchan97/ICAFusion (`data/multispectral/FLIR-align-3class.yaml`, `FLIR-align.yaml`, README "Results update"). Issues #18, #44, #56.
- RSDet: https://github.com/Zhao-Tian-yi/RSDet/blob/master/configs/_base_/datasets/FLIR.py
- MMPedestron: https://github.com/BubblyYi/MMPedestron/blob/main/configs/mmpedestron/flir_exp/mmpedestron_flir_2x.py
- YOLOv11-RGBT: https://github.com/wandahangFY/YOLOv11-RGBT
- TINet: https://github.com/NNNNerd/Triple-I-Net-TINet
- RGBXFusion: https://github.com/dsriaditya999/RGBXFusion
- SGADA: https://github.com/avaapm/SGADA
- PearlGAN: https://github.com/FuyaLuo/PearlGAN
- UniRGB-IR: https://github.com/PoTsui99/UniRGB-IR
- M-SpecGene: https://github.com/CalayZhou/M-SpecGene
