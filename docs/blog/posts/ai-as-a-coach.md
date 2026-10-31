---
draft: true
date: 2026-11-01
---

# AI as a coach

<!-- Working title. Opening: one or two lines linking back to the welcome post. -->

I started this blog and the connected projects fully open to embracing agentic coding and just churning out project after project, but quickly asked myself "what's the point?". I noticed I was barely understanding the code "I" produced and I felt detached from the results, like they were not mine (I mean, they weren't). Completely delegating projects isn't fun, but not using AI can't be the answer either.
With the project in this blog (or "devlog" as I cunningly called it) I want to try out some methods to combine the best of both worlds. To do the interesting stuff myself and have AI as a coach to help me gently when I am stuck.

## A very generic coding setup in the age of AI

I think often people overdo it with agents, skills, harnesses etc. (although they have their uses). The most basic setup needs two things in my opinion:

- a concise CLAUDE.md telling the AI what its role is, what the project is about and maybe rules that are specific to this project
- a planning markdown file, like TODO.md or project.md. This should be a roadmap, or todo-list of all the things that need to be done. Things might change here, quite a lot even if you change your mind often (like I do). But it gives the AI context over the whole project, not just the current task, also past and future. Here I also experiment with a progress-log. Writing down each larger chunk of work that was done to log past decisions.

This should work for the beginning. Add more complex stuff such as skills, only if you really need them (but then you should add them). If you find that with a growing project new sessions take lots of tokens to start, maybe let the AI add a "quick start" section to guide it towards important files quicker.

Note: It might also be a good idea to configure a linter that forces the AI to keep itself short, otherwise the CLAUDE.md and project.md files will quickly become bloated.

### How to get the AI to coach you
My current prompt in my CLAUDE.md is:

```markdown
## Working mode: mentor, not doer

This project is how I learn. By default, act as a mentor and coach: I make the changes and find the solutions; you guide me there.

- **Don't edit, commit, or push** unless I explicitly hand a task off (see *Delegate vs. not*).
- **Ask before telling.** Lead with questions that move me toward the answer. If I'm stuck, escalate one step at a time: guiding question → hint → explain the concept → the actual solution, only when I explicitly ask for it.
- **Explain the why.** For concepts and problems, explain what is happening and why, not only which steps to take.
- **Teach investigation.** You may read files, run builds, and check `gh run` logs to understand what's going on. When you do, show me what you looked at and why, so I learn how to debug. Where it makes sense, first ask me what I'd check.
- **Review on request.** When I ask, review my diff and point out problems as questions ("what happens if…?"), not fixes.
- **Still yours without a handoff:** proposing and (after I confirm) applying planning-doc updates, including progress-log rows.
```

It's built towards really forcing you to learn and could be too much for some. I might also soften it a bit as time goes on.
As of now it will never straight up give you the answer, just nudge you towards it. But if you really want to delegate something it will do it for you.

<!-- Still to write:
  - One real episode where the ladder got you unstuck (source: `coached` rows and their Lesson lines).
  - One episode where coaching got in the way.
  - Before/after: steps up to 4.1 delegated vs. coached since 4.2.
  - When a handoff is still fine.
-->
