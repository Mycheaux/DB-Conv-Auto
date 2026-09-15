# DB Converter Agent Startup

This file is for Codex, Claude, or another coding agent that opens this repo locally.

When the user types one of the commands below in the agent chat, treat it as an instruction to start the matching workflow. These are chat commands, not shell commands.

## Agent Chat Commands

`!build_custom_NN`

Start the standard automatic mapper and inverter creation workflow. Read `Agent Guide/custom_mapper_inverter_guide.md`. Inspect the existing preprocessed data and determine cohort A and cohort B feature dimensions. If the data are missing, ask where the user's data are saved. If the data are stored somewhere else, update `config/data_path.yaml` after confirming the intended paths. Then update `config/architecture.yaml` and create a mapper and inverter whose input and output dimensions match the data.

`!run_feature_selection`

Start the beta feature-selection workflow. Read `Agent Guide/beta_variable_selection_agent_guide.md` and `feature selection/README.md`. This workflow is for expert/pro agent users only. Read the feature selection form and the cohort documentation, add candidate suggestions in red, and stop for human review. Do not preprocess data or change model code from suggestions alone.

`!build_preprocessed_data`

Start preprocessing after the user has reviewed and saved the feature selection form edited by the agent. Use exactly the variables that remain in the reviewed form. If the user does not want a variable used, they must delete it from the reviewed form before running this command. If they want another variable used, they must add it to the reviewed form before running this command.

## Core Rule

For most users, skip beta feature selection. Use already selected and preprocessed data, then build the custom mapper and inverter from the actual data dimensions.
