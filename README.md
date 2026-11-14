# UCalgary research outreach

The script prepares short, professor-specific research inquiry emails from a static profile page and your resume. It attaches the same resume and transcript PDFs to every sent message. It skips professors in `do_not_contact.csv`, anyone already marked `sent` in `sent_log.csv`, and profiles the AI judges to lack research information or a credible fit.

## Set up

1. Install Python 3.10 or newer.
2. Install dependencies: `python -m pip install -r requirements.txt`
3. Copy `.env.example` to `.env`, then set an AI provider, API key, model, graduation date, and mail backend credentials.
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

## Mail backends

- `EMAIL_BACKEND=smtp`: set `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURITY`, `SMTP_USER`, and `SMTP_PASS`. For Microsoft 365, SMTP AUTH may need to be enabled for the account. Use an app password only if your provider supports it.
- `EMAIL_BACKEND=graph`: set `MS_CLIENT_ID` for a Microsoft Entra app configured for public client/device-code sign-in and grant delegated `Mail.Send`. On the first send, the script prints a sign-in code and URL. It sends from the signed-in account.

Set `AI_PROVIDER=anthropic` with an Anthropic Console key, or `AI_PROVIDER=openai` with an OpenAI API key. Set `AI_MODEL` to a model currently available to that key. Resume PDF text is sent to the selected AI provider to tailor messages. The transcript is attached to email but is not sent to the AI provider.

Profile pages that depend on JavaScript may have too little readable text and will be logged as errors/skipped for browser rendering. The script does not currently use a browser to scrape those pages.

## Files

- `professors.csv`: professor list
- `do_not_contact.csv`: address suppression list
- `sent_log.csv`: sent status and per-row skips/errors
- `outreach_drafts.csv`: generated draft text (ignored by Git)
- `.env`: local secrets and personal configuration (ignored by Git)
