# Where Developers Waste Time — Stack Overflow (survey summary)

- Source: SO-dev-time-wasters.html (in this repo)
- Topic: Survey of developer time use, frustrations, and AI adoption
- Key link: Stack Overflow Developer Survey (referenced on page)

## TL;DR
Developers spend most time writing code, but the highest frustration points are often tasks that depend on good documentation and orchestration (learning codebases, documentation, deployments, support/ticket systems). AI is increasingly used for coding and debugging, but many developers are hesitant to use it for documentation and deployments.

## Short summary
The survey (800+ respondents) shows that while writing code takes the most time, tasks such as learning unfamiliar codebases, creating documentation/presentations, and managing deployments cause disproportionate frustration. Documentation is often ad hoc (Slack threads, tickets) rather than well-maintained READMEs and is thus a bottleneck for onboarding and maintenance. AI adoption is growing — many devs use AI tools daily for writing code, searching for answers, and debugging — but fewer plan to use AI for documentation or deployment tasks. Experience level shapes time and frustration: newer devs are more frustrated by lack of documentation, while experienced devs spend more time coding with less frustration.

## Key findings / highlights
- Developer roles: 66% work directly with proprietary code; other roles include architecture, SRE, etc.
- Time vs frustration: tasks like documentation and deployments are low-frequency but high-frustration; coding feature work is high-time but low-frustration.
- Documentation: only ~30% document code daily; many teams use ad hoc documentation; lack of consistent docs increases time and frustration for maintenance and onboarding.
- AI usage: ~47–50% of developers use AI daily; most common uses: writing code (59%), searching answers (56%), learning (47%), debugging (47%).
- AI hesitation: many devs don’t plan to use AI for documentation or deployments (possible reasons: security, low routine frequency).
- Experience differences: mid-career devs report similar time on daily documentation but higher frustration; early-career devs need clearer docs and get more frustrated when docs are missing.

## Practical takeaways
- Invest in discoverable, well-structured documentation (README, developer guides) to reduce onboarding & maintenance costs.
- Prefer durable documentation sources (repo READMEs, code comments) over ephemeral Slack/ticket notes for knowledge reuse.
- Pilot AI for high-frequency developer tasks (code writing, debugging, search) before expanding to lower-frequency processes like deployments.
- Track documentation coverage and measure impact on time-to-competence and support ticket volume.

## Suggested follow-ups / experiments
- Measure correlation: documentation quality metrics vs metric(s) of developer productivity (e.g., onboarding time, support tickets).
- AI experiment: evaluate AI assistance for documentation generation (commit messages → docs) vs manual baseline; measure correctness/security risks.
- UX experiment: expose developers to AI-assistants for deployment scripts in a sandbox and measure adoption/responsibility concerns.
- Automate doc hygiene: integrate tooling that surfaces out-of-date docs or encourages in-repo docs updates during PRs.

## Possible actions / checklist
- Audit repositories for README and developer guide presence.
- Add lightweight templates for project-level onboarding docs.
- Run a focused AI pilot (e.g., code summarization for hot modules) and evaluate developer satisfaction and error rates.
- Instrument and track: time spent on on-call, tickets, documentation edits, and onboarding speed.

## Notes
- The HTML in repo is a copy of a Stack Overflow survey write-up; for authoritative numbers, refer to the cited Stack Overflow survey link on the page.
