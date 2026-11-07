# Codex CLI YOLO Task

[![skills.sh](https://skills.sh/b/evgenyigumnov/codex)](https://skills.sh/evgenyigumnov/codex)

An agent skill for running an explicitly authorized programming task in a separate Codex CLI process with YOLO mode. It saves the submitted prompt, JSONL execution log, and final response as timestamped files.

## Install

```bash
npx skills add evgenyigumnov/codex --skill codex-cli-yolo-task
```

## Use

Ask your agent to run the task through Codex CLI in YOLO mode and provide the project folder. For example:

> Use the Codex CLI YOLO task skill to implement the requested feature in `/path/to/project`.

YOLO mode disables the sandbox and approval prompts. The skill requires clear authorization for that mode and a project folder named by the user. The CLI's `--cd` option sets its working directory but does not confine file access to that folder.

Run artifacts are saved in `codex-prompt-log-result` in the current working directory unless another location is requested.
