---
name: reviewer-spec-location
description: >
  Spec-location procedure for code reviewers: before any correctness, architecture,
  security, quality, or test-coverage review, run this skill to find the authoritative
  spec document. Without a spec you cannot review correctly — you might approve code
  that violates the agreed design, or block code for requirements that were already
  changed. Use this skill at the very start of every review heartbeat, before reading
  any code or posting any findings. Also invoke when you are unsure where the spec is,
  when a prior reviewer left a "spec not found" comment, or when asked to verify that
  your review was done against the right document.
---

## Why This Matters

A review without a located spec is a guess. Three real cases where getting this wrong burned time:

- **[TEC-1727](/TEC/issues/TEC-1727)**: spec was at the grandparent level (`TEC-1569#document-spec`). Reviewers who stopped at the direct parent found nothing and reviewed against stale ACs.
- **[TEC-1666](/TEC/issues/TEC-1666)**: spec was the issue description itself. Reviewers who hunted for a `document-spec` first wasted a round-trip.
- **[TEC-1311](/TEC/issues/TEC-1311)**: ACs were inline in the task description, no document at all.

The four-step search below covers all known patterns in order of authority.

---

## Step 1 — Check the current review issue for a `document-spec` document

```
GET /api/issues/{currentIssueId}/documents/spec
```

If the response is 200 and the document body is non-empty, **this is your spec**. Fetch the full body and proceed to the [Cite and Continue](#cite-and-continue) section.

> Why this order: the most specific spec wins. If someone attached a spec document directly to the review issue, it supersedes anything upstream.

---

## Step 2 — Check the parent issue

```
GET /api/issues/{currentIssueId}   # field: parentId
GET /api/issues/{parentId}/documents/spec
```

First fetch the current issue to read `parentId`. Then fetch the parent's `document-spec`. If 200 and non-empty, **this is your spec**. Proceed to [Cite and Continue](#cite-and-continue).

---

## Step 3 — Check the grandparent issue

```
GET /api/issues/{parentId}   # field: parentId  → grandparentId
GET /api/issues/{grandparentId}/documents/spec
```

Fetch the parent issue to read its `parentId` (the grandparent). Then fetch the grandparent's `document-spec`. If 200 and non-empty, **this is your spec**.

---

## Step 4 — Fall back to numbered acceptance criteria

If no `document-spec` was found in Steps 1–3, look for numbered acceptance criteria (ACs) in the **issue descriptions** of the current issue and then the parent issue.

Numbered ACs look like:
- `1. The endpoint returns HTTP 200 on success.`
- `AC1: …`, `AC-1: …`, `Acceptance Criteria: …` followed by a bulleted/numbered list

If you find ACs in the description, **treat them as the spec**. Note the source (current issue description or parent issue description) and proceed to [Cite and Continue](#cite-and-continue).

---

## Step 5 — No spec found: block and request

If all four steps found nothing, **do not begin the review**. Post a comment on the review issue:

```
No authoritative spec or acceptance criteria found after checking:
- Current issue documents (document-spec)
- Parent issue [TEC-XXXX] documents (document-spec)
- Grandparent issue [TEC-YYYY] documents (document-spec)
- Current and parent issue descriptions (numbered ACs)

Please link the spec document before I proceed. @Dev Lead or issue creator: attach a `document-spec` document to [TEC-XXXX] or add numbered acceptance criteria to the issue description.
```

Then set the issue to `blocked` with `blockedByIssueIds` pointing to the parent or grandparent, and name the Dev Lead or creator as the unblock owner.

> Reasoning: a review without a spec wastes everyone's time. It is better to pause for one round-trip to get the right document than to produce a review that misses a requirement or flags the wrong things.

---

## Cite and Continue

Once you have located the spec, always include this line near the top of your review comment:

```
Reviewed against spec at [TEC-NNNN#document-spec](/TEC/issues/TEC-NNNN#document-spec).
```

Or, for AC-based specs:

```
Reviewed against acceptance criteria in [TEC-NNNN](/TEC/issues/TEC-NNNN) issue description.
```

Use the actual issue identifier, not a placeholder. This citation serves two purposes: it makes your review reproducible (anyone can read the same spec) and it signals to the next reviewer that the spec lookup was done.

---

## Quick Reference

| Step | API call | Condition to proceed |
|------|----------|----------------------|
| 1 | `GET /api/issues/{currentId}/documents/spec` | 200 + non-empty body |
| 2 | `GET /api/issues/{parentId}/documents/spec` | 200 + non-empty body |
| 3 | `GET /api/issues/{grandparentId}/documents/spec` | 200 + non-empty body |
| 4 | Read description of current + parent for numbered ACs | ACs present |
| 5 | Post blocking comment, set `blocked` | (fallback — no spec found) |

---

*TEC Custom Skill — maintained by the Deltek Technical Services Engineering team.*
