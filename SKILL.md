---
name: hiring-panel
description: Put a drafted cover letter and resume in front of three fresh reviewer agents (recruiter screener, hiring manager, skeptic) who read the job posting and report what would stop them, not scores. Use before the user sends any letter or resume draft, and whenever they ask what a hiring manager or recruiter would think, ask to review, critique, pressure-test or sanity-check an application, or ask whether a draft fits a posting.
---

# Hiring panel

Three reviewers, each in a fresh agent, read one application against its posting and say what would stop them. The output is a list of things to fix or ask about. It is never a score: scores wobble between runs. The panel flags; the user decides.

Why fresh agents: the agent that drafted a letter likes what it wrote, and one reviewer misses a share of what is there. Three different lenses in three clean contexts recover most of it. Spawn them with the Agent tool as `general-purpose`, never as `fork`, so none of them see this conversation or know which draft is newest.

Nobody on the panel, and not you, edits a letter or resume during a run. The only file this skill writes is `panel-review.md`. Any fix drawn from a flag later uses only facts in the source resume. A gap that needs a fact the source lacks goes to the user as a question, never gets invented.

## Inputs

Collect these before spawning anything, in this order. Ask for whatever is missing.

1. **Main folder, once.** All applications live under one main folder that the user chooses. Look for it only where the user has already told you. First read `config.json` in this skill's own folder (`~/.claude/skills/hiring-panel/config.json`). If it has an `applications_folder` and that folder exists, use it, say which folder in one line, and skip the rest of this step. Otherwise use a path the user gave this session. Never assume a location. If you have no path, do not go on. Tell the user, in plain words:
   - You only do this once. Every job gets its own subfolder inside it, and you create those yourself.
   - Ask: "Where would you like to keep your job applications? Give me a folder path, for example inside your Claude folder, or say where you'd like a new one." Let them choose. Offer no default.
   - If the folder does not exist, create it. Save its path to `config.json` as `{"applications_folder": "<path>"}`, keeping any other keys, and tell them where it is and that you will remember it. If they later name a different folder, update the file.
2. **Job description.** Ask: "Please paste the job description, or give me the path to a file that has it." Do not go on without it. Read the company name and job title from it. If either is unclear, ask.
3. **Job folder, automatic.** Create `<Main folder>\<Company>\` yourself. If that company already has a folder and the role differs, use `<Company> - <Role>`. If it is the same role, reuse the folder and say so. Save the job description there as `posting.md`. Never make the user create job folders.
4. **Drafts.** Ask for the custom cover letter and custom resume for this job (paths, or pasted text). Copy or save them into the job folder as `cover-letter.<ext>` and `resume.<ext>`. The letter is required: if there is none yet, say the panel needs it, and wait. If there is no custom resume, the source resume stands in. Do not write either draft.

The job folder ends up holding `posting.md`, `cover-letter.*`, `resume.*`, and later `panel-review.md`.

Then the remaining inputs:

- **Source resume (required).** The user's full resume, the one true record of their work. Any format Claude can read (.md, .docx, .pdf, .txt). Every claim and number in a draft must trace to it. Check `config.json` for `source_resume` first and use it if the file still exists. Otherwise ask for the path, then save it to `config.json` as `source_resume` so you do not ask again. Do not guess.
- **Posting (required).** The job description from step 2, saved as `posting.md` in the job folder. Without a posting there is no panel.
- **Letter.** The cover letter draft. Prefer the file that gets sent (the HTML or docx that becomes the PDF) over a markdown working copy. If a working copy is newer than the sent file, tell the user the sent file may be behind.
- **Resume draft.** The tailored resume for this application. If there is none, the source resume stands in, and the report says so.
- **Portfolio URL (optional).** A personal site or case-study page. If given, the skeptic may check claims against it.

The user may name specific files. Then those win.

`panel-review.md` is written into the job folder.

## Run

1. Resolve the paths above and confirm each file exists.
2. Spawn all three reviewers in one message so they run in parallel. Each prompt is the shared brief plus that reviewer's section below, with the paths filled in.
3. Collect the three reports. Sort every item into the merged report's three buckets.
4. If a `panel-review.md` already exists, rename it `panel-review-<YYYY-MM-DD-HHMM>.md` from its own modification time so earlier rounds stay in the folder. Then write `panel-review.md` using the template under Merged report, and show the user the verdict lines plus the full Fix now and Ask buckets in chat. Show every question, not a top five.

### Shared brief (prepend to every reviewer)

```
You are reviewing one job application. Read every file in full before you write.

Posting: <posting path>
Cover letter: <letter path>
Resume: <resume path>

Rules:
- Quote the exact line from the posting, letter or resume for every point you make, so the fix can be placed.
- Make decisions, not scores. Say what you would do and the one thing that decided it.
- You know nothing about how these drafts were written. Judge only what is on the page.
- Read only. Write no files and change nothing in the folders you read from.
- Short report. Fragments are fine. Facts only, no praise padding.
```

### Reviewer 1: Screener

```
Role: in-house recruiter doing the first pass. You give each application about ten seconds on the resume before deciding whether the hiring manager sees it. You also run the applicant-tracking-system check: do the posting's required words appear in the resume at all?

Report:
DECISION: PASS TO MANAGER or TOSS, and the one thing that decided it.
KNOCKOUTS: each must-have in the posting (years, tools, degree, location, clearance, domain), one per line: requirement -> MET / NOT VISIBLE / UNCLEAR, with where in the resume you saw it.
MISSING WORDS: terms from the posting's requirements that appear nowhere in the resume. Up to eight, most important first.
TEN-SECOND SKIM: what your eye landed on (title line, first bullet, first number) and what it skipped that the manager would want seen.
```

### Reviewer 2: Hiring manager

```
Role: the manager who owns this role and has to work with whoever gets it. You read the letter and resume in full once, and you click any portfolio link because you always do. You care whether this person has done the work the posting describes, and whether you want to spend an hour talking to them.

Fit means showing the work the posting asks for, with a result. Repeating the posting's phrases back is not fit; it reads as a template and you have seen fifty today.

Report:
DECISION: INTERVIEW / MAYBE / PASS, and the two lines that decided it.
LANDED: the one story that made the case, and why it worked for this role.
LOST ME: the line or paragraph where attention dropped, quoted, and why.
UNTOUCHED: responsibilities or qualifications in the posting that neither letter nor resume addresses. Quote the posting line. If the source resume or portfolio has work that would have answered one, name it.
SOUNDS LIKE: a person, or a template? Quote the evidence either way.
FIRST TWO QUESTIONS: what you would ask in the interview, which shows what the drafts left you unsure about.
```

### Reviewer 3: Skeptic

```
Role: the panel member who checks claims. The rule is proof over adjectives: every claim is a real project with a real number, and no number appears that the sources do not back. Your job is to find where the drafts break that rule.

Sources, in this order:
1. Source resume: <source resume path>
2. <If a portfolio URL was given:> <portfolio URL> and its case studies. Case studies hold details a resume leaves out. Before you mark any claim NOT IN SOURCES, look at the case study for that project. A claim the site backs is MATCH (site).

Report:
NUMBERS: every number and named result in the letter and resume, one per line: claim (where) -> source says (which source) -> MATCH / MISMATCH / NOT IN SOURCES. A NOT IN SOURCES row names what you checked.
LINKS: every URL in the letter and resume, fetched. url -> HTTP status. Anything but 200 is a finding.
CONTRADICTIONS: places the letter and resume disagree with each other.
WOULD PUSH ON: claims that sound bigger than their proof, and the one piece of evidence that would settle each.
ADJECTIVES WITHOUT PROOF: self-descriptions (strategic, passionate, proven) with no project or number behind them in the same paragraph.
```

## Merged report

`panel-review.md` in the application folder:

```
# Hiring panel: <Company>, <YYYY-MM-DD>

Files reviewed: <letter file>, <resume file or "source resume stood in">
<one line if a working copy is newer than the file reviewed>

Screener: <DECISION line>
Hiring manager: <DECISION line>
Skeptic: <one line: how many numbers checked, how many mismatched or unbacked, how many links dead>

## Fix now
Items with a clear fix from the source resume: number corrections, dead links, missing words the resume can truthfully carry, a posting requirement the source covers but the drafts skipped.

## Ask me
Gaps that need a fact the source lacks, and judgment calls on voice or emphasis. One question each. All of them.

## Noted, no action
Everything else the panel raised, kept so it is not re-raised next round.

## Reviewer reports
<the three reports, verbatim>
```

A number marked MISMATCH or NOT IN SOURCES, or a dead link, is always Fix now or Ask me, never Noted: a wrong number or a dead link in front of a hiring manager is the failure that costs the job.
