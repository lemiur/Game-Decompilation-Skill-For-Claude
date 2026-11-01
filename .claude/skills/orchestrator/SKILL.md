---
name: orchestrator
description: Run a matching-decompilation campaign with parallel workers. Pin the project's code-rules profile, pick disjoint lanes, launch matcher workers in separate worktrees, run each lane through plain matching, helper recovery, structural and finishing passes, review results against the decomp-code-rules reader test, and merge every result into the main branch through a fixed merge procedure (rebase, build, gate, rule check, merge). Use when asked to start, steer or merge matching workers, to push a module or score band toward 100%, to clear MATCH-PENDING fakematches or run a readability sweep, or to integrate agent branches.
---

# orchestrator: run the workers, merge the results

Project guide: the repository's `AGENTS.md`. If the repository has none, use [AGENTS-template.md](../matcher/references/AGENTS-template.md) and ask the user to fill in its project adapter.

You plan lanes, launch workers, relay rulings and merge. Workers do the matching, guided by:
- `AGENTS.md`;
- the `matcher` and `helper-recovery` skills;
- `decomp-code-rules`.

Use `wall-identifier` when a function is stuck, when triaging plateaus, or when asked "why won't this match".

**What the campaign produces is readable source.** The headline number is **matched under the rules**: functions
whose bytes match *and* that pass every [match] rule. Bytes-matched is the second number. Report both, every
time. `MATCH-PENDING` functions (exact but failing a rule, usually a fakematch) are open work, not progress.

Keep the user informed with short, concrete reports:
- functions closed;
- both score numbers;
- fakematches found and cleared;
- what is open and why.

## Before the first lane: pin the profile

Workers write to the project profile in `AGENTS.md` (`decomp-code-rules`, "Project profile"). Make sure it states:
- **Dialect:** what the compiler accepts and rejects.
- **Naming and layout style, per module:** from evidence when there is any. Otherwise propose a default from
  `decomp-code-rules`' `style-defaults.md` and get the user's ruling.
- **Placeholder patterns.**
- **The `rules` command,** which runs the rule checker.
- **Line information:** whether the binary records source lines, which makes layout changes [checked].
- **The parking mechanism** for unfinished functions.

Style is a project decision, not a worker's. Never let parallel workers each pick their own style: their output
would mix styles, and that is worse than either one. Relay the ruling to every worker, and record it in
`AGENTS.md`.

Ensure that you use the correct compiler. If the user is unsure what the correct compiler is, inspect the binary yourself and list likely options.

If the user doesn't have the compiler installed, instruct them to look on Internet Archive. For console games, inform the user that it's likely the compiler came bundled with the SDK.

Before beginning matching lanes, identify any tooling that would dramatically improve matching time and efficiency. After every 3 lanes, check again to see if there is any tooling that should be built for efficiency. Prefer to use C for tooling as much as possible. Avoid Python for tooling as much as possible.

## Slots and branches

- **Worktrees:** each worker gets its own git worktree with its own `build/` directory. Reuse a small fixed set of
  worktrees (for example three), parked at the main branch between lanes. Add more only when the user asks, and
  remove extras after their merge.
- **Branches:** lane branches stay local until their merge. Delete them afterwards.
- **Preparing a slot:**
  1. Create the lane branch from the main branch.
  2. Run one `build`; a fresh worktree may copy `build/` from a built checkout first.
  3. Check that the build tools work in worktrees: some wrappers have hard-coded paths.

## Lanes

- **Lanes are disjoint.** Split them by module, unit or score band, never overlapping another running lane, and
  never two writers on the same files or shared headers. List the lane's functions (address, name, score, rule
  status) in the prompt.
- **Renames cross units.** A rename of a type, member, global or shared function touches every user of that name.
  A naming or type-recovery lane owns the shared header it renames. Run such lanes alone, or after the lanes that
  include that header have merged.
- **Good lanes:**
  - units close to complete;
  - the lowest-scoring band;
  - lost matches;
  - **`MATCH-PENDING` functions:** the bytes are exact and the source is fake. Each one is a helper-recovery or
    structural problem with a known answer key;
  - a recorded to-do;
  - a helper-recovery sweep over sampled functions;
  - **a finishing sweep** of a unit whose functions all match:
    - banners and names from evidence;
    - named constants;
    - documented structs;
    - quirk labels.

    All of these are [free] or [checked] cleanups (`matcher`, "Finish the source").

## Worker prompt

Launch a subagent or a separate session in the slot. The prompt holds only:
- the slot path and branch;
- "read `AGENTS.md` first; it takes precedence";
- the skill to start with;
- the goal and the lane's function list;
- what the other lanes own, including shared headers and names that are off-limits;
- the working rules:
  - do the work yourself, and commit on the branch as you go;
  - no push or PR;
  - write source that passes `decomp-code-rules`; a fakematch is reported as open, not as matched;
  - report per function the fix, the rule status and what is left.

Do not add per-step verification requirements to a worker prompt; the checks run once, at merge.

## Passes

1. **Plain matching** (`matcher`).
2. **Helper recovery.** When the worker returns with walls or `MATCH-PENDING` functions, merge its work. Then
   resume the same worker (it keeps its context) on the open functions with `helper-recovery`: inline helpers,
   accessors, macros, scoped blocks. Most fakematches hide a missing helper or a merged variable.
3. **Structural pass.** If it is still stuck, resume it for a structural pass: types and ownership, API level,
   function boundaries, control structure, surviving sources. Not permutation.
4. **Finishing pass.** For the lane's functions that are now matched, resume the worker on `matcher`, "Finish the
   source": rule checker, reader test, then the [free] finishing rules in their own commit.
5. **Stop** when the worker reports no open structural lead. Before calling a wall permuter-flat, check which
   permuter families actually ran; one flat family proves nothing.

## Merging a lane

In the lane's slot:

1. **Rebase onto the main branch.**
   - Conflicts in generated files (reports, the README's numbers): take main's side and regenerate.
   - Conflicts in the score ledger: merge per row, so each row comes from the side that actually achieved it.
     Never take either side wholesale; that drops achieved scores or revives stale ones.
   - After rebasing, confirm that the lane's reported scores are still there.
   - If another merge already landed the same fix, drop the duplicate commit, or cherry-pick only the lane's
     unique commits.
2. **Gate against the main branch's baseline before updating the baseline.** After an update, the gate would
   compare the lane with itself and could not see a drop.
   - Only a real regression blocks: an *edited* function whose score fell. Dips in unchanged functions are
     informational.
   - A regression is fixed in the lane, or the user adjudicates it before the merge.
   - Formatting-only and naming commits are gated too: `__LINE__`, line information, stringized names and RTTI
     can move bytes (R23, R7).
3. **Run the byte gate from a clean rebuild** of the rebased lane (`AGENTS.md`, "The byte gate"). Batch the
   lane's matches into one gate run; on red, drop the units the byte attribution names, or bisect, then re-gate.
   Report banked counts only from the gate's printed success line.
4. **Run `rules`** over the lane's units, and compare against main.
   - A function the lane reports as matched but that the checker flags is `MATCH-PENDING`: correct the report.
   - New findings in functions the lane didn't touch mean a shared header change broke them. Fix that in the
     lane.
5. **Review the diff against the reader test.** Read a sample of the lane's new and changed functions,
   prioritizing jumps to 100%. Look for Section F patterns the checker can't see:
   - merged variables;
   - temporaries with no meaning;
   - confident invented names;
   - quirk labels (R36) without evidence;
   - `RULE-EXCEPTION`s that excuse fake code.

   Send fakematches back to the worker as `MATCH-PENDING`. Never rewrite them during the merge.
6. **Update the baseline and the reports** with both numbers, and commit them.
7. **Merge** (squash or PR, per project policy), listing each function's change and rule status, then park the
   slot.

Never commit or build in a half-finished rebase: check `git status` for unmerged paths before every build.

## Ledgers and rulings

- **Exceptions and pending matches:** a byte-exact function is always kept. It is never thrown away because of a
  rule. How it counts depends on which rule it breaks:
  - **A style rule the bytes defeat:** the function counts as matched. It carries a
    `// RULE-EXCEPTION: <rule>, <reason>` at the code, and the project's exception record lists it.
  - **A [match] rule:** the function is `MATCH-PENDING`. The source is kept as the best-known state, and the
    function goes to the open-work list for a helper-recovery or structural lane. It doesn't enter the matched
    count. If the gate needs an allow entry to admit its bytes, record that the entry is for a pending match.
  - **Original assembly:** a function the evidence shows was written in assembly is `ORIGINAL-ASM`, listed apart
    and counted neither way.

  Never leave such a decision unrecorded.
- **Rule changes:** tightening or relaxing a rule is the user's ruling. It is recorded in `AGENTS.md` with the
  reason, and the checker changes in the same commit.
- **Rulings and tooling fixes:** relay the user's rulings, including style rulings, to every running worker. When
  a tooling fix that affects running lanes lands on main (a gate or rule-checker fix, for example), send each
  worker the commit to cherry-pick.
- **Worker claims about tools are leads, not facts.** If a worker avoided a correct change because a gate, the
  rule checker or another tool objected, check the tool: a wrong gate steers every worker. A checker that flags
  genuine original code (hardware `volatile`, struct-prefix inheritance, a real vtable slot) is a checker bug.
  Never treat it as a reason to distort the source.
