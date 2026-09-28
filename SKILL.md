---
name: short-technical-writing
description: Rewrite code comments, PR descriptions, or docs into Simplified Technical English and cut their length by a target percentage. Use when asked to simplify, shorten, or de-jargon prose in a diff or document.
---

# Short Technical Writing

Two passes, in this order. Do not merge them.

**Pass 1 — rewrite everything using the Simplified Technical English spec.** Every line in
scope, not only the long ones. Short prose can still be passive, figurative, or vague, and a
line you skip here is a line you will never fix. Ignore length entirely on this pass; the
output may be longer than the input. That is fine.

**Pass 2 — cut the result by 50%.** Measure the original, measure the rewrite, and trim until
the count hits the target.

Default target: **50% fewer words**. Honor an explicit target if one is given.

The order matters. Cutting first produces text that is short and still bad: you spend the
budget defending sentences you should have rewritten, and you end up trimming words out of a
weak line instead of replacing it with a strong one. Write it well, then make it short.

**The 50% default assumes unedited prose.** Text that has already been through this skill has
no padding left, so a second run hits a floor well short of 50%: Pass 1 still finds style
defects, but Pass 2 has nothing structural to remove. Report the floor and the reason. Do not
manufacture a number by deleting a sentence the reader needs.

**A target never outranks an explicit instruction.** If the author asked the comment to explain
a specific thing, that sentence stays even when it is the only thing standing between you and
the target. Say you missed the target and why.

## Handle --help

If the argument is `--help`, `-h`, or `help`, print this and stop:

```
/short-technical-writing - Rewrite prose in Simplified Technical English and cut its length

USAGE:
  /short-technical-writing                     # comments added/changed in the current diff
  /short-technical-writing <path>              # one file
  /short-technical-writing pr <number>         # a PR description
  /short-technical-writing --target=30         # cut 30% instead of 50%

WHAT IT DOES:
  1. Define the scope, then COUNT the baseline words (the target is measured, not felt)
  2. PASS 1 — rewrite every line to Simplified Technical English, ignoring length
  3. PASS 2 — cut the rewrite to the target; re-measure and iterate
  4. Verify nothing broke (lint, typecheck, affected tests)
```

## Pass 1: the rules

Simplified Technical English (ASD-STE100, adapted). Apply all of them:

- **One idea per sentence.** Split compound sentences at the colon or semicolon.
- **Active voice.** "The upsert uses the number", not "the number is used by the upsert".
- **Present tense.** "Twilio reports these fields", not "Twilio would have reported".
- **Max ~20 words per sentence.**
- **One meaning per word.** Pick a term and reuse it. Do not alternate between "route", "destination", and "endpoint" for the same thing.
- **No metaphor, idiom, or figurative language.** Delete "load-bearing", "the hard way", "no way in", "right shape wrong moment", "blind-retry", "round-trip" (as a verb).
- **No rhetorical questions.** `"where does this number point?"` becomes "the target is ambiguous".
- **Keep articles.** "The guard satisfies the compiler", not "Guard satisfies compiler".
- **Keep domain terms.** `fail closed`, `fingerprint`, `409`, `upsert`, and identifier names are precise. Do not "simplify" them into vagueness.
- **No em dashes.** Use a period, comma, or semicolon.
- **No personal or customer data.** Never write a real phone number, email address, name,
  address, policy or claim number, account SID, token, or customer identifier into a comment,
  a description, or a published document. Describe the shape instead: "an E.164 number", "the
  account SID". Rewriting is when this leaks, because a real value from the code under review
  is the nearest example to hand.

Prose you publish outside the repo has a wider audience than the code it came from. Before
publishing, also strip internal identifiers a reader outside the team should not need: the
name of a database escape hatch, an internal service or script path, and product jargon.
Replace them with what they are, not what they are called.

## Pass 2: cutting to target

Only start this once every line in scope has been through Pass 1. You are now cutting good
prose, which is why the cuts are structural (delete a line, drop a sentence) rather than
word-level.

Cut in this order. Stop when the measurement hits the target.

1. **Delete comments that restate the code.** A comment above `unique(...)` saying "this is unique" earns nothing. This is the cheapest 20%.
2. **Drop the second sentence** when the first carries the reason. Most two-sentence comments have one load-bearing clause.
3. **Collapse multi-line blocks.** A four-line paragraph is usually one sentence plus three of elaboration.
4. **Compress, then re-measure.** Word-level tightening is the least productive move; do it last.

Never cut the **why**. A comment that survives should explain a decision the code cannot show. If a cut would remove the only reason a reader needs, keep it and take the words from somewhere else.

Never cut the **what**. This is the more common failure and it is easy to miss, because the
result still reads like fluent English. When you delete the sentence that carried the content,
the surviving claim keeps its shape and loses its referent:

> `- **Application and trunk routes restore correctly.**`

Correctly compared to what? Restored by whom, from what state? Every noun is abstract and no
change is named. The reader cannot act on it, and the author cannot see the problem because
the grammar is intact.

Rewritten with the same word budget, naming a subject and a change:

> `- **Rollback restores TwiML application and SIP trunk routes.** Updates cleared the application SID and had no trunk field.`

## Self-check before you stop

Read each rewritten line cold, as someone who has not seen the diff. For each one:

- **Name the subject.** Which function, table, field, or endpoint does this describe? "Routes", "the control", "the state" are not subjects.
- **Name the change.** What is different now? An adverb (`correctly`, `properly`, `as expected`) is not a change.
- **Point to the code.** If the line cannot be tied to a specific line of the diff, it is decoration.

A line that fails any of these is not short, it is empty. Restore the concrete noun and the
concrete verb, then take the words back from a line that is genuinely padded.

**Be harder on intent comments than feels comfortable.** A comment asserting that two things
"agree", "match", or "stay in sync", without naming either thing or the mechanism, reads as
meaningful and says nothing:

> `// The capture predicate, so the page and the action agree.`
> `// A misconfigured map blocks the switch; the page still renders.`

Both survived a 50% pass and were deleted immediately afterward by the author. If a comment
would not change what a reader does, it is padding no matter how true it is.

## Read the code before you rewrite the comment

You cannot compress an explanation you do not understand, and a comment is the one place where
a confident paraphrase of the wrong thing survives review. Open the code the comment sits on
and the identifiers it names.

This is also where the best comments come from. Reading `rawDb.transaction` and the
`pg_advisory_xact_lock` call underneath it showed that the transaction exists to scope the
lock, and that Postgres rolling back does **not** roll back the Twilio write the block
performs. The second fact is the only one a reader cannot get from the code, and no amount of
rewording the original comment would have produced it.

When the code shows the comment is wrong or hollow, fix the comment's content, then apply the
passes. Shortening a false statement produces a shorter false statement.

## Prefer a name over a comment

Renaming is a cheaper cut than rewording, and it cannot go stale. A helper called `withEnv`
needs a comment explaining that it restores the previous values; `withTemporaryEnv` does not.

After any rename, re-read the comments around it. The ones that existed to compensate for a
vague name are now dead weight, and they are easy to miss because you were looking at the
identifier, not the prose.

## Line width

Keep each comment on **one line** when it fits. Read the formatter config first
(`.prettierrc` `printWidth`, `.editorconfig`, ruff/black line-length) and count the
**indentation** against that budget. Never assume 80.

If a comment cannot fit on one line, break it into complete sentences, one per line,
rather than wrapping mid-clause.

## Do not touch

- **Lint-required justification comments.** Some repos enforce a comment before an escape hatch, such as a raw-database client that bypasses row-level security. Shorten the text, keep the required prefix and a real justification. Deleting one breaks the build.
- License headers, codegen banners, `@ts-expect-error` / `eslint-disable` directives, JSDoc type annotations that tooling reads.
- **Markdown and config markers.** YAML frontmatter and HTML comments in a `.md` file are often read by tooling, not by people. A changeset's `<!-- docs-reviewed -->` gates a CI docs check and is stripped from the changelog by config. It looks exactly like a comment you would delete.
- Generated files. In a diff, exclude lockfiles and schema snapshots (e.g. drizzle `snapshot.json`) before counting.

**Before editing any file that contains a marker, grep the repo for that literal string.**
One grep separates "stale note" from "build gate":

```bash
grep -rn "docs-reviewed" --include="*.ts" --include="*.js" . | grep -v node_modules
```

## Procedure

### 1. Scope and baseline

For a diff, find the merge base and extract only **added or changed** comment lines:

```bash
BASE=$(git merge-base HEAD origin/main)
git diff "$BASE"...HEAD -- ':!**/snapshot.json' ':!**/*.lock' \
  | grep -E "^\+\s*(//|#|\*)" | sed 's/^+//' | wc -w
```

Record the number. The target is a fraction of it. **Do not skip this** — "feels shorter" is reliably 20 points off.

### 2. Pass 1 — rewrite every line

Apply the Pass 1 rules to every line in scope, including the ones already short enough to keep.
Do not look at the word count during this pass. Judge each line only on whether it is active,
concrete, present tense, and free of metaphor.

For more than ~10 edits, write a script with explicit
`(old, new)` pairs and **assert each old string matches exactly once** before
replacing. A silent zero-match is the main failure mode of bulk edits:

```python
n = src.count(old)
if n != 1:
    failed.append(f"{rel}: {n} matches for {old[:70]!r}")
    continue
src = src.replace(old, new, 1)
```

Print the failures and fix them. Do not let a partial application pass as done.

Allow an explicit expected count for a string that legitimately repeats, but treat the
repetition as a finding: the same sentence copied into three call sites usually belongs in one
place, or nowhere. Delete it before you spend words rewording it three times.

### 3. Pass 2 — cut, re-measure, iterate

Measure first, so you know how far Pass 1 left you from the target. Compare the working tree
against the same base (note: `$BASE` not `$BASE...HEAD`, so uncommitted edits count):

```bash
git diff "$BASE" -- ':!**/snapshot.json' | grep -E "^\+\s*(//|#)" | wc -w
```

Then cut in the order given above and measure again. Expect **two or three rounds of cutting**.
The first lands near 30%, not 50%. Budget for that instead of declaring victory early.

### 4. Verify

Prose changes must not change behavior. Run, at minimum:

- the linter (this catches deleted lint-required comments)
- the typechecker (this catches a mangled line)
- the test suites covering the touched files

Check that no comment line now exceeds the width budget.

If a gate fails, **prove whether it is yours**: stash the changes, run the same gate on
committed HEAD, and compare. Report a pre-existing failure as pre-existing, with the
evidence. Never silently bypass a gate.

### 5. Report

Give the measured numbers, not adjectives:

```
BEFORE: 2846 words / 219 lines
AFTER:  1389 words / 146 lines
CUT:    51.2% words, 33.3% lines
```

## PR descriptions

Same rules, two extra constraints:

- **Preserve bot-generated blocks verbatim.** Split on the marker (e.g. `<!-- CURSOR_SUMMARY -->`), rewrite only the human half, reassemble. Measure the cut against the human half alone.
- Keep every link, ticket ID, and identifier. They are the parts a reader acts on.

```bash
gh pr view <n> --json body -q .body > /tmp/pr-current.md
# rewrite the head half, reassemble, then:
gh pr edit <n> --body-file /tmp/pr-new.md
```

Verify after publishing that the bot block survived and the word count is what you claim.

GitHub's API returns 503 and empty bodies during incidents. A `gh pr view` that yields zero
words is usually the API, not a bad publish. Retry the read before you conclude anything, and
never re-publish from an empty fetch: you will overwrite the description with nothing.

## Changesets

A changeset's bullets become the published CHANGELOG, so the reader is a user of the package,
not a reviewer of the diff. Keep the frontmatter and any marker untouched, and measure the cut
against the bullet body alone.

When the same fact appears in a code comment, the PR description, and the changeset, converge
on one wording. Three paraphrases of one change read as three changes.

## Long documents

For a spec or design doc, the target is met structurally, not sentence by sentence. Enumerated
tables are usually most of the word count and the cheapest thing to collapse: 56 test cases
across four tables became one table of counts and coverage areas, which was most of a 4862 to
2207 word cut. A reader who needs case U07's assertions opens the source document.

Keep at full fidelity anything a reader would otherwise have to open the source to trust:
truth tables, contracts and field names, thresholds, fixture IDs, and any table that assigns
an owner. Link back to the full document from the top.

Publishing to Drop: `drop download <slug>` prints the source to stdout, and works on pages
that WebFetch cannot read because of SSO. `drop create <file> --slug <slug> --name "..."`
publishes. Deploys are private by default.

## Examples

| Before | After |
|---|---|
| `// Order is load-bearing: clients round-trip this positional array back to the API.` | `// The order is necessary.` |
| `// Stale means configuration stopped approving it, not that the spelling differs: comparing spellings would flag every route, because the names all contain encoded colons.` | `// Stale means configuration stopped approving the route.` |
| `// One managed route per number, and the join key back to the inventory row. Unique rather than a plain index because two rows for one number would make "where does this number point?" ambiguous, which is the question the whole control exists to answer.` | *(deleted — restates `unique(...)`)* |
| `// Twilio reports these whether or not they were set, so a constructed route omitting them never matches the read-back, failing verification after moving Twilio. Found by e2e.` | `// Twilio reports these fields always.` |
| `A number routed by a TwiML application or a SIP trunk can now be restored. Every update erased the application SID, and no update could set a trunk SID. Only plain voice-URL routes survived a rollback.` | `Rollback restores TwiML application and SIP trunk routes. Updates cleared the application SID and had no trunk field.` |
| `// The state commits only on success.` above `rawDb.transaction(...)` | *(deleted, restates `transaction`)* |

The last row shows the third sentence being dropped as a restatement: "only voice-URL routes
survived" is derivable from "the application SID was cleared and no trunk could be set". Cut the
consequence, keep the cause.
