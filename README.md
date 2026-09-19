# TFM Code: Structural Inertia in Academic Collaboration — Gender and Co-authorship Networks in Spanish Social Science

This holds the code for my Master's Thesis in Computational Social Science at UC3M: a comparative network analysis testing whether gendered patterns of co-authorship network centrality and homophily in Spanish social science departments moved around two recent gender-equality laws, across four Madrid-based universities (UC3M, UCM, URJC, UAM).

## Repository structure

```
TFM-UC3M-SAM/
├── README.md
├── Tim_pipeline.html             # HTML render after running the quarto document
├── Thesis.Rproj                  # open this in RStudio to get the correct working directory
├── session_info.txt              # exact R + package versions used
├── tfm_pipeline.qmd               # the full pipeline: fetch, networks, models, homophily, community detection, export
└── data_cache/                   # created automatically on first run; caches all OpenAlex/genderize API pulls and results
    ├── *_core_works_*.rds
    ├── *_broad_works_edges_lean_v2_*.rds
    ├── genderize_cache.rds
    ├── all_model_coefficients.csv
    ├── all_model_fit_stats.csv
    └── ...
```

## 1. Get the data

No manual download is required — all data is fetched programmatically from the [OpenAlex](https://openalex.org/) API on first run, with gender inferred separately via the [genderize.io](https://genderize.io/) API.

1. (Recommended, not required) Get an OpenAlex API key and a genderize.io API key, and set them as environment variables before starting R:
   ```r
   Sys.setenv(OPENALEX_API_KEY = "your-key-here")
   Sys.setenv(GENDERIZE_API_KEY = "your-key-here")
   ```
   The pipeline runs without either key, just at lower rate limits.
2. Every fetch stage is cached to `data_cache/` and checkpointed in per-batch `.rds` files, so an interrupted run resumes from the last completed batch rather than restarting. If `data_cache/` is shared separately (e.g. archived alongside this repository), dropping it in next to the `.qmd` files skips re-fetching entirely.

## 2. Set up the R environment

Requires R ≥ 4.2 and the packages `openalexR`, `dplyr`, `tidyr`, `stringr`, `igraph`, `purrr`, `MASS`, `pscl`, `lme4`, `sandwich`, `lmtest`, `performance`, `jsonlite`, and `ggplot2`. The exact versions used to produce the thesis results are recorded in `session_info.txt`.

## 3. Run the scripts

1. `tfm_pipeline.qmd` — the full pipeline, run top to bottom. Covers data fetch, network construction, gender/seniority variables, degree and betweenness models, homophily, community detection, the temporal extension, and results/figure export. End to end, this takes roughly an hour on the author's machine, almost entirely spent on the mixed-effects model fits.

## Notes

1. **Memory-safe fetching.** UCM and UAM are too large to download via the "full works" approach without exceeding available RAM, so their data is fetched in a lean, per-batch format (only the fields needed, discarded and rebuilt into a flat edge list immediately after each batch) rather than held in memory as complete OpenAlex work objects.
2. **Convergence checking.** Every mixed-effects model fit is validated through a tiered check (relative gradient, then a warm-started refit, then a full multi-optimizer sweep if needed).
3. `data_cache/` is created automatically; nothing needs to be created by hand before running the pipeline.
