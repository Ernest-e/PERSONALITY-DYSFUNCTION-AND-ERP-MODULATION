# PERSONALITY-DYSFUNCTION-AND-ERP-MODULATION
code for statistical analysis

Analysis notebooks for a spatial cueing study of saccadic behavior, target-locked P3a, and cue-locked contingent negative variation (CNV). The analyses relate responses to 80% and 50% cue-validity contexts to self and interpersonal functioning and maladaptive personality traits.

The notebooks implement the analyses reported in the manuscript and generate numerical and graphical outputs when run. Study data are required to reproduce the results. Synthetic examples, where included, illustrate code behavior and are not participant data.

## Files and execution order

Keep the notebooks and `requirements.txt` in the same project folder. Open JupyterLab from that folder so that relative paths resolve consistently.

| File | Purpose | Main output |
|---|---|---|
| `00_saccade_detection.ipynb` | Automatic detection and visual review, one participant at a time | `<participant_id>_saccades_auto.xlsx` |
| `01_saccades.ipynb` | Classify reviewed saccades and calculate behavioral measures | `saccades_summary.xlsx`; Table 2 |
| `02_eeg_features.ipynb` | Extract P3a and CNV amplitudes from prepared EEG recordings | `eeg_features.xlsx`; participant condition averages in `evoked/` |
| `03_erp_behavior_statistics.ipynb` | Behavioral tests, personality associations, and P3a models M0–M2 | `erp_behavior_statistics.xlsx`; Tables 1–5; Figures 2–3 |
| `04_cnv_statistics.ipynb` | CNV models M3–M6, reliability, and CNV–P3a coupling | `cnv_statistics.xlsx`; Table 6; Figures 4–5 |
| `05_saccade_overlap_control.ipynb` | Assess overlap of target- and saccade-related EEG activity | `saccade_overlap_control.xlsx`; Table S1 |
| `requirements.txt` | Python dependencies | — |
| `.gitignore` | Exclude local data, results, and environment files from Git | — |

Run 00 → review annotations → 01 → 02 → 03. Notebooks 04 and 05 then use the preceding outputs. Notebook 05 does not require the output of 04.

## Installation

Use Python 3.12 in a dedicated environment. On macOS or Linux:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m jupyter lab
```

On Windows, create the environment with `py -3.12 -m venv .venv` and activate it in PowerShell with `.venv\Scripts\Activate.ps1`; then run the same `python -m ...` commands. Select the kernel belonging to this environment. `ipywidgets` supports the interactive trial browser in notebook 00; the static trial view is also available.

Each notebook has a configuration cell near the beginning. Replace path placeholders, check participant identifiers and filename patterns, enable the analysis flag, restart the kernel, and run all cells in order. Notebook 00 uses `RUN_DETECTION`; 01–02 use `RUN_STUDY_ANALYSIS`; 03–05 use `RUN_ANALYSES`. Enable the export flag to create files: `SAVE_AUTOMATIC_XLSX` in 00 or `EXPORT_OUTPUTS` in 01–05. The overwrite flags default to `False`.

Set input and output paths to private folders outside the public code folder. A path such as `PATH_TO_EEG_OUTPUTS` is a placeholder, not a folder to create literally. Review quality-control tables before proceeding to the next notebook. Numerical results are calculated without intermediate rounding; formatting is applied when presenting results.

## Required data

### 1. CNT recording and reviewed saccade annotations

Notebook 00 reads one Neuroscan CNT recording with horizontal EOG channels `EOG1` and `EOG2` and task annotations. Check `EOG_ANODE`, `EOG_CATHODE`, and `SIGN_RIGHT` for the recording's polarity. `PARTICIPANT_ID` is a pseudonymous identifier used consistently across all inputs.

Preserve the automatic workbook. Make a separate reviewed copy named `<participant_id>_saccades.xlsx` for notebook 01. Review detections visually and correct `latency_ms` and `direction` in the first worksheet. Keep every trial, including undetected responses, and preserve trial identifiers and event codes. Keep the event-alignment and recording metadata sheets for reference.

| Required column | Definition |
|---|---|
| `trial` | Zero-based retained target-epoch identifier, unique within the participant |
| `event` | Experimental target code: 20, 21, 22, 23, 30, 31, 32, or 33 |
| `latency_ms` | Reviewed onset relative to target onset, in milliseconds; negative values allowed; blank if undetected |
| `direction` | −1 for left, +1 for right; 0 or blank if undetected |

Do not use latency 0 to mean missing: it denotes onset exactly at the target. Notebook 01 recalculates detection, correctness, and behavioral categories from these columns; manually assigned categories are not required.

The `trial` identifier is not an EEG sample index. Notebook 02 checks complete target counts and event order before linking reviewed annotations to EEG data. Dropped target events must be resolved before combining the modalities.

### 2. Prepared continuous EEG files

Notebook 02 requires two continuous FIF versions per participant:

- P3a input prepared with a 1–30 Hz passband.
- CNV input prepared with a 0.05–30 Hz passband that preserves slow activity.

Apply the intended EEG reference and artifact-component correction before creating these inputs. Notebook 02 performs epoching, baseline correction, and epoch rejection; it does not perform CNT-to-FIF conversion, ICA, filtering, or resampling. These preprocessing steps must be documented with the data.

Both FIF versions must have the same sampling frequency, first-sample index, and full target-event sequence in the same sample coordinate system. Retain task annotations, channel names, and electrode coordinates for scalp maps. Set `ERP_FILE_PATTERN` and `CNV_FILE_PATTERN` so that each participant matches exactly one file. Notebook 05 uses the P3a FIF recordings, including horizontal EOG needed for continuous saccade detection.

The target codes distinguish cue and target direction:

| Target codes | Condition | Cue–target relationship |
|---|---|---|
| 20, 30 | `valid_80` | Same side, 80% context |
| 21, 31 | `invalid_80` | Opposite sides, 80% context |
| 22, 32 | `valid_50` | Same side, 50% context |
| 23, 33 | `invalid_50` | Opposite sides, 50% context |

Cue codes 12 and 13 indicate left and right, respectively. Codes 20–23 have a left target; codes 30–33 have a right target. In notebook 02, cues are matched to the first following target before another cue, with a cue–target interval of 1.25–1.60 s.

### 3. Participant characteristics and questionnaire scores

Notebook 03 reads one Excel table with one row per participant, including participants without questionnaire data. Leave unavailable scores blank. Set `PARTICIPANT_FILE` and `PARTICIPANT_SHEET`. In `COLUMN_MAP`, each key is an analysis variable and its value is the corresponding column header in the input file.

| Analysis variable | Contents |
|---|---|
| `subject` | Pseudonymous identifier matching the EEG and behavioral inputs |
| `age` | Age in years |
| `diagnostic_group` | Diagnostic label mapped by `DIAGNOSTIC_LABEL_MAP` |
| `identity`, `self_direction`, `empathy`, `intimacy` | SIFS domain scores, before analysis standardization |
| `negative_affectivity`, `detachment`, `dissociality`, `disinhibition`, `anankastia` | FFiCD domain scores, before analysis standardization |

The analysis groups are `no_mental_disorder`, `personality_disorder`, and `schizotypal_disorder`. Map the labels in the input table explicitly; unmapped labels stop the analysis. The notebooks do not score individual questionnaire items.

## Analysis conventions

### Saccades and EEG selection

- A detected saccade has a finite latency and direction −1 or +1. Anticipatory saccades have latency <90 ms, express saccades 90–<120 ms, and regular saccades ≥120 ms.
- Response latency includes correctly directed express and regular saccades. Anticipatory and direction-error percentages use detected saccades as their denominator.
- The anticipatory probability contrast pools valid and invalid trial counts within each context, then subtracts the 50% percentage from the 80% percentage. It is expressed in percentage points.
- CNV excludes undetected and pretarget saccades (<0 ms), retaining detected saccades with latency ≥0 ms regardless of direction. Thus a 0–89 ms saccade is behaviorally anticipatory and passes the CNV timing criterion.
- P3a uses all EEG-retained target trials, without the CNV saccade exclusions. The measurement is the mean at F3, Fz, F4, C3, Cz, and C4 from 240–320 ms.
- CNV is measured at Fz from 350–1,000 ms after the cue. Both cue sides are pooled. At least 30 retained trials are required in each probability context; additional participant exclusions are listed explicitly in notebook 02.
- EEG epoch rejection uses a 100 µV **peak-to-peak** threshold, not an absolute ±100 µV amplitude criterion. P3a and CNV epochs have separate rejection and inclusion records.

### Scores, models, and scaling

In notebook 03, each SIFS domain is standardized across participants with an observed score, using the sample SD (`ddof=1`). Each pair of domain z-scores is averaged, and the resulting self or interpersonal composite is standardized again. Pair means use the available domain when only one is observed; score coverage is reported. Table 1 uses raw domain-pair means. Age is standardized across the EEG cohort; FFiCD domains are standardized across participants with available scores.

Notebook 04 reuses the model scores exported by 03 in `Participant_scores`, without restandardizing them in the smaller CNV sample. Notebook 05 uses `self_comp` from the `Participant_ERP` sheet of 03 for the participant-level rank correlations.

Validity is coded −0.5 for valid and +0.5 for invalid trials; probability is coded −0.5 for 50% and +0.5 for 80%. Probability modulation is `(invalid_80 − valid_80) − (invalid_50 − valid_50)`. P3a models use participant random intercepts and slopes for validity, probability, and their interaction. Mixed models use REML with the Powell optimizer; their exact formulas, samples, and diagnostics are exported.

For CNV–P3a coupling, `cnv_w_z` is the participant-centered CNV amplitude divided by the pooled sample SD of the deviations across all paired trials. Centering pools both contexts. `cnv_b_z` standardizes participant means repeated across paired trial rows and is therefore weighted by paired-trial counts. These predictors are constructed before model-specific exclusion for missing scores. Coefficients involving `cnv_w_z` are per one pooled within-participant SD; increasing this predictor means less negative CNV. The separately exported negativity contrasts reverse the sign for interpretation.

Notebook 03 reports the defined Benjamini–Hochberg families for behavioral correlations, FFiCD moderation, and diagnostic-group comparisons. Notebook 04 contains exploratory analyses with unadjusted p-values. Inspect model convergence and warnings alongside coefficients and intervals; a returned coefficient table alone does not establish a satisfactory fit.

### Saccade-overlap control

Set `ERP_FIF_DIR`, `ERP_FILE_PATTERN`, `EEG_FEATURES_FILE`, and `ERP_STATISTICS_FILE` in notebook 05. The default FIF pattern is `{subject}*ica.fif`; change it to match exactly one prepared recording per participant. The two workbooks come from notebooks 02 and 03.

The four estimation methods are `epoch_all`, `epoch_detected_ge90`, `regression_targets`, and `regression_targets_saccades`: conventional ERP averaging, saccade-latency-restricted averaging, target-only regression, and regression with separate left- and right-saccade predictors. Notebook 05 detects saccades from the continuous 1–40 Hz HEOG signal. For `epoch_detected_ge90`, it selects the first detected saccade with 0 < latency ≤600 ms after each target and retains that trial only if this latency is ≥90 ms. It does not require correct saccade direction. The same continuous detector supplies all saccade events for regression. These events serve a different purpose from the manually reviewed target-epoch annotations in 00–01 and do not replace those annotations.

The EEG is not filtered or resampled in notebook 05. It preserves channel types stored in each FIF file; notebook 02 explicitly classifies A1/A2 as miscellaneous channels. This distinction can change which channels contribute to EEG rejection. Regression processing uses 1-s blocks and a 100 µV peak-to-peak artifact threshold, with `tstep=1` and `decim=1`. BAD annotations are used by epoching but not by the regression path. `Main_ERP_comparison` checks conventional averages against the main ERP results, including amplitudes, retained counts, and sample selection. Differences are flagged for inspection rather than automatically corrected.

`saccade_overlap_control.xlsx` contains Table S1, method agreement, group probability tests, participant contrasts, condition amplitudes, the comparison with the main ERP analysis, trial QC, continuous saccade events, processing QC, warnings, software versions, input files, and settings. Set `OUTPUT_DIR` and enable `EXPORT_OUTPUTS` to write this workbook. `SAVE_EVOKEDS` defaults to `False`; when enabled, target estimates are also saved as `evoked/<subject>_<method>-ave.fif`.

In `Condition_amplitudes`, `n_events` is the retained epoch count for averaging. For regression, it is the number of supplied events of that type before regression sample rejection; it is not an accepted-trial count.

## Outputs and reproducibility

The workbooks contain numerical results, participant/trial counts, quality-control records, model diagnostics, analysis settings, and software versions. Notebook 02 saves `<subject>_erp-ave.fif` and `<subject>_cnv-ave.fif` for figure generation. Keep `SAVE_EVOKEDS = True` when the later waveform and scalp-map figures are required. Figure exports in 03–04 are vector PDF and 600-dpi TIFF.

The principal handoffs are `Trial_annotations` from 01; `ERP_trials`, `CNV_trials`, and `ERP_CNV_pairs` from 02; and `Participant_scores` and `Participant_ERP` from 03. Notebook 05 also requires `Event_alignment` and `EEG_input_QC` from 02. Preserve workbook sheet names and column names. Hash checks in downstream analyses verify that the EEG workbook matches the one used to construct earlier results.

Runtime and model convergence depend on the number of participants and trials and on the local numerical libraries. Use the recorded software versions when comparing runs. Confirm participant counts, trial counts, estimates, and figure values against the manuscript after executing the notebooks with study data.

## Data access and public repository contents

The study data are not publicly available because they contain sensitive information from individuals undergoing forensic psychiatric evaluation. Anonymized data may be made available from the corresponding author, Ernest I. Rabinovich (`rabinovichernest@gmail.com`), upon reasonable request and subject to institutional approval.

The study was not preregistered.

The public repository contains code and documentation. Keep raw recordings, reviewed annotations, participant tables, generated workbooks, participant FIF averages, and participant-level notebook outputs outside it. Before committing notebooks, clear cell outputs and saved widget state that contain participant information or local data paths. `.gitignore` helps exclude common data and output files but does not remove files already tracked by Git.
