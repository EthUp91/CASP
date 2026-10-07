# Gongguan demand-stress paper

**Title:** Demand-Dependent Operating Ranges of Traffic-Signal Controllers at Gongguan Roundabout

- [Four-page PDF](Gongguan_paper.pdf)
- [Editable Word document](Gongguan_paper.docx)
- [Editable Markdown draft](paper_draft.md)
- [Validated paper package](Gongguan_paper_ready_package.zip)

Open a file and choose **Download raw file** to download it from GitHub.
The package includes editable LaTeX, publication figures (PDF/PNG), seed-level
metrics, complete result tables, selected paired statistics, validation reports,
and reproducible analysis/build scripts. The PDF uses a generic two-column
four-page layout; Word pagination may differ.

## Research status

The study evaluates Fixed-Time and newly trained source-based DQN, PPO and RPPO
policies across100–170% demand using matched seeds42–46 (160 completed runs).
Each RL policy was trained for512,000 transitions. All160 raw runs were audited;
confidence intervals use five seed-level observations, not pooled vehicles.

The original repository's DDQN label actually calls plain SB3 DQN. Published
checkpoints were unavailable; CPU, portable paths and independent SUMO workers
were necessary adaptations. These results do not claim exact published-model
reproduction or a universal algorithm ranking. Fixed-Time remains competitive
at moderate demand; RPPO has the highest extreme-demand mean completion, with
important five-seed uncertainty.

Before submission, complete the original publication's bibliographic citation,
replace anonymous author metadata, and apply the selected venue template.
No additional experiment is required for the narrow descriptive case-study claim.

Model binaries and the full raw simulation archive are not included in this
paper upload. The package README documents the complementary archives needed
for re-running the full raw-data/artifact audit.
