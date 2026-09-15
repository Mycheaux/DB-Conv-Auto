# DB Converter Agent Guide

This folder is for Codex, Claude, or another coding agent that needs to adapt this DB-converter repository to a new pair of tabular cohorts.

Most users should bring already selected, preprocessed, numeric data. In that normal path, the agent should inspect the data dimensions, update the YAML files, and create the mapper and inverter accordingly. Start with [custom_mapper_inverter_guide.md](custom_mapper_inverter_guide.md).

Variable selection from cohort documentation is a beta feature for advanced or pro agent users only. It requires careful reading of codebooks, human review, and explicit approval before preprocessing. Use [beta_variable_selection_agent_guide.md](beta_variable_selection_agent_guide.md) only when the user intentionally asks for that workflow or provides a completed feature selection form and documentation.

## Which Guide to Use

- Standard path: use [custom_mapper_inverter_guide.md](custom_mapper_inverter_guide.md) when the user already has selected and preprocessed data, or can provide the final columns directly.
- Beta path: use [beta_variable_selection_agent_guide.md](beta_variable_selection_agent_guide.md) when the user wants an agent to suggest comparable variables from cohort documentation.
- Form location: beta variable selection uses [../feature selection](../feature%20selection), especially `Feature selection form.docx`.

## Agent Chat Commands

Users can type these commands in their Codex or Claude agent chat after downloading the repo locally:

- `!build_custom_NN`: start automatic custom mapper and inverter creation from the available preprocessed data. If the data are missing or saved elsewhere, ask the user for the data location and update `config/data_path.yaml` as needed.
- `!run_feature_selection`: start the beta documentation-based variable suggestion workflow.
- `!build_preprocessed_data`: after human review of the feature selection form, create preprocessed data from the variables that remain in the reviewed form.

## Safety Rule

Do not let beta variable selection automatically decide the final variables. The agent may suggest candidates in red, but the user must approve the final feature list before preprocessing or model changes.
