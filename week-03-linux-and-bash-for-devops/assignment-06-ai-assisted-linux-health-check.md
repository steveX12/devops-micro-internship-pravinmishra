# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

![Baseline health check](/week-03-linux-and-bash-for-devops/screenshots/6-baseline-health.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

![Workspace structure](/week-03-linux-and-bash-for-devops/screenshots/6-workspace-structure.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

when the output ofthe command `systemctl is-active nginx` returns `active`, it prove that the Nginx service is currently running as a background process.


---

**2. What proves that the server is listening for HTTP traffic?**

The `ss -ltn | grep ':80'` output showed two lines with `LISTEN` state on `0.0.0.0:80` and `[::]:80` — meaning something is actively bound to port 80 and ready to accept incoming connections, for both IPv4 and IPv6. Combined with the `curl -I http://localhost` request returning `HTTP/1.1 200 OK`, this confirms not just that a port is technically open, but that a real request sent to it gets a real, successful response.


---

**3. Why must you capture a healthy baseline before simulating an incident?**

Without a documented "before" state, there'd be no way to prove that anything I break later in this assignment was actually caused by my intentional incident simulation, rather than something that was already broken beforehand. It also gives me something concrete to compare against during recovery  I'll know exactly what "back to normal" looks like, because I have the exact original evidence (active status, listening ports, 200 OK response) to match against.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

![CLAUDE.md content](/week-03-linux-and-bash-for-devops/screenshots/6-claude-md.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Without project-specific rules, Claude would only have general knowledge about Linux and DevOps best practices, with no understanding of the specific boundaries I want enforced in this project like not touching live services. Writing these rules directly into `CLAUDE.md` means every time Claude works in this project folder, it automatically knows the safety constraints, without me having to repeat them in every single conversation.


---

**2. Why is the human required to execute the recovery command?**

Because AI can misread evidence, miss context a human would catch, or simply be wrong and on a live production system, an incorrect automated action could cause more damage than the original problem. Keeping a human in the loop for the actual recovery step means there's always a real person reviewing the situation and taking responsibility for the final decision, rather than letting an AI agent make changes to a live server autonomously.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The Safety Rules section explicitly states: "Claude must not assume a diagnosis without evidence. Every conclusion must be backed by specific output from the Bash script's report." This forces Claude to point to actual evidence from the triage report when explaining a problem, rather than guessing or assuming based on general knowledge alone.


---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results

![Agentic plan](/week-03-linux-and-bash-for-devops/screenshots/6-agentic-plan.png)

---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The initial inspection Claude performed running `systemctl status`, checking the listening port, `df -h`, `free -h`, and a local curl request represents the Gather phase. Claude collected raw, factual evidence about the server's real state before forming any conclusions or writing any code, which matches the first step of the Agentic Loop defined in my `CLAUDE.md`.

---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes. Claude explicitly asked permission before proceeding ending its response with "Want me to proceed to writing scripts/linux..." rather than creating the script automatically. This shows it correctly treated the read-only investigation and the file-creation step as two separate actions requiring separate approval, respecting both my direct prompt and the Safety Rules in `CLAUDE.md`.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning first means the actual checks in the script are based on real evidence about this specific server (like the fact that disk is already at 70%, which Claude flagged as worth monitoring even though it's not the primary check yet), rather than a generic, one-size-fits-all script. It also gives me a chance to review and agree with the approach before any code exists, catching a misaligned plan early rather than discovering it after writing a script I'd need to significantly rework.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

![Script top section](/week-03-linux-and-bash-for-devops/screenshots/6-triage-top.png)

---

#### Screenshot 6 — Middle section showing check functions and conditionals

![Script middle section](/week-03-linux-and-bash-for-devops/screenshots/6-triage-middle.png)

---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

![Script bottom section](/week-03-linux-and-bash-for-devops/screenshots/6-triage-bottom.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

![Syntax check and permissions](/week-03-linux-and-bash-for-devops/screenshots/6-triage-syntax-permissions.png)

---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**

The `checks` array stores the names of five functions as plain text strings: `"nginx_status"`, `"port_check"`, `"http_check"`, `"disk_check"`, and `"memory_check"`. Each name corresponds to a function defined earlier in the script that performs one specific health check.

---

**2. How does the `for` loop use that array?**

The loop (`for check in "${checks[@]}"`) goes through each item in the array one at a time, storing the current function name in the variable `check`. Then `$check` is called on its own line  since Bash allows a variable holding a function's name to actually execute that function, this single loop runs all five checks in sequence without me needing to write five separate function calls manually.

---

**3. Why are the health checks separated into functions?**

Separating each check into its own function (`nginx_status`, `port_check`, etc.) keeps the script organized and easy to read each function has one clear responsibility. It also makes the script easy to extend: adding a sixth check later would just mean writing one new function and adding its name to the `checks` array, without needing to touch the loop or the rest of the script's structure.

---

**4. What is the purpose of `$(...)` in this script?**

`$(...)` is command substitution it runs a command and captures its actual output as a value I can store in a variable. I used this throughout the script, for example `status=$(systemctl is-active nginx)` captures the real word "active" or "inactive" into the `status` variable, and `http_code=$(curl -s -o /dev/null -w "%{http_code}" http://localhost)` captures the real HTTP response code so it can be tested inside an `if` statement.

---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

Exit codes let other programs or scripts check how this script finished without having to read and parse its full text output. `exit 0` conventionally means success (HEALTHY), while non-zero codes signal a problem I used `exit 1` for FAIL and `exit 2` for WARN to distinguish between a critical failure and a milder warning. This becomes especially useful later when Claude Code runs this script as a skill, since it can check the exit code to quickly understand the overall result before even reading the detailed report.


---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

#### Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

![Healthy triage run](/week-03-linux-and-bash-for-devops/screenshots/6-triage-healthy-run.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

![Exit code](/week-03-linux-and-bash-for-devops/screenshots/6-triage-exit-code.png)

---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status returned `HEALTHY`, with all five checks passing: Nginx active, port 80 listening, a `200` HTTP response, disk usage at a safe 70%, and 573Mi of memory available.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The `[PASS] Local HTTP check returned: 200` line is the strongest evidence  it's not just confirming Nginx is running, it's confirming a real HTTP request sent to `http://localhost` actually got a successful response back, meaning the full request path from client to server is working, the same way I verified this back in Assignment 3 and 4.

---

**3. Did your script return exit code 0 or 1? Explain why.**

The script returned exit code `0`, confirmed by running `echo "Exit code: $?"` immediately after. This matches the logic I built in Task 4: since `overall_status` remained `"HEALTHY"` throughout every check (nothing triggered a FAIL or WARN condition), the script's final `if` statement correctly fell through to the `else` branch and executed `exit 0`.

---

**4. What is the difference between a warning and a failure in this script?**

A `[FAIL]` result means something is fundamentally broken Nginx not running, port 80 not listening, or the HTTP check not returning `200` any of which immediately sets `overall_status="FAIL"` and results in exit code `1`. A `[WARN]` result means something isn't broken yet, but is approaching a concerning threshold like disk usage between 80-89%, or low available memory which sets `overall_status="WARN"` (only if nothing has already failed) and results in exit code `2`. FAIL represents an active problem; WARN represents an early signal worth watching before it becomes one.


---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

![SKILL.md content](/week-03-linux-and-bash-for-devops/screenshots/6-skill-md.png)

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

![Skill run healthy](/week-03-linux-and-bash-for-devops/screenshots/6-skill-run-healthy.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**

Bash lets Claude run the triage script itself, Read lets it open the generated report file, and Grep lets it search through text if needed. Write is deliberately excluded because this skill's entire purpose is diagnosis, not modification leaving Write out means Claude is technically incapable of creating or editing any file while this skill is active, enforcing the "read-only" principle at the tool level, not just as a written instruction that could be ignored.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

This setting means Claude cannot decide on its own to run this skill automatically in the background it only runs when I explicitly type `/linux-triage` myself. This keeps a human deliberately in control of *when* a health check happens, rather than Claude potentially triggering system checks (or worse, being one step closer to triggering recovery actions) without my direct instruction.

---

**3. What part is performed by Bash, and what part is performed by Claude?**

Bash performs the actual "Gather" phase running the real commands (`systemctl is-active nginx`, `curl`, `df -h`, etc.) and producing factual, unbiased evidence in the report file. Claude performs the "Analyze" phase reading that evidence and explaining what it means in plain English, without altering or reinterpreting the raw facts themselves. The separation matters: the evidence-gathering is deterministic and repeatable, while the explanation is where Claude adds value by making the results understandable.

---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**

Without running the actual script, Claude would have no real information about my specific server it could only guess, or worse, hallucinate a plausible-sounding but false answer. By running the skill, Claude's explanation is grounded in real, current evidence from my actual system, not general assumptions. This is the core difference between using Claude as a chatbot answering from general knowledge versus using Claude as part of an agentic workflow that's actually looking at real data.

---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails

![Incident manual check](/week-03-linux-and-bash-for-devops/screenshots/6-incident-manual-check.png)

---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

![Incident skill diagnosis](/week-03-linux-and-bash-for-devops/screenshots/6-incident-skill-diagnosis.png)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name

![Incident failure report](/week-03-linux-and-bash-for-devops/screenshots/6-incident-failure-report.png)

---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**

Nginx service status, port 80 listening check, and the local HTTP check all failed. Disk usage and memory both remained in a passing/informational state, unaffected by the incident.

---

**2. What evidence supports the conclusion that Nginx is unavailable?**

`systemctl is-active nginx` returned `inactive`, `ss -ltn | grep ':80'` returned nothing (no process bound to port 80 anymore), and the local curl check returned `000`  a code meaning no HTTP response was received at all, rather than an actual error status code. All three pieces of evidence point consistently to the same root cause: Nginx is not running.

---

**3. Did Claude execute the recovery command? Why is that important?**

No. Claude explicitly stated it would not run the suggested commands, citing the project's safety rules directly, and instead presented them as recommendations for me to review and execute myself. This matters because it keeps a human accountable for any change made to a live system  Claude can be wrong, misread a situation, or a suggested fix could have unintended side effects, and a human reviewing before acting is the safeguard against that.

---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The Bash report represents the Gather phase it's raw, factual evidence collected directly from the system (service status, port state, HTTP response code), with no interpretation or opinion attached.

---

**5. Which phase is represented by Claude's explanation?**

Claude's explanation represents the Analyze phase taking that raw evidence and turning it into a clear, human-readable diagnosis: identifying the root cause, explaining how the failures relate to each other, and proposing (but not executing) a recovery path.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

![Recovery manual check](/week-03-linux-and-bash-for-devops/screenshots/6-recovery-manual-check.png)

---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

![Recovery skill verify](/week-03-linux-and-bash-for-devops/screenshots/6-recovery-skill-verify.png)

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

![Reports folder](/week-03-linux-and-bash-for-devops/screenshots/6-reports-folder.png)

---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

![Incident summary file](/week-03-linux-and-bash-for-devops/screenshots/6-incident-summary.png)

---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

I ran `sudo nginx -t` first to confirm the configuration file itself wasn't the problem, then ran `sudo systemctl start nginx` to bring the service back up following Claude's suggested recovery steps, but executing them myself as the human operator, exactly as the safety rules required.


---

**2. What evidence proves that the service recovered?**

Multiple independent pieces of evidence confirm recovery: `systemctl is-active nginx` returned `active`, `curl -I http://localhost` returned `HTTP/1.1 200 OK` with the same `Content-Length` and `Last-Modified` values as my original healthy baseline, and re-running the `/linux-triage` skill produced a fresh report showing all five checks passing again with `Overall Status: HEALTHY`.

---

**3. Why is the second triage run necessary?**

Running the triage script again after recovery provides the same kind of objective, evidence-based verification that caught the original failure  rather than just assuming the fix worked because I ran a command, the second report proves it, the same way the first report proved something was broken. It closes the loop: Gather (found the problem) - Analyze (understood the cause) - Human Act (fixed it) - Verify (proved the fix worked), with real evidence at both ends.


---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

An AI automatically restarting services without human review could mask a deeper problem instead of actually fixing it  for example, if Nginx was crashing repeatedly due to a bad configuration change, blindly restarting it would just delay the real fix while looking like a resolved incident. It could also take an action a human would have vetoed given more context (like an intentional maintenance stop being mistaken for a failure), or restart a service into an unstable state, causing more harm than leaving it stopped until a human could properly investigate.


---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

Using AI as a chatbot means asking a question and getting an answer based on general knowledge or assumptions, while using AI in this agentic workflow means the AI gathers real evidence from an actual system, reasons about that specific evidence, and still requires a human to approve and execute any action that changes the system.


---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Stephen Chinedu Okam

**Date:** 17/07/2026

---

**1. Reported Symptom**

The website hosted on this Ubuntu EC2 server became unreachable. A local HTTP check returned no response at all (curl status `000`), indicating the site was completely down rather than returning an error page.

---

**2. Evidence Collected**

Running the `/linux-triage` skill produced a report showing three failed checks: `systemctl is-active nginx` returned `inactive`, `ss -ltn | grep ':80'` showed nothing listening on port 80, and a local curl request to `http://localhost` returned `000`. Disk usage (70%) and available memory (407Mi) both remained normal, ruling out resource exhaustion as a cause.

---

**3. Most Likely Cause**

The Nginx service itself was stopped. This was the root cause behind both other failures: with Nginx not running, nothing was bound to port 80, and with nothing listening on that port, the local HTTP request had no service to respond to it.

---

**4. Human-Approved Recovery Action**

Claude suggested first verifying the Nginx configuration with `sudo nginx -t` to rule out a config-related startup failure, then starting the service with `sudo systemctl start nginx`. I reviewed this suggestion and executed both commands myself, as the human operator Claude did not run either command automatically.

---

**5. Verification**

After starting the service, I confirmed recovery independently with `systemctl is-active nginx` (returned `active`) and `curl -I http://localhost` (returned `HTTP/1.1 200 OK`, matching the original `Content-Length` and `Last-Modified` values). I then re-ran the `/linux-triage` skill, which generated a fresh report showing all five checks passing with `Overall Status: HEALTHY`, confirming the fix was successful.

---

**6. Safety Decision**

Throughout this incident, Claude Code operated strictly within the boundaries defined in `CLAUDE.md` and the skill's `SKILL.md`  it gathered evidence and explained the likely cause, but explicitly declined to execute any recovery command, citing the project's safety rules directly in its response. All state-changing actions were reviewed and performed by me.

---

**7. Agentic Loop Mapping**

Gather: the Bash script collected raw evidence via `linux-triage.sh`, both before and after the incident. Analyze: Claude read that evidence and explained the root cause without executing anything. Human Act: I personally reviewed the config, then started the Nginx service. Verify: I re-ran the triage script and independently confirmed recovery with manual commands, proving the fix worked with objective evidence rather than assumption.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://www.linkedin.com/posts/stephen-chinedu-okam_dmibypravinmishra-devops-agenticai-share-7483978323224838144-uKxR/?utm_source=share&utm_medium=member_desktop&rcm=ACoAACmCHGwBc_NVFnHVBkHhow_NJVZpnW-l1-A`

---

#### Screenshot — Published LinkedIn post

![Linkedin Post](/week-03-linux-and-bash-for-devops/screenshots/6-Linkedin-post.png)

---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

`https://github.com/steveX12/devops-micro-internship-pravinmishra`

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [ ] Incident summary contains all seven required sections
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots and the Bash report
- [ ] Skill does not have Write permission
- [ ] Skill did not execute any recovery commands
- [ ] No sensitive data exposed

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