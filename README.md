# Bash Unfucked

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**10 real-world Bash rescue scenarios.** Each one: what went wrong, what to run, and why it works.

No fluff. No theory. Just the fixes you need when Bash has you cornered.

## Get the full version — 30 scenarios, $2

This repo is a free sampler. The full **Bash Unfucked** reference covers all 30 rescue scenarios including pipe gotchas, cron failures, portability traps, process management, and encoding nightmares.

**[Get Bash Unfucked on Gumroad →](https://sadafade.gumroad.com/l/bash-unfucked)**

---

## Free guides on our blog

- [5 Git Mistakes Every Developer Makes (And How to Fix Them)](https://highfadefree.vercel.app/blog/git-mistakes)
- [The Git Recovery Cheatsheet](https://highfadefree.vercel.app/blog/git-recovery-cheatsheet)
- [How to Undo a Git Commit (The Right Way)](https://highfadefree.vercel.app/blog/undo-git-commit)
- [How to Recover a Deleted Git Branch](https://highfadefree.vercel.app/blog/recover-deleted-git-branch)
- [How to Recover a Lost Git Stash](https://highfadefree.vercel.app/blog/git-stash-recovery)
- [Git Reflog: The Complete Recovery Guide](https://highfadefree.vercel.app/blog/git-reflog-tutorial)

---

## 1. Unquoted variable breaks on paths with spaces

```bash
# Broken
cp $FILE /backup/

# Fixed
cp "$FILE" /backup/
```

**Why:** Without quotes, the shell word-splits `$FILE`. A path like `my project/file.txt` becomes two arguments. Always double-quote variables.

---

## 2. `rm -rf "$DIR/"` with empty variable deletes everything

```bash
# Dangerous — if DIR is empty, this runs: rm -rf /
rm -rf "$DIR/"

# Safe — abort if DIR is unset or empty
rm -rf "${DIR:?'DIR is not set'}/"
```

**Why:** `${VAR:?msg}` exits with an error if `VAR` is unset or empty. This is the canonical guard against accidental root deletion.

---

## 3. Script continues after a command fails

```bash
# Broken — keeps running even if cd fails
cd /nonexistent
rm -rf *  # now deletes files in the WRONG directory

# Fixed — exit on any error
set -euo pipefail
cd /nonexistent  # script exits here
```

**Why:** By default, bash ignores non-zero exit codes. `set -e` exits on failure, `-u` errors on undefined variables, `-o pipefail` catches failures in pipelines.

---

## 4. Pipe hides the real error

```bash
# Broken — exit code is from wc, not the failing curl
curl http://broken-url | wc -l
echo $?  # 0

# Fixed
set -o pipefail
curl http://broken-url | wc -l
echo $?  # non-zero
```

**Why:** Without `pipefail`, the exit code of a pipeline is the exit code of the last command only. `pipefail` makes it the exit code of the rightmost command that failed.

---

## 5. Variable set inside a pipe loop is empty afterward

```bash
# Broken — pipe creates a subshell
count=0
cat file.txt | while read line; do
    ((count++))
done
echo "$count"  # prints 0

# Fixed — redirect from file
count=0
while read line; do
    ((count++))
done < file.txt
echo "$count"  # correct count
```

**Why:** Each side of a pipe runs in a subshell. Variables set in a subshell don't propagate to the parent.

---

## 6. `for f in *` breaks on filenames with spaces

```bash
# Broken — word splitting
for f in $(ls); do echo "$f"; done

# Fixed — use glob directly
for f in *; do echo "$f"; done
```

**Why:** `$(ls)` output undergoes word splitting. Glob expansion (`*`) preserves filenames as-is. Never parse `ls` output.

---

## 7. Script works in terminal but fails in cron

```bash
# Broken — cron has a minimal PATH
#!/bin/bash
convert input.png output.jpg  # command not found

# Fixed
#!/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
/usr/local/bin/convert input.png output.jpg
```

**Why:** Cron uses a minimal environment. Either set `PATH` at the top of your script or use absolute paths.

---

## 8. `^M` / carriage return in script (Windows line endings)

```bash
# Symptom: /bin/bash^M: bad interpreter

# Fix
sed -i 's/\r$//' script.sh

# Or
dos2unix script.sh

# Prevention
git config --global core.autocrlf input
```

**Why:** Windows uses `\r\n` line endings; Unix uses `\n`. The `\r` breaks the shebang line and syntax.

---

## 9. `#!/bin/sh` script uses bash-only features

```bash
# Broken — arrays and [[ are bash-only
#!/bin/sh
arr=(1 2 3)        # syntax error in dash
[[ $x == "y" ]]    # syntax error in dash

# Fixed — use bash shebang
#!/bin/bash
arr=(1 2 3)
[[ $x == "y" ]]
```

**Why:** `/bin/sh` is often `dash` on Debian/Ubuntu, not bash. Arrays, `[[`, `(( ))`, and brace expansion fail. Use `#!/bin/bash` or write POSIX sh.

---

## 10. Background job dies when terminal closes

```bash
# Broken — background job receives SIGHUP
./long-running-task.sh &

# Fixed — use nohup
nohup ./long-running-task.sh > output.log 2>&1 &

# Or disown it
./long-running-task.sh &
disown
```

**Why:** When a terminal closes, it sends `SIGHUP` to all child processes. `nohup` ignores that signal. For production, use a real process manager.

---

## Quick Reference: Common Safe Patterns

| Pattern | Unsafe | Safe |
|---------|--------|------|
| Variable usage | `$var` | `"$var"` |
| Loop over files | `for f in $(ls)` | `for f in *` |
| Script header | `#!/bin/sh` (with bashisms) | `#!/bin/bash` + `set -euo pipefail` |
| Temp files | `tmp=/tmp/myfile` | `tmp=$(mktemp); trap 'rm -f "$tmp"' EXIT` |
| Printing | `echo -e` / `echo -n` | `printf` |
| Empty var guard | `rm -rf $DIR/` | `rm -rf "${DIR:?}/"` |
| Read input | `read line` | `IFS= read -r line` |

---

## Want all 30 scenarios?

This is 10 of 30 scenarios from **Bash Unfucked** — the complete interactive Bash recovery reference.

The full version includes:
- **30 real-world scenarios** (quoting, pipes, cron, encoding, portability, and more)
- **Interactive HTML reference** — searchable, works offline
- **Quick-reference cheat sheet**

**[$2 on Gumroad →](https://sadafade.gumroad.com/l/bash-unfucked)**

---

*Also check out [Git Unfucked](https://github.com/highfadehq01/git-unfucked) — 32 Git recovery scenarios.*

---

Built by **[High Fade Free](https://highfadefree.vercel.app)** — minimal, precision developer tools.

*Ship clean. Stay sharp.*
