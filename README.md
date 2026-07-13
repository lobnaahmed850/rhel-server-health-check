# RHEL Server Health Check

A bash script that performs remote SSH-based health checks across multiple RHEL/Linux servers — checking uptime, disk usage, memory usage, and failed SSH login attempts — using strict-mode error handling and safe input parsing.

Built as a DevOps capstone project to apply bash scripting fundamentals (variables, arrays, functions, loops, `getopts`, text processing) in a realistic sysadmin automation context.

## Features

- Health checks across any number of servers listed in a simple text file
- Reports for each server: system uptime, root partition disk usage, memory usage, and failed SSH login attempt count
- Strict mode (`set -euo pipefail`) so the script fails fast and loud instead of silently continuing after an error
- Input validation via `getopts` (`-f` for server list, `-u` for remote user, `-h` for usage help)
- Logging to a temporary log file (created with `mktemp`), with automatic cleanup on exit
- Skips blank lines and `#`-comment lines in the server list file
- Safe file-reading and argument-handling patterns (`IFS= read -r`, quoted `"$@"`/array expansions) to avoid word-splitting bugs

## Usage

```bash
chmod +x server_health_check.sh
./server_health_check.sh -f servers.txt -u <remote_user>
```

| Flag | Description |
|------|-------------|
| `-f` | Path to a file listing target servers (one hostname/IP per line) |
| `-u` | Remote SSH username to connect as |
| `-h` | Show usage help |

Copy `servers.txt.example` to `servers.txt` and edit it with your real target hosts before running.

## Requirements

- SSH access (as the specified user) to every target host
- The remote user must be able to log in via SSH — either password authentication enabled on the target, or SSH keys set up (`ssh-copy-id`)
- For the failed-login check to return real data, the remote user needs read access to the server's auth log (`/var/log/secure` on RHEL, `/var/log/auth.log` on Debian/Ubuntu) — see **Known Limitations** below

## Known Limitations

- **Auth log path is RHEL-specific.** The script reads `/var/log/secure`. Debian/Ubuntu-based targets would need this changed to `/var/log/auth.log`.
- **Auth log permissions.** `/var/log/secure` is root-readable only by default. Running the check as a non-root user requires a scoped, passwordless sudo rule, e.g.:
  ```
  <user> ALL=(root) NOPASSWD: /usr/bin/grep -c Failed\ password /var/log/secure
  ```
  Note sudoers doesn't interpret `"quoted strings"` the way bash does — the space must be backslash-escaped, not quoted, or the rule silently won't match.
- **No SSH key automation built in.** The script currently relies on whatever auth method (password or key) is already configured on the target; it doesn't set up keys itself.
- **Sequential, not parallel.** Servers are checked one at a time — fine for a handful of hosts, but would need backgrounding/`xargs -P` to scale to many servers efficiently.

## Possible Improvements

- Switch from password-based SSH to key-based authentication by default
- Support both RHEL (`/var/log/secure`) and Debian (`/var/log/auth.log`) automatically based on remote OS detection
- Run health checks in parallel instead of sequentially
- Add a `--json` output mode for feeding results into monitoring/alerting tools
