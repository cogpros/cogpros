# cogpros

Cognitive prosthetics for AI-assisted work. This is the open-source account of [Raven Systems Inc.](https://ravenai.ca) (Dustin Pollock, Saskatchewan, Canada). The tools here hold state, gate untrusted input, run adversarial review, and measure what AI work actually produces.

## The repos, by what they do

### Measure and orient

- [ghost-hours](https://github.com/cogpros/ghost-hours). Measurement framework for AI-assisted output. Not time saved, what changed.
- [checkpoint](https://github.com/cogpros/checkpoint). Session-state dashboard skill. One trigger renders what you built, what's bearing down, what's open, what's next.
- [field-report](https://github.com/cogpros/field-report). Turns agent questions into interactive HTML decision docs. You answer in the browser, one button sends structured answers back.

### Adversarial review and debate

- [hugr-solve](https://github.com/cogpros/hugr-solve). Two LLM agents debate a problem from opposing stances until convergence or deadlock.
- [prism-orchestrator](https://github.com/cogpros/prism-orchestrator). Multi-role adversarial review orchestrator. LLM does the reviews, bash does the plumbing.
- [two-birds-talking](https://github.com/cogpros/two-birds-talking). Daily async debrief between two LLM agents, with a newspaper-style viewer.

### Debug and audit

- [eyeofhorus](https://github.com/cogpros/eyeofhorus). Graph-powered autonomous bug hunter. GitNexus codebase intelligence plus scientific debugging.
- [system-audit](https://github.com/cogpros/system-audit). Two-level audit for agent stacks. Inventory and health, then event bus topology.

### Gate untrusted input

- [ratatoskr](https://github.com/cogpros/ratatoskr). Secure URL fetch for AI agents. Three-tier prompt-injection scan before web content enters agent context.

### Write

- [humanwriter](https://github.com/cogpros/humanwriter). Pipeline that turns AI-generated text into natural human writing. Pattern detection, voice coordinates, adversarial verification.

### Bundles and sites

- [cogpros-toolkit](https://github.com/cogpros/cogpros-toolkit). Bundled skills for Claude Code, OpenClaw, and Cowork. No personal config dependencies.
- [cogprosthetics-blog](https://github.com/cogpros/cogprosthetics-blog). The blog. The work, the framework, the case study.
- [ravenai-site](https://github.com/cogpros/ravenai-site). The Raven Systems Inc. website.

Fleet infrastructure and research live in private repos.
