# Feature Selection

This is an optional beta workflow for expert/pro agent users. Most users should skip it and provide already selected, preprocessed data.

Use this folder when you want an agent to help suggest comparable variables across two cohorts before preprocessing.

## What To Put Here

- `Feature selection form.docx`
- Cohort A documentation, such as a data dictionary, codebook, PDF, Excel sheet, CSV, TSV, DOCX, or text file
- Cohort B documentation in the same kind of format

Use the exact variable IDs from the documentation or raw data whenever possible. Variable labels are helpful, but IDs are what the agent should use for preprocessing.

## Steps

1. Open `Feature selection form.docx`.
2. Fill in the minimum variables or constructs you require for each cohort.
3. Include each variable ID when you know it.
4. Copy or place the cohort documentation files into this folder.
5. In Codex or Claude, run:

```text
!run_feature_selection
```

6. The agent reads the form and documentation, then adds suggested comparable variables in red.
7. Review the edited form carefully. Delete anything you do not want used, add anything missing, and make sure the final kept variables have IDs.
8. Save the reviewed form.
9. Run:

```text
!build_preprocessed_data
```

The variables left in the reviewed form are the variables the agent will use for preprocessing.

## Human Review Rule

Do not treat red suggestions as approved automatically. The user must approve the final variable list before preprocessing or model changes.

Before preprocessing, confirm:

- final cohort A variable IDs and column order
- final cohort B variable IDs and column order
- raw data file locations
- missingness and categorical recoding rules
- split proportions and random seed
- output folder

After approval, the agent should create normalized train, validation, and test arrays, then update `config/data_path.yaml` and `config/architecture.yaml` so the mapper and inverter workflow can run.
