# UCalgary research outreach

The script prepares short, professor-specific research inquiry emails from a static profile page and your resume. It attaches the same resume and transcript PDFs to every sent message. It skips professors in `do_not_contact.csv`, anyone already marked `sent` in `sent_log.csv`, and profiles the AI judges to lack research information or a credible fit.

## Set up

1. Install Python 3.10 or newer.
2. Install dependencies: `python -m pip install -r requirements.txt`
3. Edit your existing `.env` file. Set `AI_PROVIDER=gemini`, add your Gemini key as `GEMINI_API_KEY` (or `AI_API_KEY`), set `AI_MODEL` to an available Gemini model, and configure the Microsoft Graph values described below. Keep `.env` private; do not paste the key into chat or commit it.
4. Save the two PDFs in this folder as `resume.pdf` and `transcript.pdf`, or change `RESUME_PDF` and `TRANSCRIPT_PDF` in `.env`.
5. Review the starter list in `professors.csv` and add/remove rows as you like. It includes `name,email,profile_url` plus fit, funding, and contact notes; the notes help the generator avoid assuming that a lab has current salary funding.
6. Add any addresses to suppress to `do_not_contact.csv` under its `email` column. Matching ignores capitalization and surrounding whitespace. Exclusions are checked before the script fetches the profile or calls the AI API.

`GRADUATION_DATE` is set to December 2027 based on your clarification. `TARGET_TERM` defaults to Summer 2027 and can be changed.

## Summer 2027 funding timing

UCalgary has renamed PURE to UREX. Summer 2027 applications open November 23, 2026; the student deadline is January 25, 2027 and the supervisor deadline is January 29. With a December 2027 graduation target, you should still be an undergraduate at the application deadline, but confirm that you will meet UREX's continuing-undergraduate requirement for Fall 2027. The university's current NSERC USRA rules also require registration in a bachelor's degree when applying and that degree requirements remain incomplete at the application deadline; citizenship/permanent-resident criteria also apply to the general USRA. A paid position directly supported by a professor's grant is a separate possibility and should be asked about directly.

Official details: [UREX Summer dates and awards](https://signaturelearning.ucalgary.ca/undergraduate-research/urex-summer), [UREX Award eligibility](https://signaturelearning.ucalgary.ca/undergraduate-research/urex-funding/urex-award), and [UCalgary USRA eligibility](https://signaturelearning.ucalgary.ca/undergraduate-research/urex-summer/usra).

## Run

Dry-run is the default. Start with the first five rows:

```powershell
python .\send_outreach.py --dry-run --limit 5
```

The generated messages are printed and written to `outreach_drafts.csv` (replaced on each run). Nothing is sent in dry-run mode. Once configuration is ready, send eligible rows with:

```powershell
python .\send_outreach.py --send --limit 5
```

The script pauses a random 45–150 seconds between successful sends and stops after 40 successful sends per UTC day. `--limit` caps the number of professor rows considered. Successful deliveries are logged in `sent_log.csv`, and reruns skip those addresses. Errors are recorded and do not stop later rows.

## Microsoft Graph email

The sender uses Microsoft Graph with your UCalgary Microsoft 365 account; it does not use SMTP or an email password. In Microsoft Entra, register a public client app for your UCalgary organization, enable public client/device-code flow, and add the Microsoft Graph **delegated** `Mail.Send` permission. Put the app's Application (client) ID in `MS_CLIENT_ID` and its Directory (tenant) ID in `MS_TENANT_ID` in `.env`. Do not create a client secret. If app registration or consent is blocked by university policy, ask UCalgary IT Support at `it@ucalgary.ca` whether they can approve or provide a client app for delegated Microsoft Graph `Mail.Send`.

When `--send` first runs, the script displays a device sign-in code. Open the displayed Microsoft sign-in page and sign in as `nada.mohamed1@ucalgary.ca`. Graph sends from that signed-in account and saves a copy in Sent Items. Microsoft documents `Mail.Send` as the least-privileged delegated permission for `/me/sendMail`, including messages with file attachments: [sendMail API](https://learn.microsoft.com/en-us/graph/api/user-sendmail?view=graph-rest-1.0). Device-code authentication uses a public client app: [Microsoft Entra public client guidance](https://learn.microsoft.com/en-us/entra/identity-platform/msal-client-applications).

## Gemini and privacy

Set `AI_PROVIDER=gemini`, `GEMINI_API_KEY` (or `AI_API_KEY`), and `AI_MODEL` in `.env`. The script calls Gemini's `generateContent` REST API using the key in an HTTP header. Resume PDF text is sent to Gemini to tailor messages; the transcript is attached to email but is not sent to the AI provider. Google documents the API key header and `generateContent` endpoint in its [Gemini API reference](https://ai.google.dev/api/generate-content).

Profile pages that depend on JavaScript may have too little readable text and will be logged as errors/skipped for browser rendering. The script does not currently use a browser to scrape those pages.

## Files

- `professors.csv`: professor list
- `do_not_contact.csv`: address suppression list
- `sent_log.csv`: sent status and per-row skips/errors
- `outreach_drafts.csv`: generated draft text (ignored by Git)
- `.env`: local secrets and personal configuration (ignored by Git)
