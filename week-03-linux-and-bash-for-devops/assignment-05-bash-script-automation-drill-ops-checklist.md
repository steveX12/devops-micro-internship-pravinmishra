# Assignment 5 — Bash Script Automation Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will practice Bash scripting by building a series of small automation scripts covering environment setup, variables, arrays, loops, file conditionals, if-else logic, and functions. These scripts form the foundation of real-world Linux automation used in DevOps, cloud, and production support environments.

---

# Task 1 — Bash Environment & Workspace Setup

## Goal

Verify that Bash is available on your system and create a clean workspace for this assignment.

### Evidence

#### Screenshot 1 — Output of `echo $SHELL` and `bash --version`

![Bash version check](/week-03-linux-and-bash-for-devops/screenshots/5-bash-version.png)

---

#### Screenshot 2 — Output of `pwd` and `ls -lah` showing the scripts directory

![Workspace setup](/week-03-linux-and-bash-for-devops/screenshots/5-workspace-setup.png)

---

### Notes

Answer the following in your own words:

**1. What is Bash?**

Bash which means (Bourne Again Shell) is a command-line interpreter and scripting language used on Linux and Unix-based systems. It's the program that reads the commands I type, interprets them, and tells the operating system what to do. Beyond running one-off commands interactively, Bash can also read a saved file full of commands (a script) and run them all in sequence 

---

**2. What is the difference between shell and Bash?**

"Shell" is a general term for any program that provides a command-line interface between the user and the operating system it's a category, not one specific program. Bash is one specific implementation of a shell it is the most common one on Linux systems. 

---

**3. Why is it important to confirm the Bash version before writing scripts?**

it is important to confirm the bash version before writing script because different Bash versions support different features some newer syntax won't work correctly, or at all, on older versions. Confirming the version before writing helps you avoids writing a script that works perfectly on one machine but silently fails or behaves unexpectedly on a different server with an older Bash version.

---

# Task 2 — Your First Bash Script

## Goal

Create your first Bash script, make it executable, and run it from the terminal.

### Evidence

#### Screenshot 1 — Content of `first-script.sh`

![first-script.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-first-script-content.png)

---

#### Screenshot 2 — Output of `./first-script.sh`

![first-script.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-first-script-output.png)

---

#### Screenshot 3 — Output of `ls -l first-script.sh` showing executable permission

![first-script.sh permissions](/week-03-linux-and-bash-for-devops/screenshots/5-first-script-permissions.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of `#!/bin/bash`?**

#!/bin/bash is popularly known as "shebang," it tells the operating system exactly which interpreter should be used to run the script. Since a Linux system might have multiple shells installed, this removes any ambiguity it explicitly says "run this file using Bash," not some other shell that might interpret certain commands differently.

---

**2. Why do we use `chmod +x` before running a script?**

we use "chmod +x" before running a script in other to give permission to user of the script, chmod +x explicitly grants execute permission

---

**3. What is the difference between running a script using `./script.sh` and `bash script.sh`?**

./script.sh runs the file directly as an executable program, which requires the file to have execute permission first (via chmod +x) and relies on the shebang line to know which interpreter to use. bash script.sh instead explicitly tells Bash to read and execute the file's contents

---

# Task 3 — Variables: User Information Script

## Goal

Use variables to store and display user-related information.

### Evidence

#### Screenshot 1 — Content of `user-info.sh`

![user-info.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-user-info-content.png)

---

#### Screenshot 2 — Output of `./user-info.sh`

![user-info.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-user-info-output.png)

---

### Notes

Answer the following in your own words:

**1. What is a variable in Bash?**

A variable is a named container that stores a piece of data like a name, a number, or a file path so it can be reused throughout a script without retyping the actual value every time. In my script, name, role, group, and location are all variables holding text values that I reference later using $name, $role, and so on.

---

**2. Why should we avoid spaces around the `=` sign when creating variables?**

Because Bash treats spaces as separators between commands and their arguments. If I write name = "Stephen" with spaces, Bash doesn't interpret this as "assign this value to the variable name" it tries to interpret name as a command, = as another separate argument, and so on, which causes an error.

---

**3. How do you access the value stored inside a Bash variable?**

By putting a $ symbol before the variable's name, like $name. This tells Bash to substitute the variable's actual stored value in that spot, rather than treating "name" as literal text. 

---

# Task 4 — Arrays & Loops: Tools Checklist Script

## Goal

Use arrays and loops to print a checklist of tools used in Bash scripting.

### Evidence

#### Screenshot 1 — Content of `tools-checklist.sh`

![tools-checklist.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-tools-checklist-content.png)

---

#### Screenshot 2 — Output of `./tools-checklist.sh`

![tools-checklist.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-tools-checklist-output.png)

---

### Notes

Answer the following in your own words:

**1. What is an array in Bash?**

An array is a single variable that can hold multiple values at once, instead of just one. In my script, tools is one array variable containing six separate tool names, rather than needing six separate variables like tool1, tool2, and so on.

---

**2. Why are arrays useful in scripts?**

Arrays let you group related data together and process it efficiently, especially when combined with loops. Instead of writing a separate echo line for every single tool, I stored all six tools in one array and let a loop handle printing each one automatically. This makes scripts shorter, easier to update (adding a new tool just means adding one item to the array), and less repetitive.

---

**3. What does `"${tools[@]}"` mean?**

This syntax means "expand every element in the tools array, treating each one as a separate, complete item." The @ specifically tells Bash to keep multi-word items (like "AWS EC2") intact as a single element, rather than accidentally splitting them into separate words. This is why "AWS EC2" printed correctly as one line, not two.

---

**4. What is the purpose of the `for` loop in this script?**

The for loop automatically goes through every item in the tools array, one at a time, and runs the same block of code (printing a checkmark and the tool's name) for each one. This avoids manually writing six separate echo statements the loop handles repeating the action for however many tools are in the array, even if I added more later.

---

# Task 5 — Loops: Number Counter Script

## Goal

Use loops to repeat a task multiple times.

### Evidence

#### Screenshot 1 — Content of `counter.sh`

![counter.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-counter-content.png)

---

#### Screenshot 2 — Output of `./counter.sh`

![counter.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-counter-output.png)

---

### Notes

Answer the following in your own words:

**1. What is a loop?**

A loop is a way to repeat a block of code multiple times automatically, instead of writing the same instruction over and over manually. In this script, instead of writing five separate echo statements for numbers 1 through 5, the loop runs the same echo "Count: $i" line five times, with $i changing value each time.

---

**2. Why do we use loops in Bash scripting?**

Loops save time and reduce repetition, especially as the number of repeated actions grows. Writing five echo lines by hand is manageable, but if a script needed to count to 1000, or process 500 files, manually writing that out wouldn't be practical. 

---

**3. How many times did the loop run in your script?**

The loop ran 5 times, printing "Count: 1" through "Count: 5," matching the five numbers listed in for i in 1 2 3 4 5.

---

**4. What would you change if you wanted the loop to run 10 times?**

I would change the list of numbers to include all ten values: for i in 1 2 3 4 5 6 7 8 9 10. Alternatively, Bash also supports a range shorthand, for i in {1..10}, which achieves the same result without manually typing every number useful for larger ranges.

---

# Task 6 — Files & Conditionals: File Validation Script

## Goal

Use file checks and conditionals to verify whether files and directories exist.

### Evidence

#### Screenshot 1 — Output of `ls -lah ../test-folder`

![test-folder listing](/week-03-linux-and-bash-for-devops/screenshots/5-test-folder.png)

---

#### Screenshot 2 — Content of `file-check.sh`

![file-check.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-file-check-content.png)

---

#### Screenshot 3 — Output of `./file-check.sh`

![file-check.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-file-check-output.png)

---

### Notes

Answer the following in your own words:

**1. What does `-d` check in Bash?**

-d checks whether a given path exists and is specifically a directory (a folder), not a file. In my script, [ -d "$folder" ] confirmed that /home/ubuntu/test-folder exists as an actual directory.

---

**2. What does `-f` check in Bash?**

-f checks whether a given path exists and is specifically a regular file, not a directory. I used this twice once to confirm sample.txt genuinely exists, and once to correctly detect that does-not-exist.txt does not.

---

**3. Why should file and directory paths be stored in variables?**

Storing paths in variables like folder, file, and missing_file means the actual path is only written once, at the top of the script. If the path ever needed to change, I'd only update it in one place instead of hunting through the entire script for every place it's used. It also makes the script more readable

---

**4. What happens if the file does not exist?**

The -f check returns false, so the script's if condition fails and it runs the else block instead in my case, printing x File not found: [path]. This is exactly what happened with does-not-exist.txt.

---

# Task 7 — Conditionals: Pass or Retry Script

## Goal

Use if-else conditionals to make decisions based on a variable value.

### Evidence

#### Screenshot 1 — Content of `score-check.sh` with `score=85`

![score-check.sh content score 85](/week-03-linux-and-bash-for-devops/screenshots/5-score-check-85-content.png)

---

#### Screenshot 2 — Output showing `Result: Pass`

![score-check.sh output Pass](/week-03-linux-and-bash-for-devops/screenshots/5-score-check-85-output.png)

---

#### Screenshot 3 — Content of `score-check.sh` with `score=55`

![score-check.sh content score 55](/week-03-linux-and-bash-for-devops/screenshots/5-score-check-55-content.png)

---

#### Screenshot 4 — Output showing `Result: Retry`

![score-check.sh output Retry](/week-03-linux-and-bash-for-devops/screenshots/5-score-check-55-output.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of if-else in Bash?**

If-else lets a script make a decision and take a different action depending on whether a condition is true or false, instead of always running the same fixed set of instructions. In my script, the same code checks the score, but produces a different result "Pass" or "Retry" depending on the actual value, without me needing to write two separate scripts.

---

**2. What does `-ge` mean?**

-ge stands for "greater than or equal to." It's used specifically for comparing numbers inside Bash's [ ] conditional syntax. [ "$score" -ge 60 ] checks whether the value stored in score is 60 or higher.

---

**3. Why should conditions be tested with different values?**

Testing only one value (like just 85) only proves the "Pass" path works it doesn't prove the "Retry" path works correctly, or that the condition's boundary (exactly 60) behaves as expected. Testing with both 85 and 55 confirms the script correctly handles both outcomes, not just the one I happened to try first. 

---

**4. How can conditionals help in automation scripts?**

Conditionals let scripts respond intelligently to real, changing conditions instead of blindly running the same steps every time. For example, a real deployment script could check if a service is healthy before continuing, or check if disk space is critically low before proceeding with an installation reacting appropriately either way, rather than assuming everything is always fine.

---

# Task 8 — Functions: Final Bash Automation Script

## Goal

Create a final Bash script using functions to organize reusable code.

### Evidence

#### Screenshot 1 — Content of `final-automation.sh`

![final-automation.sh content](/week-03-linux-and-bash-for-devops/screenshots/5-final-automation-content.png)

---

#### Screenshot 2 — Output of `./final-automation.sh`

![final-automation.sh output](/week-03-linux-and-bash-for-devops/screenshots/5-final-automation-output.png)


---

#### Screenshot 3 — Output of `ls -lah` showing all created scripts

![All scripts listing](/week-03-linux-and-bash-for-devops/screenshots/5-all-scripts.png)

---

### Notes

Answer the following in your own words:

**1. What is a function in Bash?**

A function is a named, reusable block of code that can be defined once and then run ("called") multiple times by simply writing its name, instead of rewriting the same code repeatedly. In my final script, show_user_info, list_tools, check_score, and check_file are each functions the actual logic only appears once, and I trigger each one by name at the bottom of the script.

---

**2. Why are functions useful in scripts?**

Functions keep scripts organized and easier to read instead of one long, unbroken block of code, the script is broken into clearly labeled, self-contained pieces, each responsible for one specific task. They also avoid repeating code: if I needed to check the score again somewhere else in the script, I could just call check_score again instead of retyping the whole conditional block.

---

**3. Which functions did you create in this script?**

I created four functions: show_user_info (prints my name and group using variables), list_tools (loops through the tools array and prints each one), check_score (runs the pass/retry conditional from Task 7), and check_file (runs the file existence check from Task 6).

---

**4. How does this final script combine variables, arrays, loops, conditionals, files, and functions?**

Each concept from this assignment became one building block inside this script: variables (name, group, score) store core data; the tools array holds a list of related values; a for loop iterates through that array inside list_tools; an if-else conditional evaluates the score inside check_score; a file existence check (-f) confirms a real file inside check_file; and functions wrap each of these pieces into named, reusable blocks that get called in a clear, deliberate order at the bottom of the script. Individually, each concept was simple but combined through functions, they form one organized, working automation script.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`https://www.linkedin.com/posts/stephen-chinedu-okam_dmibypravinmishra-devops-agenticai-activity-7483677383749885952-wMlq?utm_source=share&utm_medium=member_desktop&rcm=ACoAACmCHGwBc_NVFnHVBkHhow_NJVZpnW-l1-A`

---

#### Screenshot — Published LinkedIn post

![LinkedIn post screenshoot](/week-03-linux-and-bash-for-devops/screenshots/5-Linkedin-Post.png)

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- All script files must be created and run successfully
- Required notes must be answered clearly for every task
- Do not expose sensitive information (keys, passwords, credentials)

---

# Completion Checklist

- [ ] Task 1: Environment setup verified, workspace created (Screenshots 1–2, Notes answered)
- [ ] Task 2: First script created, executed, permissions verified (Screenshots 1–3, Notes answered)
- [ ] Task 3: Variables script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 4: Arrays and loops script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 5: Counter loop script created and run (Screenshots 1–2, Notes answered)
- [ ] Task 6: File validation script created and run (Screenshots 1–3, Notes answered)
- [ ] Task 7: Pass/Retry conditional script tested with both values (Screenshots 1–4, Notes answered)
- [ ] Task 8: Final automation script created and run (Screenshots 1–3, Notes answered)
- [ ] All scripts run without errors
- [ ] Full Name visible in all required screenshots
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