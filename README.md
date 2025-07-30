# Brute-Force-Attack Detection---LetsDefend-Lab
Brute Force Attack Simulation Lab - LetsDefend.io

![44-bruteforce-q75](https://github.com/user-attachments/assets/a416193d-bc15-44d0-9a64-f2fa8474f5c0)

## OVERVIEW
Brute force attacks are one of the most straightforward — but still alarmingly effective — techniques in a cybercriminal’s playbook. By relentlessly attempting different username and password combinations, attackers exploit weak or default credentials to gain unauthorized access to systems. While it might sound primitive, this method still succeeds far too often — especially when combined with automation or large-scale botnets.

In this write-up, I’ll walk through the full investigative process: identifying the attacker’s IP, tracking down the brute force path, filtering out suspicious HTTP POST traffic, and isolating the successful login credentials. We’ll also examine SSH access logs and correlate failed and successful login attempts.

Using tools like Wireshark, we’ll trace the attack back to its origin, identify the targeted IP address and services, examine the intensity of the login attempts, and pinpoint which user accounts may have been breached. Then, we’ll validate our findings by analyzing the authentication logs to confirm whether the attacker successfully accessed the system — and if so, how.

This brilliant LetsDefend Lab is not only a great test of technical skills, but also a practical reminder of why securing login interfaces and monitoring for abnormal access patterns is critical in defending modern systems.

Let’s jump in and unravel the attack, one packet at a time.

## SCENARIO
Our web server has been compromised, and it’s up to you to investigate the breach. Dive into the system, analyze logs, dissect network traffic, and uncover clues to identify the attacker and determine the extent of the damage. Are you up for the challenge?

File Location: /root/Desktop/ChallengeFile/BruteForce.7z

File Password: infected

Link to the LAB: https://app.letsdefend.io/challenge/brute-force-attacks

## WRITE-UP
Firstly, we start by powering the VM machine and access the lab. Once inside, we need to access the folder containing the files and Unzip the target .zip folder to extract all the files.

<img width="1501" height="821" alt="0  unzip the file" src="https://github.com/user-attachments/assets/dd2c2aec-cd6e-4737-9956-e8cb61b86c9b" />

Once extracted, we now start our analysis.

### 1. What is the IP address of the server targeted by the attacker’s brute-force attack?
   
We start by analyzing the .pcap file heading straight to the
`Statistics > Endpoints > IPv4`

