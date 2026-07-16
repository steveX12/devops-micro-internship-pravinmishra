# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![React App Browser](/week-03-linux-and-bash-for-devops/screenshots/3-react-app-browser.png)

---

#### Screenshot 2 — Output of `ip a`

![IP Address](/week-03-linux-and-bash-for-devops/screenshots/3-ip-address.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![SS Tulpen](/week-03-linux-and-bash-for-devops/screenshots/3-ss-tulpen.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![UFW Status](/week-03-linux-and-bash-for-devops/screenshots/3-ufw-status.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

In my ss -tulpen output, I can see a line showing tcp LISTEN 0.0.0.0:80 owned by nginx, with three worker which include (("nginx",pid=11949,fd=5),("nginx",pid=11948,fd=5),("nginx",pid=11947,fd=5)) processes attached to it. The 0.0.0.0 address means Nginx is accepting connections from any IP, not just from the server itself. This matches what I see in my browser  I can reach the app from my own laptop using the public IP, which confirms Nginx really is listening for outside traffic on port 80.

---

**2. What proves SSH is active on port 22?**

The same ss -tulpen output shows tcp LISTEN 0.0.0.0:22 owned by sshd. This is the exact port my SSH command connects to from my Windows laptop. Since I'm currently connected to the server through SSH while running these commands, that's direct proof the service is active and listening.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No. Besides Nginx (80) and SSH (22), the only other listening services were systemd-resolved (DNS) and chronyd (time sync), and both of these are bound to 127.0.0.... meaning they only accept connections from the server itself, not from the outside network. I also checked ufw status and found it inactive, but my EC2 instance's AWS Security Group is handling firewall duties instead, only allowing ports 80 and 22 through. So overall, no unexpected or risky ports were open.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![Nginx Status](/week-03-linux-and-bash-for-devops/screenshots/3-nginx-status.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![Nginx Test](/week-03-linux-and-bash-for-devops/screenshots/3-nginx-t.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![SS Port 80](/week-03-linux-and-bash-for-devops/screenshots/3-ss-port80.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart in production, the server will stop responding to any web requests on port 80, which means the React app will become completely unreachable to any user, which then result in downtime. Since I confirmed Nginx is enabled in systemd, it will still try to start automatically after a reboot. But if the failure is caused by something like a broken configuration file, a restart attempt would fail immediately, and systemctl status nginx would show failed instead of active (running). This is exactly why checking nginx -t before every reload is important catching a bad config before it takes the whole site down, rather than after.

---

**2. What's your basic rollback plan?**

Firstly, if a config change breaks Nginx, I will restore the previous working config file (which is why keeping backups or using git to track config changes matters) and then i will run sudo nginx -t to confirm it's valid before reloading. Secondly, if the issue is with the deployed React build itself rather than Nginx's config, I will redeploy the last known-good build to /var/www/html/. Thirdly, systemctl status nginx --no-pager and the Nginx error logs would be my first stop to diagnose what actually went wrong before making any changes, so I'm fixing the real problem instead of guessing.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![Access Log](/week-03-linux-and-bash-for-devops/screenshots/3-access-log.png)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![Error Log](/week-03-linux-and-bash-for-devops/screenshots/3-error-log.png)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![Journalctl Nginx](/week-03-linux-and-bash-for-devops/screenshots/3-journalctl-nginx.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No real errors were found. The Nginx error log only contained a single [notice] level line about inherited sockets during a restart, this is informational, not an actual error, and is the lowest severity Nginx logs. The access log did show a number of 400 Bad Request responses, but these were from bots on the internet sending malformed or invalid requests (like TLS handshake attempts on a plain HTTP port), not real errors in my application or Nginx configuration. Nginx correctly rejected these itself.

**2. If there were no errors, what does that indicate about the system?**

An empty (or near-empty) error log indicates Nginx is stable and correctly configured it's not struggling to find files, hitting permission issues, or crashing internally. Combined with the access log showing plenty of real traffic being handled successfully (200 responses) alongside bad requests being properly rejected (400 responses), it shows the system is working exactly as it should: serving valid requests and safely rejecting invalid ones, without any internal failures.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. My curl -I http://51.20.5.189 request appeared in the access log as a HEAD / HTTP/1.1 request from IP 51.20.5.189 with a 200 response and the user-agent curl/8.18.0. This proves the full traffic flow is working end-to-end: a request I made reached the public IP, passed through to Nginx, was processed, and a response was logged confirming the request path from client to server is functioning exactly as expected, not just that the app "looks" reachable.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![Uptime](/week-03-linux-and-bash-for-devops/screenshots/3-uptime.png)

---

#### Screenshot 2 — Output of `free -h`

![Free H](/week-03-linux-and-bash-for-devops/screenshots/3-free-h.png)

---

#### Screenshot 3 — Output of `df -h`

![DF H](/week-03-linux-and-bash-for-devops/screenshots/3-df-h.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![DU Var](/week-03-linux-and-bash-for-devops/screenshots/3-du-var.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

Disk is the resource I'd watch most closely. CPU load average was 0.00 across all three time windows its essentially idle. Memory showed 569Mi which is genuinely available out of 908Mi total, with zero swap usage, meaning RAM has never even been stressed. Disk, however, is already at 60% used (4.0G out of 6.7G) on a fairly small root volume. Breaking it down with du -sh /var/*, most of that usage comes from normal OS and package data (/var/lib, /var/cache), not from my actual app (/var/www is only 1.2M) or logs (19M). It's not an emergency today, but unlike CPU and memory which have wide headroom, disk is the one resource that only grows over time and is worth checking periodically going forward.

---

**2. What happens if disk becomes 100% full in a production server?**

When a disk hits 100%, the server can't write any new data, Nginx would be unable to write new log entries, which can actually cause it to stop serving requests to users entirely. Any application trying to save files, write temporary data, or even system processes trying to write normal operational data could start failing. In the worst case, the server can become unstable or partially unresponsive.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![LS Var WWW](/week-03-linux-and-bash-for-devops/screenshots/3-ls-var-www.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![Grep Deployed By](/week-03-linux-and-bash-for-devops/screenshots/3-grep-deployed-by.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![Grep Try Files](/week-03-linux-and-bash-for-devops/screenshots/3-grep-try-files.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

Firstly, I checked /var/www/html and saw all deployed files shared the exact same timestamp, showing they came from one clean, complete build rather than a partial or stale deployment. Secondly, I searched the deployed JavaScript bundle for my name (which I added to App.js as personalization) using grep -R -o -i "okam", and found it inside main.e66e3d00.jswhich proves my specific code changes made it all the way from source code into the actual files being served, not just sitting unused in GitHub. Finally, I confirmed the Nginx config has the correct try_files $uri $uri/ /index.html; rule active, which is required specifically for React's client-side routing to work correctly confirming the server isn't just serving a set of files, but is correctly configured for this type of application.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![Nginx T Fail](/week-03-linux-and-bash-for-devops/screenshots/3-nginx-t-fail.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![Nginx T Pass](/week-03-linux-and-bash-for-devops/screenshots/3-nginx-t-pass.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Curl Recovery](/week-03-linux-and-bash-for-devops/screenshots/3-curl-recovery.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

I removed a semicolon from the end of the try_files $uri $uri/ /index.html; line in the Nginx config. Every directive in an Nginx config file must end with a semicolon, and removing it broke the parser's ability to read the file correctly.

---

**2. How did you fix the issue?**

I reopened the config file with sudo nano /etc/nginx/sites-available/default and restored the missing semicolon. I then ran sudo nginx -t to confirm the syntax was valid again before applying anything live. Once the test passed, I ran sudo systemctl reload nginx to apply the fixed configuration to the running service, and confirmed recovery with curl -I, which returned a 200 OK response.

---

**3. How can you avoid this kind of issue in real production systems?**

I will avoid it by always running sudo nginx -t after any config change, before reloading or restarting the service this catches syntax errors before they can cause downtime. I will also form the habit of backing up the config file before editing it (which I did with cp), so a broken change can be reverted instantly if needed.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![Curl Fail](/week-03-linux-and-bash-for-devops/screenshots/3-curl-fail.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![Curl Recovery Task 7](/week-03-linux-and-bash-for-devops/screenshots/3-curl-recovery-task7.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

I simulated a broken deployment by renaming index.html in /var/www/html, removing the main entry file the React app depends on. When I tested the site with curl -I, it returned 403 Forbidden rather than the expected 200 OK. This happened because Nginx's try_files rule fell back to checking if /var/www/html/ was a valid directory (which it was), but since directory listing is disabled for security, Nginx refused to display its contents resulting in a 403 instead of a straightforward "file not found" error.

---

**2. How did you fix the issue and restore the application?**

I restored the file by moving it back to its original name and location with sudo mv /var/www/html/index.html.bak /var/www/html/index.html. I then verified the fix with curl -I, which returned 200 OK again, along with the same Content-Length and Last-Modified timestamp as before confirming the exact original file was restored, not a different or corrupted version.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

firstly, deployments should be automated and scripted rather than manually moving files around, which removes the chance of a human accidentally deleting or renaming a critical file. Secondly, keeping the app's source code and build process in version control (git) means a broken deployment can be quickly redeployed from a known-good build rather than manually reconstructing files. Third, basic monitoring or uptime checks on the live site would catch this kind of failure within minutes rather than relying on someone noticing manually, which matters a lot for minimizing real downtime.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

Passwords can be guessed, reused across services, or leaked through phishing, and they're vulnerable to brute-force attacks, automated bots trying thousands of password combinations. In my own access logs from Task 3, I saw exactly this kind of automated scanning happening against my server already, even without SSH being targeted directly. Key-based authentication uses a cryptographic key pair instead, a private key that stays on my own machine and never gets transmitted, and a public key on the server that can only be unlocked by its matching private key. This makes brute-forcing essentially impossible, since there's no password to guess at all, an attacker would need to physically steal my private key file, not just guess a string of characters.

---

**2. Why should only required ports be open on a production server?**

Every open port is a potential entry point for an attacker. In Task 1, I confirmed only two ports were actually open and listening for outside connections: 80 (Nginx, for the web app) and 22 (SSH, for my own access), everything else was bound only to localhost and unreachable from outside. If unnecessary ports were open, each one would be another opportunity for someone to find a vulnerable service running on it. Minimizing open ports reduces what's called the "attack surface", the smaller the number of ways in, the smaller the chance something can be exploited.

---

**3. Why is it important for Nginx to be enabled on boot?**

In Task 2, I confirmed Nginx was set to enabled in systemd, meaning it automatically starts if the server reboots, whether from a planned maintenance restart, an AWS-level instance stop/start, or an unexpected crash. Without this, a reboot would cause a real outage until someone manually logged in and started the service again. For a production system, that delay directly translates to downtime and lost availability, so having critical services start automatically is essential for reliability.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

If something like my .pem SSH key file, AWS credentials, or API keys were accidentally exposed for example, pushed to a public GitHub repo or shown in a screenshot anyone who found it could gain the same level of access I have. This could mean someone taking full control of my EC2 instance, deleting or modifying my deployed app, using my AWS account to run up costs on resources they spin up, or using my server as a launching point for further attacks. This is exactly why I've been careful throughout this assignment to avoid exposing sensitive information like account IDs or keys in my screenshots.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Cloud resources like EC2 instances typically cost money for every hour they run, whether or not they're actually being used. Leaving unused resources running unnecessarily increases costs for no benefit. There's also a security angle an idle, forgotten server is still a live target on the internet (as I saw firsthand in Task 3's access logs, full of scanning bots), and one that's no longer being actively monitored or maintained is more likely to become a security risk over time if left unattended indefinitely.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

``

---

#### Screenshot — Published LinkedIn post

![LinkedIn Post](/week-03-linux-and-bash-for-devops/screenshots/3-LinkedIn-Post.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
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