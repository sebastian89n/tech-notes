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

Jobs has "Git Polling Log" that displays information about polling attempts.

![[Pasted image 20250812083814.png]]

# Webhooks instead of polling

## GitHub

- In Jenkins job: check **“GitHub hook trigger for GITScm polling.”**
- On GitHub repo: **Settings → Webhooks → Add webhook**
    - Payload URL: `https://<your-jenkins>/github-webhook/`
    - Content type: `application/json`
    - Events: “Just the push event” (or what you need)
- Remove your Poll SCM schedule (or leave empty) to avoid double builds
## GitLab

- Install/enable the **GitLab Plugin** (or use Generic Webhook Trigger).
- In Jenkins job: check **“Build when a change is pushed to GitLab (webhook)”** or configure the Generic Webhook trigger.
- On GitLab repo: **Settings → Webhooks**
    - URL: `https://<your-jenkins>/project/<job-name>` (GitLab Plugin) or `https://<your-jenkins>/gitlab-webhook/`
    - Events: Push (and Merge Request if desired)
## Bitbucket
- Enable **“Build when a change is pushed to Bitbucket”** in job (Bitbucket plugin).
- In Bitbucket: **Repository settings → Webhooks**
    - URL: `https://<your-jenkins>/bitbucket-hook/`
    - Event: Push
- Remove your Poll SCM schedule (or leave empty) to avoid double builds.

![[Pasted image 20250812083637.png]]