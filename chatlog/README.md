# 📚 Documentation Index — chatlog

This folder is the reference layer for **capturing, versioning, and reviewing ChatGPT conversation transcripts**. Use it to understand *what the workflow delivers, why it exists, and how it runs in production*.

---

## 📁 Contents (at a glance)

| File | Purpose |
|---|---|
| `README.md` | This file — overview, navigation, change rules. |
| `chatlogs/<thread>/<timestamp>.md` | Individual conversation transcripts in Markdown, committed per thread/run. |

> If you add or rename a doc, update this table in the same PR.

---

## 🧭 How to navigate

- **Start here:** Each run generates a Markdown transcript stored under `chatlogs/<thread_id>/<timestamp>.md`.  
- **Review changes:** Every transcript is committed on a **feature branch** and raised as a **Pull Request (PR)** for review.  
- **Track threads:** `thread_id` maps back to the originating conversation; timestamps ensure unique file paths.

---

## 🔄 Workflow design (CHATLOG_TO_GITHUB)

1. **Inbound**  
   - Triggered by **Webhook** (`POST`) or **Cron** (daily run).  
   - Payload may include `thread_id` and/or new message text.  

2. **Thread handling**  
   - Create thread if missing.  
   - Always standardise `thread_id` for downstream nodes.  

3. **Conversation update**  
   - Add new message(s) to thread.  
   - Run Assistant to process.  

4. **Transcript build**  
   - Fetch all messages from the thread (`List Messages`).  
   - Convert JSON to Markdown transcript.  

5. **File metadata**  
   - Generate file path (`chatlogs/<thread>/<timestamp>.md`).  
   - Build commit message string.  

6. **Branch + PR**  
   - Create branch from `main`.  
   - Commit transcript file to new branch.  
   - Open PR into `main` with context (thread + path).  

7. **Outbound**  
   - Webhook responds with JSON containing:  
     - `ok`  
     - `thread_id`  
     - `path`  
     - `html_url` (file in repo)  
     - `raw_url` (raw Markdown)  
     - `pr_url` (link to PR)

---

## 🚀 Running in production

- **Ad-hoc run**:  
  Trigger the **Webhook URL** with a `POST` request:  
  ```bash
  curl -X POST "https://<n8n-host>/webhook/chatlog" \
    -H "Content-Type: application/json" \
    -d '{"thread_id":"thr_abc123","message":"Test message"}'
  ```

- **Scheduled run**:  
  Add a **Cron node** at the start:  
  - Mode: `Every Day`  
  - Time: e.g. `08:00 AEST`  
  - Connect Cron → main flow.  
  This ensures one transcript capture daily.

---

## 🔄 Change management

- **One PR = one transcript intent.**  
- Every commit lands on a **branch**, then a **PR** is raised.  
- **Supersession:** If a transcript supersedes a prior one, note it in the PR description.  
- **Timestamps:** Use ISO format (`YYYY-MM-DD-HH-MM-SS`) to guarantee uniqueness.  

---

## ✅ Authoring rules (short)

- All transcripts are stored as **Markdown** (`.md`).  
- **Roles** (`USER`, `ASSISTANT`) are clearly separated with headings.  
- Keep **timestamps explicit** in filenames.  
- No manual edits to transcripts — any modifications must go through the workflow.  

---

## 🔗 Quick links

- Production repo: [ops-hub](https://github.com/BenElphick79/ops-hub)  
- Example PR URL is included in each webhook response (`pr_url`).  

---

_Last updated: 2025-09-03 (AEST)._
