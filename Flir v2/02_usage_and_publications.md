# FLIR ADAS v2: recent usage (Q2) and papers in major venues (Q3)

**Research date:** 2026-11-02
**Dataset:** Teledyne FLIR "FREE ADAS Thermal Dataset v2" (v2.0.0, released 19 Jan 2022). Specs: 26,442 annotated frames (9,711 thermal stills, 9,233 RGB stills, 7,498 matched RGB+thermal video frames), about 520k boxes, 15 classes. Homepage: `adas-dataset-v2.flirconservator.com`. Catalogue DOI: 10.57702/3yajgd26 ([TIB LDM](https://service.tib.eu/ldmservice/dataset/teledyne-flir-free-adas-thermal-datasets-v2)).
**Group:** B (see `../PLAN.md`)

---

## TL;DR

- **v2 use is small but growing, and v1 still leads.** Some top-venue papers use v2 directly. However, most RGB-T fusion detection papers, including CVPR, ECCV, ICCV, AAAI and TCSVT papers, still use **FLIR-aligned**. FLIR-aligned is Zhang et al.'s 2020 cleaned subset of **v1**: 5,142 pairs, 4,129 train and 1,013 test, 3 classes. In my full-text check of arXiv papers since 2022 that mention FLIR in the abstract, aligned-v1 papers outnumber explicit-v2 papers by about **3 to 1** (about 20 vs about 7).
- **Trend.** Papers with "FLIR ADAS" in the title or abstract (Semantic Scholar): 6 (2022), 7 (2023), 7 (2024), 21 (2025), 26 (2026 so far). The broader OpenAlex search, which includes full text where available, finds 30, 54, 56, 81 and 97. Papers that explicitly write "FLIR ADAS v2" are a small slice: OpenAlex gives 1, 1, 6, 6, 9 for 2022–2026.
- **Compared with other RGB-T datasets.** From 2023 on, LLVIP and M3FD grow much faster than FLIR. For 2026, OpenAlex counts are LLVIP 324, M3FD 244, "FLIR ADAS" 97 and "KAIST multispectral" 27. FLIR has **no citable paper**, so its use is undercounted; papers cite only a URL. Many papers that cite the v2 URL actually use aligned v1, because FLIR's old v1 page now redirects to the v2 form.
- **Downloads.** The main Kaggle mirror of v2 (`samdazel`) has **12,434 downloads and 17,299 views**. The v1 mirror (`deepnewbie`, since 2020) has 14,958. On Hugging Face, FLIR_aligned (v1) gets about 1.1k downloads a month, while the only v2-video mirror gets about 80.
- **Q3 table.** I found **11 papers in major venues where v2 use is confirmed or very likely**: CVPR ×2, AAAI, ACM MM, IEEE RA-L, ICIP, IJCNN, CVPR/ICCV workshops ×4. 5 of the 11 have their own public code repo. There are also **10 major-venue papers where the version is ambiguous**, and **13 papers in lower-tier venues or preprints** that use v2. I did **not** reach 20 confirmed major-venue v2 papers, and I don't think that many exist.
- **What v2 is used for.** Mostly **thermal-only** tasks: detection, adversarial and backdoor attacks on IR detectors, 14/16-bit tone mapping, RGB→thermal generation, and open-vocabulary/zero-shot detection. It is rarely used for paired RGB-T fusion, because the v2 pairs are not registered (Group A/E topic).

---

## 1. Methodology and limits

| Source | What I did | Limits |
|---|---|---|
| Semantic Scholar Graph API, `/paper/search/bulk` | Phrase queries over titles and abstracts: `"FLIR ADAS"`, `FLIR +(thermal\|infrared) +dataset`, `"LLVIP"`, `"M3FD"`, `"KAIST" +(multispectral\|thermal)`. Counted per year. | Titles and abstracts only. Most papers name the dataset only in the experiments section, so these are **lower bounds**. The full-text `snippet/search` endpoint returned HTTP 429 every time (no API key). |
| OpenAlex `/works?search=` | Same phrases plus `"FLIR ADAS v2"`, `"FLIR v2"`, `"FLIR-aligned"`, filtered by year. | OpenAlex searches title, abstract and full text where it has full text, so recall is uneven. The free daily budget ran out partway through. 2026 covers Jan–Oct. Relevance ranking can include loose matches. |
| arXiv API + full-text grep | Pulled every arXiv paper with FLIR AND (thermal OR infrared) in the abstract (72 total, 52 since 2022). Downloaded the PDFs and grepped for version markers: `v2`, `26,442`, `10,742`, `9,711`, `adas-dataset-v2` vs `5,142/4,129/1,013/aligned/Zhang 2020` vs `v1_3/10,228/2018`. Read the context by hand. | Small sample (abstract mention required). Gives the v1:v2 ratio, not totals. |
| Google Scholar | Not usable: programmatic access was blocked as bot-check circumvention. | Manual queries for the user: `"FLIR ADAS v2"`, `"FLIR v2" thermal`, `"26,442" FLIR`, `"Teledyne FLIR" "v2" thermal dataset`. |
| Papers with Code | **Closed**: `paperswithcode.com/dataset/flir` now redirects to HF "trending". The 2025-07-28 archive snapshot ([pwc-archive/datasets](https://huggingface.co/datasets/pwc-archive/datasets)) has **no FLIR entry**. For comparison it lists `num_papers`: LLVIP 116, KAIST Multispectral 23, DroneVehicle 18. | No PwC count is available for FLIR v1 or v2. |
| GitHub search API (repos) | Searched repo names and descriptions only. Code search needs authentication. | Undercounts repos that use FLIR internally. |
| Kaggle API / HF API | Download, view and vote counts for the mirrors. | Kaggle counts are cumulative. HF `downloads` covers the last 30 days. Mirrors are unofficial. The official download (Conservator / flir.com form) publishes no counts. |
| Venue and version check | Read each candidate in its arXiv or CVF PDF and recorded the version, image counts and classes it states. | Some IEEE/Wiley/ScienceDirect papers were paywalled (HTTP 403), so their version comes from abstracts or snippets and is marked as such. |

**How I decided the version** (the papers usually just say "FLIR"):
- **v2**: the paper says v2 or v2.0, or gives 26,442 / 9,711 / 10,742 / 3,749 / 15 classes, or cites `adas-dataset-v2`, or is dated 2022 or later and uses 14-bit RAW plus the 15 classes.
- **aligned (v1)**: 5,142 pairs, or 4,129/1,013 pairs, or 3 classes (person/car/bicycle), or cites Zhang et al. ICIP 2020.
- **v1**: says v1_3, 10,228 frames or 8,862 train, or the 2018 citation.
- Everything else is **(ambiguous v1/v2)**.

Warning: **citing the v2 URL does not mean using v2.** MSCoTDet (TCSVT) and ThermEval cite the v2 page but use the aligned v1 pairs.

---

## 2. Q2: How much is it used? (trends 2022–2026)

### 2.1 Papers per year

| Query (source) | 2018 | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026* |
|---|---|---|---|---|---|---|---|---|---|
| `"FLIR ADAS"` title/abstract (Semantic Scholar bulk; total 76) | 0 | 2 | 2 | 5 | 6 | 7 | 7 | 21 | 26 |
| `FLIR +(thermal\|infrared) +dataset` title/abstract (S2; total 437) | 4 | 10 | 17 | 25 | 37 | 47 | 68 | 104 | 100 |
| `"FLIR ADAS"` (OpenAlex, incl. full text where available) | 0 | 4 | 10 | 31 | 30 | 54 | 56 | 81 | 97 |
| `"FLIR ADAS v2"` (OpenAlex) | – | – | – | – | 1 | 1 | 6 | 6 | 9 |
| `"FLIR v2"` (OpenAlex) | – | – | – | – | 0 | 2 | 4 | 2 | 4 |
| `"FLIR-aligned"` (OpenAlex), i.e. **v1-derived** | – | – | – | 0 | 0 | 11 | 12 | 11 | 32 |
| `"Teledyne FLIR" dataset` (OpenAlex), name used after 2021 | – | – | 0 | 21 | 74 | 137 | 155 | 166 | 200 |
| arXiv papers with FLIR + thermal/IR in abstract | 1 | 0 | 4 | 8 | 5 | 7 | 8 | 8 | 24 |

*2026 runs to 2026-11-02.

How to read this:
- Use of FLIR in general (v1 + v2) roughly **tripled between 2023 and 2026**, with a jump in 2025–26. This is mostly driven by YOLO-style IR detectors in MDPI, IEEE Access and conference papers, and by VLM/thermal-foundation-model work.
- Explicit "v2" mentions grow, but they stay at about 5–10% of FLIR mentions. Most authors do not name the version. From the counts in §2.2 and my manual checks, a reasonable estimate is that **roughly 25–35% of recent FLIR-using papers are on v2** (mostly thermal-only work), and the rest use aligned v1 or are unclear.
- **Full-text check of the arXiv sample (2022–2026, 52 papers, FLIR in abstract):**
  - **explicit v2: about 7.** Examples: 2204.08745, 2305.00691, 2507.12017, 2509.06333, 2509.21696, 2603.24327, 2610.33118.
  - **FLIR-aligned (v1): about 20.** Examples: TFDet, D3T, CMM, ModPrompt, FD2-Net, IC-Fusion, WiSE-OD, LYNRED-MDS, InfraNet, CFGPNet, A2DINOv3, WD-FQDet, 2610.41204 and others.
  - The rest are ambiguous or unrelated (FLIR as a camera brand, e.g. the FLIR-IISR / FLIR-IVSR super-resolution datasets shot with a FLIR T1050sc).

### 2.2 Comparison with v1 and with other RGB-T datasets

| Dataset | S2 title/abstract hits 2022 / 23 / 24 / 25 / 26 | OpenAlex hits 2022 / 23 / 24 / 25 / 26 | Citable paper (S2 citations) | GitHub repos (name/desc), created 2026 | Kaggle top mirror downloads | HF top mirror (30-day downloads) |
|---|---|---|---|---|---|---|
| **FLIR ADAS (v1+v2)** | 6 / 7 / 7 / 21 / 26 | 30 / 54 / 56 / 81 / 97 | **none** (URL only) | 25 total, 13 in 2026 (`FLIR ADAS`) | v2: 12,434 (`samdazel/teledyne-flir-adas-thermal-dataset-v2`); v1: 14,958 (`deepnewbie/flir-thermal-images-dataset`) | v1-aligned `jsonhash/FLIR_aligned` 1,084; v2 video `Maxim37/flir_adas_video_pairs` 83 |
| LLVIP (2021) | 5 / 26 / 42 / 102 / 143 | 14 / 77 / 105 / 222 / 324 | ICCVW 2021 paper; PwC archive `num_papers`=116 | 55 total, 33 in 2026 | 1,547 (`afradhossain/llvip-dataset`) | `jsonhash/LLVIP` 3,965 |
| M3FD (TarDAL, CVPR 2022) | 0 / 13 / 26 / 63 / 131 | 2 / 31 / 60 / 123 / 244 | TarDAL: 1,391 | 14 total, 9 in 2026 | 999 (`nus1998/m3fd-dataset`) | `Frencis/M3FD_RGBT` 1,436 |
| KAIST Multispectral (CVPR 2015) | 20 / 35 / 23 / 33 / 43 | 21 / 23 / 28 / 23 / 27 | 1,163 | 16 total, 5 in 2026 | small mirrors only | `koifisharriet/KAIST-…` 5,806 |

Sources: Semantic Scholar bulk-search API, OpenAlex API, Kaggle API (`/api/v1/datasets/view`), HF API, GitHub search API. All were queried on 2026-11-02.

Takeaways:
1. **FLIR v2 vs v1.** On Kaggle, v2 has almost caught up with the older v1 mirror in downloads: 12.4k vs 15.0k, and v1 had about 2 extra years. In papers, though, **v1 (as FLIR-aligned) stays the default benchmark for RGB-T fusion detection**, because it is registered and has fixed 4,129/1,013 splits that make results comparable. v2 is preferred for **thermal-only** work because it has more images, 15 classes, 14/16-bit RAW and video with track IDs.
2. **Against LLVIP and M3FD.** Both overtook FLIR in 2023. In 2025–26 they appear about 2–3× more often in abstracts. They are registered, they have citable papers, and they are the standard pair for image fusion and fusion detection.
3. **Against KAIST.** KAIST is flat at about 20–40 papers a year. FLIR (v1+v2) has passed it in abstract mentions since 2025.
4. **GitHub.** Repos with "FLIR ADAS" in the name or description jumped in 2026: 13 of 25. Most are student projects (YOLOv8 tutorials, RGB-T fusion homework) and several are spam-like Chinese "dataset + YOLOv8" repos. Some useful v2 tooling repos are listed in §4.

---

## 3. Q3: Papers in major venues

### 3.1 Table A: major venues, v2 confirmed or very likely (sorted by year)

Variant codes: **official** = official v2 splits. **custom subset** = filtered or re-split by the authors.

| # | Year | Title | Venue | Task | FLIR variant (evidence) | Code |
|---|---|---|---|---|---|---|
| A1 | 2022 | Augmentation of Atmospheric Turbulence Effects on Thermal Adapted Object Detection Models (Uzun et al.) | CVPR 2022 Workshops (PBVS) | Thermal detection, augmentation, RGB→thermal domain adaptation | **v2 official** thermal stills. PDF: "FLIR-ADAS v2 … 26,442 images, 15 classes … 9,711 thermal and 9,233 RGB". [CVF](https://openaccess.thecvf.com/content/CVPR2022W/PBVS/papers/Uzun_Augmentation_of_Atmospheric_Turbulence_Effects_on_Thermal_Adapted_Object_Detection_CVPRW_2022_paper.pdf), [arXiv 2204.08745](https://arxiv.org/abs/2204.08745) | none found |
| A2 | 2023 | HOTCOLD Block: Fooling Thermal Infrared Detectors with a Novel Wearable Design (Wei et al.) | AAAI 2023 | Adversarial attack on IR person detection | **v2.0 custom subset.** "Since Teledyne FLIR released an updated version in Jan 2022, we use v2.0". Person boxes >120 px: 1,255 imgs (878 train / 377 test). [arXiv 2212.05709](https://arxiv.org/abs/2212.05709), [AAAI](https://ojs.aaai.org/index.php/AAAI/article/view/26777) | [weihui1308/HOTCOLDBlock](https://github.com/weihui1308/HOTCOLDBlock) |
| A3 | 2023 | IR Reasoner: Real-Time Infrared Object Detection by Visual Reasoning (Gündoğan et al.) | CVPR 2023 Workshops (PBVS) | Thermal detection (YOLOv4/YOLOR/YOLOv7 + self-attention) | **v2 official splits, 14-bit RAW.** Classes kept: person, car, sign, light, bike (others dropped as too rare, e.g. dog). [CVF](https://openaccess.thecvf.com/content/CVPR2023W/PBVS/html/Gundogan_IR_Reasoner_Real-Time_Infrared_Object_Detection_by_Visual_Reasoning_CVPRW_2023_paper.html) | [tlgksy/ir-reasoner](https://github.com/tlgksy/ir-reasoner) |
| A4 | 2024 | Physical Backdoor: Towards Temperature-based Backdoor Attacks in the Physical World (Yin et al.) | CVPR 2024 | Backdoor attack on IR detection (YOLOv5/v3, Faster R-CNN) | **v2 custom subset** "Flir_v2_T": thermal images only, 13,460 images, results on person + car. [arXiv 2404.19417](https://arxiv.org/abs/2404.19417) | none found |
| A5 | 2024 | Thermal Chameleon: Task-Adaptive Tone-mapping for Radiometric Thermal-Infrared Images (Lee et al.) | IEEE RA-L 2024 | Tone mapping of RAW thermal for detection (and depth) | **v2** ("we used the FLIR-ADAS v2 dataset for object detection"): 10,724 train / 2,151 test. [arXiv 2410.18340](https://arxiv.org/abs/2410.18340) | [donkeymouse/ThermalChameleon](https://github.com/donkeymouse/ThermalChameleon) |
| A6 | 2024 | Data Generation Scheme for Thermal Modality with Edge-Guided Adversarial Conditional Diffusion Model (ECDM) | ACM MM 2024 | RGB→thermal translation / data generation for detection | **v2.0** ("We utilize v2.0"). 5 classes: person, bike, car, light, sign. Also uses LLVIP and PRW. [arXiv 2408.03748](https://arxiv.org/abs/2408.03748) | [lengmo1996/ECDM](https://github.com/lengmo1996/ECDM) |
| A7 | 2024 | Defending Against Physical Adversarial Patch Attacks on Infrared Human Detection (Strack, Waseda et al.) | ICIP 2024 | Adversarial defense, IR person detection | **Likely v2 custom subset.** Says "Following Wei et al. [HOTCOLD] we filtered the original dataset for human detection". Teledyne FLIR naming. Version not stated. [arXiv 2309.15519](https://arxiv.org/abs/2309.15519) | none found |
| A8 | 2025 | MS-YOLO: Infrared Object Detection for Edge Deployment via MobileNetV4 and SlideLoss | IJCNN 2025 | Thermal detection (lightweight YOLOv8) | **v2 custom subset.** 9 classes (person, bike, car, motor, bus, truck, light, hydrant, sign). 15,094 imgs: 10,474 train / 3,493 test / 1,127 val. [arXiv 2509.21696](https://arxiv.org/abs/2509.21696) | none (uses ultralytics) |
| A9 | 2025 | Task-Driven Neural Adaptive Gain Control: 16-bit-to-8-bit Thermal Tone Mapping for Superior Object Detection (Javidnia) | ICCV 2025 Workshops (MIRA) | Tone mapping for detection (YOLOX, Faster R-CNN) | **v1 + v2** 16-bit frames ("FLIR ADAS v1+v2 benchmark"); person and car AP50. [CVF](https://openaccess.thecvf.com/content/ICCV2025W/MIRA/html/Javidnia_Task-Driven_Neural_Adaptive_Gain_Control_16-bit-to-8-bit_Thermal_Tone_Mapping_for_ICCVW_2025_paper.html) | [hosseinjavidnia/NAGC](https://github.com/hosseinjavidnia/NAGC) |
| A10 | 2026 | Thermal-Det: Language-Guided Cross-Modal Distillation for Open-Vocabulary Thermal Object Detection (Ranasinghe et al.) | CVPR 2026 | Open-vocabulary / zero-shot thermal detection | **v2 ("FLIR-V2") and FLIR-Aligned** as separate benchmarks. Table 1 zero-shot AP on both. [arXiv 2605.10130](https://arxiv.org/abs/2605.10130), [CVPR page](https://cvpr.thecvf.com/virtual/2026/poster/39220) | not released (as of 2026-11) |
| A11 | 2026 | Le MuMo JEPA: Multi-Modal Self-Supervised Representation Learning with Learnable Fusion Tokens | CVPR 2026 Workshop | Self-supervised RGB-T representation, transfer to detection | **v2** (cites "Teledyne FLIR free ADAS thermal dataset v2", `adas-dataset-v2…`), used for RGB-thermal training and transfer. [arXiv 2603.24327](https://arxiv.org/abs/2603.24327) | none found |

**Summary counts for Table A (11 papers)**
- **By venue:**
  - CVPR main ×2 (A4, A10)
  - CVPR Workshops ×3 (A1, A3, A11)
  - ICCV Workshops ×1 (A9)
  - AAAI ×1 (A2)
  - ACM MM ×1 (A6)
  - IEEE RA-L ×1 (A5)
  - ICIP ×1 (A7)
  - IJCNN ×1 (A8)
- **By task:**
  - thermal object detection ×3 (A1, A3, A8)
  - adversarial / backdoor security ×3 (A2, A4, A7)
  - tone mapping for detection ×2 (A5, A9)
  - RGB→thermal generation ×1 (A6)
  - open-vocabulary detection ×1 (A10)
  - self-supervised RGB-T ×1 (A11)
  - **RGB-T fusion detection on v2: 0**
- **By variant:**
  - official splits ×3 (A1, A3, A5)
  - custom subset ×5 (A2, A4, A6, A7, A8)
  - v1 + v2 ×1 (A9)
  - v2 + aligned-v1 ×1 (A10)
  - unclear subset ×1 (A11)
- **Own public code:** 5/11 (A2, A3, A5, A6, A9). A8 uses stock ultralytics only. A10 has not released code.

### 3.2 Table B: major venues, version ambiguous (v1/v2)

These papers name only "FLIR ADAS" or "FLIR" and give counts that fit neither release cleanly. Later agents should check them before using their numbers.

| Year | Title | Venue | Task | Why ambiguous | Code |
|---|---|---|---|---|---|
| 2023 | Physically Adversarial Infrared Patches with Learnable Shapes and Locations (Wei, Yu, Huang) | CVPR 2023 | Adversarial attack on IR pedestrian detection | "FLIR ADAS", 7,873 train / 2,027 test person images; no version given. **(ambiguous v1/v2)** [arXiv 2303.13868](https://arxiv.org/abs/2303.13868) | none |
| 2023 | Spectral Transfer Guided Active Domain Adaptation for Thermal Imagery (STGADA) | CVPR 2023 Workshops (PBVS) | Active DA, thermal classification/detection | 3 classes (bicycle 4,137 / car 43,734 / person 26,294 boxes), follows SGADA (2021, v1). **(ambiguous, likely v1)** [arXiv 2304.07031](https://arxiv.org/abs/2304.07031) | [avaapm/STGADA](https://github.com/avaapm/STGADA) |
| 2025 | DuaDiff: Dual-Conditional Diffusion Model for Guided Thermal Image Super-Resolution | IEEE TNNLS 2025 | RGB-guided thermal SR | "FLIR-ADAS" + CATS, version not visible in abstract. **(ambiguous v1/v2)** DOI 10.1109/TNNLS.2025.3640168 | none found |
| 2025 | MDSF: A Plug-and-Play Block for Boosting Infrared Small Target Detection in YOLO-Based Networks | IEEE TGRS 2025 | IR (small) object detection | "FLIR ADAS" in abstract; full text paywalled. **(ambiguous v1/v2)** DOI 10.1109/TGRS.2025.3566889 | none found |
| 2026 | Robust Thermal Image Object Detection via Appearance-Guided Mixture of Experts (Aakerberg et al.) | WACV 2026 Workshops (RWS) | Thermal detection under drift | Uses LTDv2 + "FLIR ADAS" with simulated drift. **(ambiguous v1/v2)** [project page](https://andreasaakerberg.github.io//publication/2026-01-01-ag-moe) | none found |
| 2026 | Few-Shot LoRA Adaptation of a Flow-Matching Foundation Model for Cross-Spectral Object Detection | WACV 2026 Workshops | Synthetic-IR generation, few-shot detection | Synthetic IR from LLVIP and FLIR ADAS RGB; regex hit v1 markers. **(ambiguous, leans v1)** [arXiv 2601.04381](https://arxiv.org/abs/2601.04381) | none |
| 2026 | Knowledge-Guided Adversarial Training for Infrared Object Detection via Thermal Radiation Modeling (KGAT) | IJCV 2026 | Adversarial robustness, IR detection | Cites "FLIR-ADAS (FLIR 2018)", multi-class; no counts. **(ambiguous, leans v1)** [arXiv 2603.25170](https://arxiv.org/abs/2603.25170) | [shukunxiong/KGAT](https://github.com/shukunxiong/KGAT) |
| 2026 | ProtoHGF-Net: Prototype HyperGraph Fusion … for RGBT Object Detection | ACM MM 2026 | RGB-T fusion detection | "FLIR" compared against CFT/LRAF-Net, which use aligned. **(ambiguous, very likely aligned v1)** [arXiv 2608.11595](https://arxiv.org/abs/2608.11595) | none found |
| 2026 | Impact of Inter-Modality Misalignment on Multispectral Object Detection | IEEE OJVT 2026 | Registration + fusion detection | Generalisation test on "FLIR ADAS aligned RGB-LWIR pairs" (+3.54 mAP). **(ambiguous v1/v2)** [Univ. Galway record](https://research.universityofgalway.ie/en/publications/impact-of-inter-modality-misalignment-on-multispectral-object-det/) | none found |
| 2026 | Shifting Focus by Reliability: Geometric Fusion Modulation for Robust RGB-T Detection | WACV 2027 (accepted) | RGB-T fusion detection | aligned markers. **(likely aligned v1)** [arXiv 2610.41204](https://arxiv.org/abs/2610.41204) | none found |

### 3.3 Table C: other venues and preprints that use v2

Useful for code mining and class-filtering practice. Version confirmed unless marked otherwise.

| Year | Title | Venue | Task | FLIR variant | Code |
|---|---|---|---|---|---|
| 2023 | Joint tone mapping and denoising of thermal IR images via multi-scale Retinex and multi-task learning | SPIE DCS 2023 | Tone mapping / denoising | v2 (16-bit) | repo cited in paper: [HensoldtOptronicsCV/ToneMappingIQA](https://github.com/HensoldtOptronicsCV/ToneMappingIQA) (may be an IQA tool, not the paper's code) |
| 2024 | Infrared multi-target detection and tracking in dense urban traffic scenes (Zha et al.) | IET Image Processing 2024 | Detection + MOT | v2: 10,742 train / 2,292 test, 8-bit; reports 78.6% mAP; notes v2 is "considerably harder than v1". [Wiley](https://ietresearch.onlinelibrary.wiley.com/doi/full/10.1049/ipr2.13053) | none found |
| 2024 | The Use of Multimodal LLMs to Detect Objects from Thermal Images: Transportation Applications | MDPI *Automation* 2024 | VLM (GPT-4V, Gemini) detection | v2 (26,442 frames, 15 classes) [arXiv 2406.13898](https://arxiv.org/abs/2406.13898) | none |
| 2024 | TID-Attack: digital thermal infrared detector attack via free velocity and rollback mutation | Infrared Physics & Technology 2024 | Black-box adversarial attack | FLIR-ADAS V2 + CVC-09 + KAIST (from abstract) [ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S1350449524001695) | none found |
| 2025 | How to Augment for Atmospheric Turbulence Effects on Thermal Adapted Object Detection Models? | *Scientific Reports* 2025 (arXiv 2405.06383) | Detection + augmentation | v2 thermal stills (9,711), car + person, 90/10 train/val, official val used as test. [arXiv](https://arxiv.org/abs/2405.06383), [Sci Rep PDF](https://www.nature.com/articles/s41598-025-86830-1.pdf) | none |
| 2025 | MDCFVit-YOLO: nighttime IR small-target vehicle and pedestrian detection | PLOS ONE 2025 | Thermal detection | FLIR_ADAS_v2 (67% mAP) [PLOS](https://journals.plos.org/plosone/article?id=10.1371%2Fjournal.pone.0324700) | none found |
| 2025 | LIWL-YOLO: lightweight IR target detection for land and water surfaces | *Scientific Reports* 2025 | Thermal detection | FLIR-v2 + own data; classes with <100 samples dropped. [Nature](https://www.nature.com/articles/s41598-025-21550-0) | none found |
| 2025 | SS-DC: Spatial-Spectral Decoupling and Coupling Across Visible-Infrared Gap for Domain Adaptive Object Detection | arXiv | RGB→IR unsupervised DA detection | **v2**: 9,233 RGB / 9,711 IR. New protocol: 5,663 labelled RGB source, 4,856 unlabelled IR target, 1,144 IR eval. 5 classes (person, bike, car, sign, light). Also aligned-v1 with D3T protocol. [arXiv 2507.12017](https://arxiv.org/abs/2507.12017) | none |
| 2025 | Multi-Modal Camera-Based Detection of Vulnerable Road Users | arXiv | RGB + thermal VRU detection (YOLOv8) | v2: 10,319 RGB / 10,742 thermal train, 1,085 / 1,144 val, 3,749 video test pairs; merged with KITTI/BDD class map. [arXiv 2509.06333](https://arxiv.org/abs/2509.06333) | none |
| 2026 | Fs2PA: Full-Scale Feature Synergistic Perception Architecture for Vehicular IR Object Detection | journal (PubMed-indexed, likely MDPI Sensors) | Thermal detection | FLIR v2 + M3FD; mAP50 64.06% [PubMed](https://pubmed.ncbi.nlm.nih.gov/41978041/) | none found |
| 2026 | XCalib: Depth-Guided Geometric Optimization for Dense Thermal-Visible Video Registration | arXiv | RGB-T **registration** of v2 videos | v2 video, day/night sequences of 672 / 1,040 pairs. [arXiv 2609.21770](https://arxiv.org/abs/2609.21770) | none |
| 2026 | Two-Stage Dataset Curation via Class Pruning for Improved AI-Based Thermal Image Object Detection | IEEE eIT 2026 | Class pruning / curation | "FLIR ADAS" in abstract, 15-class pruning implies v2 **(likely v2)**. DOI 10.1109/eIT68936.2026.11670425 | none found. **Relevant to Group E class filtering** |
| 2026 | ThermEval: A Structured Benchmark for Evaluation of VLMs on Thermal Imagery | arXiv (KDD submission?) | VLM benchmark | cites v2 page, "≈13,000 aligned pairs" **(ambiguous v1/v2)** [arXiv 2602.14989](https://arxiv.org/abs/2602.14989) | [AyushShrivstava/ThermEval_KDD](https://github.com/AyushShrivstava/ThermEval_KDD) |

### 3.4 Table D: high-profile papers that use FLIR-aligned (v1), not v2 (for contrast and to avoid mislabelling)

| Year | Title | Venue | FLIR variant evidence | Code |
|---|---|---|---|---|
| 2023 | TFDet: Target-Aware Fusion for RGB-T Pedestrian Detection | IEEE TNNLS | aligned 5,142 pairs | — |
| 2024 | HalluciDet | WACV 2024 | "sanitized and aligned paired sets by Zhang et al.", 4,129 / 1,013, person only | [heitorrapela/HalluciDet](https://github.com/heitorrapela/HalluciDet) |
| 2024 | D3T: Distinctive Dual-Domain Teacher | CVPR 2024 | aligned FLIR, RGB→thermal DA | [EdwardDo69/D3T](https://github.com/EdwardDo69/D3T) |
| 2024 | Causal Mode Multiplexer (CMM) | CVPR 2024 | FLIR test 1,013 (aligned) | — |
| 2024 | ModTr: Modality Translation for OD Adaptation | ECCV 2024 | aligned 4,129 / 1,013, dog removed (bicycle, car, person) | [heitorrapela/ModTr](https://github.com/heitorrapela/ModTr) |
| 2024 | MSCoTDet | IEEE TCSVT | aligned 4,129 / 1,013, **but cites the v2 page** | — |
| 2025 | FD2-Net | AAAI 2025 | aligned 5,142 (Zhang 2020) | — |
| 2025 | Visual Modality Prompt (ModPrompt) | ICCV 2025 | aligned | [heitorrapela/ModPrompt](https://github.com/heitorrapela/ModPrompt) |
| 2025 | ThermalGen | NeurIPS 2025 | FLIR 4,129 / 1,013 (aligned) | project page only |
| 2025 | DiffV2IR | arXiv 2025 | eval on FLIR-aligned (4,489 of 5,142 pairs); "FLIR" also in IR-500K pretraining pool (version unclear) | [LidongWang-26/DiffV2IR](https://github.com/LidongWang-26/DiffV2IR) |
| 2025 | F-ViTA | arXiv 2025 (ICCV-style) | "aligned version" via a FLIR-align repo, 4,890 / 126 **(origin unclear, Group A should check)** | [JayParanjape/F-ViTA](https://github.com/JayParanjape/F-ViTA) |
| 2026 | WiSE-OD | WACV 2026 | FLIR ALIGNED 4,129 / 1,013 (FLIR-C corruptions built on it) | — |

---

## 4. Code and tooling links for later agents (class filtering and preprocessing)

**Paper code that uses v2:**
- [weihui1308/HOTCOLDBlock](https://github.com/weihui1308/HOTCOLDBlock): person filter, height >120 px.
- [tlgksy/ir-reasoner](https://github.com/tlgksy/ir-reasoner): 14-bit RAW, 5-class subset.
- [donkeymouse/ThermalChameleon](https://github.com/donkeymouse/ThermalChameleon): RAW tone mapping, 10,724 / 2,151 split.
- [lengmo1996/ECDM](https://github.com/lengmo1996/ECDM): 5 classes.
- [hosseinjavidnia/NAGC](https://github.com/hosseinjavidnia/NAGC): 16-bit v1+v2.

**Community v2 tooling:**
- [GokulEdakkePuram/ThermalObjectDetection](https://github.com/GokulEdakkePuram/ThermalObjectDetection/blob/main/docs/00-the-dataset.md). An **audit of v2**: 30,787 frames on disk vs 26,442 claimed; 11,886 thermal / 11,403 RGB train+val vs the claimed 9,711 / 9,233; a "deer" class not in FLIR's list; 24 Hz vs 30 fps; track_id in the video annotations. Key input for Groups A and E.
- [ax2mazhr/FLIR2YOLO](https://github.com/ax2mazhr/FLIR2YOLO): Conservator/COCO→YOLO converter for v2 folder layout.
- [sarisemih/customize-flir-dataset](https://github.com/sarisemih/customize-flir-dataset): class selection, Conservator→YOLO.
- [bilibilistar/Infrared_Imgae_Detection_Using_YoloV5](https://github.com/bilibilistar/Infrared_Imgae_Detection_Using_YoloV5): v1 and v2 YOLOv5.
- [Ariazhao006/groundingdino-lora-flir](https://github.com/Ariazhao006/groundingdino-lora-flir).
- [xi666666/FLIR-ADAS-UDAOD](https://github.com/xi666666/FLIR-ADAS-UDAOD): RGB→IR UDA split, 2026.
- [Danial-Qashqai/RGBT-ObjectDetection](https://github.com/Danial-Qashqai/RGBT-ObjectDetection).

**Aligned-v1 code (often mistaken for v2):** D3T, HalluciDet, ModTr, ModPrompt, STGADA, KGAT (links in the tables above).

**v2 mirrors:**
- Kaggle [samdazel/teledyne-flir-adas-thermal-dataset-v2](https://www.kaggle.com/datasets/samdazel/teledyne-flir-adas-thermal-dataset-v2): 11.9 GB, 12,434 downloads.
- Kaggle `kushagraborse/thermal-images-flir-adas-dataset`: 248 downloads.
- Kaggle `rajababuadigarla/teledyne-flir-free-adas-thermal-dataset-v2`: 94 downloads.
- HF `Maxim37/flir_adas_video_pairs`.
- Roboflow "Flir ADAS v2" export: a **car-only** subset; don't confuse it with the full dataset.

---

## 5. Uncertainties

1. **No complete count exists.** Google Scholar was not accessible programmatically, Semantic Scholar full-text search was rate-limited, and Papers with Code is closed and its archive has no FLIR entry. The yearly numbers are therefore title/abstract lower bounds (S2) or partial full-text counts (OpenAlex).
2. **Version inference.** Many papers write only "FLIR". The aligned-v1 subset is so dominant in fusion work that I list a "FLIR" paper as v2 only with explicit evidence. Table B entries could move to Table A or Table D after a full read. Paywalled papers (TGRS, TNNLS, IET, ScienceDirect, OJVT) were judged from abstracts or snippets.
3. **Fewer than 20 major-venue v2 papers.** Even counting workshops, ICIP and IJCNN, I confirmed 11. Adding the ambiguous major-venue papers gives 21. If the downstream plan needs v2 benchmarks from top venues, the evidence base is thin, and FLIR-aligned (v1) is the benchmark the community actually compares on.
4. **Venue status of preprints.** SS-DC, VRU, XCalib and ThermEval had no confirmed venue as of 2026-11-02. Thermal-Det code is not yet released.
5. **Counts in the dataset itself are inconsistent.** Different papers report 26,442 / 30,787 frames and 10,724 vs 10,742 train images. This is a Group A/E issue, but it means split sizes in the papers differ even under "official v2".
