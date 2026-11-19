---
name: rule-check
description: Check a finished clip against technical specs, campaign rules and banned content before posting. Use before every upload, or when the user asks "is this ok to post".
---

# Rule check

Run:

`python "${CLAUDE_PLUGIN_ROOT}/scripts/check.py" <clip.mp4> --campaign campaigns.md`

The script checks technical properties and limited caption keywords. Its campaign parser searches the whole file and cannot reliably select one campaign. Do not treat its PASS as campaign approval. Select the exact campaign and check its sourced rules manually; flag missing or ambiguous rules as UNKNOWN. Verify permitted creator, platform, duration, credit, hashtags, deadline and submission separately. Do not approve a clip when a required rule is unknown.

Then:

1. Show PASS or FAIL per check in a short list.
2. On FAIL: fix the cause (re-cut with `clip-cutter`, add the credit, trim) and re-run. Do not argue with a failed check.
3. Read the caption yourself too. It must credit the creator in the text, no invented facts, no "official" wording, and match the platform's caption rules in the playbook.
4. The script cannot judge everything. If the campaign rules are `unknown` or missing, ask the human to paste them.
5. Only after all checks pass, send the human the clip, caption and credit and wait for "yes". The human posts it. Never auto-upload.
