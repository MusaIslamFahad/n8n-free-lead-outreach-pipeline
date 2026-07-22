<div align="center">

![sentinelmcp Banner](https://raw.githubusercontent.com/MusaIslamFahad/n8n-free-lead-outreach-pipeline/main/assets/banner.png)

![License](https://img.shields.io/badge/license-MIT-green.svg)
![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?logo=n8n&logoColor=white)
![Hunter.io](https://img.shields.io/badge/enrichment-Hunter.io-FF7A59)
![Google Sheets](https://img.shields.io/badge/database-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blueviolet.svg)
![Stars](https://img.shields.io/github/stars/MusaIslamFahad/n8n-lead-outreach-pipeline?style=social)

An end-to-end, self-hosted lead generation and cold outreach pipeline built as a single [n8n](https://n8n.io) workflow. It finds contacts at target companies, emails them automatically, watches your inbox for replies, and sends you a daily digest - all backed by a Google Sheet as the "database."

</div>

---

## What it does

The workflow is organized into four independent branches that share one Google Sheet:

| Branch | Trigger | What happens |
|---|---|---|
| 🔵 **A — Lead Generation** | Daily @ 7:00 AM (or manual) | Picks an unprocessed domain from `TargetDomains`, looks up contacts at that domain via [Hunter.io](https://hunter.io), and writes them into `Leads`. Marks the domain as processed. |
| 🟢 **B — Send Outreach** | Daily @ 9:00 AM | Finds every lead with `Status = New`, sends each one a personalized email, throttles 30s between sends, then marks them `Sent`. |
| 🟠 **C — Reply Detection** | Live (IMAP inbox watch) | When a new email lands in your inbox, checks if the sender matches a known lead. If so, logs it to `Replies` and updates that lead's status to `Replied`. |
| 🟣 **D — Morning Digest** | Daily @ 8:00 AM | Emails you an HTML summary of every reply received since the last digest, then marks those replies as notified. |

```
TargetDomains ──▶ [Hunter.io lookup] ──▶ Leads ──▶ [Send email] ──▶ Sent
                                            ▲
                                            │
Inbox (IMAP) ──▶ [Match sender] ──▶ Replies ┘──▶ [Daily digest email]
```

---

## Requirements

- A running [n8n](https://n8n.io) instance (cloud or self-hosted, v1.x+)
- A Google account with a Google Sheet (see [Sheet setup](#1-create-the-google-sheet))
- A [Hunter.io](https://hunter.io) account and API key (free tier works for testing)
- An email account for sending (SMTP) and for watching replies (IMAP) — these can be the same inbox

---

## Setup

### 1. Create the Google Sheet

Create a new Google Sheet with three tabs, using the exact column headers below. Ready-to-import CSVs for each tab are provided in [`sheet-templates/`](./sheet-templates).

**`TargetDomains`**

| Domain | Processed |
|---|---|
| example.com | No |

**`Leads`**

| Email | FirstName | LastName | Title | Company | Status | DateAdded | DateSent |
|---|---|---|---|---|---|---|---|

**`Replies`**

| Email | Name | Subject | ReceivedDate | Notified |
|---|---|---|---|---|

Copy the Sheet's ID from its URL — you'll need it in step 4:
`https://docs.google.com/spreadsheets/d/`**`THIS_PART_IS_THE_ID`**`/edit`

### 2. Import the workflow

In n8n: **Workflows → Add workflow → Import from File**, and select [`workflows/lead-gen-outreach-reply-digest.json`](./workflows/lead-gen-outreach-reply-digest.json).

### 3. Add credentials

In n8n's **Credentials** panel, add:

- **Google Sheets — OAuth2** (used by all the Google Sheets nodes)
- **SMTP** (used by the "Send Outreach Email" and "Send Morning Digest" nodes)
- **IMAP** (used by "Watch Inbox For Replies")
- **Query Auth** for Hunter.io - name the parameter `api_key` and paste your Hunter.io API key as the value (used by "Hunter - Find Emails At Domain")

### 4. Wire up the sheet references

Open each Google Sheets node in the workflow and, in the **Document** field, select your spreadsheet (or paste in its ID from step 1). Confirm the **Sheet** dropdown points to the right tab (`TargetDomains`, `Leads`, or `Replies`).

### 5. Set your email addresses

Two nodes have placeholder addresses to replace:

- `Send Outreach Email` → **From** field: set to your sending address
- `Send Morning Digest` → **From** and **To** fields: set From to your sending address, and To to the inbox where you want to receive the daily digest

### 6. Customize the outreach email

Open the `Build Email Content` node and edit the `Subject` and `Body` fields to your own pitch. The included copy is just a starting example — swap in your own offer, company name, and voice.

### 7. Test each branch

Every trigger node can be run individually from the n8n canvas before going live:

1. Run `Manual Trigger - Test Lead Gen` → confirm rows appear in `Leads`
2. Run `Schedule - Daily Send 9AM` manually → confirm a test email sends and the lead's `Status` flips to `Sent`
3. Send yourself a test email from a `Leads` address → confirm it's caught and logged to `Replies`
4. Run `Schedule - Morning Digest 8AM` manually → confirm the digest email arrives

### 8. Activate

Once everything checks out, toggle **Active** in the top-right of the n8n editor.

## Notes & customization ideas

- **Rate limiting**: the `Throttle Before Next Send` node waits 30 seconds between outreach emails to avoid tripping spam filters - adjust as needed for your sending domain's reputation and volume.
- **Hunter.io limits**: the free tier caps monthly domain searches; the `limit` query parameter on the Hunter request is set to `10` contacts per domain.
- **Matching replies to leads**: reply detection matches purely on sender email address against the `Leads` sheet, so make sure leads reply from the same address you emailed.
- **Swap the data store**: the workflow uses Google Sheets for simplicity, but the same shape (append/update/read by column) maps cleanly onto Airtable, Postgres, or Notion if you'd rather not use Sheets.

---

## 🤝 Contributing

Contributions are welcome! If you'd like to improve the project:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

**Ideas for contributions:** a Postgres/Airtable backend as an alternative to Google Sheets, support for other enrichment providers (Apollo, Clearbit) alongside Hunter.io, a follow-up sequence (2nd/3rd touch) branch, unsubscribe/bounce handling, or a Dockerized n8n setup for one-command local testing.

---

## 👤 Author

**Md. Musa Islam Fahad**  
CSE (Data Science) · Daffodil International University, Dhaka  
📧 musa.islam.fahad@gmail.com  
🌐 [Portfolio](https://musaislamfahad.vercel.app) · [GitHub](https://github.com/MusaIslamFahad) · [LinkedIn](https://linkedin.com/in/md-musa-islam-fahad-b18759249)

---

## 📄 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) for details.  
Free to use, modify, and deploy.

---

## 🙏 Acknowledgements

- [n8n](https://n8n.io) - The workflow automation engine this project is built on
- [Hunter.io](https://hunter.io) - Email discovery API used for lead enrichment
- [Google Sheets API](https://developers.google.com/sheets/api) - Lightweight data store for leads and replies

---

<div align="center">

Built as a demonstration of automated, no-code lead generation and outreach.

**[⬆ Back to Top](#lead-gen--outreach--reply-digest-n8n-workflow)**

</div>
