# Off-Limits-FTP-What-I-Learned
learning-only writeup for a CodePath CYB102 lab (Directory Traversal / FTP)
It does **not** contain exploit code or raw PCAPs. The goal is to document the small but important shell/bash lessons I learned while finishing the `attack.sh` script.

---

## TL;DR

* Start background processes with `&` so the script can keep running.
* Capture the background process PID with `$!` if you need to stop the process later.
* Variable assignment in bash **must not** contain spaces around `=`: `NAME=value` (not `NAME = value`).
* When using a variable, prefix it with `$` to expand it: `echo "$VAR"` or `node script.js "$URL"`.

---

## The three mistakes I fixed

### 1) Backgrounding vs foreground

**Problem:** I launched the server without backgrounding it. Because the server process ran in the foreground, the script blocked and never reached the attack step.

**Fix:** Start the server in the background and capture the PID:

```bash
# start server in background
node scripts/start-server.js &
# capture background PID
vulnpid=$!
```

**Why it matters:** `$!` gives the PID of the most recently backgrounded process. If you want to kill or otherwise manage that process later, you need that PID. If you don't background the server, `$!` will be empty or will not point to the intended process.

---

### 2) Waiting for things to start (don’t background the sleep)

**Problem:** I used `sleep 2 &` which backgrounded the sleep command. That meant the script did *not* actually wait, and the attack ran before the server was ready.

**Fix:** Use `sleep 2` (no `&`) to pause the script:

```bash
# wait for server to start
sleep 2
```

**Why it matters:** Backgrounding a command with `&` returns control to the shell immediately. For waits you usually want the script to pause so the service becomes available.

---

### 3) Variable assignment and expansion

**Problem A:** I had `ATTACK_PATH = "http://localhost:8888/timmy/reports_original.txt"` — the spaces around `=` caused the shell to treat `ATTACK_PATH` as a command and the string as an argument.

**Problem B:** I passed `ATTACK_PATH` (without `$`) to `node`, which supplied the literal word `ATTACK_PATH` instead of the URL.

**Fix:** Assign without spaces and use `$` when referencing:

```bash
ATTACK_PATH="http://localhost:8888/timmy"
node scripts/attack.js "$ATTACK_PATH"
```

**Why it matters:** Bash syntax requires `NAME=value`. Any spaces make the shell parse the parts as separate tokens (commands/arguments). Also, variables are expanded only when you use `$NAME` or `${NAME}`.

---

## A short checklist I now use when writing small bash orchestration scripts

* [ ] Make long-running services run in the background if the script must continue.
* [ ] Capture PIDs (`$!`) right after backgrounding if you intend to stop the process later.
* [ ] Use `sleep` (foreground) to wait for services to become ready — only background sleeps if intentionally parallelizing.
* [ ] No spaces around `=` when assigning variables.
* [ ] Use quotes around variable expansions to avoid word-splitting: `"$VAR"`.
* [ ] Test the command manually first (e.g., `node scripts/attack.js "http://localhost:8888/timmy"`) to confirm it behaves before putting it into a script.

---

## Safety & Ethics

This writeup is educational. Do **not** run attack tools against any system you do not own or have explicit permission to test.

---


*Created while completing CodePath CYB102 — Directory Traversal project.*
