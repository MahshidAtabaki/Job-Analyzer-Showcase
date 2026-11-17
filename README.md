# Job Analyzer

**Know if a job is worth your time, before you apply.**

Job Analyzer is a personal tool for job hunting in the UK on a work visa. Every saved job raises the
same three questions: *can they sponsor my visa, do I really fit, and is the company healthy?*
Answering them by hand takes 20 to 30 minutes per job. Job Analyzer answers them in about three
minutes, then helps you apply.

https://github.com/user-attachments/assets/c5b1b958-797c-4e6f-a215-0819fd5f4a3a

> This repository is a showcase only: the overview and the demo video. The source code is private.

## How it works

1. **Save jobs on LinkedIn, as usual.** On your laptop or your phone. A small browser extension
   picks up your saved jobs and brings each one into the dashboard, with the logo, the company and
   the full job description. There's no "Add job" form to fill in.
2. **Pick a job and go step by step.** Nothing runs until you ask, and each step shows its result
   and waits for you before the next one
   (see [Product thinking](#product-thinking-spend-only-on-what-decides)):
   1. **Visa sponsorship, free:** today's **UK Register of Licensed Sponsors** (from gov.uk) is checked
      for the company. No AI is used. If they can't sponsor your visa, you can stop right there.
   2. **The company:** Claude researches it on the web (what it does, funding, revenue, runway,
      headcount and risks), with a small, fixed research budget. Your CV isn't sent.
   3. **Your fit:** if you like the company, Claude compares the job with your CV and profile, reusing
      the company research instead of searching again.
   4. **Cover letter, tailored CV and interview prep:** only for the jobs you decide to apply to.
3. **Get a straight answer.** One card with a fit score out of 10, a verdict (for example
   *Apply with caveats*) and a one line summary, followed by:
   - **Visa:** whether the company can sponsor a **Skilled Worker** or **Scale-up** visa, with the
     evidence from the register.
   - **Company health:** stable, growing or risky, with the numbers behind it.
   - **Why you fit / Watch outs:** your real strengths for the role and the gaps to be ready for.
     Nothing is stretched and no experience is invented.
   - **Worth checking:** questions about things the role asks for that your CV doesn't show yet.
4. **Apply with Claude beside you.** A chat panel sits next to the report. Ask for a tailored CV or a
   cover letter, or paste application questions, and download the result as a PDF.
5. **Track where every job stands:** Saved → Analyzed → Applying → Applied.

## Product thinking: spend only on what decides

**The problem.** Every analysis costs part of the user's Claude plan. The first version ran everything at
once for every job: visa check, company research, fit against the CV, a cover letter and interview prep,
about 40k to 70k tokens per job. User research into the user's main concern showed the decision doesn't
work that way. People drop a job the moment one deal breaker shows up: if the company can't sponsor a
visa, nothing else in the report matters, and if the company looks unstable or unappealing, the fit score
doesn't matter either. So most of the tokens were spent on reports for jobs the user had already ruled
out, and the user had to read past a lot of information they didn't want.

**The decision.** I ranked the signals by how decisive they are, so that if one fails the rest is never
needed, and made each later step something the user asks for:

| Step | What it answers | If it fails | Cost (tokens, estimated) |
|---|---|---|---|
| 1. Sponsor register | Can they sponsor my visa? | Stop. Nothing else matters. | 0 (no AI) |
| 2. Company research | What is it, is the money and runway healthy? | User stops here. | ~10k to 20k, on request |
| 3. Fit with my CV | Am I a match, and where are the gaps? | User stops here. | ~12k to 18k, on request |
| 4. Cover letter, CV, interview prep | Help me apply | | ~15k to 20k each, on request |

**Steps added**

- **A free sponsorship gate** before any AI: a register lookup in code. Jobs that fail never reach Claude.
- **A company only step** that sends no CV and has a hard research budget (4 searches, 2 fetches; there was
  no limit before).
- **User approval checkpoints:** every step shows its result and waits for a click before the next one
  (Research the company, Check my fit, Cover letter, Interview prep).
- **A fit step with no web tools**, reusing the company research instead of searching again.
- **Cover letter and interview prep on demand**, instead of being written for every job.

**Tokens saved per job analysis** (estimates from prompt sizes and the research budget; real numbers are
logged for every run):

| What happened to the job | Before | Now | Saved |
|---|---|---|---|
| Company doesn't sponsor visas | 40k to 70k | 0 | all of it (100%) |
| Sponsors, but the user doesn't like the company | 40k to 70k | 10k to 20k | ~30k to 50k (about 70%) |
| User likes the company and checks fit | 40k to 70k | 25k to 35k | ~15k to 35k (about 40%) |
| Cover letter and interview prep | in every job | only for jobs being applied to | ~3k to 6k of writing per job not applied to |

Per user, the saving depends on how many saved jobs fail early. For example, if half the saved jobs don't
sponsor visas, a quarter are dropped after the company step and a quarter go on to the fit check, an
average job costs about 11k tokens instead of about 55k: **roughly 80% less per job analysis**, and the
user only reads what helps the next decision.

**The right model for each step.** I also looked at what each step really needs from the model, and
stopped using the same model for everything:

| Step | Model | Why |
|---|---|---|
| 1. Sponsor register | none | A lookup in code. |
| 2. Company research | Haiku (small, fast) | Searching and summarising public facts. A bigger model adds cost, not accuracy. |
| 3. Fit with my CV | Sonnet | Judging gaps honestly, without stretching experience, needs careful reasoning. |
| 4. Cover letter, CV, interview prep | Sonnet | Writing quality and tone matter here. |

Haiku uses a fraction of what Sonnet does per token, so the company step, which runs on far more jobs
than the later steps, now costs a fraction of what it did, on top of the savings above. The bigger model
is kept for the steps where its judgement changes the result.

## Your profile

Upload your CV (PDF or Word) and the CV text is rebuilt automatically. Add your LinkedIn and portfolio,
plus notes, and every analysis uses them.

## Private by design

- **Runs on your own laptop.** The analysis uses Claude Code with your own Claude subscription.
  There's **no API key** and no per-request bill.
- **Your data stays with you.** Jobs, reports and your CV are stored locally.
- **Optional online dashboard.** It's useful on your phone. The website only queues the work; your
  laptop does the analysis and sends the result back.

## Built with

Node.js · Claude Code · a Chrome extension (Manifest V3) · the gov.uk sponsor register · PDF
generation for CVs and cover letters.

---

© 2026 @MahshidAtabaki on GitHub. All rights reserved. No licence is granted to use, copy, modify or
distribute this project, its design or its demo.

*The person in the demo, "Alex Morgan", is fictional. Company information shown comes from a real
analysis of a public job listing.*
