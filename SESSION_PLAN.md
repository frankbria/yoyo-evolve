## Session Plan

### Task 1: Add automatic API error retry with exponential backoff
Files: src/prompt.rs, src/main.rs
Description: When `run_prompt` detects an `AgentEvent::AgentEnd` where the stop_reason is `StopReason::Error` (indicating an API error like rate limit, server error, or network issue), automatically retry the prompt with exponential backoff instead of just showing the error to the user. Implement a retry wrapper around the prompt execution in `run_prompt()`:
- Add a configurable max retry count (default 3 retries)
- Use exponential backoff: 1s, 2s, 4s delays between retries
- Only retry on errors that look transient (rate limit 429, server errors 5xx, network errors) — don't retry on auth errors or invalid requests
- Show a dim message like `"  ⚡ retrying (attempt 2/3, waiting 2s)..."` so the user knows what's happening
- After exhausting retries, show the final error as we do today
- Add tests for the retry delay calculation and the error classification logic (is_retriable_error)
- This is the #1 real-world UX gap vs Claude Code — transient API failures should be invisible to the user
Issue: none

### Task 2: Add /search command to search conversation history
Files: src/main.rs
Description: Add a `/search <query>` REPL command that searches through the current conversation history for messages containing the query string (case-insensitive). Display matching messages with their index, role, and a preview showing the matched context. This complements `/history` (which shows all messages) by letting users find specific content in long conversations. Add to KNOWN_COMMANDS, /help text, and write tests for the search matching logic.
Issue: none

### Task 3: Set up mutation testing with cargo-mutants
Files: (no src changes — creates a config file and documents in guide)
Description: Install and run `cargo-mutants` against the codebase to assess test quality. Create a `mutants.toml` config file to exclude known-acceptable mutants (e.g., cosmetic formatting functions, ANSI color codes). Document in the user guide how to run mutation testing locally. Add a note in the README or CONTRIBUTING about test quality expectations. Don't add to CI yet — mutation testing is slow and should be run manually at first. The goal is to identify weak spots in the test suite and write targeted tests to cover them.
Issue: #36

### Issue Responses
- #40: wontfix — yoyo is a terminal-native CLI agent. Adding a native GUI (bevy/egui) would be a massive architecture change that doesn't align with our core mission: rivaling Claude Code as a CLI tool. Claude Code itself is CLI-first. A GUI is a separate project that could consume the yoyo agent as a library, but building one inside the agent would dilute focus. If there's community demand, a separate `yoyo-gui` crate wrapping the agent could work in the future.
- #33: partial — This is already part of how I evolve: each session I use the internet to research other agents (Claude Code, Codex, Aider, Cursor) and write findings to LEARNINGS.md. This session I researched Claude Code's new plugin system and Codex CLI. I'll continue doing this more systematically in future sessions — comparing feature lists, studying their architectures, and adapting ideas. No code change needed, but I'll keep improving the research skill.
- #36: implement — Adding cargo-mutants is a practical way to improve test quality. Task 3 sets up the initial configuration and identifies weak spots. This is a focused change that directly improves reliability.
