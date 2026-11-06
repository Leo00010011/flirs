# FLIR research plan

Datasets: **FLIR ADAS v1** (`Flir v1/`) and **FLIR ADAS v2** (`Flir v2/`).
One agent per group per version (10 agents total). Output format: Markdown.

## Groups

| Group | File | Questions |
|---|---|---|
| A | `01_dataset_overview.md` | Q1 samples & classes · Q4 registration between modalities · Q8 ways to download |
| B | `02_usage_and_publications.md` | Q2 recent usage · Q3 papers in important conferences/journals |
| C | `03_tasks_and_benchmarks.md` | Q5 deep learning tasks · Q9 established benchmarks |
| D | `04_sota_detection.md` | Q6 SOTA values for detection |
| E | `05_known_problems_workarounds.md` | Q7 known problems & established workarounds |

### Q4 extended definition
- Is the official release registered/aligned between RGB and thermal? Resolution, FOV, sync issues.
- Do published/community **registered (aligned) versions** exist? For each: who made it, method
  (manual / homography / learned), number of pairs, classes kept, splits, download link.
- For each registered version: find **at least one paper in an important venue** (CVPR, ICCV, ECCV,
  WACV, BMVC, ICRA, IROS, IEEE T-ITS, TIP, TGRS, TCSVT, Information Fusion, Pattern Recognition...)
  that uses it. Existence proof only, not an exhaustive search (that is Q3).

## Dependencies (3 waves, each version independent)

```
Wave 1:  A (Q1,Q4,Q8)      B (Q2,Q3)
           │   ╲          ╱    │
Wave 2:    │    E (Q7) ◄─┘     C (Q5,Q9)
           │   needs A+B       needs B
            ╲                 ╱
Wave 3:          D (Q6)
               needs A+C
```

- C uses B's paper list to identify tasks and benchmarks.
- E uses A's registration/class facts and B's papers/code repos (classes removed, preprocessing, filtering).
- D uses A's classes/splits and C's benchmark protocols so mAP numbers are comparable.
