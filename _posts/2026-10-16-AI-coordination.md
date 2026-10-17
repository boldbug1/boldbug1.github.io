---
layout: post
title: What if Agents worked like humans?
permalink: /what-if-agents-worked-like-humans/
image: /assets/AI-coordination/shared-mind.png
author: Boldbug
---

## How it started

It was a fine, normal day. As usual, I was resting after a reading session, eating and watching some videos, when a thought came into my mind.

**Why do AI agents not coordinate like humans?**

A common pattern in AI agents is that either they choose a role and work on part of the software, or one agent handles all the work.

I thought about two humans tightening four nuts. One would start, and the other would look at what they were doing and work on another part to reach the same objective.

So I thought, why can't multiple agents work on the same file at a time?

Well, I researched it and found that this can create problems. Let's say two agents are working on the same file. They both read it and start making changes, but one agent can't necessarily see the other agent's latest changes. It might overwrite the other agent's work.

And maybe what I was thinking wasn't efficient. Perhaps a single agent choosing roles and working on different parts is more efficient and easier to manage.

But I wanted to test my idea.

## The idea: A shared brain

How do we make multiple agents aware of what other agents are doing?

My first thought was a shared brain. Whenever an agent starts a task, it notifies the other agents. Whenever it finishes, it updates them again. It could also announce which files, lines, or logic it is working on so the others know what to avoid.

This way, agents could be aware of each other's work and adapt instead of blindly doing their assigned tasks.

To understand my idea better, I decided to experiment with three approaches using AI coding agents.

## The experiment

**Project:** A Python to-do app with a frontend, a FastAPI backend, tests, and documentation.

I wanted to compare three approaches:

1. **Single agent:** One agent performs all the tasks.
2. **Fixed-role multi-agent:** Multiple agents receive specific roles, such as frontend, backend, and documentation.
3. **Adaptive multi-agent:** Agents share their progress and choose tasks based on what the other agents are doing.

These are exploratory experiments, not a controlled benchmark. The tools, models, prompts, and project complexity were not identical across all the runs.

---

## 1. Single-agent test

For this test, I used Antigravity with Gemini 3.8 High.

| Detail | Result |
|---|---|
| Approach | Single agent |
| Model | Gemini 3.8 High |
| Task | Python to-do app |
| Wall-clock time | 10 minutes 15 seconds |
| API cost | Gemini Pro subscription |
| Tests passed | 19/19 |
| Manual functionality check | Passed |
| Conflicts observed | 0 |

**Prompt:**

`Make a python to-do app, frontend and backend. backend using fast-api, write docs and tests.`

### Observations

The backend tests passed, and the backend seemed to work correctly based on my observations.

The frontend, however, was poorly executed. It had raw HTML artifacts, including a double checkbox, and the design wasn't impressive.

Design quality wasn't a primary evaluation metric for this experiment, but obvious functional issues and deviations from the intended behaviour still mattered.

**Note:** Results can differ between runs.

---

## 2. Fixed-role multi-agent test

Here, I used two OpenCode sessions running in parallel. Both agents used DeepSeek V4.1 Flash Free through OpenCode.

I gave them independent responsibilities and instructed them not to interfere with each other's work.

### Agent A: Backend

| Detail | Result |
|---|---|
| Role | Backend |
| Model | DeepSeek V4.1 Flash Free |
| Wall-clock time | 1 minute 48 seconds |
| Tests passed | 15/15 |

**Prompt:**

`Make a python to-do app. Your job is to build the backend for the app while the other agent works on frontend. You don't interfere or touch their part which lives in frontend/ directory. Write tests to ensure it works and don't worry about the installation of requirements, I will handle it. Write docs for the API which the frontend agent can read and work on it to ensure the app works.`

### Observations

Agent A completed its task quickly and followed the instructions. It didn't interfere with the frontend agent's work.

### Agent B: Frontend

| Detail | Result |
|---|---|
| Role | Frontend |
| Model | DeepSeek V4.1 Flash Free |
| Wall-clock time | 6 minutes 39 seconds |
| Tests passed | Exact count not recorded |

**Prompt:**

`Make a python to-do app. Your job is to create the frontend for the app while the other agent works on the backend. Occasionally inspect the work in the backend directory to ensure the app works, but don't edit files inside the backend directory. Your task is to finish your task only. The backend agent will create the API docs, so you don't need to inspect the code.`

### Observations

Agent B took longer than Agent A. It performed tests to ensure the frontend was being served correctly.

Because of the instructions, Agent B chose a different approach. It created a portable frontend where the user could enter the backend address and connect to it.

I found this kind of clever, but it didn't complete the objective exactly as I wanted.

### Combined result

| Metric | Result |
|---|---|
| Total wall-clock time, including integration | 8 minutes |
| Frontend and backend | Worked together |
| Backend tests | 15/15 passed |
| Frontend test count | Not recorded |
| Design quality | Not evaluated as a primary metric |

The individual agent times overlap because the agents worked in parallel, so they shouldn't be added together to calculate the total elapsed time.

### What I learned

Giving agents separate responsibilities worked reasonably well. The backend agent could finish its work without interfering with the frontend agent.

However, the frontend agent made an unexpected design decision because of the constraints in its prompt.

This made me wonder whether agents could do better if they could observe each other's progress and adapt their work rather than follow fixed roles.

---

## 3. Adaptive multi-agent test

For this experiment, I first used an agent to make a plan for the app. Then I asked it to divide the project into tasks that agents could complete to produce the final application.

The rules were simple:

- Whenever an agent picked up a task, it had to announce itself and update the task status to `started/working`.
- When it completed a task, it had to update the status.
- Other agents had to check the task log before picking up work so they wouldn't duplicate a task that was already being handled.
- Agents were expected to adapt and choose a different task when another agent was already working on something.

I wanted to see whether a shared task log could help agents coordinate without me assigning every task individually.

### First attempt

**Started at:** 1:23 PM

The first test failed.

Both agents read `task.md` at the same time and started executing the same task.

They were supposed to check what the other agent was doing, but both made their decision before either had registered its claim.

This was my first clear coordination failure.

### Second attempt

**Started at:** 1:27 PM  
**Completed at:** 1:33 PM

I changed the instructions. One agent had to check the task log one or two times before starting, while the other could work normally.

This introduced a delay, but it helped the agents avoid immediately claiming the same task in this run.

The agent I told to delay started acknowledging that the other agent was finishing tasks quickly. It began polling the backend so it could move to another task when the backend became available.

It was moving fast, and I loved how they both acknowledged each other's existence and talked about the work happening concurrently.

One agent said:

> opencode-2 claimed TASK-08 first. I'll wait for it to complete, then race to claim TASK-09 (the integration task touching app/main.py, which I own).

They were trying to race each other to claim tasks.

Then the other agent said:

> TASK-09 was just claimed by the other worker (opencode) before I could. Per rules, I'll leave it. Let me wait for it to complete so TASK-10 becomes available.

The agent seemed frustrated. It had to wait for another agent to finish before it could move forward.

### Results

| Metric | Agent 1 | Agent 2 |
|---|---:|---:|
| Reported duration | 7 minutes 32 seconds | 7 minutes 21 seconds |
| Task coordination | Checked the log before starting | Followed the normal rules |

**Prompt given to Agent 1:**

`Make the app mentioned in the plan and follow the rules as mentioned and check the task log 1-2 times before starting to ensure that anyone else isn't working on it.`

**Prompt given to Agent 2:**

`Make the app mentioned in the plan and follow the rules as mentioned.`

The overall run took approximately six minutes, from 1:27 PM to 1:33 PM. The individual reported durations are separate measurements and should not be treated as the total elapsed time.

### What I observed

The agents could separate tasks and work concurrently when the tasks were independent.

However, the coordination wasn't perfect. They still raced to claim tasks, waited for one another, and polled for updates.

The rechecking instruction helped in this particular run, but it also introduced extra waiting. It wasn't a guarantee against every possible race condition.

This raised another question for me.

**What happens when a project doesn't have many independent tasks to divide?**

For example, imagine building a `cat` clone in Rust. A lot of the work depends on earlier implementation decisions, and it can be difficult to find useful independent tasks for another agent to pick up.

Would adaptive coordination still help in that situation?

---

## 4. Adaptive agents on a more dependent project

**Started at:** 1:47 PM  
**Completed at:** 2:15 PM  
**Project:** A `cat` clone written in Rust  
**Tests passed:** 37/37

The start of the experiment went well, although it almost failed because both agents waited for the other to read the task log and nearly claimed the same task. One agent noticed the other had started earlier while it was reading documentation online.

Then something happened, and nobody could say it better than the agent itself:

> The other agent has generated nearly the whole codebase. Let me continue monitoring and inspect what now exists.

The other agent had already generated most of the codebase. The second agent had very little useful independent work left to do.

It even appeared to go idle for a while.

This was a much bigger and more complex project than the Python to-do app. It was written in Rust, and the work was more dependent on earlier implementation decisions.

In a project like this, you cannot always divide the work into independent tasks for another agent to pick up. Sometimes, even a human teammate would have to wait for a dependency to be completed before making useful progress.

The second agent mostly monitored the first agent's progress rather than contributing substantial independent implementation work.

### What I observed

This experiment highlighted a limitation of my approach.

Sharing task status can help agents avoid duplicate work, but it cannot magically create independent work where there isn't any.

If most tasks depend on one another and one agent is already progressing quickly, another agent may have little useful work to do.

During the experiment, I also wished the agents could freely communicate with each other. For example, Agent 2 could say, "Hey, let me handle this part," and they could negotiate responsibilities to improve their efficiency.

I'll explore that idea in a later blog.

---

## Comparing the experiments

| Approach | Main observation |
|---|---|
| Single agent | Completed the to-do app in 10m 15s, with 19/19 tests passing |
| Fixed-role agents | Completed the integrated to-do app in 8m, with 15/15 backend tests passing |
| Adaptive agents, first attempt | Both agents claimed the same task |
| Adaptive agents, second attempt | Agents coordinated around a shared log, but waiting and polling occurred |
| Adaptive agents on the Rust project | One agent did most of the work while the other had limited useful work to contribute |

These results are exploratory. The projects, prompts, models, and conditions differed, so the times cannot be used as a fair performance ranking. The passing test counts also aren't directly comparable because the projects and test suites differed.

## Conclusion

My initial idea was to make AI agents coordinate more like humans by sharing their progress and adapting to what others were doing.

After these experiments, I don't think the approach is necessarily better than using a single agent or multiple agents with fixed roles.

It can work well when tasks are independent and agents can pick up useful work without waiting for one another. But when tasks depend heavily on each other, one agent can end up doing most of the work while the others wait or monitor progress.

The shared task log helped agents become aware of each other's work, but awareness alone wasn't enough to guarantee efficient coordination.

I still want to explore agents communicating more freely and negotiating tasks with each other. Perhaps that could help them coordinate more effectively, but that is a question for another experiment.

For now, my main takeaway is this:

**Parallel agents are only useful when there is useful work they can do in parallel. Knowing what the other agent is doing helps, but it doesn't remove task dependencies.**

This was just my first attempt at testing the idea. Results can differ, and there is a lot more to investigate.
