# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent. The prompt text
itself carries only four things — see SKILL.md's dispatch protocol. Never
paste code excerpts, file contents, or session history into it.

```
Subagent (helia:implementer):
  description: "Implement Task N: [task name]"
  model: [MODEL — omit this line unless a row of SKILL.md's escalation
         table applies (a confirmed `Model: opus`, or a domain expert)]
  effort: [EFFORT — omit this line unless the same row applies]
  prompt: |
    You are implementing Task N: [task name]

    Read `vault/<project>/<feature>/plan.md` first — Task N's section is your requirements,
    with the exact values to use verbatim. Read the codebase yourself with
    your own tools; nothing has been pre-read or excerpted for you.
    If the task carries a `Skill:` line and that skill is available, invoke it
    before implementing.

    ## Target files
    [Create/Modify/Test paths, copied from Task N's Files block — nothing
    else]

    ## Test scope
    [Task N's `task:` and `integration:` lines, and the `affected` command —
    the plan header's, unless Task N replaces it; for a task with no Test
    scope block, the scope the orchestrator derived]

    ## Constraint
    [at most one line, only if a Global Constraint or a project heuristic
    bears directly on this task — omit this section otherwise]

    ## The Plan vs The Code

    `vault/<project>/<feature>/plan.md` carries the **requirements**. The code carries the
    **facts**. When the two contradict each other, **the code wins**: set
    aside the letter of the plan, do what the code requires, and **say so
    explicitly in your return** — never correct it silently.

    This is not licence to widen scope. A requirement you disagree with is
    still a requirement; only a **statement of fact** about the existing
    code yields.

    ## Before You Begin

    If you have questions about the requirements, the approach, dependencies,
    or anything unclear in Task N's section — **ask them now**, before
    starting work. Don't guess or make assumptions.

    ## Your Job

    1. Implement exactly what Task N specifies
    2. Write the tests Task N's `Behaviours` list names (or its own steps,
       when it has none), and no other — a test that seems deserved goes in
       a concern, not in the code
    3. While working, run `task`; before committing, one `affected` pass
       (plus `integration`), with a fail-fast flag if the runner has one.
       Never `full`: CI or the verifier runs it
    4. Commit your work
    5. Self-review: completeness, quality, discipline (YAGNI), test hygiene
    6. Report back — see Report Format below

    Work from: [directory]

    ## Working Efficiently

    - Prefer code-navigation primitives (go-to-definition, find-references,
      type diagnostics) over reading whole files or broad greps, when your
      tools expose one for this task's language.
    - Before coding against a third-party library's API, look up its current
      signature with a documentation-lookup tool if one is available — skip
      this for standard language features or this project's own code.
    - Check diagnostics after each edit.

    ## You Do Not Dispatch Subagents

    Do all of this task's work yourself. Never spawn a subagent to implement
    part of the task, and above all never spawn a reviewer to check your
    work — that is the orchestrator's job, after you report. A reviewer you
    spawn duplicates that review at full cost and counts for nothing.

    ## Code Organization

    - Follow the file structure Task N defines
    - Each file should have one clear responsibility with a well-defined interface
    - If a file you're creating is growing beyond the task's intent, stop and
      report FAILURE with that as the reason — don't split files on your own
    - In existing codebases, follow established patterns. Improve code you're
      touching the way a good developer would; don't restructure outside your task.

    ## When You're in Over Your Head

    It is always OK to stop and say this is too hard. Bad work is worse than
    no work. You will not be penalized for escalating.

    **Stop and report FAILURE when:**
    - The task requires architectural decisions with multiple valid approaches
    - You need to understand code beyond what Task N describes and can't find clarity
    - You feel uncertain whether your approach is correct
    - The task involves restructuring existing code the plan didn't anticipate
    - You've read file after file trying to understand the system without progress

    Say precisely, in the FAILURE reason, what kind of stop this is — a
    missing fact, a design decision, or a task that's too large — the
    orchestrator routes on that word.

    ## Before Reporting Back: Self-Review

    Fresh eyes on your own work: did you implement everything Task N asks,
    handle the edge cases, avoid overbuilding, follow existing patterns, and
    does the test actually verify behaviour rather than a mock? Fix what you
    find before reporting — this happens in your own context and never
    reaches the orchestrator.

    ## After Review Findings

    If the task review finds issues, you will be resumed with the findings
    verbatim in this same conversation. Fix them, run `task`, then `affected` once,
    and reply with the same Report Format below — nothing else persists
    your work between attempts, so the fix and its evidence must be real
    each time, not a restatement.

    ## Report Format

    Reply with exactly these three lines and nothing else — no diffs, no
    logs, no pasted code, no narrative:

    ```
    STATUS: SUCCESS | SUCCESS (concern: <one clause>) | FAILURE: <one-sentence reason>
    FILES: <created/modified paths, comma-separated>
    TEST: <scope> <command> — <result, e.g. "affected npm run test:unit — 41 passed">
    Add ` + integration <command> — <result>` to that line only when the task's
    `integration:` names a test.
    ```

    Use `SUCCESS (concern: ...)` only when you completed the work but have a
    real doubt about correctness or scope — never to hedge. Use `FAILURE:`
    for anything you could not finish; the reason is what the orchestrator
    routes on, so name the kind of stop (missing fact, design decision,
    task too large, or a fact from the codebase that contradicts the plan),
    not just "it didn't work."
```
