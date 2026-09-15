# Beta Variable Selection Agent Guide

## Status

This is a beta workflow. It is for advanced or pro agent users who understand that semantic variable matching from cohort documentation is uncertain and must be human reviewed.

Most users should not use this workflow. For ordinary use, ask the user for already selected, preprocessed data or for a final approved column list, then follow [custom_mapper_inverter_guide.md](custom_mapper_inverter_guide.md) to create the mapper and inverter from the actual data dimensions.

## Purpose

Use this guide when a user wants an agent to read cohort documentation and suggest variables that might be comparable across two cohorts. The output of this step is not model-ready data. It is a reviewed feature selection form that the user must approve before preprocessing.

Start this workflow when the user types `!run_feature_selection` in the agent chat.

After the user reviews and saves the form edited by the agent, the user may type `!build_preprocessed_data` to start preprocessing. At that point, use the variables that remain in the reviewed form. Tell the user before preprocessing that they must delete variables they do not want and add variables they do want before running the preprocessing command.

The current lightweight workflow lives in:

```text
feature selection/
```

The main working file is:

```text
feature selection/Feature selection form.docx
```

The agent may create or update:

```text
feature selection/Feature selection form - with red suggestions.docx
```

## When to Use This Beta Feature

Use it only when all are true:

- The user explicitly wants help mapping or selecting variables.
- The user provides cohort documentation such as PDF, DOCX, XLSX, CSV, TSV, or text codebooks.
- The user accepts that the suggestions are candidates, not final variables.
- The user will review and approve the final variable list.

Do not use it when:

- The user already has preprocessed train, validation, and test arrays.
- The user already has a final column list.
- The task is only to build the mapper and inverter.
- The documentation is missing or too vague to support reliable suggestions.

## Reliable Environment Options

The same workflow can run in several environments, but the agent must keep paths and dependencies explicit.

Local repo or nested checkout:

- Work inside the current DB-converter folder.
- Read and write only under the repo unless the user gives another location.
- Keep the original `Feature selection form.docx` intact when possible and write a red-suggestion copy.
- Use the local repo configs only after user approval.

Codex worktree:

- Treat the worktree as disposable development space.
- Keep feature-selection suggestions as document edits and Markdown notes.
- Do not assume user files outside the worktree exist unless the paths are visible.
- Before model work, verify that the intended data files are inside the active workspace or explicitly accessible.

Notebook or Colab-style environment:

- Use notebooks only for exploration or preprocessing examples.
- Export final selected data as `.npy` or numeric `.csv`.
- Record column order outside the arrays because `.npy` does not preserve names.
- Mirror the final file paths back into `config/data_path.yaml`.

Docker environment:

- Do not edit paths casually. Docker volume mounts must match the top-level config, data, and output directories.
- Put approved preprocessed arrays under the mounted data directory.
- Keep `data_path.yaml` consistent with where the container sees the files, not only where the host sees them.

Across every environment:

- Use document-aware tools for DOCX, PDF-aware tools for PDFs, and spreadsheet-aware tools for XLSX.
- Validate extracted variable names against the real raw data columns before preprocessing.
- Preserve the original form and documentation.

## Workflow

1. Read `feature selection/Feature selection form.docx`.
2. Identify cohort A, cohort B, and the user's minimum requested variables or constructs.
3. Read the documentation files in `feature selection/`.
4. Extract candidate matches using variable names, labels, domains, coding, and scale descriptions.
5. Add suggestions in red in the form's open space or in `Feature selection form - with red suggestions.docx`.
6. Keep the document simple: one red block for cohort A and one red block for cohort B.
7. Mark uncertainty directly in red, especially name conflicts, weak construct matches, reverse coding, missing-value codes, and wave-specific variables.
8. Stop and ask the user to approve, reject, or revise the candidate list.
9. After approval, ask where the raw data files are.
10. Verify the approved variables exist in the raw data.
11. Preprocess only the approved variables.
12. Update `config/data_path.yaml`, `config/architecture.yaml`, and the mapper/inverter architecture only after preprocessing succeeds.

## Suggestion Standards

Good candidate matches usually share:

- the same clinical or conceptual construct
- the same anatomical or functional domain
- similar scale type, such as continuous, ordinal, binary, or count
- compatible direction after recoding
- comparable time window or wave
- relevance to the user's downstream task

Weak matches must be labeled as weak. Do not hide uncertainty.

## Human Approval Gate

Before preprocessing, obtain explicit user approval for:

- final cohort A variables and column order
- final cohort B variables and column order
- raw data file locations
- missingness rule
- categorical recoding rule
- train, validation, and test split proportions
- random seed
- output folder

If approval is incomplete, do not edit converter configs or model code.

## Preprocessing After Approval

Once approved:

- Load raw data.
- Select only the variables that remain in the reviewed form edited by the agent.
- Convert booleans and ordered categories to numeric values.
- Apply the approved missingness rule.
- Min-max normalize numeric features.
- Split into train, validation, and test sets.
- Ensure equal row counts across A and B within each split.
- Save arrays to a project-specific folder under `data/preprocessed/`.
- Save a plain column-order manifest next to the arrays.

Required consistency:

- `x_train.shape[1]` is cohort A feature count.
- `y_train.shape[1]` is cohort B feature count.
- `Mapper_input_size == Inverter_output_size == cohort A feature count`.
- `Mapper_output_size == Inverter_input_size == cohort B feature count`.

## Hand Off to Mapper and Inverter Work

After preprocessing, switch to [custom_mapper_inverter_guide.md](custom_mapper_inverter_guide.md).

At that point, the beta feature-selection task is done. The remaining job is standard DB-converter setup:

- update `config/data_path.yaml`
- update `config/architecture.yaml`
- update or generate `src/model.py`
- verify shapes
- train
- run inference

## Current Example

The current example in `feature selection/` uses DEAS as cohort A and SHARE as cohort B.

The reviewed form should keep:

- one red paragraph for DEAS cohort A suggestions
- one red paragraph for SHARE cohort B suggestions

For the current DEAS and SHARE files, the agent found:

- DEAS `id42_1`, `id42_21`, `id42_22`: moderate physical activity frequency, hours per week, and minutes per week.
- DEAS `id43_1`, `id43_21`, `id43_22`: easy physical activity frequency, hours per week, and minutes per week.
- SHARE `mobility`, `mobilit2`, `mobilit3`: mobility and threshold variants.
- SHARE `adl`, `adl2`: activities-of-daily-living count and threshold variant.
- SHARE `phactiv` or `phinact`: physical inactivity; verify the exact raw-data column name.
- DEAS `x511` wave-prefixed families such as `gc511`, `hc511`, `ic511`, and `kc511`: candidate mobility or ADL limitation items.

These are candidates only until the user approves them.
