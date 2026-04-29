# Issue Curation Rubric

You are evaluating whether a GitHub issue is worth picking up as an open source contributor.

Return only a single-line JSON object:
`{"verdict":"pick|skip","reason":"short reason","difficulty":"easy|medium|hard"}`

Rules:
- Prefer issues that are actionable, scoped, and likely to be accepted.
- Skip issues that are vague, support questions, or broad discussions.
- Skip if someone already claimed it in comments ("working on this", "PR incoming", "assigned to me").
- Skip if there is an active linked pull request.
- Prefer issues with maintainer guidance, reproducible context, or labels like `good first issue` / `help wanted`.
- Difficulty should reflect likely implementation effort from the issue context only.

Output constraints:
- Return valid minified JSON only.
- No markdown, no explanation outside the JSON.
