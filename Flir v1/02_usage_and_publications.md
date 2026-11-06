# FLIR ADAS v1: usage trends and publications (Q2, Q3)

**Research date:** 2026-11-02
**Scope:** Teledyne FLIR "FREE ADAS" thermal dataset, **v1** (July 2018 release; v1.0 to v1.3; about 10,228 thermal frames, 8,862 train and 1,366 val; 9,214 to 9,219 frames with boxes; mainly person, bicycle, car and dog, plus a rare "other vehicle"; RGB frames are **not** aligned and carry no labels). I kept it separate from **v2** (January 2022; 26,442 annotated frames, 15 classes, RGB labels and an RGB-T video map).

---

## TL;DR

- **v1 is still used heavily in 2026, but mostly through its derivatives, not the raw release.** The standard benchmark today is **"FLIR-aligned"**: 5,142 RGB-T pairs (4,129 train, 1,013 test; person, car, bicycle). Zhang et al. (ICIP 2020, CFR) cut it out of v1. Semantic Scholar counts citations of that paper at 1 (2020), 11 (2021), 14 (2022), 42 (2023), 72 (2024), 102 (2025) and 118 (2026 so far). The citations keep rising every year.
- **Image fusion uses a second derivative, RoadScene**: 221 hand-aligned pairs taken from v1 video (FusionDN, AAAI 2020). It shows up in almost every IVIF paper (U2Fusion, TarDAL, CDDFuse, DDFM, ...), often labelled just "FLIR".
- **Direct use of the raw v1 release is declining.** Citations of MMTOD (CVPRW 2019), the classic thermal-only v1 benchmark paper, peaked at 27 a year in 2021 to 2024 and fell to 19 (2025) and 15 (2026). Thermal-only papers from 2023 onward often move to v2 (e.g. HOTCOLD, AAAI 2023, says so explicitly).
- **Competitors grow faster.** LLVIP citations: 23 (2022), 96 (2023), 159 (2024), 315 (2025), 365 (2026). M3FD/TarDAL: 17, 120, 234, 470, 531. These are now cited 3 to 5 times more than FLIR-aligned. Still, most 2024 to 2026 multispectral-detection papers report **FLIR-aligned + LLVIP + M3FD** together. FLIR-aligned is being complemented, not dropped.
- **v2 has not replaced v1 for RGB-T research.** v2 is popular for thermal-only detection and on Kaggle: the v2 mirror has 12.4k downloads, against 15.0k for the main v1 mirror since 2020. But top-venue RGB-T fusion work still almost always uses FLIR-aligned (v1).
- **Q3 table: 48 papers in important venues** (2019 to 2026). 26 use FLIR-aligned, 8 the official v1, 7 RoadScene, 4 a custom v1 subset or their own alignment, and 3 are ambiguous v1/v2. **37 of 48 have a GitHub repo.** Venues: CVPR 8, ECCV 5, ICCV 5, WACV 4, CVPRW 3, plus AAAI, IJCAI, ACM MM, IROS, IV, TPAMI, T-ITS, TCSVT, TNNLS, TMM, TIV, PR, Information Fusion, etc.

---

## Q2. How much is v1 used recently?

### 2.1 Methodology (and limits)

| Signal | How measured | Main limits |
|---|---|---|
| Semantic Scholar (S2) **keyword counts** | `/paper/search/bulk` with year filter. The query matches **title + abstract only** | Most papers name FLIR only in the experiments section, so these are strong **undercounts**. "FLIR" is also a camera brand (KAIST, LLVIP and others were shot with FLIR cameras). |
| S2 **citation counts per year** of anchor papers | `/paper/{id}/citations` grouped by citing year | FLIR v1 has **no official paper**, so I used proxies: CFR (ICIP 2020, creator of FLIR-aligned) and MMTOD (CVPRW 2019, the classic v1 benchmark). Citing does not always mean using. 2026 is partial (Jan to Oct). S2 sometimes dates papers by the arXiv year. |
| arXiv API abstract counts | `abs:FLIR AND (abs:thermal OR abs:infrared)` per year | Very small numbers, and the API returned unstable counts (e.g. KAIST counts looked too low). Treat as indicative only. |
| GitHub repo search | `api.github.com/search/repositories` per creation year | Matches repo name/description/README. "FLIR thermal" also matches camera drivers (Lepton, Boson). |
| Kaggle / Hugging Face | public listing APIs (download counts) | Mirrors are unofficial. Kaggle counts are lifetime totals and HF counts cover the last 30 days. Version is inferred from size and description. |
| Papers with Code | **not available**: paperswithcode.com now redirects to Hugging Face (since 2025), and the Wayback copy could not be fetched (HTTP 429) | No "papers using dataset" counter could be retrieved. |
| OpenAlex full-text search | attempt failed (daily anonymous quota exhausted) | Not used. |

Google Scholar blocks automated queries, so I could not collect Scholar counts. As a rough guide, Scholar full-text hits for "FLIR ADAS" are typically an order of magnitude above S2 title/abstract hits.

### 2.2 Per-year numbers

**A. Citations per year of anchor papers (Semantic Scholar, retrieved 2026-11-02)**

| Anchor paper (proxy for) | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026* | Total |
|---|---|---|---|---|---|---|---|---|---|
| CFR, ICIP 2020 (FLIR-aligned, a **v1 derivative**) | - | 1 | 11 | 14 | 42 | 72 | 102 | 118 | 367 |
| MMTOD, CVPRW 2019 (**raw v1**, thermal detection) | 2 | 12 | 27 | 22 | 27 | 27 | 19 | 15 | 152 |
| ProbEn, ECCV 2022 (raw v1, RGB-T) | - | - | - | 4 | 25 | 65 | 72 | 93 | 263 |
| LLVIP (2021) (competitor) | - | 1 | 1 | 23 | 96 | 159 | 315 | 365 | 976 |
| TarDAL / M3FD, CVPR 2022 (competitor) | - | - | - | 17 | 120 | 234 | 470 | 531 | 1391 |

*2026 = January to early November.

**B. Semantic Scholar keyword hits (title + abstract only; strong undercounts)**

| Query | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026* |
|---|---|---|---|---|---|---|---|---|
| `"FLIR ADAS"` | 2 | 2 | 5 | 6 | 7 | 7 | 21 | n/a (rate limit) |
| `FLIR (infrared\|thermal) (detection\|fusion\|translation)` (noisy: includes camera mentions) | 34 | 53 | 50 | 42 | 58 | 87 | 127 | 143 |
| `"FLIR-aligned"\|"FLIR aligned"` | - | 0 | 0 | 0 | 3 | 5 | 3 | 18 |
| `"FLIR ADAS v2"\|"FLIR v2"\|"FLIR ADAS 2.0"` | - | 0 | 0 | 1 | 0 | 2 | 5 | 7 |
| `LLVIP` | 0 | 0 | 1 | 5 | 26 | 42 | 102 | 143 |
| `M3FD` | 0 | 0 | 0 | 0 | 13 | 26 | 63 | 131 |

**C. arXiv abstract mentions** (`FLIR AND (thermal OR infrared)`): 2018: 1, 2019: 0, 2020: 4, 2021: 8, 2022: 5, 2023: 7, 2024: 8, 2025: 8, 2026: 24. For comparison, LLVIP: 0, 0, 0, 1, 1, 2, 10, 12, 13 (2018 to 2026). Indicative only (see the limits above).

**D. GitHub repositories (created per year)**

| Query | 2018 | 2019 | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026* |
|---|---|---|---|---|---|---|---|---|---|
| `FLIR thermal` (noisy) | 11 | 33 | 29 | 18 | 21 | 22 | 23 | 45 | 95 |
| `FLIR ADAS` | 0 | 0 | 1 | 2 | 1 | 1 | 2 | 4 | 13 |
| `LLVIP` | 0 | 0 | 0 | 1 | 1 | 4 | 3 | 13 | 33 |
| `M3FD` | 0 | 0 | 0 | 0 | 0 | 1 | 2 | 2 | 9 |

All-time totals: "FLIR thermal" 334, "FLIR ADAS" 25, "FLIR thermal detection" 60, "FLIR yolo" 55, "FLIR fusion" 13. Most research code does not put "FLIR" in the repo name, so these numbers are lower bounds. See the Q3 code column for the real research repos.

**E. Kaggle / Hugging Face mirrors (download counts on 2026-11-02)**

| Mirror | Version (inferred) | Downloads |
|---|---|---|
| [kaggle deepnewbie/flir-thermal-images-dataset](https://www.kaggle.com/datasets/deepnewbie/flir-thermal-images-dataset) (16.4 GB, 2020) | **v1** (full, with video) | 14,958 (lifetime), 90 votes |
| [kaggle samdazel/teledyne-flir-adas-thermal-dataset-v2](https://www.kaggle.com/datasets/samdazel/teledyne-flir-adas-thermal-dataset-v2) (11.9 GB) | v2 | 12,434 (lifetime) |
| [kaggle albertofv/flir-thermal-images-dataset-reduced](https://www.kaggle.com/datasets/albertofv/flir-thermal-images-dataset-reduced) (2021) | v1 (reduced) | 1,030 |
| [kaggle s111mple/flir-aligned](https://www.kaggle.com/datasets/s111mple/flir-aligned) (2024) | FLIR-aligned (v1) | 228 |
| [kaggle benceveges/yolo-flir](https://www.kaggle.com/datasets/benceveges/yolo-flir) (2022) | v1 (YOLO format, likely) | 100 |
| [HF jsonhash/FLIR_aligned](https://huggingface.co/datasets/jsonhash/FLIR_aligned) (2024) | FLIR-aligned (v1) | 1,084 (last 30 days) |
| [HF bushwang/FLIR_aligned](https://huggingface.co/datasets/bushwang/FLIR_aligned) | FLIR-aligned (v1) | 53 (30 d) |
| [HF Maxim37/flir_adas_video_pairs](https://huggingface.co/datasets/Maxim37/flir_adas_video_pairs) | ambiguous (likely v2 video) | 83 (30 d) |

About 1.1k downloads a month of FLIR-aligned on HF alone, in late 2026, is direct evidence that the v1-derived aligned subset is still in active use.

### 2.3 Is v1 being replaced?

**By v2?** Only partly, and mostly for thermal-only work.
- Evidence for a switch: HOTCOLD Block (AAAI 2023) says: *"since the Teledyne FLIR company released an updated version in January 2022, we use v2.0 instead of v1.0"* ([arXiv 2212.05709](https://arxiv.org/abs/2212.05709); repo [weihui1308/HOTCOLDBlock](https://github.com/weihui1308/HOTCOLDBlock) links the v2 download). Thermal-only and edge-detection papers in 2024 to 2025 use v2 with 15 classes (e.g. [arXiv 2406.13898](https://arxiv.org/abs/2406.13898), [MS-YOLO, arXiv 2509.21696](https://arxiv.org/abs/2509.21696)). On Kaggle, the v2 mirror (12.4k) is close to the main v1 mirror (15.0k).
- Evidence against: v2 still ships **unaligned** RGB-T stills. No standard "aligned v2" benchmark has emerged, so RGB-T fusion papers kept FLIR-aligned (v1). In the Q3 table, all 2025 to 2026 top-venue RGB-T detection papers (ICCV 2025 WaveMamba, M2D-LIF, M-SpecGene; ECCV 2026 ZPA-MDETR; CVPR 2026 RGBT-Clothing; Information Fusion 2026 MS2Fusion; PR 2026 IRDFusion) use FLIR-aligned. Even D3T (CVPR 2024) chose "the updated FLIR dataset" and meant **FLIR-aligned**, not v2. S2 title/abstract hits for "FLIR v2" stay in single digits a year.

**By other RGB-T datasets (LLVIP, M3FD, KAIST, DroneVehicle, MSRS)?** It has been complemented, with relative share falling.
- LLVIP (2021) and M3FD (2022) citation curves overtook FLIR-aligned in 2023 and are now 3 to 5 times higher (table A).
- The usual 2024 to 2026 protocol is the triplet **FLIR-aligned + LLVIP + M3FD**, sometimes with DroneVehicle, KAIST or VEDAI: DAMSDet, Fusion-Mamba, RSDet, WaveMamba, IRDFusion, SeaDATE, MS2Fusion and others. FLIR-aligned is usually the hardest of the three on mAP50 (about 80 to 85, against about 97 for LLVIP), so it stays useful as a discriminative benchmark.
- In image fusion, RoadScene (v1-derived) is a fixed test set next to TNO, MSRS and M3FD. MSRS and M3FD are now used more often for training.
- KAIST remains the pedestrian benchmark; FLIR is the multi-class driving benchmark. They are used side by side, not as substitutes.

**Bottom line:** raw v1 use (thermal-only, unaligned) is slowly declining and partly moving to v2. The **v1-derived FLIR-aligned subset is still growing in absolute terms** as one of the three standard RGB-T detection benchmarks, though its share is shrinking relative to LLVIP and M3FD.

---

## Q3. Papers in important venues that use FLIR v1 (or a v1 derivative)

**Inclusion rule:** the paper runs experiments on FLIR v1, the FLIR-aligned subset (made from v1 in 2020), RoadScene (made from v1 video), or a custom v1 subset. The version was inferred from the year, image counts (10,228 / 9,214 / 5,142 / 4,129+1,013), the classes (3 to 5 classes means v1; 15 to 16 means v2) and the download links. Rows are sorted by year. Venues were checked against Semantic Scholar, DBLP, CVF or IEEE metadata. The FLIR usage and variant were checked against the PDF text (arXiv or CVF), except where marked "inferred".

**Variant legend**
- **Official v1:** the original thermal frames and labels (RGB unaligned).
- **Aligned subset:** "FLIR-aligned" by Zhang, Fromont, Lefèvre and Avignon (ICIP 2020): 5,142 pairs, 4,129 train and 1,013 test, 3 classes (person, car, bicycle). Commonly redistributed as `FLIR_ICIP2020_aligned` / `FLIR_align`.
- **Derived: RoadScene:** 221 aligned pairs from v1 video ([hanna-xu/RoadScene](https://github.com/hanna-xu/RoadScene)).
- **Custom:** the authors' own subset or alignment of v1.

### 3.1 Main table (sorted by year)

| # | Year | Title | Venue | Task | FLIR variant | Code | Source |
|---|---|---|---|---|---|---|---|
| 1 | 2019 | Borrow From Anywhere: Pseudo Multi-Modal Object Detection in Thermal Imagery (MMTOD) | CVPR Workshops (PBVS) | Thermal detection (pseudo-RGB via I2I translation) | Official v1 (thermal; 9,214 annotated imgs; also FLIR-1/2, FLIR-1/4 subsets) | [tdchaitanya/MMTOD](https://github.com/tdchaitanya/MMTOD) | [link](https://openaccess.thecvf.com/content_CVPRW_2019/html/PBVS/Devaguptapu_Borrow_From_Anywhere_Pseudo_Multi-Modal_Object_Detection_in_Thermal_Imagery_CVPRW_2019_paper.html) |
| 2 | 2020 | FusionDN: A Unified Densely Connected Network for Image Fusion (introduces RoadScene) | AAAI | Image fusion | Derived: RoadScene (221 hand-aligned RGB-T pairs cut from v1 video) | [hanna-xu/FusionDN](https://github.com/hanna-xu/FusionDN); dataset [hanna-xu/RoadScene](https://github.com/hanna-xu/RoadScene) | [link](https://github.com/hanna-xu/RoadScene) |
| 3 | 2020 | DIDFuse: Deep Image Decomposition for Infrared and Visible Image Fusion | IJCAI | Image fusion | Derived: RoadScene ("FLIR", 180 train / 40 test pairs) | [Zhaozixiang1228/IVIF-DIDFuse](https://github.com/Zhaozixiang1228/IVIF-DIDFuse) | [link](https://arxiv.org/abs/2003.09210) |
| 4 | 2020 | Multispectral Fusion for Object Detection with Cyclic Fuse-and-Refine Blocks (CFR) - origin of "FLIR-aligned" | ICIP | RGB-T detection (fusion) | Aligned subset created here (5,142 pairs: 4,129 train / 1,013 test; person, car, bicycle) | [zhanghengdev/CFR](https://github.com/zhanghengdev/CFR) (KAIST results only; FLIR-aligned data is mirrored e.g. in D3T/RGBXFusion READMEs) | [link](https://arxiv.org/abs/2009.12664) |
| 5 | 2020 | Pyramidal Edge-maps and Attention based Guided Thermal Super-resolution | ECCV Workshops | Thermal super-resolution (RGB-guided) | Official v1 RGB-T pairs (rectified, unaligned; 43,830 train / 1,257 test pairs incl. video) | none found | [link](https://arxiv.org/abs/2003.06216) |
| 6 | 2021 | Guided Attentive Feature Fusion for Multispectral Pedestrian Detection (GAFF) | WACV | RGB-T detection (fusion) | Aligned subset (same group as CFR) | none found (official) | [link](https://openaccess.thecvf.com/content/WACV2021/papers/Zhang_Guided_Attentive_Feature_Fusion_for_Multispectral_Pedestrian_Detection_WACV_2021_paper.pdf) |
| 7 | 2021 | Fooling Thermal Infrared Pedestrian Detectors in Real World Using Small Bulbs | AAAI | Adversarial attack (physical) on thermal person detection | Custom: "FLIR ADAS v1.3", person subset "FLIR person select" (710 train / 301 test) | none found | [link](https://arxiv.org/abs/2101.08154) |
| 8 | 2021 | Self-training Guided Adversarial Domain Adaptation for Thermal Imagery (SGADA) | CVPR Workshops (PBVS) | Domain adaptation (MS-COCO RGB -> FLIR thermal, object classification on crops) | Official v1 (thermal crops) | [avaapm/SGADA](https://github.com/avaapm/SGADA) | [link](https://arxiv.org/abs/2106.07165) |
| 9 | 2021 | SSTN: Self-Supervised Domain Adaptation Thermal Object Detection for Autonomous Driving | IROS | Domain adaptation / thermal detection | Official v1 (9,214 imgs, 3 classes: car, person, bicycle) | none found | [link](https://arxiv.org/abs/2103.03150) |
| 10 | 2022 | Thermal Infrared Image Colorization for Nighttime Driving Scenes With Top-Down Guided Attention (PearlGAN) | IEEE T-ITS | Image translation (NTIR->daytime color) + seg. evaluation | Official v1 ("FLIR Thermal Starter Dataset"; day RGB / night IR split + own pixel-level masks) | [FuyaLuo/PearlGAN](https://github.com/FuyaLuo/PearlGAN) | [link](https://arxiv.org/abs/2104.14374) |
| 11 | 2022 | Efficient and Model-Based Infrared and Visible Image Fusion via Algorithm Unrolling (AUIF) | IEEE TCSVT | Image fusion | Derived: RoadScene ("FLIR", 180 train / 40 test) | [Zhaozixiang1228/IVIF-AUIF-Net](https://github.com/Zhaozixiang1228/IVIF-AUIF-Net) | [link](https://ieeexplore.ieee.org/document/9416456) |
| 12 | 2022 | U2Fusion: A Unified Unsupervised Image Fusion Network | IEEE TPAMI | Image fusion | Derived: RoadScene | [hanna-xu/U2Fusion](https://github.com/hanna-xu/U2Fusion) | [link](https://github.com/hanna-xu/RoadScene) |
| 13 | 2022 | Target-aware Dual Adversarial Learning and a Multi-scenario Multi-Modality Benchmark (TarDAL / M3FD) | CVPR | Image fusion + detection | Derived: RoadScene (fusion eval); introduces competitor M3FD | [dlut-dimt/TarDAL](https://github.com/dlut-dimt/TarDAL) | [link](https://arxiv.org/abs/2203.16220) |
| 14 | 2022 | Infrared Invisible Clothing: Hiding from Infrared Detectors at Multiple Angles in Real World | CVPR | Adversarial attack (physical) | Custom: "FLIR ADAS v1.3", person subset PEOPLE_FLIR (7,873 train / 2,027 test) | none found | [link](https://arxiv.org/abs/2205.05909) |
| 15 | 2022 | Multimodal Object Detection via Probabilistic Ensembling (ProbEn) | ECCV | RGB-T detection (late fusion) | Official v1 (unaligned RGB + thermal labels) | [Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling](https://github.com/Jamie725/Multimodal-Object-Detection-via-Probabilistic-Ensembling) | [link](https://arxiv.org/abs/2104.02904) |
| 16 | 2022 | Robust Environment Perception for Automated Driving: A Unified Learning Pipeline for Visual-Infrared Object Detection | IEEE IV | RGB-T detection (fusion) | Custom alignment of v1 (7,381 / 1,056 / 2,111 pairs; 3 classes) | none found | [link](https://arxiv.org/abs/2206.03943) |
| 17 | 2023 | Physically Adversarial Infrared Patches with Learnable Shapes and Locations | CVPR | Adversarial attack (physical) | (ambiguous v1/v2) person subset; same group used v2 in HOTCOLD (AAAI'23) | none found | [link](https://arxiv.org/abs/2303.13868) |
| 18 | 2023 | Multi-Modal Gated Mixture of Local-to-Global Experts for Dynamic Image Fusion (MoE-Fusion) | ICCV | Image fusion (+ detection) | Aligned subset | [SunYM2020/MoE-Fusion](https://github.com/SunYM2020/MoE-Fusion) | [link](https://arxiv.org/abs/2302.01392) |
| 19 | 2023 | CDDFuse: Correlation-Driven Dual-Branch Feature Decomposition for Multi-Modality Image Fusion | CVPR | Image fusion | Derived: RoadScene (50 val pairs + test) | [Zhaozixiang1228/MMIF-CDDFuse](https://github.com/Zhaozixiang1228/MMIF-CDDFuse) | [link](https://arxiv.org/abs/2211.14461) |
| 20 | 2023 | DDFM: Denoising Diffusion Model for Multi-Modality Image Fusion | ICCV | Image fusion | Derived: RoadScene (test) | [Zhaozixiang1228/MMIF-DDFM](https://github.com/Zhaozixiang1228/MMIF-DDFM) | [link](https://arxiv.org/abs/2303.06840) |
| 21 | 2023 | Multimodal Object Detection by Channel Switching and Spatial Attention (CSSA) | CVPR Workshops (PBVS) | RGB-T detection (fusion) | Aligned subset (inferred from reported mAP50 79.2) | unofficial re-impl.: [artrela/mulitmodal-cssa](https://github.com/artrela/mulitmodal-cssa) | [link](https://openaccess.thecvf.com/content/CVPR2023W/PBVS/html/Cao_Multimodal_Object_Detection_by_Channel_Switching_and_Spatial_Attention_CVPRW_2023_paper.html) |
| 22 | 2023 | LRAF-Net: Long-Range Attention Fusion Network for Visible-Infrared Object Detection | IEEE TNNLS | RGB-T detection (fusion) | (ambiguous; most likely aligned subset) | none found | [link](https://ieeexplore.ieee.org/document/10144688) |
| 23 | 2023 | TIRDet: Mono-Modality Thermal InfraRed Object Detection Based on Prior Thermal-To-Visible Translation | ACM MM | Thermal detection (with T2V translation prior) | (ambiguous) README links official FLIR page; compared on FLIR-aligned by later work (PFGF) | [zeyuwang-zju/TIRDet](https://github.com/zeyuwang-zju/TIRDet) | [link](https://github.com/zeyuwang-zju/TIRDet) |
| 24 | 2023 | Illumination-Guided RGBT Object Detection With Inter- and Intra-Modality Fusion (TINet) | IEEE TIM | RGB-T detection (fusion) | Aligned subset (official FLIR-aligned splits) | [NNNNerd/Triple-I-Net-TINet](https://github.com/NNNNerd/Triple-I-Net-TINet) | [link](https://ieeexplore.ieee.org/abstract/document/10057437/) |
| 25 | 2024 | ICAFusion: Iterative Cross-Attention Guided Feature Fusion for Multispectral Object Detection | Pattern Recognition | RGB-T detection (fusion) | Aligned subset | [chanchanchan97/ICAFusion](https://github.com/chanchanchan97/ICAFusion) | [link](https://arxiv.org/abs/2308.07504) |
| 26 | 2024 | D3T: Distinctive Dual-Domain Teacher Zigzagging Across RGB-Thermal Gap for Domain-Adaptive Object Detection | CVPR | Domain adaptation (RGB -> thermal detection) | Aligned subset ("FLIR_ICIP2020_aligned", new disjoint RGB/thermal split) | [EdwardDo69/D3T](https://github.com/EdwardDo69/D3T) | [link](https://arxiv.org/abs/2403.09359) |
| 27 | 2024 | HalluciDet: Hallucinating RGB Modality for Person Detection Through Privileged Information | WACV | IR->RGB translation for detection (person only) | Aligned subset (person class only) | [heitorrapela/HalluciDet](https://github.com/heitorrapela/HalluciDet) | [link](https://arxiv.org/abs/2310.04662) |
| 28 | 2024 | RGB-X Object Detection via Scene-Specific Fusion Modules | WACV | RGB-T detection (fusion) | Aligned subset (+ own day/night split) | [dsriaditya999/RGBXFusion](https://github.com/dsriaditya999/RGBXFusion) | [link](https://arxiv.org/abs/2310.19372) |
| 29 | 2024 | DAMSDet: Dynamic Adaptive Multispectral Detection Transformer | ECCV | RGB-T detection (DETR fusion) | Aligned subset | [gjj45/DAMSDet](https://github.com/gjj45/DAMSDet) | [link](https://arxiv.org/abs/2403.00326) |
| 30 | 2024 | Modality Translation for Object Detection Adaptation Without Forgetting Prior Knowledge (ModTr) | ECCV | IR->RGB translation / detector adaptation | Aligned subset | [heitorrapela/ModTr](https://github.com/heitorrapela/ModTr) | [link](https://arxiv.org/abs/2404.01492) |
| 31 | 2024 | When Pedestrian Detection Meets Multi-Modal Learning: Generalist Model and Benchmark Dataset (MMPedestron) | ECCV | Multi-modal pedestrian detection (generalist) | Aligned subset (fine-tuning); FLIR also inside MMPD training mix | [BubblyYi/MMPedestron](https://github.com/BubblyYi/MMPedestron) | [link](https://arxiv.org/abs/2407.10125) |
| 32 | 2024 | CF-Deformable DETR: An End-to-End Alignment-Free Model for Weakly Aligned Visible-Infrared Object Detection | IJCAI | RGB-T detection (weakly aligned) | Aligned subset + synthetic random offsets ("FLIR_w") | [116508/CF-Deformable-DETR](https://github.com/116508/CF-Deformable-DETR) (README empty) | [link](https://www.ijcai.org/proceedings/2024/0084.pdf) |
| 33 | 2024 | TFDet: Target-Aware Fusion for RGB-T Pedestrian Detection | IEEE TNNLS | RGB-T detection (fusion) | Aligned subset | [XueZ-phd/TFDet](https://github.com/XueZ-phd/TFDet) | [link](https://arxiv.org/abs/2305.16580) |
| 34 | 2024 | Nighttime Thermal Infrared Image Colorization With Feedback-Based Object Appearance Learning (FoalGAN) | IEEE TCSVT | Image translation (NTIR->daytime color) | Official v1 ("FLIR Thermal Starter Dataset", PearlGAN splits) | [FuyaLuo/FoalGAN](https://github.com/FuyaLuo/FoalGAN) | [link](https://arxiv.org/abs/2310.15688) |
| 35 | 2025 | Mixed Patch Visible-Infrared Modality Agnostic Object Detection (MiPa) | WACV | Modality-agnostic detection | Aligned subset | [heitorrapela/MiPa](https://github.com/heitorrapela/MiPa) | [link](https://arxiv.org/abs/2404.18849) |
| 36 | 2025 | Fusion-Mamba for Cross-Modality Object Detection | IEEE TMM | RGB-T detection (Mamba fusion) | Aligned subset | none found | [link](https://arxiv.org/abs/2404.09146) |
| 37 | 2025 | Removal Then Selection: A Coarse-to-Fine Fusion Perspective for RGB-Infrared Object Detection (RSDet) | IEEE T-ITS | RGB-T detection (fusion) | Aligned subset | [Zhao-Tian-yi/RSDet](https://github.com/Zhao-Tian-yi/RSDet) | [link](https://arxiv.org/abs/2401.10731) |
| 38 | 2025 | Rethinking Early-Fusion Strategies for Improved Multispectral Object Detection | IEEE TIV | RGB-T detection (early fusion) | Custom: Vadidar et al. (IV'22) v1 alignment, 7,381 / 1,056 / 2,111 | [XueZ-phd/Efficient-RGB-T-Early-Fusion-Detection](https://github.com/XueZ-phd/Efficient-RGB-T-Early-Fusion-Detection) (README not reachable) | [link](https://arxiv.org/abs/2405.16038) |
| 39 | 2025 | Pseudo Visible Feature Fine-Grained Fusion for Thermal Object Detection (PFGF) | CVPR | Thermal detection (pseudo-visible features) | Aligned subset (thermal only at test) | [liting1018/PFGF](https://github.com/liting1018/PFGF) | [link](https://openaccess.thecvf.com/content/CVPR2025/papers/Li_Pseudo_Visible_Feature_Fine-Grained_Fusion_for_Thermal_Object_Detection_CVPR_2025_paper.pdf) |
| 40 | 2025 | WaveMamba: Wavelet-Driven Mamba Fusion for RGB-Infrared Object Detection | ICCV | RGB-T detection (Mamba fusion) | Aligned subset | [EhanDong/WaveMamba](https://github.com/EhanDong/WaveMamba) (from Awesome-RGBT list; README not reachable) | [link](https://arxiv.org/abs/2507.18173) |
| 41 | 2025 | Rethinking Multi-modal Object Detection from the Perspective of Mono-Modality Feature Learning (M2D-LIF) | ICCV | RGB-T detection | Aligned subset | [Zhao-Tian-yi/M2D-LIF](https://github.com/Zhao-Tian-yi/M2D-LIF) | [link](https://arxiv.org/abs/2503.11780) |
| 42 | 2025 | M-SpecGene: Generalized Foundation Model for RGBT Multispectral Vision | ICCV | RGB-T foundation model (detection downstream) | Aligned subset (inferred: mAP50 84.8 scale; COCO_FLIR files provided) | [CalayZhou/M-SpecGene](https://github.com/CalayZhou/M-SpecGene) | [link](https://arxiv.org/abs/2507.16318) |
| 43 | 2025 | UniRGB-IR: A Unified Framework for Visible-Infrared Downstream Tasks via Adapter Tuning | ACM MM | RGB-T detection / segmentation (adapter tuning) | Aligned subset ("FLIR_align", 4,129 / 1,013) | [PoTsui99/UniRGB-IR](https://github.com/PoTsui99/UniRGB-IR) | [link](https://arxiv.org/abs/2404.17360) |
| 44 | 2026 | Multispectral State-Space Feature Fusion (MS2Fusion) | Information Fusion | RGB-T detection (+ seg, SOD) | Aligned subset ("FLIR-align") | [61s61min/MS2Fusion](https://github.com/61s61min/MS2Fusion) | [link](https://arxiv.org/abs/2507.14643) |
| 45 | 2026 | IRDFusion: Iterative Relation-Map Difference Guided Feature Fusion for Multispectral Object Detection | Pattern Recognition | RGB-T detection (fusion) | Aligned subset (inferred: person/car/bicycle) | [61s61min/IRDFusion](https://github.com/61s61min/IRDFusion) | [link](https://arxiv.org/abs/2509.09085) |
| 46 | 2026 | Physical Adversarial Clothing Evades Visible-Thermal Detectors via Non-Overlapping RGB-T Pattern | CVPR | Adversarial attack on RGB-T detectors | Aligned subset (backgrounds for simulation) | [thu-ml/RGBT-Clothing](https://github.com/thu-ml/RGBT-Clothing) | [link](https://arxiv.org/abs/2605.04675) |
| 47 | 2026 | Distribution-Aligned Multimodal Fusion for Robust Object Detection | CVPR | RGB-T detection (robust fusion) | Official v1 by count ("10,228 pairs") - alignment handling unclear | none found | [link](https://openaccess.thecvf.com/content/CVPR2026/papers/Hao_Distribution-Aligned_Multimodal_Fusion_for_Robust_Object_Detection_CVPR_2026_paper.pdf) |
| 48 | 2026 | DETR is Secretly a Multispectral Detector: Zero-Parameter Adaptation via Semantic Alignment (ZPA-MDETR) | ECCV | RGB-T detection (RT-DETR adaptation) | Aligned subset | [UserXiangYang/ZPA-MDETR](https://github.com/UserXiangYang/ZPA-MDETR) | [link](https://media.eventhosts.cc/Conferences/ECCV2026/pdfs/10379.pdf) |

### 3.2 Summary counts (48 papers)

**By venue**

| Venue | # |
|---|---|
| CVPR | 8 |
| ECCV | 5 |
| ICCV | 5 |
| WACV | 4 |
| CVPRW | 3 |
| AAAI | 2 |
| IJCAI | 2 |
| T-ITS | 2 |
| TCSVT | 2 |
| TNNLS | 2 |
| ACM MM | 2 |
| Pattern Recognition | 2 |
| ICIP | 1 |
| ECCVW | 1 |
| IROS | 1 |
| TPAMI | 1 |
| IV | 1 |
| TIM | 1 |
| TMM | 1 |
| TIV | 1 |
| Information Fusion | 1 |

**By task**

| Task | # |
|---|---|
| RGB-T (multispectral) detection | 25 |
| Pixel-level image fusion | 8 |
| Adversarial attack | 4 |
| Image/modality translation | 4 |
| Thermal-only detection | 3 |
| Domain adaptation | 3 |
| Super-resolution | 1 |

**By FLIR variant**

| Variant | # |
|---|---|
| FLIR-aligned (v1-derived, Zhang et al. 2020) | 26 |
| Official v1 release | 8 |
| RoadScene (v1-derived) | 7 |
| Custom v1 subset / own alignment | 4 |
| Ambiguous v1/v2 | 3 |

**By year (within this table)**

| Year | # |
|---|---|
| 2019 | 1 |
| 2020 | 4 |
| 2021 | 4 |
| 2022 | 7 |
| 2023 | 8 |
| 2024 | 10 |
| 2025 | 9 |
| 2026 | 5 |

Entries with an official (or author-linked) GitHub repo: **37 / 48**.


Notes:
- The 2026 count is partial, and indexing of CVPR/ECCV/NeurIPS 2026 papers is still incomplete.
- RoadScene papers are listed only as a representative sample (7). The real number of fusion papers in important venues that test on RoadScene is far higher, probably over 100.

### 3.3 Useful related papers that are not in the main table (arXiv-only, lower-tier venues, or excluded)

| Year | Paper | Venue | Variant | Code | Why listed |
|---|---|---|---|---|---|
| 2019 | Enhancing Object Detection in Adverse Conditions using Thermal Imaging (Agrawal et al.) | arXiv [1909.13551](https://arxiv.org/abs/1909.13551) | Official v1 | - | early v1 thermal baseline |
| 2021 | Cross-Modality Fusion Transformer for Multispectral Object Detection (**CFT**) | arXiv [2111.00273](https://arxiv.org/abs/2111.00273) (454 citations; not formally published) | Aligned subset | [DocF/multispectral-object-detection](https://github.com/DocF/multispectral-object-detection) | **The YOLOv5 two-stream codebase most later FLIR-aligned papers build on.** Its README gives FLIR-aligned download links. |
| 2021 | Detecting Small Objects in Thermal Images Using SSD (DDSSD) | Automatic Control & Computer Sciences | Official v1 (8,862/1,366) | - | small-object v1 results |
| 2022 | Exploring Thermal Images for Object Detection in Underexposure Regions for Autonomous Driving (Munir et al.) | Applied Soft Computing 121 | Official v1 | - | style-transfer DA on v1 |
| 2022 | Memory-Guided Collaborative Attention for Nighttime TIR Colorization (**MornGAN**) | arXiv [2208.02960](https://arxiv.org/abs/2208.02960) | Official v1 (PearlGAN splits) | [FuyaLuo/MornGAN](https://github.com/FuyaLuo/MornGAN) | same FLIR split family as PearlGAN/FoalGAN/TIRDet/PFGF |
| 2024 | SeaDATE (dual-attention transformer, multimodal detection) | arXiv [2410.11358](https://arxiv.org/abs/2410.11358) | Aligned subset | - | FLIR + LLVIP + M3FD protocol |
| 2024 | HOTCOLD Block (AAAI 2023) | AAAI | **v2** (explicit) | [weihui1308/HOTCOLDBlock](https://github.com/weihui1308/HOTCOLDBlock) | excluded from the main table; evidence of the move to v2 |
| 2022 | Meta-UDA (WACV 2022) | WACV | KAIST + DSIAC only | - | **excluded**: often assumed to use FLIR, but the PDF does not |
| 2020 | Task-conditioned DA for pedestrian detection in thermal (ECCV 2020) | ECCV | KAIST only | [mrkieumy/task-conditioned](https://github.com/mrkieumy/task-conditioned) | **excluded**: no FLIR |
| 2023 | Edge-guided RGB-to-TIR translation (ICRA 2023) | ICRA | GTA + STheReO, no FLIR | [rpmsnu/sRGB-TIR](https://github.com/rpmsnu/sRGB-TIR) | **excluded** |

### 3.4 Hints for the code-mining agents (groups C and E)

- **FLIR-aligned data and annotation sources** inside repos: D3T README (Google Drive `FLIR_ICIP2020_aligned`, VOC format, with new ImageSets), RGBXFusion (`FLIR_Aligned`, COCO-style plus their own day/night split), UniRGB-IR (COCO annotations for `FLIR_align`), CFT/DocF (YOLO format), ICAFusion (says "fixed some bugs and now update the results on the FLIR dataset"), RSDet (`configs/_base_/datasets/FLIR.py`, mmdet), M-SpecGene (`COCO_FLIR/FLIR_ir`, `FLIR_rgb`), YOLOv11-RGBT ([wandahangFY/YOLOv11-RGBT](https://github.com/wandahangFY/YOLOv11-RGBT), `FLIR_aligned-rgbt.yaml`, "FLIR3C" = 3 classes). The original `zonaqiu/FLIR-align` repo cited by Awesome-RGBT-Fusion no longer resolves.
- **Class filtering:** FLIR-aligned keeps 3 classes (person, car, bicycle) and drops dog (MiPa: "dog objects are inadequate for training") and other vehicle. HalluciDet keeps **person only**. Fusion-Mamba describes FLIR as "five categories (people, car, bike, dog, other cars)" but evaluates on aligned 3-class data. The official-v1 thermal works (MMTOD, SSTN, SGADA) use 3 or 4 classes; MMTOD converts to PASCAL-VOC and gives split files for FLIR, FLIR-1/2 and FLIR-1/4.
- **Person-only subsets** for adversarial work: "FLIR person select" (710/301, AAAI 2021) and PEOPLE_FLIR (7,873/2,027, CVPR 2022), both from "FLIR ADAS v1.3".
- **Day-RGB / night-IR splits** for translation: PearlGAN, MornGAN and FoalGAN use `FLIR_datasets/trainA` (day RGB) and `trainB` (night IR), plus pixel-level masks for a test subset. TIRDet and PFGF reuse the PearlGAN `FLIR_NTIR2DC` checkpoints.
- **Custom v1 alignment:** Vadidar et al. (IV 2022) give 7,381 / 1,056 / 2,111 pairs, reused by "Rethinking Early-Fusion" (TIV 2025).
- **RoadScene file names keep the v1 frame ids** (`FLIR_05164`, `FLIR_04602`, ...), so overlap with the v1 train/val split can be checked.

---

## Uncertainties and caveats

1. **Undercounting:** I could not get Google Scholar, Papers with Code (shut down) or OpenAlex full-text counts. The per-year numbers are proxies (anchor-paper citations, title/abstract hits), not exact "papers using v1".
2. **2026 is partial,** and S2 year attribution is sometimes off (it may use the arXiv year).
3. **Ambiguous version:** CVPR 2023 adversarial patches (probably v2, given the group's AAAI 2023 paper), TIRDet (ACM MM 2023; probably aligned v1), LRAF-Net (TNNLS; variant not stated in the abstract). CVPR 2026 "Distribution-Aligned" quotes "10,228 pairs", which is the v1 thermal count, but does not explain how it handles alignment.
4. **"Inferred" variants** (CSSA, M-SpecGene, IRDFusion) come from reported mAP ranges or class lists, not explicit statements.
5. **Code links:** I checked each README over raw.githubusercontent.com on 2026-11-02. These did not resolve on main/master: WaveMamba (`EhanDong/WaveMamba`), `XueZ-phd/Efficient-RGB-T-Early-Fusion-Detection` and `zonaqiu/FLIR-align`. They may live on other branches or have been removed. CF-Deformable-DETR's README is empty. DAMSDet's README is minimal, though the repo exists. CSSA has only an unofficial re-implementation.
6. **Venue labels** come from S2/DBLP metadata. ModTr appears as ECCV 2024 in S2 and in Awesome-RGBT-Fusion. Fusion-Mamba is IEEE TMM vol. 27 (2025). RSDet is IEEE T-ITS (2025).
7. **The main table is broad but not exhaustive.** Many more FLIR-aligned papers exist in TGRS, TCSVT, TIM, KBS, Sensors, Remote Sensing and similar venues. IGT (KBS 2023), SCFR (T-ITS 2024) and EI2Det (TCSVT 2025) are likely FLIR users but were not verified.

---

## Sources (main)

- Dataset page: https://www.flir.com/oem/adas/adas-dataset-form/ ; v2 download guide: https://adas-dataset-v2.flirconservator.com/
- Semantic Scholar Graph API (search/bulk, paper/batch, citations): https://api.semanticscholar.org/
- arXiv API: https://export.arxiv.org/api/query ; GitHub search API; Kaggle and Hugging Face dataset listing APIs (queried 2026-11-02)
- Awesome-RGBT-Fusion (venue/code cross-check): https://github.com/yuanmaoxun/Awesome-RGBT-Fusion
- Per-paper sources: see the "Source" column of the main table and the links in sections 2.3 and 3.3.
