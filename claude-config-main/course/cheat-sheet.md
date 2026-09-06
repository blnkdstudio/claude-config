# Systems Architecting with AI — Cheat Sheet

> Keep this handy. One page is all you need.

---

## The Process

```
THINK  →  DESCRIBE  →  REVIEW  →  ITERATE
  |          |            |          |
What do    Tell AI      Check if   Fix what's
I want?    to build it  it's right wrong
```

**You are the architect. AI is the builder.**

---

## Starting a New Project

Open your terminal, create a folder, start Claude Code:

```bash
mkdir my-project
cd my-project
claude
```

Then run the build command:

```
/development build
```

This walks you through:
1. Defining your MVP (what to build)
2. Picking your tech stack (how to build it)
3. Scaffolding the project (creating all the files)
4. Pushing to GitHub (saving your work online)

---

## Building Features

After your project is set up, build features one at a time:

```
/development plan          # Design the feature (interactive)
/development implement     # Build it with AI agents
/development ship          # Save and push your changes
```

---

## How to Talk to AI

### Be Specific (Good)

| Instead of... | Say... |
|--------------|--------|
| "Make it better" | "Sort the list by date, newest first" |
| "Fix it" | "The save button doesn't work — clicking it does nothing" |
| "Add a feature" | "Add a search bar that filters bookmarks by title" |
| "Make it look nice" | "Use a card layout with rounded corners and a blue header" |

### The Magic Formula

```
I want [WHAT] that does [BEHAVIOR] when [TRIGGER].
```

Examples:
- "I want a button that saves the form data when clicked"
- "I want a list that shows all bookmarks sorted by newest first"
- "I want a search bar that filters results as I type"

---

## When Things Go Wrong

**Error message?**
→ Copy the entire error and tell Claude: "I'm getting this error: [paste]. Fix it."

**Wrong behavior?**
→ Describe what you expected vs. what happened: "When I click save, nothing happens. It should save the bookmark and show it in the list."

**Looks wrong?**
→ Describe what you see vs. what you want: "The text is too small and there's no spacing between items. Make the text 16px and add spacing between list items."

**Totally stuck?**
→ Describe the situation: "I'm trying to add search to my bookmark app but I don't know where to start. The app currently shows a list of bookmarks. I want to add a search bar above the list."

---

## Thinking Like an Architect

Before you ask AI to build anything, answer these:

```
1. What am I building?        → [one sentence]
2. Who is it for?              → [the user]
3. What are the 3 features?   → [list them]
4. What does the user see?    → [describe the screen]
```

This takes 2 minutes and saves 20 minutes of confusion.

---

## Useful Commands

| Command | What It Does |
|---------|-------------|
| `claude` | Start Claude Code |
| `/development build` | Start a new project from scratch |
| `/development plan` | Plan a new feature |
| `/development implement <path>` | Build a planned feature |
| `/development ship` | Save and push your changes |
| `/development list` | See all your features and their status |
| `/development debug` | Fix a bug with AI help |

---

## Your First Week After the Workshop

| Day | Challenge |
|-----|-----------|
| **Day 1** | Take the app you built in the workshop. Add one more feature. |
| **Day 2** | Start a brand new project — something you actually want to use. |
| **Day 3** | Build 2 features for your new project. |
| **Day 4** | Show someone what you built. Explain it to them. |
| **Day 5** | Deploy your app so others can use it (ask Claude how). |
| **Weekend** | Think about what else you could build. Write down 3 ideas. |

---

## Key Mindset Shifts

| Old Thinking | New Thinking |
|-------------|-------------|
| "I need to learn to code first" | "I need to learn to describe what I want" |
| "I should understand every line" | "I should understand the system design" |
| "Errors mean I failed" | "Errors are feedback — paste them and iterate" |
| "Building software takes months" | "Building an MVP takes hours" |
| "I need a developer" | "I need a clear vision of what to build" |

---

*Built with Claude Code — the same tool you used today.*
