# CLAUDE LOOK AT SPEC. CLAUDE THINK. CLAUDE WRITE TRUTH.

> **Caveman format. No fluff. Just rock-hard facts and fire-hot ambiguities.**

---

## CONFIDENCE NUMBER: 7 OUT OF 10

Claude can build 80% of this. But fire in several places. Read below.

---

## WHAT CLAUDE CAN BUILD GOOD (strong hand, no fire)

```
MARKDOWN FILES       — CLAUDE STRONG HERE
  SKILL.md           ✅ full
  prompt-contract.md ✅ full
  task-contracts.md  ✅ full
  context-routing.md ✅ full
  validator.md       ✅ full
  examples.md        ✅ full

JSON FILES           — CLAUDE STRONG HERE
  plugin.json        ✅ full
  hooks.json         ✅ full (format known)
  prompt-optimizer.config.json ✅ full

JS SCRIPTS (structure + logic)
  optimize-prompt.mjs         ✅ can write; cannot test Ollama at runtime
  check-context-freshness.mjs ✅ can write; cannot test CI runtime
  nudge-short-prompt.mjs      ✅ can write

DOCS
  README, TEST_PLAN, METRICS, ADOPTION, TROUBLESHOOTING ✅ full

TEST SCAFFOLDING
  tests/fixtures/  ✅ skeleton
  tests/golden/    ✅ skeleton (BUT test cases = guesses without real repo)
```

---

## WHAT CLAUDE CANNOT BUILD WITHOUT HUMAN HAND

```
OLLAMA RUNTIME TEST     🔥 need Ollama running. Claude cannot ping localhost.
PLUGIN LOAD TEST        🔥 need --plugin-dir flag in real Claude Code session.
REAL DOCS/AI/ PACKS     🔥 need actual target repo. Claude will write stubs only.
GOLDEN TEST CONTENT     🔥 before/after pairs need real repo + human judgment.
ENV VAR SECRETS         🔥 human must set PROMPT_OPTIMIZER_API_KEY in shell.
CI WORKFLOW WIRING      🔥 .github/workflows/*.yml needs real repo + human to push.
```

---

## AMBIGUITY LIST — EVERY FOGGY ROCK CLAUDE SEE

### AMB-01: Plugin directory discovery mechanism — WHERE IS PLUGIN?

**Spec says:** plugin loads via `--plugin-dir` flag.  
**Problem:** Spec does not say if Claude Code reads `--plugin-dir` from a config file, env var, or only CLI. If team wants plugin to auto-load without the flag every session, there is no spec for how.  
**Missing:** How does a developer "install" this plugin permanently? Does it go in `~/.claude/plugins/`? Is there a global config entry? Spec is silent.

---

### AMB-02: Hooks format — IS THIS THE REAL FORMAT?

**Spec says:** `hooks/hooks.json` with `SessionStart` + `UserPromptSubmit`.  
**Problem:** Claude Code hooks format has changed. The spec shows a nested `{"hooks": {"SessionStart": [{"hooks": [...]}]}}` structure — but current Claude Code uses `settings.json` hooks, not a separate `hooks.json` inside the plugin. The spec references [plugins-reference] but the URL `code.claude.com/docs/en/plugins-reference` may be stale or wrong.  
**Missing:** Verified current hooks schema for plugins (not user-level hooks). Does Claude Code even support per-plugin hooks.json?

---

### AMB-03: Skill invocation path — HOW DOES CLAUDE READ SKILL FILES?

**Spec says:** `SKILL.md` references `context-routing.md`, `validator.md`, `task-contracts.md` by relative name.  
**Problem:** Does Claude Code automatically inject all files in `skills/enhance/` into context when the skill runs? Or does `SKILL.md` need explicit `@file` references? Or does the model just "see" a folder? The spec assumes Claude reads sibling files but never says HOW.  
**Missing:** Mechanism by which support files (non-SKILL.md files in the skill directory) are made available to the model.

---

### AMB-04: `optimize-prompt.mjs` execution model — WHO RUNS THE SCRIPT?

**Spec says:** skill delegates to `scripts/optimize-prompt.mjs`.  
**Problem:** Claude Code skills run as model instructions, not as node processes. A SKILL.md cannot directly `exec` a .mjs file. The mechanism would need to be: (a) the model calls a bash tool to run the script, or (b) the script is wired as a hook, or (c) this is aspirational pseudocode. None of these paths are specified.  
**Missing:** How does a SKILL.md instruction cause a Node.js script to execute? What Claude Code primitive bridges these?

---

### AMB-05: `CLAUDE.md` location — WHICH REPO'S FILE?

**Spec says:** "Always read `CLAUDE.md` if present."  
**Problem:** The plugin lives in its own repo. The target repo is a different directory. When the developer runs `/prompt-enhancer:enhance` in their project, which `CLAUDE.md` does "always read" refer to? The plugin's own? The target project's? Both? The spec implies the target repo's but never states it.  
**Missing:** Explicit statement of CWD context when skill runs — does the skill operate in the plugin directory or the developer's current working directory?

---

### AMB-06: Context pack path resolution — RELATIVE TO WHAT?

**Spec says:** load `docs/ai/*.md` packs from the target repo.  
**Problem:** Same issue as AMB-05. If the plugin is installed globally/elsewhere, `docs/ai/` is a relative path. Relative to what? The plugin root? The CWD of the Claude Code session? The target repo root?  
**Missing:** Path resolution strategy for context packs when plugin and target repo are separate directories.

---

### AMB-07: Token counting — HOW IS "~500 tokens" ACTUALLY ENFORCED?

**Spec says:** size classes have token budgets (~500 / ~1000 / ~2000). Validator enforces this.  
**Problem:** The Validator runs as model instructions reading `validator.md`. The model cannot programmatically count tokens — it estimates. The optional `count-tokens.mjs` script would count accurately, but it is marked "optional" and the invocation mechanism has the same gap as AMB-04.  
**Missing:** Whether token enforcement is fuzzy (model estimates) or hard (script runs). "Reject outputs over budget" in the spec implies hard enforcement, but the only hard enforcer needs the script-execution gap solved first.

---

### AMB-08: Golden test runner — HOW DO TESTS ACTUALLY RUN?

**Spec says:** golden set in `tests/golden/`, re-run on every skill change.  
**Problem:** There is no test runner specified. No Jest config, no shell script, no npm test command. "Re-run whenever SKILL.md changes" implies automation but there is no test harness described.  
**Missing:** Test execution mechanism. Is this a human-manual review checklist? A script? A CI job? The spec says "full procedures in docs/TEST_PLAN.md" — but TEST_PLAN.md is a file to be created, not a reference to something existing.

---

### AMB-09: `module-index.md` population — WHO WRITES THIS?

**Spec says:** Context Selector resolves file paths from `module-index.md` and `ownership.md`.  
**Problem:** These files are listed in "what each target repository provides" (§8.2). They must be written by the team maintaining the target repo. But the spec gives no template, no minimum viable format, and no tooling to auto-generate them from the repo structure.  
**Missing:** Format spec for `module-index.md` (what fields? what granularity? per-file or per-module?). Without this, two teams will produce incompatible formats and the Context Selector will not reliably find paths.

---

### AMB-10: Fallback signal — HOW DOES SKILL KNOW SCRIPT FAILED?

**Spec says:** if `optimize-prompt.mjs` is unreachable, skill falls back to in-agent.  
**Problem:** If the script exits with a "signal" (spec's word), what is that signal? An exit code? A specific stdout string? A JSON flag? The SKILL.md instruction would need to parse this. But SKILL.md is markdown instructions for a model, not code. The model cannot reliably parse subprocess exit codes.  
**Missing:** Precise fallback protocol between the script and the skill instruction layer.

---

### AMB-11: `UserPromptSubmit` hook — WHAT COUNTS AS "SHORT"?

**Spec says:** nudge fires if prompt is "very short and looks like a coding task."  
**Problem:** "Very short" is undefined. 10 words? 50 characters? The `nudge-short-prompt.mjs` script must implement this heuristic but the spec gives no threshold or detection logic.  
**Missing:** Numeric threshold and keyword heuristic for the short-prompt detector.

---

### AMB-12: Per-repo config override — MERGE OR REPLACE?

**Spec says:** target repo can have its own `prompt-optimizer.config.json`.  
**Problem:** Does this file fully replace the plugin's default config, or do fields deep-merge? If a target repo sets only `provider`, does it inherit the plugin's `providers.*` objects or must it redefine everything?  
**Missing:** Config merge strategy (replace vs. deep-merge vs. field-level precedence).

---

### AMB-13: Version of Claude Code — WHICH VERSION IS THIS WRITTEN FOR?

**Spec says:** uses `--plugin-dir`, `SessionStart`, `UserPromptSubmit`, plugin `SKILL.md` format.  
**Problem:** The execution plan (claude-code-execution-plan.md) explicitly warns "plugins/hooks are relatively new and have had breaking changes." The spec references `code.claude.com/docs/en/plugins-reference` — this URL pattern is not the current Claude Code docs domain. Plugin support format as of June 2026 may differ from what the spec was written against.  
**Missing:** Pinned Claude Code version this spec was validated against.

---

### AMB-14: "Bring your own model" privacy claim — IS THIS ACTUALLY PRIVATE?

**Spec says:** with local Ollama, prompts stay on the developer's machine. SUITABLE FOR REGULATED CODEBASES.  
**Problem:** The skill still runs in Claude Code (the Anthropic-hosted model) for validation. The raw prompt and context packs are always sent to Anthropic's model at minimum (for the Validator role). The privacy benefit of Ollama only applies to the optimization step, not the full flow. The claim "raw prompt + repo context never leave the machine" is only true for the optimization sub-step, not the full plugin execution.  
**Missing:** Honest scoping of the privacy guarantee. Developers in regulated environments may misread this as full local execution.

---

## CONFIDENCE BREAKDOWN BY AREA

| Area | Confidence | Why |
|---|---|---|
| Markdown skill files (SKILL.md, contracts, routing) | 9/10 | Pure text. Claude strong. |
| JSON configs (plugin.json, optimizer config) | 9/10 | Simple structure, well-specified. |
| JS scripts (optimize-prompt.mjs, freshness check) | 7/10 | Logic is clear; runtime untestable. |
| Hooks (hooks.json + nudge script) | 5/10 | AMB-02: format may be wrong for current Claude Code. |
| Golden test content | 4/10 | Needs real repo. Claude will produce plausible stubs only. |
| Plugin actually loading and running | 6/10 | AMB-03, AMB-04: execution bridge unclear. |
| End-to-end Ollama flow | 5/10 | Needs Ollama running; AMB-04 gap. |
| Context pack templates | 8/10 | Format is clear enough; content is repo-specific. |
| **OVERALL** | **7/10** | Good skeleton. Human must verify runtime behavior. |

---

## WHAT HUMAN MUST DO AFTER CLAUDE BUILDS

```
1. INSTALL CLAUDE CODE     — npm install -g @anthropic/claude-code
2. RUN WITH --plugin-dir   — verify /prompt-enhancer:enhance loads
3. FIX HOOKS FORMAT        — if hooks.json does not fire, check current Claude Code hooks spec
4. WRITE REAL docs/ai/     — for your actual target repo (Claude makes stubs)
5. RUN OLLAMA              — ollama pull qwen2.5-coder:7b && ollama serve
6. SET ENV VAR             — export PROMPT_OPTIMIZER_API_KEY=... in shell
7. CURATE GOLDEN TESTS     — replace stubs with real before/after pairs
8. WIRE CI                 — add .github/workflows/context-freshness.yml to target repo
```

---

## BOTTOM LINE FOR CAVEMAN

**CLAUDE BUILD HOUSE FRAME. HUMAN PUT WINDOWS AND FIRE.**

Spec is solid. Architecture is sound. Ambiguities live at the edges where model instructions meet runtime execution — and those edges require a human with a real Claude Code install to poke and fix. Expect 2–4 iteration rounds after the initial build before the plugin runs end-to-end.
