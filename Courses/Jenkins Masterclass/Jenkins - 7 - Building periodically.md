### Scheduling Jobs Periodically in Jenkins

In Jenkins, you can configure jobs to run automatically at specified times using the **"Build periodically"** option in the job configuration. This uses the **CRON-like syntax** provided by Jenkins, where the schedule format is:

```
MINUTE HOUR DOM MONTH DOW
```

- **MINUTE** — 0–59
- **HOUR** — 0–23
- **DOM** (Day of Month) — 1–31
- **MONTH** — 1–12 or names (e.g., Jan, Feb)
- **DOW** (Day of Week) — 0–7 (0 or 7 = Sunday) or names (e.g., Mon, Tue)

Special characters:
- `*` — any value
- `,` — multiple values (e.g., `0,15,30,45`)
- `-` — range (e.g., `9-17`)
- `/` — step (e.g., `*/5` = every 5 units)

**Examples:**
- `H/15 * * * *` — Every 15 minutes, using `H` to distribute load evenly.
- `0 2 * * *` — Every day at 02:00.
- `0 8-18/2 * * 1-5` — Every 2 hours between 8:00 and 18:00 on weekdays.
- `H 0 * * 0` — Once every Sunday at midnight.
The `H` token is Jenkins-specific and hashes the job name to spread builds evenly across available minutes/hours, avoiding all jobs starting at the same time.
### SCM Polling in Jenkins

**SCM polling** automatically checks your Source Control Management (SCM) repository (e.g., Git, Subversion) at regular intervals, and triggers a build if changes are detected.  
You configure it in the **"Poll SCM"** option in the job configuration.

The schedule uses the same **CRON-like syntax** as "Build periodically".

![[Pasted image 20250812083814.png]]

![[Pasted image 20250812083637.png]]