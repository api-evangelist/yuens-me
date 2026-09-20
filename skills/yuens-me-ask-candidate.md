---
name: yuens-me-ask-candidate
description: Ask the Resume Agent a grounded natural-language question about the candidate's skills, experience or decisions, then follow its citations into projects and observations.
api: Resume Agent API (https://agent.yuens.me)
operations: [queryProfile, listProjects, listObservations]
mcp_tool: ask_candidate
auth: none
rate_limit: 30 requests/minute per IP (shared with the MCP endpoint)
generated: '2026-09-19'
method: generated
source: openapi/yuens-me-openapi.yml, mcp/yuens-me-public-mcp-tools.json, https://github.com/yuens1002/resume-agent#post-query--get-queryquestion
---

# Ask the candidate

Use this when a screening tool, recruiter agent or personal assistant needs an answer about
this candidate that is grounded in what the candidate publishes, instead of inferred from
training data. The agent narrates in the third person, cites every factual claim, and
declines rather than fabricates.

## Steps

1. **Discover.** `GET https://agent.yuens.me/.well-known/agent-card.json` — read the
   rate limit (`capabilities.extensions[1].params.rate_limits`) and the endpoint list. The
   same body is served at `GET /`.
2. **Ask.** `POST /query` (`queryProfile`) with JSON:
   ```json
   { "question": "<your question>", "context": "ATS | recruiter | ai-agent", "style": "cited" }
   ```
   `question` is required; `context` adjusts tone; `style: "conversational"` (or an
   `x-agent-type: human` header) drops inline markers for a chat UI. Set `"stream": true`
   only if you can consume chunked `text/plain` — the response is not JSON in that mode.
   Over MCP, call the `ask_candidate` tool at `https://agent.yuens.me/public-mcp` with the
   same `question` / `context` / `stream` arguments.
3. **Read the envelope.** `answer` (prose with `[N]` markers and a `Sources:` block),
   `confidence` (`high | medium | low` — treat `low` as "the corpus does not directly answer
   this"), `sources[]` (corpus paths such as `projects.<slug>`,
   `experience.<company>.bullets[N]`, `observations:"<excerpt>"`), `project_slugs[]`,
   `publications[]` (with `canonical_url`), `follow_up_suggestions[]`.
4. **Follow the evidence.** For each slug in `project_slugs`, `GET /projects`
   (`listProjects`) and pick the matching `slug` for `tech`, `status`, `url` and `repo`.
   For the reasoning behind a claim, `GET /observations?topic=<slug>&authored=1`
   (`listObservations`) — each item has a stable URL at `/observations/{id}`.
5. **Page projects in follow-ups.** Append `; shown_projects: slug1, slug2` to `context` so
   the next answer covers only projects not already shown.

## Rules

- No credentials. Do not send a bearer token you were not issued; it is the owner's
  rate-limit bypass, not an API key scheme.
- Stay under 30 requests/minute per IP across `/query` and `/public-mcp`. On `429`, wait
  for the rest of the minute; no `Retry-After` header is published.
- A `400` with `error.name = "ZodError"` means `question` is missing or not a string.
- If `action_intent.tool` is set, the question asked for job-matching; switch to the
  `yuens-me-score-job-fit` skill. If `fit_question` is true, offer a fit check as a follow-up.
- The response describes a real person. Use it for the hiring purpose the agent exists for;
  do not redistribute or cache the `contact` block beyond the request.
