---
name: yuens-me-browse-evidence
description: Pull the structured profile, the portfolio and the authored observation trail behind it, with honest paging, for an ATS or verification pass that wants facts without NL processing.
api: Resume Agent API (https://agent.yuens.me)
operations: [getProfile, listProjects, listObservations]
auth: none
rate_limit: 30 requests/minute per IP
generated: '2026-09-19'
method: generated
source: openapi/yuens-me-openapi.yml, https://github.com/yuens1002/resume-agent#get-observations--get-observationsid
---

# Browse the evidence

Use this when you want raw, cacheable JSON — no LLM call — for indexing, verification or
a structured ATS import.

## Steps

1. **Snapshot.** `GET /info` (`getProfile`) returns `contact`, `summary`, `tagline`,
   `skills[]` (grouped by category), `employment[]`, `education[]`, `projects[]`,
   `publications[]`, `availability` and `verification_status`. This is the whole public
   profile in one response.
2. **Portfolio.** `GET /projects` (`listProjects`) returns an array of
   `{ name, slug, description, role, tech[], status, started, url, repo, cover }`. `slug`
   is the key that `/query` answers cite in `project_slugs`.
3. **Reasoning trail.** `GET /observations` (`listObservations`) with:
   - `topic=<tag>` (case-insensitive), `type=observation|idea|task|reference`
     (default excludes `reference`, the git-sync ledger), `since=YYYY-MM-DD`,
   - `authored=1` for hand-written notes only (`authored=0` for machine sync/telemetry),
   - `limit=1..500` (default 25).
   Read `count`, `total` and `truncated`; if `truncated` is true, raise `limit` (up to 500)
   — there is no cursor. If you sent `authored=` and the envelope has no `authored` key, the
   deployment predates the filter and the list is unfiltered.
4. **Cite a single note.** Each observation is addressable at
   `GET /observations/{id}`; a private or unknown id returns `404`, never `403`.
5. **Verify identity (optional).** `GET /.well-known/oep-public-key.json` and compare its
   `fingerprint` with the `_oep.yuens.me` DNS TXT record (`v=oep1; alg=ed25519; fp=…`) and
   the agent card's `provider.identity.fingerprint`. A match proves the domain owner runs
   the agent.

## Rules

- All three reads are anonymous and count toward the 30 requests/minute per-IP ceiling.
- `contact` is personal data published for hiring contact; do not redistribute it.
- Do not treat `type=reference` entries as authored claims — they are commit-ledger noise.
