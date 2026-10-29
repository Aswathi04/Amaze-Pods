# Free-Trial Lead Handler

**Google Forms + Google Sheets + Apps Script + AI**

Turns a free-trial enquiry into a personalised draft reply in seconds. A human always reviews the draft before anything is sent to a parent.

Built as a small automation concept for Amaze Pods, using only tools a non-developer team can manage: Google Workspace and an AI API.

## The problem

Parents who ask for a free trial expect a quick, personal reply. Writing each one by hand is slow and repetitive, and leads can get lost when the team is busy.

## How it works

1. A parent submits the Google Form.
2. The response lands in a Google Sheet.
3. Apps Script sends the details to an AI model, using a prompt written in the brand's tone and limited to facts about the service.
4. The suggested reply is saved in the `AI Draft` column and as a **Gmail draft** addressed to the parent.
5. The team gets an alert email with the enquiry details and the suggested reply.
6. A team member reviews and edits the draft in Gmail, then sends it.
7. Once the draft is sent, `Status` changes from **New** to **Contacted** automatically. The team sets **Trial booked** manually.

## Setup (about 15 minutes)

1. Create a Google Form with these questions: `Parent name`, `Parent email`, `Child's grade`, `Curriculum`, `Subjects`, `Preferred timings`, `Anything else we should know?`
2. In the Form, open Responses and link it to a new Google Sheet.
3. In the Sheet, open Extensions > Apps Script and paste in `Code.gs`.
4. Get an API key from Google AI Studio. In Apps Script, go to Project Settings > Script Properties and add `GEMINI_API_KEY`. Never put the key in the code or commit it to GitHub.
5. In `CONFIG`, set `NOTIFY_EMAIL` to the team inbox that should get new-lead alerts.
6. Run `setup()` once and approve the permissions. This adds the columns, status dropdown, colours and form trigger.
7. Run `setupStatusSync()` once. This checks every 15 minutes whether drafts have been sent.
8. Submit a test response. The draft appears in the sheet and in Gmail Drafts, and the alert email arrives.

## Customising

All settings are at the top of `Code.gs`:

- **Tone and facts:** edit `SYSTEM_PROMPT`.
- **Form question names:** edit `CONFIG.FIELDS`. Matching ignores case, spaces and punctuation.
- **Model:** change `CONFIG.MODEL`. Model names get retired, so this is the first thing to check if the API returns a 404.
- **Fallback model:** set `CONFIG.FALLBACK_MODEL` to a second model to try if the first keeps failing.
- **Statuses:** edit `CONFIG.STATUSES`.
- **Alerts:** set `NOTIFY_EMAIL` to an empty string to turn them off.

## Testing

- `testLastRow()` re-runs the handler on the latest response without resubmitting the form.
- To test the status sync, send a draft to a second address you own, then run `syncStatuses()`. The row should change to Contacted.
- Use fake data only. Do not test with real parents' details.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `ERROR: API 404` in `AI Draft` | The model name was retired. Update `CONFIG.MODEL`. |
| `ERROR: API 503` | Temporary overload. The script retries automatically, so run `testLastRow()` again after a few minutes. |
| `ERROR: Missing GEMINI_API_KEY` | Add the key in Script Properties. |
| Parent name is blank or a placeholder | Form question titles don't match `CONFIG.FIELDS`. |
| No draft or alert email | Run `testLastRow()` once and approve the Gmail permission prompt. Also check the Triggers page. |
| Status stays New | Only rows created after the Draft ID column was added can sync. Submit a fresh test response. |

## Design choices

- **Human in the loop.** The AI writes the first version, a person checks it and presses Send. Nothing is sent to a parent automatically.
- **No invented facts.** The prompt only allows details from the public website and forbids made-up prices, mentor names, timings or guarantees.
- **Reliable by default.** Temporary API failures are retried with growing waits, and a fallback model can be configured.
- **Modular.** The prompt, field names, model and statuses all live in one config block, so someone who isn't a developer can change them.
- **Provider-independent.** Only `callModel_` talks to the AI provider, so switching to another API means changing one function.

## Limitations

- Drafts appear in the Gmail account that ran `setup()`. For a real team, use a shared inbox.
- `Trial booked` can't be detected from email, so a person sets it.
- If a draft is deleted and the parent is emailed separately, the status still becomes Contacted, which is acceptable since they were contacted.

## Possible next steps

- Reminder alert for leads still marked New after 2 days.
- Instant auto-acknowledgement ("we got your request") for low-risk first contact, keeping the personalised reply under review.
- Move to a shared inbox and a paid API tier once volume grows.
- Track how often drafts are edited before sending, to decide when more automation is safe.
