---
name: codex-cli-yolo-task
description: Use when a user explicitly requests a YOLO-mode Codex CLI run in a named project folder and wants the prompt, log, and result saved.
---

# Run a task through Codex CLI in YOLO mode

Use this skill only when the user explicitly asks for a separate Codex CLI run in YOLO mode, or otherwise clearly authorizes disabling the sandbox and approval prompts for this task. The user must identify the project folder. If either point is unclear, ask before launching Codex CLI.

YOLO mode disables the sandbox and approval prompts. The `--cd` option sets Codex CLI's working directory; it does **not** restrict file access to that directory. Do not describe the project folder as a security boundary.

## Before launching

1. Use only the project folder the user named. Verify that it exists and is a directory. Do not infer it from the current working directory. Create it only if the user explicitly asks to start a new project or create that folder.
2. Check the installed CLI's supported flags with `codex exec --help`. Prefer `--yolo`. If the CLI lacks that flag but supports `--dangerously-bypass-approvals-and-sandbox`, use that full flag. If neither is available, stop and report the incompatibility; do not substitute another mode.
3. Keep the CLI's default model and configuration. Do not pass `-m` or `--model`, set a model through `-c`, or change profiles.
4. Prepare the prompt directly from the user's task. Remove only instructions addressed to this assistant about choosing or launching the skill/tool (for example, “run this through Codex CLI”). Preserve the requested work, paths, constraints, and preferences. Do not expand it into a detailed specification or add requirements.

   Do not inspect the project, search its files, or do external research to enrich or reinterpret the task before launching Codex CLI. Do not add research findings, inferred file locations, diagnoses, implementation ideas, or acceptance criteria to the prompt. Keep the prompt to the user's request and details the user supplied; repository investigation is for Codex CLI to do after launch. If the request is ambiguous, preserve it as written rather than researching assumptions into it; ask the user only if the ambiguity prevents carrying out the requested task safely.

## Run and save the artifacts

Save each run's prompt, JSONL execution log, and final response under `codex-prompt-log-result/<repository-name>/<run-folder>` in the current working directory. If the user specifies another artifact directory, use it as the root instead. Use the basename of the user-named project folder as `<repository-name>`. Name `<run-folder>` with the timestamp format `YYYY-MM-DD-HH_MM_SS` followed by a short, lowercase, hyphen-separated description of the task, such as `2020-01-02-17_33_55-add-task-new`. Keep dates out of the artifact filenames: use `prompt.md`, `execution.jsonl`, and `result.md`.

Use `--json` for the event log and `--output-last-message` (or `-o`) for the final response file. Redirect both stdout and stderr to the log. Pass the prepared prompt to Codex CLI on stdin.

```bash
PROJECT_DIR="/absolute/path/to/the/project"
REPO_NAME="$(basename -- "$PROJECT_DIR")"
RUN_STAMP="$(date +%Y-%m-%d-%H_%M_%S)"
# Use a short English slug that describes the user's task.
TASK_SLUG="add-task-new"
RUN_DIR="$PWD/codex-prompt-log-result/$REPO_NAME/$RUN_STAMP-$TASK_SLUG"
PROMPT_FILE="$RUN_DIR/prompt.md"
LOG_FILE="$RUN_DIR/execution.jsonl"
RESULT_FILE="$RUN_DIR/result.md"
mkdir -p "$RUN_DIR"

cat > "$PROMPT_FILE" <<'PROMPT'
<the user's task with only tool-selection instructions removed>
PROMPT

if codex exec --yolo \
  --cd "$PROJECT_DIR" \
  --json \
  --output-last-message "$RESULT_FILE" \
  - < "$PROMPT_FILE" \
  > "$LOG_FILE" 2>&1; then
  CLI_EXIT=0
else
  CLI_EXIT=$?
fi
```

If `codex exec --help` showed only `--dangerously-bypass-approvals-and-sandbox`, use that full option in place of `--yolo`.

After the process exits, record its exit code and artifact paths. Do not open, read, analyze, quote, or summarize the execution log. If the result file exists, read it and return its contents to the user, including when the CLI exits unsuccessfully. If it does not exist, report that fact without inspecting the log.

Do not delete prompts, logs, or results automatically. If the user asks to remove old runs, delete only this skill's artifacts in `codex-prompt-log-result`; clarify the date range if “old” is ambiguous.

Do not run tests or other checks through the CLI unless the user requested them. In the final response, report only actions and checks that actually occurred. Include the project path, `CLI_EXIT`, all artifact paths, and the result file's contents (or state that no result file was created).
