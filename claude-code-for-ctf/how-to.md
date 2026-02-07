## WSL bash script that creates the full CTF-ready structure (memory + modular rules + logs) in the current directory (or an optional target dir).

```bash
cat > ctf-init.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

TARGET="${1:-.}"

mkdir -p "$TARGET/.claude/rules" "$TARGET/notes"

# Project memory entrypoint
cat > "$TARGET/.claude/CLAUDE.md" <<'MD'
# Project: CTF challenge

Always read the logs first:
- @notes/ATTEMPTS.md
- @notes/FAILURES.md

Project rules:
- @.claude/rules/ctf.md
MD

# Modular CTF rules (anti-repeat + loop discipline)
cat > "$TARGET/.claude/rules/ctf.md" <<'MD'
# CTF Rules (Project)

## 0) Prime directive: do not repeat known failures
- Before proposing any action, read:
  - @notes/ATTEMPTS.md
  - @notes/FAILURES.md
- If a command/payload/approach is in FAILURES.md, DO NOT repeat it.
- Exception: you may retry only if you explicitly state what has changed
  (new evidence, corrected assumption, different input/endpoint/build, etc.).

## 1) Loop discipline (anti-random-walk)
Repeat:
1) Observe: collect the minimum needed facts.
2) Hypothesize: list 3–5 candidate paths ranked by likelihood × cost.
3) Test: run the smallest falsifying test.
4) Verify: compare output vs expected signal.
5) Log: append the exact attempt and result.
6) Decide: continue / pivot / deepen.

## 2) Logging format (so future sessions can resume instantly)
Append to ATTEMPTS.md after every attempted action:

### [YYYY-MM-DD HH:MM] <short label>
- Hypothesis:
- Action (exact commands/requests):
- Expected signal:
- Actual result:
- Conclusion:
- Next:

If it fails or is ruled out, also append to FAILURES.md:

### <short label>
- What was tried (exact):
- Why it failed:
- What must change to justify retrying:

## 3) Stop conditions (prevent endless loops)
- If 3 consecutive tests fail with no new information, STOP and:
  - summarize current knowledge,
  - list missing info,
  - propose 2 new angles not attempted yet.

## 4) Category triage checklists (first 10 minutes)
### Web
- Map endpoints/auth/roles/params/uploads.
- Identify cookies/JWT/session, CSRF, CORS, rate limits.
- Do targeted fuzzing only after you have a hypothesis.

### Pwn
- Identify arch + mitigations (PIE/NX/RELRO/canary), libc linkage, IO style.
- Repro crash or primitive with minimal PoC; log offsets/versions.
- Prefer leak-first strategies; keep steps reproducible.

### Rev
- Identify input format and checks (length/charset/transforms).
- Extract constants/tables/keys; confirm with a tiny harness.
- Log confirmed behaviors and pivot points.
MD

# Logs
cat > "$TARGET/notes/ATTEMPTS.md" <<'MD'
# Attempts log
(Empty at start — append entries as you go.)

## Template
### [YYYY-MM-DD HH:MM] <short label>
- Hypothesis:
- Action (exact commands/requests):
- Expected signal:
- Actual result:
- Conclusion:
- Next:
MD

cat > "$TARGET/notes/FAILURES.md" <<'MD'
# Failures log
(Empty at start — add items only when something fails or is ruled out.)

## Template
### <short label>
- What was tried (exact):
- Why it failed:
- What must change to justify retrying:
MD

echo "CTF project initialized at: $TARGET"
echo "Created:"
echo "  - $TARGET/.claude/CLAUDE.md"
echo "  - $TARGET/.claude/rules/ctf.md"
echo "  - $TARGET/notes/ATTEMPTS.md"
echo "  - $TARGET/notes/FAILURES.md"
EOF

chmod +x ctf-init.sh

```

In a new challenge folder:

```bash
mkdir -p ~/ctf/chal1 && cd ~/ctf/chal1
/path/to/ctf-init.sh .
```

Or create everything into a target directory from anywhere:

```bash
./ctf-init.sh ~/ctf/chal2
```

Optional (but very useful): make it a command , add alias:

```bash
echo "alias ctfinit='$PWD/ctf-init.sh'" >> ~/.bashrc
source ~/.bashrc
```

then you can do:

```bash
ctfinit .
```

## One more thing to ensure Claude actually follows it

When you open Claude Code in that directory, start with:

“Read notes/ATTEMPTS.md and notes/FAILURES.md first, then propose 3 hypotheses with smallest tests. Log everything.”

After that, it becomes self-reinforcing because every session begins by importing the logs.

