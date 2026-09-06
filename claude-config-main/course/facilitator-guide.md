# Systems Architecting with AI — 3-Hour Workshop

> Facilitator guide for teaching non-coders how to build real software using AI.
> Designed for groups of 5-30 people. No coding experience required.

---

## Before the Workshop

### What You Need

| Item | Details |
|------|---------|
| **Your laptop** | Claude Code installed, a project idea ready for the live demo |
| **Projector/screen** | So everyone can see your live demo |
| **Wi-Fi** | Strong enough for everyone to install and use Claude Code |
| **Printed cheat sheets** | One per person (see `cheat-sheet.md`) |

### What Attendees Need

- A laptop (Mac, Windows, or Linux)
- A GitHub account (have them create one before the workshop if possible)
- No coding experience required — that's the whole point

### Pre-Workshop Setup (send 24 hours before)

Send attendees this checklist:

```
Before the workshop, please:

1. Create a GitHub account at https://github.com (if you don't have one)
2. Install Node.js from https://nodejs.org (LTS version)
3. Install Claude Code:
   - Open your terminal (Mac: Terminal app, Windows: PowerShell)
   - Run: npm install -g @anthropic-ai/claude-code
4. Get a Claude API key or Claude Max subscription
5. Bring your laptop fully charged

If you get stuck on any step, don't worry — we'll help you at the start of the workshop.
```

---

## The Workshop

---

### HOUR 1: "The World Changed" (60 min)

---

#### Part A: Live Demo (30 min)

**Goal:** Blow their minds. Build a working app in front of them without writing a single line of code.

**Setup:** Open your terminal, projector on, audience watching.

**Script:**

> "Who here has ever wanted to build an app but didn't know how to code?"
>
> (hands go up)
>
> "By the end of today, you will have built one. Not a toy — a real app that runs on the internet. I'm going to show you how."

**The Demo:**

1. Open terminal. Create an empty folder.
2. Start Claude Code.
3. Run `/development build`
4. Walk through the discovery prompts out loud — narrate your thinking:
   - "I want to build a guestbook app where visitors can leave messages"
   - "The core features are: leave a message, view all messages, maybe a like button"
   - "I'll use Next.js for the frontend, SQLite for the database — keeping it simple"
5. Let Claude scaffold the project.
6. Then plan and build the first feature live.
7. Run it. Show the app working in the browser.

**Key moments to narrate:**

- "Notice I didn't write any code. I described what I wanted."
- "When it got something wrong, I just told it what was wrong. Like talking to a colleague."
- "The skill isn't coding. The skill is knowing what to ask for."

**Time check:** 25-30 minutes for the demo. If it takes longer, that's fine — the demo IS the lesson.

---

#### Part B: Get Everyone Set Up (30 min)

**Goal:** Every person has Claude Code running and ready.

**Script:**

> "Now it's your turn. Let's get everyone set up. Open your laptops."

**Walk them through:**

1. Open terminal
2. Verify Node.js: `node --version` (help anyone who doesn't have it)
3. Verify Claude Code: `claude --version` (help anyone who needs to install)
4. Authenticate: `claude` and follow the prompts
5. Verify GitHub CLI: `gh --version` (install with `brew install gh` on Mac or guide for Windows)
6. Authenticate GitHub: `gh auth login`

**Tips:**

- Have 1-2 helpers walking the room if possible
- Pair up anyone who's stuck with someone who's done
- Don't rush this — if setup takes the full 30 min, that's okay
- Have a backup plan: if someone can't install locally, they can use claude.ai/code in the browser

**Checkpoint:** Everyone can type `claude` and see it respond.

---

### HOUR 2: "Your First System" (60 min)

---

**Goal:** Everyone builds the same app together, step by step. You lead, they follow.

**The Project:** A personal bookmark manager — simple enough to build in an hour, interesting enough to feel real.

#### Step 1: Think Before You Prompt (10 min)

**Script:**

> "Before we talk to AI, we need to think. The number one mistake people make with AI is jumping straight in without knowing what they want. So let's think first."

**Exercise:** Have everyone write down (on paper or in a notes app):

```
What am I building?
→ A personal bookmark manager

Who is it for?
→ Me — to save and organize links

What are the 3 core features?
→ 1. Save a bookmark (URL + title + optional tag)
→ 2. View all my bookmarks
→ 3. Search/filter bookmarks

What does the user see first when they open the app?
→ A list of their bookmarks with a button to add a new one
```

> "This is the architecture mindset. Before you tell AI to build, you decide WHAT to build. You are the architect. AI is the builder."

#### Step 2: Start the Build (10 min)

**Everyone together:**

1. Create a new folder: `mkdir my-bookmarks && cd my-bookmarks`
2. Start Claude Code: `claude`
3. Run: `/development build`
4. Walk through the discovery prompts together — tell them what to answer:
   - Idea: "A personal bookmark manager where I can save, tag, and search links"
   - Features: save bookmark, view list, search/filter
   - Stack: Next.js, SQLite, Tailwind CSS (keep it simple — one command to run)

> "Everyone should be seeing the same questions. Just follow along and type what I type."

#### Step 3: Build the First Feature (20 min)

After scaffold is done:

1. Run `/development plan` — plan the "save and view bookmarks" feature
2. Run `/development implement` on the generated plan
3. Watch Claude build it

**Narrate as it happens:**

> "See how it's creating the database schema first? That's because data comes before UI — you need somewhere to store things before you can display them."
>
> "Now it's building the API — the behind-the-scenes part that handles saving and fetching."
>
> "And now the frontend — what the user actually sees."

#### Step 4: Run It (10 min)

1. Start the dev server: `npm run dev`
2. Open `localhost:3000` in the browser
3. Everyone should see their app running

**Celebration moment:**

> "You just built a working web app. You have a database, an API, and a user interface. And you didn't write a single line of code. How does that feel?"

#### Step 5: Fix Something (10 min)

**Intentionally break or improve something** — this teaches the feedback loop:

> "Let's say we want the bookmarks to show newest first. Right now they're in random order. How do we fix this?"

Have them describe the change to Claude:

> "Sort the bookmarks by date added, newest first"

**Key lesson:**

> "When something isn't right, you don't need to know how to fix it in code. You just need to describe what's wrong and what you want instead. That's the skill."

---

### HOUR 3: "Now You're On Your Own" (60 min)

---

#### Part A: Solo Build (40 min)

**Script:**

> "Now it's your turn. You're going to pick your own idea and build it from scratch. Here's the rule: keep it small. 3 features max. You have 40 minutes."

**Help them pick ideas** (have this list on the projector for anyone stuck):

| Idea | Features |
|------|----------|
| **Todo list** | Add tasks, mark complete, filter by status |
| **Personal journal** | Write entries, view by date, search |
| **Recipe box** | Save recipes, tag by cuisine, search |
| **Expense tracker** | Log expenses, categorize, view totals |
| **Workout log** | Log exercises, track sets/reps, view history |
| **Reading list** | Add books, mark as read, rate them |
| **Flashcard app** | Create cards, quiz mode, track progress |
| **Contact book** | Add people, tag them, search |

**Process for each person:**

1. Write down what you're building and 3 features (2 min)
2. Create a new folder
3. Run `/development build`
4. Follow the prompts
5. Build at least one feature with `/development plan` + `/development implement`

**Your role:** Walk the room. Help people who are stuck. The most common issues will be:

- Vague prompts → help them be specific
- Scope creep → remind them: "3 features, that's it"
- Error messages → show them how to paste the error to Claude and ask for a fix
- Impatience → "Trust the process, let it finish"

#### Part B: Show & Tell (20 min)

**Script:**

> "Alright, time's up! Who wants to show what they built? You don't have to be done — showing something half-built is totally fine."

- Ask for 3-4 volunteers
- Each person gets 3-4 minutes:
  1. What did you build? (10 seconds)
  2. Show it running (1 minute)
  3. What was the hardest part? (1 minute)
  4. Quick applause

**After demos:**

> "Look around. An hour ago none of you had these apps. Now they exist. They're real. They run. Some of you have never coded in your life, and you just built software. That's the paradigm shift."

---

## Closing (5 min at the end)

**Key takeaways to reinforce:**

1. **You are the architect.** AI is the builder. The skill is knowing what to build and how to describe it.
2. **Think before you prompt.** The 2 minutes you spend thinking saves 20 minutes of back-and-forth.
3. **Be specific.** "Make it better" gets bad results. "Sort by date, newest first" gets exactly what you want.
4. **Iterate.** Nothing is perfect on the first try. The power is in the feedback loop.

**What to do after today:**

> "Here's your homework — and it's the most fun homework you'll ever get. Think of one real problem in your life that software could solve. Maybe it's organizing something, tracking something, automating something. And build it this week. You have all the tools now."

**Hand out the cheat sheet** (if you didn't already).

**Point them to resources:**

- Claude Code docs: `claude.ai/code`
- The `/development build` command — their starting point for any new project
- Your contact info (if you want to offer follow-up support)

---

## Troubleshooting Guide

Common issues during the workshop and how to handle them:

| Issue | Fix |
|-------|-----|
| "npm not found" | Node.js isn't installed. Download from nodejs.org |
| "claude not found" | Run `npm install -g @anthropic-ai/claude-code` |
| "gh not found" | Run `brew install gh` (Mac) or download from cli.github.com |
| Claude API auth fails | Check API key or subscription status |
| "Permission denied" on Mac | Prefix with `sudo` or fix npm permissions |
| Build errors after scaffold | Have them paste the error to Claude: "Fix this error" |
| Slow responses | Normal during AI generation. Patience. |
| Someone falls behind | Pair them with someone ahead. Peer help is faster than you walking over. |
| Someone finishes early | Challenge them: "Add one more feature" or "Help your neighbor" |

---

## Timing Cheat Sheet

| Time | Phase | What's Happening |
|------|-------|-----------------|
| 0:00 | Start | Welcome, context setting |
| 0:05 | Demo | You build an app live (25 min) |
| 0:30 | Setup | Everyone installs and configures (30 min) |
| 1:00 | Guided build | Everyone builds bookmark manager together |
| 1:10 | Think first | Write down the system design on paper |
| 1:20 | Scaffold | `/development build` together |
| 1:40 | First feature | `/development plan` + `/development implement` |
| 1:50 | Run it | See it working, celebrate |
| 2:00 | Solo build | Everyone picks their own idea |
| 2:40 | Show & tell | 3-4 volunteers demo their apps |
| 2:55 | Closing | Key takeaways, next steps, cheat sheet |
