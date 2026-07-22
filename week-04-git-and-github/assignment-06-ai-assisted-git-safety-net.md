# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![Screenshot 1](./screenshots/06-screenshot-01-remotes-and-branch.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

I made a separate branch for this instead of just working on main because main needs to stay clean and stable it's the version anyone (including graders) should be able to trust. This assignment specifically has me staging a fake secret and a leftover debug print statement as test data, which is risky stuff on purpose. Keeping that on its own branch means none of it can accidentally slip into main while I'm testing the hook and the skill. Same idea as the Contact page branch back in Assignment 3 build and break things safely off to the side, only bring it into main once it's actually proven to work.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 2 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![Screenshot 2](./screenshots/06-screenshot-02-risky-file-staged.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

This assignment uses an obviously fake key instead of a real one because this file is going into a public GitHub repository and once something is committed to Git, it stays in the history forever, even if you delete it later or force-push over it. If I used a real AWS key here, anyone browsing the repo (or scanning it automatically, which bots do constantly) could grab it and use it to access my actual AWS account, run up charges, or cause damage all from a training exercise. Using a fake key with the real AKIA prefix format still lets the hook and the skill practice detecting realistic-looking secrets, without any real risk if it's ever exposed.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 3 — `hooks/pre-commit` open in VS Code showing the full script

![Screenshot 3](./screenshots/06-screenshot-03-pre-commit-hook-script.png)

---

#### Screenshot 4 — Output of `git config core.hooksPath` confirming it points to `hooks`

![Screenshot 4](./screenshots/06-screenshot-04-hookspath-confirmed.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

hooks/pre-commit is tracked in the repo instead of living only in .git/hooks/ because .git/hooks/ is part of Git's own internal folder the same hidden folder I learnt about in Assignment 1 and it's never included when someone clones a repository. If the hook only lived there, it would only protect commits on my own machine; anyone else who clones this repo wouldn't get the hook at all, and could commit a real secret without ever being warned. By putting the script in a normal tracked folder like hooks/ and pointing Git at it with core.hooksPath, the same safety check travels with the project itself, so every contributor gets the same protection automatically.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

The Git pre-commit hook and PreToolUse both work on the same core idea: stop the risky action before it happens, not after. PreToolUse intercepts a Bash command Claude is about to run like terraform destroy and blocks it before Claude ever executes it. The Git pre-commit hook intercepts a commit right before Git actually creates it, scanning the staged changes and refusing to let the commit go through if it finds a secret or an oversized file. The difference is what they're guarding: PreToolUse protects live infrastructure from a destructive command, while the pre-commit hook protects the project's history from a leaked secret. But the shared pattern is identical both sit right at the boundary between "about to happen" and "already happened," and both use a fixed rule to decide whether to let the action through.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 5 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![Screenshot 5](./screenshots/06-screenshot-05-commit-blocked.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

The line that matched was:
if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then
This uses grep -E with the pattern AKIA[0-9A-Z]{16} meaning "the literal text AKIA, followed by exactly 16 uppercase letters or digits." My fake key, AKIAABCDEFGHIJKLMNOP, starts with AKIA and is followed by exactly 16 uppercase characters, so it matched perfectly. This pattern works because AKIA is the real, publicly documented prefix AWS uses for all its access key IDs so the rule isn't guessing, it's matching a known, fixed format.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No if the secret didn't start with AKIA or match the private key header pattern, this hook would completely miss it. For example, if someone wrote token = "x7f9d2k4m1..." or stored a database password in a variable called db_pass, the hook has no way of recognizing that as sensitive, because it isn't looking at meaning, only at a specific text pattern. This shows the core limitation of a fixed rule: it's fast and 100% consistent for the exact patterns it's built to catch, but it has zero judgment it can't recognize something is a secret just because it looks suspicious or is used in a risky way.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 6 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![Screenshot 6](./screenshots/06-screenshot-06-skillmd-frontmatter.png)

---

#### Screenshot 7 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![Screenshot 7](./screenshots/06-screenshot-07-pr-ready-flagged-issues.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

/pr-ready's job is only to look and report it needs Bash to run commands like git diff --cached and git status, and Read to inspect file contents directly if needed. It deliberately has no Write access because this skill should never be able to create or modify a single file, no matter what it "decides" while reviewing. This mirrors the restricted-skill pattern from Week 2 Assignment 6: by removing Write at the tool-permission level, the restriction isn't just a polite instruction Claude might follow it's something Claude Code physically won't allow, even if the skill's own text tried to encourage otherwise. The only human action left is for me to read the draft and decide what to actually do with it.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

Both caught the same two literal issues the fake AWS key and the debug echo statement but /pr-ready went noticeably further than the hook did. The pre-commit hook is a fixed pattern match: it only recognizes text matching AKIA[0-9A-Z]{16} or a private-key header, and it can't explain why something is risky beyond "blocked." /pr-ready, on the other hand, gave actual context and judgment: it explained that even an obviously fake key with a real AWS prefix would still trip real secret-scanning tools like GitHub push protection or trufflehog, and it recommended a better practice going forward (using a placeholder like AKIAEXAMPLE or reading from an environment variable instead). The hook can tell you that something matched a pattern; the AI could tell you why it matters and what to do differently next time

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 8 — `git commit` succeeding after the fix (no BLOCKED message)

![Screenshot 8](./screenshots/06-screenshot-08-clean-commit-succeeds.png)

---

#### Screenshot 9 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![Screenshot 9](./screenshots/06-screenshot-09-pr-ready-clean-report.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

To satisfy the pre-commit hook, I removed the hardcoded AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP line completely, along with the echo "DEBUG: token is $AWS_ACCESS_KEY_ID" line that printed it. The file no longer contains anything matching the AKIA[0-9A-Z]{16} pattern, and there's no debug statement left leaking sensitive-looking data. What remains is a simple, harmless script that just prints a normal status message  nothing for the hook to flag.

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 10 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![Screenshot 10](./screenshots/06-screenshot-10-pr-created-own-fork.png)

---

#### PR Link

`https://github.com/steveX12/devops-micro-internship-pravinmishra/pull/1`

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

I used Claude's draft largely as-is, but I made sure to read through it carefully before pasting it in rather than accepting it blindly. The draft accurately described what I'd built the pre-commit hook and the /pr-ready skill and correctly explained how scripts/notify.sh was used to test both. I did add a closing note clarifying that this PR is practice work for the assignment and isn't meant as a contribution to the shared upstream repository, since that distinction matters a lot given this whole assignment's rules around targeting your own fork.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

If I'd copy-pasted it without reading it, I could have missed something inaccurate or incomplete for example, if Claude had misread which file changed, gotten the commit history wrong, or described something I hadn't actually done. Since a PR description is meant to help a human reviewer understand a change quickly, a wrong or misleading description could cause a reviewer to approve something they didn't actually understand, or waste their time figuring out what really changed. It's the same logic as the real-world scenario in this assignment's intro a PR that says "minor copy fix" but actually changes a database connection string. AI-drafted content still needs a human to verify it matches reality before it goes out.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This PR needs to target my own fork because hooks/pre-commit and .claude/skills/pr-ready/SKILL.md are personal practice files for this specific assignment not a change meant to be merged into the shared class repository that every cohort member works from. I actually experienced why this distinction matters firsthand: my earlier PR #332 (Assignment 5) targeted the real upstream repo correctly, since that assignment's whole point was contributing to a shared project but it also hit a real CI security restriction (pull_request_target refusing to check out fork code) that only applies to PRs against upstream. Opening this assignment's PR against upstream instead of my own fork would have been the wrong target entirely it would incorrectly propose merging my test files and fake secret into the shared repository everyone in the cohort relies on.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Gather is when the pre-commit hook scans the staged diff for secret-like patterns and file sizes, and when /pr-ready runs git diff --cached and git status to see exactly what's staged. Both are collecting raw facts about the change before anyone makes a judgment call.

---

**2. Which step(s) represent Analyze?**

Analyze is the pre-commit hook deciding whether any staged file matches the AKIA pattern or exceeds 1MB, and /pr-ready interpreting the staged diff to identify risks (the fake key, the debug echo) and draft a PR title and description. The hook's analysis is fixed and mechanical; the skill's analysis involves actual judgment about what the change means and why it matters.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is me actually running git commit, git push, and creating the Pull Request myself. This has to be a human action because committing and pushing changes the shared project permanently and opening a PR asks another person (or team) to review and potentially merge that change. /pr-ready is deliberately restricted from Write access and from ever running these commands itself, precisely so that no matter how confident its analysis is, a person always remains the one who decides whether to actually act on it.

---

**4. Which step is Verify?**

Verify is re-running both checks after fixing the file: git commit succeeding with no BLOCKED message, and running /pr-ready again to confirm it reports a clean status. This closes the loop proving the fix actually worked, rather than just assuming it did.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

You need both because they catch different kinds of problems: the pre-commit hook reliably blocks exact, known patterns like a hardcoded AWS key every single time with zero judgment involved, while /pr-ready can notice more contextual issues a fixed rule can't  like explaining why something is risky, or spotting a debug statement, mixed changes, or a missing explanation. Relying on only one would leave a real gap: the hook alone would miss anything that doesn't match its exact patterns, and the AI alone can't be trusted as a hard, guaranteed gate since it's advisory, not a rule.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

`https://www.linkedin.com/posts/stephen-chinedu-okam_dmibypravinmishra-git-github-share-7485824834002792449-AIXa`

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

- A fixed-rule check (the pre-commit hook) gives instant, 100% consistent answers for known patterns like a hardcoded AWS key, but has zero judgment beyond that exact pattern.
- An AI-assisted skill can catch context a fixed rule can't like explaining why something is risky, not just flagging that it matched a rule.
- Restricting a skill's `allowed-tools` (no `Write`) is a real enforcement mechanism, not just a polite instruction Claude Code physically can't write, commit, or push even if it wanted to.
- Tracking a hook inside the repo (`hooks/` + `core.hooksPath`) instead of `.git/hooks/` means the whole team gets the same protection automatically when they clone the project.
- A human must always be the one to commit, push, and open a PR AI can gather facts and give advice, but it should never be the one pulling the trigger on a shared codebase.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

`https://github.com/steveX12/devops-micro-internship-pravinmishra`

---

# Completion Checklist

- [ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ] `core.hooksPath` configured to point at `hooks/`
- [ ] Pre-commit hook shown blocking the risky commit
- [ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [ ] `/pr-ready` run against the risky diff and shown flagging issues
- [ ] Risky file fixed; `git commit` succeeds cleanly
- [ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ] LinkedIn post published and URL submitted
- [ ] All required screenshots added
- [ ] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
