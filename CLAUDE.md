@AGENTS.md

## Claude Code

- Read `AGENTS.md` at session start and follow the documented agent chat commands.
- Treat `!build_custom_NN`, `!run_feature_selection`, and `!build_preprocessed_data` as project workflow triggers, not shell commands.
- If the user types `run_feature_selection` without the leading exclamation mark, treat it as `!run_feature_selection` and start the beta feature-selection workflow.
- If the user types `build_custom_NN` without the leading exclamation mark, treat it as `!build_custom_NN`.
- If the user types `build_preprocessed_data` without the leading exclamation mark, treat it as `!build_preprocessed_data`.

