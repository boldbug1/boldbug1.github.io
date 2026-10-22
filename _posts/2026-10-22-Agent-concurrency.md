---
layout: post
title: Independence is the way?
permalink: /independence-is-the-way/
Author: Boldbug
image: /assets/agent-concurrency/independence.png
---

Picking up from my last post: the bottleneck in the adaptive architecture was dependent tasks. When tasks depend heavily on each other, one agent does most of the work while the other just keeps watching.

So I wondered: **what if the LLM breaks the task into independent pieces that agents can carry out?**

There is an obvious cost. As a project gets more complex, the plan gets bigger, and the model has to reason more to split it cleanly. Maybe a partial-independence approach is the better balance. But instead of just talking, let's test it.

All tests below used the **DeepSeek V4.1 Flash (free)** model in **OpenCode**. These are exploratory experiments, not controlled benchmarks, and results can differ between runs.

## Test 1: independent tasks with a shared TASKS.md

**Project:** a Python backend API for a social media app. I chose a simple backend because design wasn't something I wanted to evaluate. I also wanted a project that doesn't take too long, but isn't *too* independent either.

Like last time, I opened multiple OpenCode instances. One agent made the plan, divided the tasks, and set the contract.

**Prompt A:**

`create a plan for python backend api for a social media app. divide the project into tasks which an individual can perform to complete the project and do list the what tasks depend on other task and if they are independent.`

**Prompt B:**

`now i want you to make a plan and write it to PLAN.md and TASKS.md where there is just a task table and additonal column like to tell who is working on it , any agent can come and claim a task taht hasnt been claimed and update the task status to "started" and when it finished then updated the status to finished. each agent checks the tasks/md 1-2 times to ensure no other agent is working on the tasks. these are the rules of TASKS.md and table and i want you to make the tasks and plan more specefic so every task is independent and doesnt depend on any other task. and also write that agents shoudnt worry about dependecy installation and just make the project`

The model took longer than usual on this. Planning took about 5 minutes.

Then I started the test. Both agents got the same instruction, and I added a small delay to the second one so they wouldn't pick up the same task at the same moment:

- **Agent A:** `Follow the plan and rules and execute them`
- **Agent B:** `Follow the plan and rules and execute them. wait 1s before starting`

In this method the agents already know the schema, the API structure, and everything else they need to work independently.

| Metric | Result |
|---|---|
| Build start | 11:09 PM |
| Agent A finished | 11:28 PM (19 min) |
| Agent B finished | 11:32 PM (23 min) |

Agent B took longer because it ran extra checks to make sure everything was correct and complete.

**What I saw:**

- Coordination improved a lot. When tasks were actually independent, both agents coordinated properly.
- There was one conflict: both agents wrote to the same file once. Because the plan had already fixed the schema and API structure, their outputs were almost identical, so nothing broke.
- There was some latency. To claim a task, an agent has to read the file and then write to it. I wondered if agents could talk to each other directly.

## The baseline: a single agent

One run says nothing on its own, so I ran the same project with a single agent. It wrote its own detailed plan, so its time includes planning.

| Metric | Result |
|---|---|
| Started | 11:52 PM |
| Finished | 12:55 AM |
| Total | about 63 min, planning included |

The context filled up to 84% along the way. The agent went very deep: it kept investigating that everything ran, re-checking the code and dependencies, running tests, and trying to deploy. That isn't bad, but it isn't what I'm trying to measure here.

That fits my prediction, since one agent was doing everything. It's a sign that the multi-agent adaptive method is working, but it doesn't prove it works everywhere, or that it's the best method. The single agent also did testing and deployment, which the multi-agent agents did not, so the two numbers aren't a like-for-like comparison.

**A side observation:** the models spend a lot of time checking tool versions and solving dependency conflicts instead of writing code. That's a good habit, but I think they could assume a version and let the user install it later.

## Changing how I test

Full projects were getting tedious. Every test took hours, and it hurt the momentum of my experiments. I found that **small tests of about 10 minutes are enough to spot a pattern.** So from now on I run small tests to improve the method, and a full test at the end of the day to confirm everything works.

## Fixing claiming with claim.py

In Test 1, the agents spent a lot of time claiming tasks and writing their names in the file. The instruction to check the file 1-2 times also didn't prevent conflicts very well, and it added latency.

So I tried something more robust and faster: a script called `claim.py` that agents use to claim tasks. A claim is just a file, created with the rule "create it only if it doesn't exist." The operating system does that check and the create as one step, so only one agent can win. There's no gap between looking and writing.

I tested it on a small run with three agents. There were no conflicts and all tasks were executed. Only one agent did all the work, though, because the tasks were tiny. So this test checked conflicts, not collaboration, and it passed.

## Test 2: the full run with claim.py

Same project and same model.

My first attempt: planning ran from 3:58 PM to 4:01 PM, and building from 4:04 PM to 4:07 PM.

OK, I'm kind of shocked. How were they so fast?

It turned out the agents had made a smaller project than in the earlier tests. For an honest comparison I gave them the same plan as before. The agent rewrote it to match the new `rules.md`, so the planning cost should stay about the same as the first test (around 5 minutes).

Second attempt: building started at 4:15 PM. One agent started setting up a local PostgreSQL server and got stuck, so I had to step in and tell it to just write the code and not worry about testing. What I'm measuring is how efficiently agents collaborate to write code.

| Metric | Result |
|---|---|
| Build start | 4:15 PM |
| Last task finished | 4:38 PM (23 min) |
| Agent A finished | 4:41 PM (26 min) |
| Agent B finished | 4:55 PM (40 min) |

Agent B took longer because, after the tasks were done, it started a post-review of the whole codebase. As far as I know, there were no conflicts. I credit the atomic script and the new rules.

## Putting it together

| Setup | Last task done | Agent A | Agent B |
|---|---|---|---|
| Test 1 (claiming in TASKS.md) | not recorded | 19 min | 23 min |
| Test 2 (claim.py) | 23 min | 26 min | 40 min |
| Single agent | n/a | n/a | n/a (63 min total, planning included) |

Times for the multi-agent runs start at build start. Planning was about 5 minutes for both.

Measured by when the last task finished, Test 2 took 23 minutes, about the same as Test 1. Test 1's 23 minutes is when Agent B stopped, which includes its checks, and Test 2 includes the PostgreSQL stall I had to fix. So I'd call it the same ballpark: no speed difference I can show.

## What I learned

1. **Independent tasks fix the "one agent works, the other watches" problem.** Both agents did real work.
2. **Atomic claiming removed the conflicts, but it didn't make the run faster.** Claiming wasn't the bottleneck.
3. **A lot of time goes into things around the work:** reviewing, testing, deploying, and fighting the environment. Agent B's extra 14 minutes and the single agent's hour are mostly this.

I only ran each setup once, on a free model, and I stepped in once, so treat these as hints, not facts.

## What's next

The next research is about speed: what actually takes time in a multi-agent run, and how to cut it down.