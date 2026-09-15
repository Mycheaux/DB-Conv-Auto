# Feature Selection

This folder is intentionally simple. It is an optional human-guided step before DB-converter preprocessing.

This is a beta feature for expert/pro agent users. Most users should skip this folder and provide already selected, preprocessed data.

Use the user's Word form as the working document:

- `Feature selection form.docx`

The user writes the minimum variables or constructs they care about in black. The agent reads the cohort documentation in this same folder, then adds suggestions in red in the open space of the form or in a copy named `Feature selection form - with red suggestions.docx`.

Keep suggestions grouped by cohort: one red block for cohort A and one red block for cohort B. Do not mix candidate variables, documentation notes, and final approved choices in the same paragraph.

Place cohort documentation files directly in this folder unless the user asks for a different organization. Accept PDF, DOCX, XLSX, CSV, TSV, or plain text documentation.

## Agent Chat Commands

Type these commands in the Codex or Claude agent chat:

- `!run_feature_selection`: the agent reads this form and the documentation files, then adds candidate variable suggestions in red.
- `!build_preprocessed_data`: after reviewing the red suggestions, the agent creates preprocessed data from the variables that remain in the reviewed form.

Before running `!build_preprocessed_data`, the user must open the form edited by the agent and save the final list. Whatever remains in the reviewed form will be used. Delete variables there if they should not be used. Add variables there if they should be used.

## Agent Workflow

1. Read `Feature selection form.docx`.
2. Read the documentation files in this folder.
3. Identify the user's required variables or constructs for cohort A and cohort B.
4. For each cohort A entry, suggest related cohort B variables in red under the cohort A block.
5. For each cohort B entry, suggest related cohort A variables in red under the cohort B block.
6. Use variable names, labels, coding notes, value ranges, questionnaire domains, and clinical meaning when judging similarity.
7. Mark weak or uncertain matches clearly in red.
8. Stop and ask the user to approve, delete, or correct the suggestions.
9. Only after approval, ask where the raw data files are.
10. Preprocess the approved columns, min-max normalize them, split them into train, validation, and test sets, and update `config/data_path.yaml` and `config/architecture.yaml`.

Do not create a large folder structure unless the user asks for it. Do not treat agent suggestions as approved variables.

## Human Review Rule

This is a beta feature. It should be used only by expert/pro agent users who want documentation-based variable suggestions.

The agent may suggest candidate variables, but the user must approve the final variable list before preprocessing or model changes.

Before preprocessing, confirm:

- final cohort A variables and column order
- final cohort B variables and column order
- raw data file locations
- missingness rule
- categorical recoding rule
- split proportions
- random seed
- output folder

## After Approval

Once the user approves the variables, this folder's job is finished. Continue with the main mapper/inverter workflow:

- create normalized train, validation, and test arrays
- update `config/data_path.yaml`
- update `config/architecture.yaml`
- create or update the mapper and inverter so their input and output dimensions match the data
