# FLIR research index: navigation guide for agents

Read this file first. It tells you **where each fact lives**, so you can open the right file and line
range instead of reading all ~3,400 lines. Research date for every file: **2026-11-02**.
How the files were produced: [PLAN.md](PLAN.md).

## 1. Layout

```
flirs/
├── INDEX.md                  ← this guide
├── PLAN.md                   ← task groups, dependencies (how the files were produced)
├── Flir v1/                  ← FLIR ADAS v1 (v1.3, 2018/2019) + derivatives FLIR-aligned, RoadScene
│   ├── 01_dataset_overview.md          Q1 counts/classes · Q4 registration · Q8 download/license
│   ├── 02_usage_and_publications.md    Q2 usage trends · Q3 48 top-venue papers + code links
│   ├── 03_tasks_and_benchmarks.md      Q5 DL tasks · Q9 protocols P1–P14
│   ├── 04_sota_detection.md            Q6 SOTA tables, organised by protocol P1–P6
│   └── 05_known_problems_workarounds.md Q7 problems P1–P11 + checklist
└── Flir v2/                  ← FLIR ADAS v2 (2022)
    ├── 01_dataset_overview.md          Q1 · Q4 · Q8
    ├── 02_usage_and_publications.md    Q2 · Q3 tables A–D + code links
    ├── 03_tasks_and_benchmarks.md      Q5 · Q9 protocols P0–P13
    ├── 04_sota_detection.md            Q6 SOTA, protocols P0–P17
    └── 05_known_problems_workarounds.md Q7 problems 1–9 + checklist
```

Every file has the same skeleton: a **TL;DR** near the top, the numbered sections, **Uncertainties**, then **References**.
For a quick answer, read the TL;DR first.

## 2. ⚠ ID namespaces collide: always qualify them

| Label | Meaning | Defined in |
|---|---|---|
| v1 `P1`–`P14` | **evaluation protocols** for v1 | [v1/03 §9.2](Flir%20v1/03_tasks_and_benchmarks.md#L178) |
| v1 `P1`–`P11` in file 05 | **problems** (not protocols) | [v1/05](Flir%20v1/05_known_problems_workarounds.md) |
| v2 `P0`–`P13` | **evaluation protocols** for v2 | [v2/03 §9.3](Flir%20v2/03_tasks_and_benchmarks.md#L140) |
| v2 `P14`–`P17` | extra v2 protocols added by the SOTA file | [v2/04](Flir%20v2/04_sota_detection.md#L26) |
| v2 "Problem 1–9" | problems | [v2/05](Flir%20v2/05_known_problems_workarounds.md) |

So v1 `P1` (FLIR-aligned RGB-T) and v2 `P1` (multi-class on val) are **different protocols**.
When you cite one, write it as `v1-P1` or `v2-P1`.

## 3. Quick facts (verified unless noted)

| Fact | FLIR v1 (v1.3) | FLIR v2 |
|---|---|---|
| Thermal frames | 14,452: train 8,862 / val 1,366 / video 4,224 (no official test) | 30,787 total in the archive (FLIR claims 26,442); thermal 10,742 / 1,144 / video test 3,749 |
| Classes with boxes | person, car, bicycle (includes motorcycles), dog; "Other Vehicle" has 0 boxes | 15 official + undocumented `deer`; many rare classes; the test split has only 10 classes |
| Thermal format | 640×512, 8-bit JPEG + 14-bit-in-16-bit TIFF | the same, plus a different camera in 2 test videos; 1,598 test frames have fake "raw" data |
| RGB annotated? | No | Yes, separately |
| Registered RGB-T? | No. Derivatives: **FLIR-aligned** (5,142 pairs, 4,129/1,013, 3 classes), RoadScene (221 pairs) | No. Only a temporal pair map for the video test; no published aligned version |
| De facto detection benchmark | v1-P1: FLIR-aligned RGB-T, mAP50/75/50:95 | none (fragmented); the only shared metric is person/car AP50 on the video test |
| SOTA headline | ~88.4 mAP50 (WaveMamba, ICCV'25); 50.7 mAP (IRDFusion, PR'26); plateau ~86–88 since 2024 | official baseline YOLOX-m 75.3/77.2 AP50 (person/car); no RGB-T results |
| Official download | FLIR form (login) → v2 guide → release notes → Box link (v1.3, 16.35 GB) | direct official zip, 11.9 GB, MD5 given |
| License | non-commercial research only, no redistribution, so all mirrors are unofficial | FLIR non-commercial; terms page returns 404 |

## 4. Topic → location lookup

### Dataset facts
| Topic | v1 | v2 |
|---|---|---|
| Frame counts per split | [01 §1.1 L45](Flir%20v1/01_dataset_overview.md#L45) | [01 §1.1 L36](Flir%20v2/01_dataset_overview.md#L36) |
| Sensors, bit depth, fps, locations | [01 §1.2 L64](Flir%20v1/01_dataset_overview.md#L64) | [01 §1.2 L69](Flir%20v2/01_dataset_overview.md#L69) |
| Annotation format and label map | [01 §1.3 L80](Flir%20v1/01_dataset_overview.md#L80) | [01 §1.3 L89](Flir%20v2/01_dataset_overview.md#L89) |
| Boxes per class per split | [01 §1.4 L97](Flir%20v1/01_dataset_overview.md#L97) | [01 §1.4 L97](Flir%20v2/01_dataset_overview.md#L97) |
| Version history / v1 vs v2 | [01 §1.5 L130](Flir%20v1/01_dataset_overview.md#L130) | [01 L20](Flir%20v2/01_dataset_overview.md#L20) |
| Official vs Kaggle (MD5-identical) | [01 §1.6 L151](Flir%20v1/01_dataset_overview.md#L151) | — |
| Registration status | [01 §4.1 L180](Flir%20v1/01_dataset_overview.md#L180) | [01 §4.1 L132](Flir%20v2/01_dataset_overview.md#L132) |
| Registered versions + one top-venue paper using each | [01 §4.2–4.3 L203](Flir%20v1/01_dataset_overview.md#L203) | [01 §4.2 L143](Flir%20v2/01_dataset_overview.md#L143) (none exist) |
| Download, mirrors, license, link status | [01 §8 L240](Flir%20v1/01_dataset_overview.md#L240) | [01 §8 L167](Flir%20v2/01_dataset_overview.md#L167) |

### Literature
| Topic | v1 | v2 |
|---|---|---|
| Usage per year + methodology | [02 §2 L19](Flir%20v1/02_usage_and_publications.md#L19) | [02 §1–2 L20](Flir%20v2/02_usage_and_publications.md#L20) |
| Is it being replaced? (LLVIP, M3FD, KAIST) | [02 §2.3 L88](Flir%20v1/02_usage_and_publications.md#L88) | [02 §2.2 L68](Flir%20v2/02_usage_and_publications.md#L68) |
| Paper table (venue, task, variant, code) | [02 §3.1 L114](Flir%20v1/02_usage_and_publications.md#L114) (48 papers) | [02 §3.1–3.4 L89](Flir%20v2/02_usage_and_publications.md#L89) (tables A: v2, B: ambiguous, C: lower-tier, D: actually aligned-v1) |
| Counts per venue / task | [02 §3.2 L167](Flir%20v1/02_usage_and_publications.md#L167) | in the TL;DR |
| **GitHub repos to mine** | [02 §3.4 L252](Flir%20v1/02_usage_and_publications.md#L252) | [02 §4 L189](Flir%20v2/02_usage_and_publications.md#L189) |

### Tasks, protocols, SOTA
| Topic | v1 | v2 |
|---|---|---|
| Task overview (mainstream vs niche) | [03 §5.0 L45](Flir%20v1/03_tasks_and_benchmarks.md#L45) | [03 §5.1 L40](Flir%20v2/03_tasks_and_benchmarks.md#L40) |
| Official leaderboard? (Papers with Code / OpenCodePapers) | [03 §9.1 L166](Flir%20v1/03_tasks_and_benchmarks.md#L166) | [03 §9.1 L122](Flir%20v2/03_tasks_and_benchmarks.md#L122) |
| **Protocol definitions table** | [03 §9.2 L178](Flir%20v1/03_tasks_and_benchmarks.md#L178) | [03 §9.3 L140](Flir%20v2/03_tasks_and_benchmarks.md#L140) |
| Protocol inconsistencies (evaluator, resolution, "mAP" = AP50) | [03 §9.4 L266](Flir%20v1/03_tasks_and_benchmarks.md#L266) | [03 §9.4 L170](Flir%20v2/03_tasks_and_benchmarks.md#L170) |
| Main SOTA table | [04 §1.1 L53](Flir%20v1/04_sota_detection.md#L53) (v1-P1) | [04 §1–4](Flir%20v2/04_sota_detection.md#L52) (one table per protocol) |
| SOTA over time | [04 §1.3 L127](Flir%20v1/04_sota_detection.md#L127) | — |
| Thermal-only SOTA | [04 §3 L188](Flir%20v1/04_sota_detection.md#L188) (P2), [§5 L238](Flir%20v1/04_sota_detection.md#L238) (P4) | [04 §1–4](Flir%20v2/04_sota_detection.md#L52) |
| Misalignment-robust / day-night | [04 §2 L148](Flir%20v1/04_sota_detection.md#L148) | — |
| Open-vocabulary / zero-shot | — | [04 §5c L300](Flir%20v2/04_sota_detection.md#L300) |
| Cross-protocol person/car AP50 | — | [04 §7 L341](Flir%20v2/04_sota_detection.md#L341) |
| **Suspicious / non-comparable numbers** | [04 §8 L294](Flir%20v1/04_sota_detection.md#L294) | [04 §6 L319](Flir%20v2/04_sota_detection.md#L319) |

### Problems and practice
| Topic | v1 (file 05) | v2 (file 05) |
|---|---|---|
| Annotation errors / re-annotations | [P1 L62](Flir%20v1/05_known_problems_workarounds.md#L62) | [Prob 1 L46](Flir%20v2/05_known_problems_workarounds.md#L46) |
| Class imbalance; **classes kept per paper/repo** | [P2 L122](Flir%20v1/05_known_problems_workarounds.md#L122) (table L146) | [Prob 2 L85](Flir%20v2/05_known_problems_workarounds.md#L85) (table L103) |
| Small objects / resolution | [P3 L185](Flir%20v1/05_known_problems_workarounds.md#L185) | in Prob 1 |
| No registration / pairing | [P4 L210](Flir%20v1/05_known_problems_workarounds.md#L210) | [Prob 4 L175](Flir%20v2/05_known_problems_workarounds.md#L175) |
| Residual misalignment in FLIR-aligned | [P5 L241](Flir%20v1/05_known_problems_workarounds.md#L241) | — |
| Missing RGB frames | [P6 L274](Flir%20v1/05_known_problems_workarounds.md#L274) | [Prob 5 L207](Flir%20v2/05_known_problems_workarounds.md#L207) |
| 8-bit vs 16-bit raw | [P7 L298](Flir%20v1/05_known_problems_workarounds.md#L298) | [Prob 6 L229](Flir%20v2/05_known_problems_workarounds.md#L229) |
| Splits, leakage, test-set checkpointing | [P8 L339](Flir%20v1/05_known_problems_workarounds.md#L339) | [Prob 3 L137](Flir%20v2/05_known_problems_workarounds.md#L137), [Prob 7 L262](Flir%20v2/05_known_problems_workarounds.md#L262) |
| License / links | [P9 L392](Flir%20v1/05_known_problems_workarounds.md#L392) ⚠ stale, see §5 | [Prob 8 L292](Flir%20v2/05_known_problems_workarounds.md#L292) |
| Domain shift | [P10 L425](Flir%20v1/05_known_problems_workarounds.md#L425) | [Prob 9 L321](Flir%20v2/05_known_problems_workarounds.md#L321) |
| v1/v2/aligned confusion | [P11 L455](Flir%20v1/05_known_problems_workarounds.md#L455) | — |
| **Recommended practice checklist** | [L478](Flir%20v1/05_known_problems_workarounds.md#L478) | [L356](Flir%20v2/05_known_problems_workarounds.md#L356) |

Line numbers are correct as of 2026-11-02 and shift if a file is edited. If one is off, use the
heading search in §6.

## 5. Errata: later findings override earlier files

Not yet applied in the files themselves. **When two files disagree, trust the one listed here.**

| File (stale claim) | Correct value | Source of the correction |
|---|---|---|
| [v1/05 L47, L398](Flir%20v1/05_known_problems_workarounds.md#L398): "official v1 link returns 404, page serves v2 only" | v1 is still officially downloadable (Box link via the v2 guide's release notes); the 404s came from FLIR blocking bots | [v1/01 §8.1](Flir%20v1/01_dataset_overview.md#L242) |
| [v1/02](Flir%20v1/02_usage_and_publications.md#L114) (row #17): CVPR 2023 learnable-patches attack marked "ambiguous v1/v2" | v1 (same 7,873/2,027 v1.3 person split as CVPR 2022) | v1/03 |
| [v1/03 L284](Flir%20v1/03_tasks_and_benchmarks.md#L284): CVPR'26 "Distribution-Aligned" ≈ 94 mAP50 | **79.5 ± 0.7** mAP50 (Table 1); still excluded from v1-P1 | v1/04 |
| [v2/03](Flir%20v2/03_tasks_and_benchmarks.md#L140): MS-YOLO evaluated on test | evaluated on **official val** (1,144 frames) | v2/04 |
| v2/02, v2/03: SS-DC "COCO mAP" | probably AP50 | v2/04 |
| v2/02 table A | add **MODIPHY / YOLO Phantom (ICIP 2024)** as a confirmed v2 paper | v2/03 |

## 6. Search recipes

Run these from `flirs/`. The folder names contain spaces, so quote the paths.

```bash
# All section headings with line numbers (a table of contents for every file)
grep -n -E '^#{1,3} ' "Flir v1"/*.md "Flir v2"/*.md

# Where a method or paper appears, across both versions
grep -n -i 'icafusion' "Flir v1"/*.md "Flir v2"/*.md

# Rows of the v1 protocol table
grep -n -E '^\| *\**P[0-9]+' "Flir v1/03_tasks_and_benchmarks.md"

# Every GitHub repo cited
grep -o -h -E 'github\.com/[A-Za-z0-9_.-]+/[A-Za-z0-9_.-]+' "Flir v1"/*.md "Flir v2"/*.md | sort -u

# Unverified or inferred claims (to re-check or treat with caution)
grep -n -i -E 'unverified|inferred|\(ambiguous' "Flir v1"/*.md "Flir v2"/*.md
```

Useful keywords: `FLIR-aligned`, `RoadScene`, `video test`, `raw`/`TIFF`/`AGC`, `dog`, `Other Vehicle`, `deer`,
`mAP50`, `AP50`, `pycocotools`, `101-point`, `checkpoint`, `random split`, `MD5`, `Box`, `license`.

## 7. Conventions and caveats for agents

- **Verification marks:** `[verified]` / "verified from archive" / "counted" mean checked against the dataset files.
  "(unverified)" or "(inferred)" means don't rely on it without re-checking.
- **Usage counts in the 02 files are lower bounds.** Scholar was blocked and the OpenAlex quota was exhausted.
- **SOTA comparisons:** compare numbers only within one protocol ID. Check the flags sections (v1/04 §8, v2/04 §6) before quoting any number.
- **Local data:** the official v1.3 download is at `/home/leonardo.ulloa/projects/data/FLIR Santa Barbara Regional Thermal Dataset`.
  Treat it as read-only. No local copy of v2 is recorded.
- **Updating these files:** keep the skeleton (TL;DR → sections → Uncertainties → References), cite links, and
  add an entry to §5 Errata here when a finding contradicts another file.
