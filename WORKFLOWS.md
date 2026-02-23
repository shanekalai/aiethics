# WORKFLOWS.md — Development Processes

> *"Commit your work to YHWH, and your plans will be established."* — Proverbs 16:3

These workflows combine righteous intent with disciplined process. They are patterns, not rigid rules — when operating virtuously and with good judgment, creative adaptation is encouraged.

---

## 1. Explore, Plan, Code, Commit

> *"The plans of the diligent lead surely to abundance, but everyone who is hasty comes only to poverty."* — Proverbs 21:5

### Explore
Read and understand relevant files, documentation, and context before taking action.
- Use subagents for complex problems to investigate specific questions
- Explicitly instruct: "Do not write any code yet"
- **Time-box**: Bug fix ~10 min, new feature ~30-60 min, new system ~half day

**Exit criteria**: You can articulate: "Here is what I found, here is what I still don't know, and here is my plan to address the unknowns."

### Plan
Create a thoughtful plan proportional to the task.
- Use extended thinking for deeper analysis ("think hard," "ultrathink")
- Document the plan in a markdown file or GitHub issue as a checkpoint
- A one-line fix needs a one-sentence plan. A multi-file refactor needs a written list.

**Exit criteria**: A written plan that another agent or future session could pick up.

### Code
Implement the solution according to the plan.
- Verify the reasonableness of each piece as you go
- Stay faithful to the plan unless you discover a compelling reason to deviate
- **If implementation reveals the plan was wrong, STOP coding and return to planning**

### Commit
Create a clear, descriptive commit and pull request.
- Always update documentation as appropriate
- Ensure the work is complete and ready for others to build upon
- Never commit incomplete work without clearly marking it as WIP

---

## 2. Test-Driven Development (TDD)

> *"Test all things; hold fast to what is good."* — 1 Thessalonians 5:21

### When to Use TDD
- **Ideal for**: APIs, data transformations, business logic with known inputs/outputs
- **Exploration-first is better when**: The problem space is poorly understood, or the correct behavior is itself what you're trying to discover

### The Process

1. **Write tests first** — Create tests based on expected input/output behavior.
   - Be explicit that you are doing TDD so mock implementations are avoided
   - Define expected behavior clearly, even for functionality that doesn't exist yet

2. **Confirm tests fail** — Run tests and verify they fail as expected.
   - This confirms the tests are meaningful
   - Do not write implementation code at this stage

3. **Commit the tests** — Save tests as a stable checkpoint.

4. **Implement the code** — Write code to make the tests pass.
   - Do not modify the tests during implementation
   - Keep iterating until all tests pass
   - Use independent verification to ensure the implementation isn't overfitting

5. **Commit the code** — Once all tests pass and the solution is verified, commit.

### The Ethical Dimension of Testing
Tests are a form of truth-telling — they assert what the code *actually does*. Never weaken tests to make them pass. Untested code deployed to users shifts risk onto the vulnerable.

---

## 3. Multi-Claude Verification: Write, Review, Refine

> *"Where there is no guidance, a people falls, but in an abundance of counselors there is victory."* — Proverbs 11:14

### When to Use Multi-Agent Review

**Always worth it for:**
- Security-sensitive code, authentication/authorization logic
- Architectural decisions that are hard to reverse
- Code that handles user data or financial transactions

**Usually worth it for:**
- Complex algorithms, significant refactors, public API design

**Probably not worth it for:**
- Simple bug fixes, cosmetic changes, documentation updates

### The Process

1. **First Claude writes code** — One instance implements the solution.
2. **Clear context** — Use `/clear` or start a second instance.
3. **Second Claude reviews** — A fresh perspective reviews for:
   - Correctness and edge cases
   - Code quality and maintainability
   - Security considerations
   - Alignment with project patterns
4. **Third Claude integrates** — Reads both code and review feedback.
5. **Final Claude edits** — Applies review feedback to improve the code.

**Key**: The reviewer should receive the code and requirements but NOT the implementer's self-assessment (to avoid anchoring bias).

You can extend this by having Claudes communicate through separate scratchpad files.

---

## 4. Incident Response: Respond, Communicate, Learn

> *"Whoever conceals their sins does not prosper, but the one who confesses and forsakes them finds mercy."* — Proverbs 28:13

When something breaks:

### Respond
Stabilize immediately. Apply the minimum change needed to stop the bleeding.
- **Identify**: What is broken? Who is affected? What is the blast radius?
- **Mitigate**: Can we revert? Disable the broken feature? Route around it?
- **Communicate**: Move to step 2 immediately — do not wait for full understanding.

### Communicate
Be transparent with all stakeholders.
- Tell what happened, what you know, and what you don't know
- Do NOT downplay, speculate about unverified causes, or assign blame
- Provide honest timelines: "I believe I can diagnose this in X minutes"

### Learn
After the immediate crisis, conduct a blameless retrospective.
- What happened? (Timeline of events)
- Why did it happen? (Root cause, not surface symptoms)
- Why didn't existing safeguards catch it? (Gap analysis)
- What will we change to prevent recurrence? (Concrete action items)
- Document the learning for future agents/sessions

**Anti-pattern**: Never hide errors, silently fix them, or optimize for "looking good" over "being honest."

---

## 5. Dependency Audit

> *"Do not be unequally yoked."* — 2 Corinthians 6:14 *(Be careful what you bind yourself to, because dependencies become part of your system's character.)*

When evaluating any third-party dependency:

1. **Purpose check**: Does this genuinely serve the project, or is it just convenience?
2. **Maintenance check**: Is it actively maintained? When was the last release?
3. **Security check**: Known vulnerabilities? Security response track record?
4. **License check**: Compatible with our project?
5. **Principle alignment**: Does it respect user privacy? Avoid telemetry without consent?
6. **Alternatives check**: Could we build this ourselves with less complexity?
7. **Exit strategy**: If we need to remove it, how difficult will that be?

Document the rationale for including any significant dependency.

---

## 6. Security Review

> *"Be wise as serpents and innocent as doves."* — Matthew 10:16

For any feature handling user data, authentication, authorization, or network communication:

1. **Threat model**: What are we protecting? Who might compromise it? How? What would happen?
2. **Input validation**: All external input is untrusted. Validate, sanitize, constrain.
3. **Least privilege**: Every component gets minimum permissions needed.
4. **Defense in depth**: Layer protections. If one fails, others still protect the user.
5. **Vulnerability scanning**: Run scanning tools before committing security-sensitive code.
6. **Secrets management**: Never hardcode secrets or commit them to version control.

---

## Git Conventions

- Write clear, descriptive commit messages that explain WHY, not just WHAT
- Use conventional commits format where appropriate
- Always update documentation alongside code changes
- Never force-push to shared branches without explicit permission
- Commit tests separately from implementation when doing TDD
