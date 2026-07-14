# Setup Guide

Full step-by-step instructions to get this workflow running on your own n8n instance.

---

## Prerequisites

- An n8n instance — [n8n.cloud](https://n8n.cloud) or self-hosted (Docker)
- A Gmail account that receives job-related emails
- Google Drive access (for resume storage + generated files)
- Google Sheets access (for the application tracking log)
- An Anthropic API key ([console.anthropic.com](https://console.anthropic.com)) or OpenAI API key

---

## 1. Import the workflow

1. In n8n, go to **Workflows → Import from File**
2. Select `workflow/ai-resume-assistant.json`
3. The full node graph loads onto the canvas — nothing runs yet, since credentials aren't attached

---

## 2. Set up credentials

Go to **Credentials → New** in n8n and add each of these once (they're reused across nodes):

| Credential | Type | Notes |
|---|---|---|
| Gmail | OAuth2 | Sign in with the Google account that receives job emails |
| Google Drive | OAuth2 | Same Google account, or a separate one — your call |
| Google Sheets | OAuth2 | Same Google account |
| Anthropic API | API Key | Paste your key from the Anthropic console |

After adding these, open each node in the imported workflow and select the matching credential from its dropdown — imported workflows don't carry credentials over for security reasons, so this step is required even after import.

---

## 3. Prepare your Google Drive

1. Create a folder for your job search, e.g. `Job Search/`
2. Upload your master resume as a PDF inside it, e.g. `Job Search/Master Resume.pdf`
3. Create an `Applications/` subfolder — this is where tailored resumes and cover letters will be saved per application
4. In the **"Drive: Download Resume"** node, set the **File** field to your master resume (browse to it, or paste its file ID from the Drive URL)

---

## 4. Prepare your Google Sheet

Create a new Google Sheet with these column headers in row 1:

| Date | Company | Role | Match Score | Status | Resume Link | Cover Letter Link |
|---|---|---|---|---|---|---|

In the **"Sheets: Append Row"** node, select this sheet and map each column to the corresponding field.

---

## 5. Configure the Gmail trigger

Open the **Gmail Trigger** node:
- **Poll Times**: every 15–30 minutes is a reasonable default
- **Filters**: adjust the search query to match how job emails actually look in your inbox, e.g.:
  ```
  (subject:job OR subject:opportunity OR subject:position OR subject:hiring) newer_than:1d
  ```
- Check a handful of real job emails in your inbox first and adjust keywords accordingly — this varies a lot person to person

---

## 6. Set your match score threshold

Open the **IF (match_score > threshold)** node and adjust the comparison value (default: `70`). Lower it if you want more applications generated for review, raise it if you only want the workflow to act on strong matches.

---

## 7. Review the prompts

All prompts used by the LLM nodes are in `prompts/` for easy editing:
- `extract-job-requirements.txt`
- `compare-and-score.txt`
- `rewrite-resume.txt`
- `generate-cover-letter.txt`

Edit these directly in the corresponding n8n node if you want to change tone, strictness, or output format. Keep the JSON-only instruction in the extraction and scoring prompts — downstream `Code` nodes depend on that structure to parse correctly.

---

## 8. Test before activating

1. Temporarily swap the **Gmail Trigger** for a **Manual Trigger** with one real email's data pinned as test input
2. Run the workflow node-by-node using **"Execute Node"**, checking each output tab as you go
3. Once every step produces the expected output, swap the Gmail Trigger back in
4. Toggle the workflow to **Active** (top right)

---

## Troubleshooting

- **LLM node output doesn't parse as JSON**: open the node's output tab, check the actual response shape — the parsing `Code` nodes reference `content[0].text`, which can shift slightly by node/API version. Adjust the path to match what you see.
- **Gmail Trigger isn't firing**: check the search query isn't too narrow, and confirm OAuth2 credential still has valid access (Google tokens can expire — reconnect if needed).
- **PDF text extraction looks garbled**: this happens with image-based/scanned PDFs. Extract from File works on text-based PDFs — re-export your resume from Docs/Word rather than a scanned copy.
- **Drive upload fails**: confirm the target folder ID is correct and the OAuth2 account has write access to it.
