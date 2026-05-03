Tweaking Infrastructure 

### What I was trying to do
Set up OpenClaw with Ollama as a local LLM provider, connect it to 
Telegram, and build the first agent (Researcher) with a working 
Google Trends scraping skill.

### What I learned

**AI Agents vs Automation**
The biggest mental model shift this session. Automation (n8n, Zapier) 
handles fixed trigger-based flows. Agents are different — they have a 
goal, a set of skills, and reason about which skill to use when called. 
You don't build agents to replace automation, you build them to handle 
the thinking parts inside an automation.

**Skills = Tools the agent can call**
Each agent has a defined set of skills registered to it. The LLM reads 
the skill descriptions and decides when and which one to invoke. The 
description quality is everything — vague descriptions = confused agent.

**Multi-agent architecture**
Each agent is isolated with its own workspace, system prompt, and skills. 
The orchestrator (OpenClaw) routes between them. Agent 1 output becomes 
Agent 2 input, and so on down the chain.

**Docker networking on Windows is tricky**
Inside Docker, localhost refers to the container itself — not your 
Windows machine. To reach Ollama running on the host, you must use 
http://host.docker.internal:11434 instead.

**Never manually write config files without knowing the schema**
We crashed OpenClaw multiple times by writing openclaw.json with the 
wrong structure. Lesson: always read the docs or use the CLI config 
commands instead of writing JSON by hand.

**Small models are quirky but workable**
Qwen 2.5 1.5B works but it's unpredictable without tight skill 
instructions. It answered a weather question then randomly asked 
"what is your favorite color?". The smaller the model, the more 
precise your skill descriptions need to be.

**Context window matters**
OpenClaw requires a minimum 16,000 token context window. Qwen 2.5 1.5B 
defaults to 8,192 which caused it to be rejected entirely. We bumped it 
to 32,768 to resolve this.

**pip3 is not included in the OpenClaw container**
Had to install it manually as root via apt-get before we could install 
any Python dependencies.

### What we built
- OpenClaw running in Docker ✅
- Ollama + Qwen 2.5 1.5B connected ✅  
- Telegram bot connected and authorized ✅
- Researcher agent created ✅
- Google Trends skill working and returning real data ✅

### What's next
- Reddit scraping skill
- RSS news scraping skill
- Agent 2: Summarizer
- Agent 3: Content Planner
- Agent 4: Idea Builder

### Honest reflection
Most of this session was fighting infrastructure, not building agents. 
That's normal when setting up a new stack for the first time. The 
important thing is the foundation is solid now — the next session 
should be much faster and focused purely on building.
But I figured that I might better be off using VPS as it might as well be much cheaper than creating your own server back at home.
