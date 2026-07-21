# Assignment 2 — CodeTrack: Tracking, Staging, Committing + Deploy to EC2

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will track and stage project files, create two meaningful Git commits in `CodeTrack`, verify your commit history, and deploy the CodeTrack static website to an EC2 instance using Nginx. This connects local version-control practice with a basic manual deployment workflow used in real DevOps environments.

---

# Task 1 — Verify Git Setup and Enter the Repository

## Goal

Confirm that Git works and that you are inside the correct `CodeTrack` repository.

### Evidence

#### Screenshot 1 — Output of `pwd` showing you're inside `CodeTrack`

![Output of pwd](/week-04-git-and-github/screenshots/02-screenshot-01-pwd-codetrack.png)

---

#### Screenshot 2 — Output of `git status` showing no "not a git repository" error

![Output of git status](/week-04-git-and-github/screenshots/02-screenshot-02-git-status-verified.png)

---

# Task 2 — Create index.html and style.css

## Goal

Create the two starter UI files inside `CodeTrack`.

### Evidence

#### Screenshot 3 — Output of `ls` showing `index.html` and `style.css`

![output of ls](/week-04-git-and-github/screenshots/02-screenshot-03-files-created.png)

---

# Task 3 — Add Starter Content

## Goal

Copy the provided starter HTML and CSS content into your local `index.html` and `style.css` files.

### Evidence

#### Screenshot 4 — Your editor showing the contents of `index.html` and `style.css`

![editor content](/week-04-git-and-github/screenshots/02-screenshot-04-editor-contents.png)

---

# Task 4 — Track and Stage Files Correctly

## Goal

Confirm both files show as untracked, then stage them individually with `git add`.

### Evidence

#### Screenshot 5 — Output of `git status` showing both files as untracked

![git status untracked](/week-04-git-and-github/screenshots/02-screenshot-05-git-status-untracked.png)

---

#### Screenshot 6 — Output of `git status` showing both files staged under "Changes to be committed"

![git status staged](/week-04-git-and-github/screenshots/02-screenshot-06-git-status-staged.png)

---

# Task 5 — Create the First Commit (Clean Initial Commit)

## Goal

Commit the staged starter files using the message `Initial UI scaffold: add index.html and style.css`, then check the log.

### Evidence

#### Screenshot 7 — Output of `git commit`

![first commit](/week-04-git-and-github/screenshots/02-screenshot-07-first-commit.png)

---

#### Screenshot 8 — Output of `git log --oneline` showing the first commit

![git log first commit](/week-04-git-and-github/screenshots/02-screenshot-08-git-log-first-commit.png)

---

# Task 6 — Modify index.html and Create a Second Commit

## Goal

Follow the instruction comment inside `index.html` to update the Student Name and Group Name, then commit that change separately using the message `Update homepage content: heading, tagline, CTA button`.

### Evidence

#### Screenshot 9 — Browser showing the updated page with your Student Name and Group Name visible

![Browser updated](/week-04-git-and-github/screenshots/02-screenshot-09-browser-updated-page.png)

---

#### Screenshot 10 — Output of `git status` showing `index.html` as modified

![git status modified](/week-04-git-and-github/screenshots/02-screenshot-10-git-status-modified.png)

---

#### Screenshot 11 — Output of `git commit`

![second commit](/week-04-git-and-github/screenshots/02-screenshot-11-second-commit.png)

---

#### Screenshot 12 — Output of `git log --oneline` showing two commits

![git log two commit](/week-04-git-and-github/screenshots/02-screenshot-12-git-log-two-commits.png)

---

# Task 7 — Deploy to EC2 with Nginx (Static Website)

## Goal

Install and start Nginx on your EC2 instance, then copy `index.html` and `style.css` into the Nginx web root.

### Evidence

#### Screenshot 13 — Output of `systemctl status nginx --no-pager` showing Nginx `active (running)`

![nginx status](/week-04-git-and-github/screenshots/02-screenshot-13-nginx-status.png)

---

#### Screenshot 14 — Output of `curl -I http://localhost` showing `HTTP/1.1 200 OK`

![curl check](/week-04-git-and-github/screenshots/02-screenshot-14-curl-check.png)

---

#### Screenshot 15 — Browser showing the CodeTrack site loaded at `http://<EC2_PUBLIC_IP>`, with your Full Name and Group Name visible

![browser live site](/week-04-git-and-github/screenshots/02-screenshot-15-browser-live-site.png)

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://www.linkedin.com/posts/stephen-chinedu-okam_dmibypravinmishra-devops-agenticai-activity-7485468602209402880-PNrR?utm_source=share&utm_medium=member_desktop&rcm=ACoAACmCHGwBc_NVFnHVBkHhow_NJVZpnW-l1-A`

---

#### Screenshot — LinkedIn post showing the deployed CodeTrack application

![Linkedin post](/week-04-git-and-github/screenshots/02-Linkedin-post.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name and Group Name must be visible in the deployed application evidence
- `git log --oneline` output must show at least two meaningful commits
- Do not expose AWS access keys, passwords, private key contents, or other sensitive information

---

# Completion Checklist

- [ ] `CodeTrack` repository verified with `git status` (Screenshots 1–2)
- [ ] `index.html` and `style.css` created and populated (Screenshots 3–4)
- [ ] Starter files staged and committed in the first commit (Screenshots 5–8)
- [ ] Student Name and Group Name updated in `index.html` (Screenshot 9)
- [ ] Second controlled commit created (Screenshots 10–12)
- [ ] Nginx active on the EC2 instance and CodeTrack reachable via its public IP (Screenshots 13–15)
- [ ] LinkedIn post published and URL submitted
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
