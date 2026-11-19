# Security audit — gtavi-claudecode

Date: 2026-11-19. Revision: `dd95d393c6f74fe2ac14b0239f8826691e5dcb0e`.

## Scope and method

Reviewed all 38 tracked files: 13 Python scripts, 16 skill instructions, two skill references, plugin manifests, dependencies, README, license, gitignore and the PNG file type. Codebase Memory excluded `scripts` and `docs`; those were inspected directly. No dependencies were installed and no untrusted media was executed. Local reproductions used temporary files and mocked network/process failures. This is a source review, not a guarantee against dependency vulnerabilities or malicious media.

## Findings

### M1 — Previous live recording deleted before new recording starts

`scripts/live.py:216-220` deletes `seg_*.ts`, chat logs and segment index for the deterministic job of the same URL, before creating `Recorder`. A failed new recording still destroys those inputs. Reproduced with a temporary job and a mocked recorder failure. Use unique session directories and explicit retention; never delete prior sessions on start.

### M2 — Campaign rule check may apply another campaign's rules

`scripts/check.py:18-29` searches the entire campaign file for the first match and has no campaign selector. A local fixture with A's maximum of 60 and B's maximum of 10 returned 60. Even an existing file with no matching rules is marked present; the result may say PASS using defaults. Require an explicit campaign identifier, parse structured per-campaign rules, fail closed on absent or ambiguous rules. Manual approval remains relevant.

### M3 — Twitch chat silently downgrades from TLS

`scripts/twitch.py:95-130` retries on port 6667 without TLS after a non-timeout connection or TLS error. Mocking a failed connection produced ports 6697 then 6667. A network observer can read or tamper with anonymous chat; forged messages can affect spike selection. No authenticated Twitch token is sent on this IRC path. Fail closed on TLS errors and expose a useful error.

### M4 — Untrusted remote content reaches agent workflow without an explicit trust boundary

`skills/campaign-finder/SKILL.md` instructs agents to read third-party pages; `scripts/questions.py`, `scripts/trends.py`, `scripts/streams.py` and `scripts/moments.py` copy outside text into reports subsequently read by agents. Malicious creator text, chat or web pages could contain instructions to influence an agent. This is a trust-boundary risk, not a demonstrated exploit or proof of data exfiltration. Label external content as data, prohibit following instructions in it, and require explicit authorization for actions and external destinations.

### L1 — Predictable voice temporary path

`scripts/translate.py:148` uses a predictable filename in the shared temporary directory. Concurrent jobs can collide; another local user able to create or manipulate that path may interfere. Exploit depends on platform and temporary-directory permissions. Use a private `TemporaryDirectory` and unique files.

### L2 — Unpinned remote model and Python packages

`scripts/facecam.py:22-33` downloads a model from a mutable branch and accepts files based on size alone. `requirements.txt` sets only lower bounds and `skills/setup/SKILL.md:12-16` directs an upgrade. Compromise or incompatible upstream changes can affect installs; no compromise or malicious payload was established. Pin model revision and checksum; use a locked, hashed dependency set, isolated environment and deliberate updates.

### L3 — Skill approval instructions conflict

`skills/playbook/SKILL.md:15` says to wait for approval and let the human post, while `skills/playbook/references/playbook.md` sections 8–11 direct publishing, scheduling and engagement. `skills/questions-to-posts/SKILL.md` also says to post after checking. Actual execution depends on the host agent and permissions; no automatic upload exists in the reviewed scripts. Make approval and human-only publication consistent across every instruction.

## Additional observations

- `cut.py` uses `ffmpeg -y` for deterministic output names, so repeating the same pick overwrites prior exports. This may be intentional but should be disclosed or guarded.
- `translate.py` sends text to Google Translate and, with `--voice`, Microsoft Edge TTS. Disclose this before processing sensitive text.
- `check.py` checks a fixed keyword list in hook/caption, not speech, frames or all campaign rules; manual review is required. Its default PASS must not be treated as a complete safety or compliance verdict.
- Python subprocesses inspected use argument arrays rather than `shell=True`; this does not eliminate risks in downstream media parsers or argument semantics.
- The plugin manifests register skills; no automatic hook or MCP server was found in tracked files. No credential-exfiltration behavior was identified in this source review.

## Limits

The campaign, deletion and TLS downgrade reproductions were run locally. No live Twitch connection, third-party services, model download, ffmpeg processing, spreadsheet application, agent prompt injection or complete end-to-end publication was tested. PNG content was identified by file type, not audited as a decoder exploit. Dependency and OS security were not audited. No severity is a claim of remote code execution.
