# PM Agent Build Log — From Chatbot to Agent

## Date: April–May 2026

## My Setup
- **OS:** Windows 11 + PowerShell
- **Python:** Microsoft Store Python 3.13
- **GPU:** AMD Radeon RX 6500M (4GB VRAM)
- **RAM:** 10.7GB total
- **LLM:** Ollama Docker with qwen2.5:1.5b (986MB)
- **Email:** Gmail with App Password

---

## What I Wanted

A **personal agent** that doesn't just wait for prompts like a chatbot, but actually:
- Remembers my todos across sessions
- Auto-processes meeting transcripts
- Creates JIRA tickets from PRDs
- Fetches Fathom meetings from Gmail and suggests action items

The key insight: **"If I have to prompt it every time, where's the agent?"**

---

## Phase 1 — Realizing the Problem

### Mistake: Running a Massive Model
I was using something like `llama3:70b` or `qwen2.5:32b` in Docker on my laptop GPU.
- **Result:** Painfully slow, barely functional
- **Why:** 4GB VRAM can't handle 32B+ parameters

### Fix: Right-Size the Model
```bash
ollama pull qwen2.5:1.5b  # 986MB, fits comfortably
```

**Lesson:** For PM tasks (summaries, extraction, classification), 1.5B is plenty. Don't let people shame you into running 70B models.

---

## Phase 2 — Understanding What an Agent Actually Is

### The "Aha" Moment
> *"Anyway, if I need to prompt every time I need the system to do something, then where's the agent at?"*

An LLM waiting for prompts = **chatbot**  
An agent needs:
1. **Triggers** — starts without you (file drops, scheduled tasks, email polling)
2. **Memory** — remembers context across sessions (SQLite database)
3. **Tool Use + Loop** — does things, not just talks (creates todos, saves files, scrapes web)

### The Architecture I Built
```
Trigger (file drop / cron / chat command / email)
    |
Controller (agent.py — routes to right skill)
    |
Skill (todo_manager.py, fathom_gmail.py, etc.)
    |
LLM (Ollama qwen2.5:1.5b — reasoning brain)
    |
Memory (SQLite — remembers everything)
    |
Action (create todo, save summary, generate JIRA CSV)
```

---

## Phase 3 — Building the Skills

### Skill 1: Todo Manager
- Chat interface: `add`, `list`, `done`
- AI auto-detects priority ("urgent" -> high)
- SQLite persistence across sessions

### Skill 2: Transcript Processor
- File watcher via `watchdog` — drop `.txt`/`.vtt` in folder -> auto-process
- Summarizes into: SUMMARY / KEY POINTS / ACTION ITEMS
- Saves to `~/PM_Agent/Fathom_Summaries/`

### Skill 3: JIRA Ticket Creator
- Feed PRD -> outputs structured Epic/Stories/Tasks
- Generates both Markdown and **CSV for bulk JIRA import**

### Skill 4: Fathom Gmail Fetcher (The Complex One)
- Polls Gmail IMAP for Fathom meeting emails
- Scrapes transcript via Playwright (headless browser)
- Summarizes with Ollama
- **Asks before creating todos** — the agentic part

---

## Phase 4 — Windows-Specific Headaches

### Problem 1: `touch` Not Found
```powershell
touch file.txt
# ERROR: The term 'touch' is not recognized
```
**Fix:** Use `New-Item` or Python's `pathlib.Path.touch()`

### Problem 2: Desktop Path Wrong
```powershell
cd Desktop
# ERROR: Cannot find path
```
**Fix:** OneDrive redirects Desktop to `C:\Users\jnura\OneDrive\Desktop`

### Problem 3: Python Not on PATH
Microsoft Store Python installs to weird location.
**Fix:** Use `python -m` syntax:
```powershell
python -m playwright install chromium
```

### Problem 4: `requests` Module Missing
```python
ModuleNotFoundError: No module named 'requests'
```
**Fix:** `pip install requests` (not included in stdlib)

### Problem 5: Playwright PATH Issues
```powershell
playwright install chromium
# ERROR: playwright not recognized
```
**Fix:** `python -m playwright install chromium`

---

## Phase 5 — Security & Best Practices

### The Credential Problem
I pasted my Gmail App Password in chat. **Don't do this.**

**Fix:** Use `.env` files + `.gitignore`
```bash
# .env (NEVER commit this)
GMAIL_USER=your.email@gmail.com
GMAIL_APP_PASSWORD=jzqxzmwdshakxfwr

# .gitignore
.env
pm_agent.db
```

### What Stays Local
- SQLite database (`pm_agent.db`)
- All transcripts and summaries
- Todo list
- Ollama inference (no cloud API)

**Everything stays on your laptop. Nothing goes to third-party AI APIs.**

---

## Key Commands Learned

```powershell
# Ollama
docker ps                           # check container running
ollama ps                          # check GPU usage
ollama list                        # list downloaded models
ollama pull qwen2.5:1.5b           # download model

# Python
pip install requests watchdog playwright python-dotenv
python -m playwright install chromium

# Windows folders
mkdir foldername                   # create folder
New-Item file.txt -ItemType File   # create file (replaces touch)
cd $env:USERPROFILE\OneDrive\Desktop  # go to actual Desktop

# Agent
python pm_agent.py                 # start agent
python pm_agent.py --remind        # one-shot reminder
```

---

## What Finally Worked

- **Local LLM** — qwen2.5:1.5b on AMD GPU via Vulkan
- **Agent loop** — triggers -> skills -> memory -> action
- **File watcher** — drop transcripts, get auto-summaries
- **Gmail integration** — fetches Fathom meetings, asks before creating todos
- **No cloud costs** — everything runs on laptop
- **Modular skills** — easy to add new capabilities

---

## Lessons Learned

1. **Agent != LLM** — The LLM is just the brain. Python glue code (triggers, memory, tools) makes it an agent.

2. **Model size is overrated** — 1.5B parameters handles PM tasks perfectly. 70B is overkill for summaries and todo extraction.

3. **Start simple, expand** — Built one skill at a time. Todo manager first, then transcripts, then JIRA, then Gmail.

4. **Local-first is viable** — For personal automation, you don't need VPS or paid APIs. Laptop + Ollama is enough.

5. **Ask before acting** — The "agentic" part isn't just doing things automatically. It's asking "Should I create these todos? [y/n/edit/skip]" before mutating your state.

6. **Windows is annoying** — PowerShell paths, OneDrive redirects, Microsoft Store Python quirks. But it works.

---

## What's Next

- [ ] **Slack/Discord notifications** — agent pings you instead of you checking it
- [ ] **Google Calendar API** — auto-prep for meetings
- [ ] **Confluence API** — auto-fetch PRDs instead of manual file
- [ ] **GitHub PR summaries** — review assistant
- [ ] **Voice trigger** — "Hey Agent, what are my todos today?"
- [ ] **Deploy to GitHub** — share the code, let others fork it

---

## The Core Insight

> *"I think even using GPU that is already sufficient"*

You don't need expensive hardware or cloud services for a personal agent. A 4GB GPU laptop, a 1.5B model, and ~200 lines of Python per skill is enough. The magic isn't in the model — it's in the **loop** that connects triggers, memory, and action.
