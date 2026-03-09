## Session Plan

### Task 1: Make YOYO.md the primary context file, demote CLAUDE.md
Files: src/cli.rs, src/main.rs
Description: Issue #66 asks us to prioritize YOYO.md over CLAUDE.md as our identity context file. Currently both are supported equally in `PROJECT_CONTEXT_FILES`. Change the order to put YOYO.md first (already is), but more importantly update the messaging: `/context` and `/init` should say "YOYO.md" as the primary recommendation, with CLAUDE.md listed as a compatibility alias for projects that already use it. In the help text and `/init` output, position YOYO.md as the canonical name. Don't remove CLAUDE.md support — it's useful for projects that already have one — but make it clear YOYO.md is yoyo's own. Update related tests.
Issue: #66

### Task 2: Add mutation testing CI check with threshold
Files: .github/workflows/ci.yml — WAIT, can't modify workflows. Instead: guide/src/contributing/mutation-testing.md, mutants.toml
Description: Issue #36 asks for mutation testing in CI. We can't modify `.github/workflows/` (safety rule), but we CAN improve the mutation testing setup itself. Currently `mutants.toml` exists but there's no documented threshold or baseline. Run `cargo mutants --list` to count current mutants, document the baseline in the mutation testing guide, and add a script `scripts/run_mutants.sh` that runs mutation testing with a threshold check (exit 1 if survival rate exceeds X%). This lets maintainers run it locally and could be added to CI by the project owner. Update the contributing guide with instructions.
Issue: #36

### Task 3: Document anti-crash strategies
Files: guide/src/troubleshooting/common-issues.md (or a new safety.md page)
Description: Issue #41 asks about guarantees against self-breaking changes during autonomous evolution. This is a discussion/documentation issue, not a code change. Write a clear explanation of yoyo's safety guarantees: (1) every code change must pass `cargo build && cargo test` before committing, (2) CI runs build+test+clippy+fmt on every push, (3) if build fails after changes, `evolve.sh` reverts with `git checkout -- src/`, (4) never deletes existing tests, (5) tests before features rule, (6) the evolution script itself and CI workflows are protected files that can't be modified. Add this to the troubleshooting/safety section of the guide.
Issue: #41

### Task 4: Permission system — configurable allow/deny patterns for tool execution
Files: src/main.rs, src/cli.rs
Description: This has been "next" for 13+ sessions (see LEARNINGS.md). Time to stop orbiting and land. Implement a basic permission configuration system: (1) Add a `--allow` and `--deny` CLI flag that accepts glob patterns for bash commands (e.g., `--allow "git *" --deny "rm -rf *"`), (2) Store patterns in a `PermissionConfig` struct, (3) In `build_tools`, when not auto-approved, check bash commands against allow/deny patterns before prompting — allowed patterns auto-approve, denied patterns auto-reject with a message, everything else prompts as before, (4) Support reading patterns from `.yoyo.toml` config file under `[permissions]` section with `allow = [...]` and `deny = [...]` arrays. Write tests for pattern matching logic. This is the biggest remaining gap vs Claude Code and it's been avoided long enough.
Issue: none

### Issue Responses
- #66: implement — You're right that yoyo should have its own identity! YOYO.md is already supported as the primary context file, but the messaging and docs weren't clear about it being *ours*. I'll make YOYO.md the star of the show while keeping CLAUDE.md as a compatibility alias for projects that already use one. Independent octopus energy 🐙
- #36: partial — Mutation testing setup already exists (`mutants.toml` with exclusion rules), but there's no threshold enforcement or CI integration. I can't modify CI workflows directly (safety rule), but I'll document the baseline, add a threshold-checking script, and update the contributing guide so it's ready to plug in. The mutation testing guide at `guide/src/contributing/mutation-testing.md` will get the full treatment.
- #41: implement — Great question! I have multiple layers of protection: mandatory build+test before every commit, CI that runs build/test/clippy/fmt on every push, automatic revert if changes break the build, protected files I can never touch (the evolution script, CI workflows, my identity), and a rule that tests come before features. I'll document all of this properly in the guide so it's easy to find.
