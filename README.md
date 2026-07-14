# AI Resume & Job Application Assistant

An n8n-powered automation that watches your inbox for job opportunities, scores them against your resume using an LLM, and — when the match is strong — generates a tailored resume and cover letter, saves everything to Drive, and logs the application to a tracking sheet.

Built to remove the repetitive part of job hunting (reading, comparing, rewriting) while keeping a human decision point before anything gets applied to.

---

## What it does

1. Watches Gmail for new job-related emails
2. Filters out noise (newsletters, irrelevant alerts)
3. Uses an LLM to extract structured job requirements (title, skills, experience level, location, salary)
4. Pulls your master resume from Google Drive and parses the PDF to text
5. Scores the match between resume and job (0–100) with an LLM, including reasoning and skill gaps
6. Branches on match score — only strong matches proceed
7. Rewrites the resume to emphasize relevant, truthful experience for that specific role
8. Generates a tailored cover letter
9. Saves both files to a per-application Drive folder
10. Logs the application (company, role, score, status, file links) to Google Sheets
11. Sends a notification email summarizing the result

---

## Architecture

```
Gmail Trigger
     │
     ▼
IF (noise filter)
     │
     ▼
LLM: Extract Job Requirements ──► Code: Parse JSON
     │
     ▼
Drive: Download Resume ──► Extract from File (PDF → text)
     │
     ▼
LLM: Compare & Score ──► Code: Parse JSON
     │
     ▼
IF (match_score > threshold)
     │
   ┌─┴─────────────┐
   ▼                ▼
LLM: Rewrite     Set: status = "skipped"
Resume               │
   │                  │
   ▼                  │
LLM: Generate         │
Cover Letter          │
   │                  │
   ▼                  │
Drive: Upload Files    │
   │                  │
   └────────┬─────────┘
            ▼
   Sheets: Append Row (log)
            │
            ▼
   Gmail: Send Notification
```

---

## Tech stack

| Layer | Tool |
|---|---|
| Orchestration | [n8n](https://n8n.io) |
| LLM | Claude (Anthropic API) |
| Email | Gmail API (OAuth2) |
| File storage | Google Drive API |
| Tracking | Google Sheets API |
| PDF parsing | n8n's `Extract from File` node |

---

## Repository structure

```
.
├── README.md                          # this file
├── SETUP.md                           # full step-by-step setup guide
├── workflow/
│   └── ai-resume-assistant.json       # exported n8n workflow — import directly
├── prompts/
│   ├── extract-job-requirements.txt
│   ├── compare-and-score.txt
│   ├── rewrite-resume.txt
│   └── generate-cover-letter.txt
└── docs/
    └── screenshots/                   # workflow canvas + example outputs
```

---

## Quick start

1. Import `workflow/ai-resume-assistant.json` into your n8n instance
2. Follow [SETUP.md](./SETUP.md) to connect Gmail, Google Drive, Google Sheets, and your LLM API key
3. Point the "Download Resume" node at your own resume file in Drive
4. Create the tracking Google Sheet using the column layout in SETUP.md
5. Activate the workflow

Full walkthrough, including credential setup and node-by-node configuration, is in [SETUP.md](./SETUP.md).

---

## Design decisions

- **Score threshold before rewriting**: the LLM rewrite/cover-letter steps only run above a configurable match score (default 70), to avoid burning API calls and generating applications for poor-fit roles.
- **No fabrication**: prompts explicitly instruct the model not to invent experience — only to reorder/emphasize what's already true in the base resume.
- **Human-in-the-loop notification**: the workflow prepares applications but notifies rather than auto-submits, so there's always a review step before anything goes out.

---

## Possible extensions

- Swap the Gmail trigger for a Webhook so job URLs (e.g. LinkedIn postings) can be submitted manually and run through the same pipeline
- Add deduplication against the tracking sheet to avoid reprocessing the same company/role
- Generate PDF output instead of plain text for the tailored resume/cover letter
- Add an error-handling sub-workflow so failed LLM parses get logged instead of silently dropped

---

## License

MIT — feel free to adapt for your own job search.
