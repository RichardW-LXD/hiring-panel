# hiring-panel

A Claude Code skill. You give it a job posting, a cover letter and a resume. It sends three fresh reviewers (a recruiter, a hiring manager, a skeptic) to read them. They tell you what would stop them. No scores.

The skeptic checks every number against your real resume, so you can catch a wrong number before a hiring manager does.

## Install (one time)

You need [Claude Code](https://claude.com/claude-code) and [Git](https://git-scm.com/downloads).

**Windows (PowerShell):**

```
git clone https://github.com/RichardW-LXD/hiring-panel.git "$HOME\.claude\skills\hiring-panel"
```

**Mac / Linux:**

```
git clone https://github.com/RichardW-LXD/hiring-panel.git ~/.claude/skills/hiring-panel
```

Restart Claude Code. Done.

## Use

1. In Claude Code, say: **"Run the hiring panel."**
2. The first time, it asks where you want to keep your applications. Pick any folder you like, for example inside your Claude folder. It creates it if it doesn't exist. You do this once.
3. It asks you to paste the job description. It makes a new folder for that job inside your applications folder by itself, and saves the description there.
4. It asks for your custom cover letter and custom resume for this job, and copies them into the job folder.
5. It asks for your source resume: the full, true record of your work, in any format (.md, .docx, .pdf). Every number in your drafts is checked against it.
6. You get `panel-review.md` in the job folder, with three lists: **Fix now**, **Ask me**, **Noted, no action**.

For the next job, say "Run the hiring panel" again. It makes another job folder for you.

The skill never edits your letter or resume. It only flags.

## Update

```
cd "$HOME\.claude\skills\hiring-panel"
git pull
```
