# GTWCI-CEH-WebAppHacking-&lt;YourStudentID&gt;

GlobalTech World Computer Institute — Cybersecurity Department
Certified Ethical Hacker (CEH) Practical Examination · Web Application Hacking
Document ref: GTWCI/CYB/CEH-PRAC/WAH-01 · Version 1.0

## Candidate details

| Field | Value |
|---|---|
| Full name | Hamzat Habeeb Adeyemi |
| Student ID / cohort | 155 |
| Date issued | 3/11/2026 |
| Submission deadline | 6/11/2026 |
| GitHub repository URL | https://github.com/Moxart-x/GTWCI-CEH-WebAppHacking-155.git|

## Candidate declaration

> I confirm that this practical examination is my own work. I carried out all testing ethically and lawfully, only against the authorised, intentionally vulnerable targets in an isolated laboratory, and at no point accessed, scanned or attacked any system I do not own or am not authorised to test. I understand that any breach of these terms, or any plagiarism or fabricated evidence, will result in a mark of zero and may be reported.

**Name:** Hamzat Habeeb Adeyemi &nbsp;&nbsp; **Student ID:** 155 &nbsp;&nbsp; **Date:** 5/11/2026

## Lab environment

- **Attacker:** Kali Linux (IP: 10.10.1.6)
- **Target server:** tc-corp-server — TechCapacity vulnerable lab VM (IP: 10.10.1.246) — isolated / host-only network, no internet exposure
- **Apps on the target:** OWASP Mutillidae II (Task 1), DVWA (Task 2)
- **Tools used:** Burp Suite, hydra, crunch, nmap, Firefox

## Contents

- [`task-1-sql-injection/report.md`](task-1-sql-injection/report.md) — SQL injection
- [`task-2-traffic-analysis/report.md`](task-2-traffic-analysis/report.md) — HTTP interception & traffic analysis
- [`task-3-bruteforce/report.md`](task-3-bruteforce/report.md) — vulnerability scan & password brute-force

> Keep only the folders for the **two** tasks you are submitting (delete the third, or keep all three and note in this README which two should be marked — the exam allows attempting all three and marking your best two).

## How to read this submission

Each task has its own folder with a write-up (`report.md`) and a `screenshots/` folder. Every screenshot has a short caption explaining what it shows. Payloads and commands are listed in the order they were tried, ending with the one that succeeded.

---
Submitted for assessment — GlobalTech World Computer Institute, Cybersecurity Department.
