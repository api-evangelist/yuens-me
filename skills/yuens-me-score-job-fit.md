---
name: yuens-me-score-job-fit
description: Score the candidate against a job description with the Resume Agent's structured fit assessment, after checking availability.
api: Resume Agent API (https://agent.yuens.me)
operations: [getAvailability, matchJob]
auth: none
rate_limit: 30 requests/minute per IP
generated: '2026-09-19'
method: generated
source: openapi/yuens-me-openapi.yml, https://github.com/yuens1002/resume-agent#post-match, https://github.com/yuens1002/resume-agent#agentic-job-match-methodology
---

# Score job fit

Use this when a role is in hand and you need a structured, evidence-graded fit assessment
rather than a keyword match.

## Steps

1. **Check availability first.** `GET /availability` (`getAvailability`) returns
   `seeking`, `status` (`open | actively-looking | not-looking`), `preferred_roles[]` and
   `remote`. If `status` is `not-looking`, say so before scoring.
2. **Submit the job description.** `POST /match` (`matchJob`) with
   ```json
   { "job_description": "<the full JD text>" }
   ```
   Send the whole posting: the agent extracts the qualities *this* JD raises (skills,
   experience characteristics, domain characteristics) and scores only those.
3. **Read the result.** `fit_score` (0-1, a weighted average over the extracted
   qualities), `matched[]`, `gaps[]`, `verdict` (prose), `recommended_action`
   (`apply | apply-with-tailoring | pass`) and `scoring`:
   - `required_qualities[]` — what the JD asked for, each `must_have` or `preferred`;
   - `scored_qualities[]` — each with `verdict` (`matched | partial | missing`) and
     `evidence_grade` (`verified` = a dated project or employment record backs it,
     `claimed` = prose only, `absent` = no support).
4. **Weight by evidence.** Prefer `verified` matches over `claimed` ones when summarising;
   a `must_have` that is `missing` should be surfaced explicitly.
5. **Go deeper if needed.** For any `verified` quality, use the `yuens-me-ask-candidate`
   skill to ask how it was applied and follow the citations to projects.

## Rules

- `job_description` is required; a `400` `ZodError` means it was missing.
- Scoring calls an LLM on the operator's side; keep to one call per JD and stay under the
  30 requests/minute per-IP ceiling.
- Do not paste confidential job descriptions you are not permitted to share with a
  third-party service; the operator logs public query traffic.
