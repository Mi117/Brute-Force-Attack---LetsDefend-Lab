# Brute-Force-Attack Detection---LetsDefend-Lab
Brute Force Attack Simulation Lab - LetsDefend.io

![44-bruteforce-q75](https://github.com/user-attachments/assets/a416193d-bc15-44d0-9a64-f2fa8474f5c0)

## OVERVIEW
Brute force attacks are one of the most straightforward — but still alarmingly effective — techniques in a cybercriminal’s playbook. By relentlessly attempting different username and password combinations, attackers exploit weak or default credentials to gain unauthorized access to systems. While it might sound primitive, this method still succeeds far too often — especially when combined with automation or large-scale botnets.

In this write-up, I’ll walk through the full investigative process: identifying the attacker’s IP, tracking down the brute force path, filtering out suspicious HTTP POST traffic, and isolating the successful login credentials. We’ll also examine SSH access logs and correlate failed and successful login attempts.

Using tools like Wireshark, we’ll trace the attack back to its origin, identify the targeted IP address and services, examine the intensity of the login attempts, and pinpoint which user accounts may have been breached. Then, we’ll validate our findings by analyzing the authentication logs to confirm whether the attacker successfully accessed the system — and if so, how.

This brilliant LetsDefend Lab is not only a great test of technical skills, but also a practical reminder of why securing login interfaces and monitoring for abnormal access patterns is critical in defending modern systems.

Let’s jump in and unravel the attack, one packet at a time.

------------------------

## SCENARIO
Our web server has been compromised, and it’s up to you to investigate the breach. Dive into the system, analyze logs, dissect network traffic, and uncover clues to identify the attacker and determine the extent of the damage. Are you up for the challenge?

File Location: /root/Desktop/ChallengeFile/BruteForce.7z

File Password: infected

Link to the LAB: https://app.letsdefend.io/challenge/brute-force-attacks

------------------------

## WRITE-UP
Firstly, we start by powering the VM machine and access the lab. Once inside, we need to access the folder containing the files and Unzip the target .zip folder to extract all the files.

<img width="1501" height="821" alt="0  unzip the file" src="https://github.com/user-attachments/assets/dd2c2aec-cd6e-4737-9956-e8cb61b86c9b" />

Once extracted, we now start our analysis.

### 1. What is the IP address of the server targeted by the attacker’s brute-force attack?
   
We start by analyzing the **.pcap** file heading straight to the
`Statistics > Endpoints > IPv4`

<img width="1478" height="776" alt="0 1 ip address invokved in the pcap file" src="https://github.com/user-attachments/assets/49aacec2-c440-4674-a5f8-bb9d4c30a75a" />

We see eight IP addresses listed, but only two of them are involved into a huge amount of traffic: **51.116.96.181 & 192.168.190.255.** Going for exclusion (as the 192.168.190.255 is a private IP address), the answer is **51.116.91.181**

<img width="1095" height="855" alt="1  ip address check" src="https://github.com/user-attachments/assets/44775974-01bd-4446-a5a6-1108fdb8e6d8" />

### 2. Which directory was targeted by the attacker’s brute-force attempt?

We keep analyzing the traffic on Wireshark, filtering it via the 'http' filter on the filter bar

<img width="1876" height="837" alt="2  http requests" src="https://github.com/user-attachments/assets/d5322ab9-1ffa-40ac-8a05-6be42586c10a" />

We clearly see hundreds of **HTTP:POST** requests (even more visual, with the appropriate filter — below): this specific request is used when a client (like your browser or a script) wants to **send data to a server for processing.** It’s commonly used when:

**- Submitting a login form (username & password)
- Uploading a file
- Sending data via APIs (like JSON or XML)
- Zoom image will be displayed**

<img width="1761" height="829" alt="2 1 request post http" src="https://github.com/user-attachments/assets/65b3f12d-7169-4b4c-bf7c-8d33f88f9300" />

The requests are targeting a specific directory, named **/index.php.**

### 3. Identify the correct username and password combination used for login.

We start by analyzing the traffic send from the server to the target IP address.

Each response we get from the server seems to be Incorrect , and we can see there are a multitude of them — raising our suspicions that someone (or a bot) is trying many username/password combination, repeatedly getting “Incorrect” messages.

To find the answer to this question we will leverage Wireshark’s search function to search for packets containing specific keywords, in our case, the keyword “correct”.

<img width="1903" height="844" alt="3  incorrect username" src="https://github.com/user-attachments/assets/ccf2d940-024d-4d36-a29a-e8e604c41312" />

Once found the right packet, we inspect the **HTTP Stream** (`Right Click on the packet > Follow > Http Stream `) **to review the full conversation between the client and server over the HTTP protocol**, eventually finding our answers:
- username: `web-hacker`
- password: `admin12345`

<img width="1278" height="842" alt="4 3 user name pass" src="https://github.com/user-attachments/assets/dae986a2-2594-4952-9c68-236ddbefe550" />

### 4.  How many user accounts did the attacker attempt to compromise via RDP brute-force?

Now we try to determine how many usernames the attacker tried to brute force. To do this we need to filter out the HTTP traffic, with focus on the HTTP POST requests to the web server. To do so we use the filter

`http && ip.dst==51.116.96.181`

<img width="1895" height="829" alt="5 2" src="https://github.com/user-attachments/assets/2086cb14-4a64-4716-9346-d3af08493ef4" />

By keeping analyzing the packets, we can see an huge amount of request to various usernames, and they are way too many to be checked singularly. How can we do? — Easy, we can use Wireshark option to **export the Packets** as a plain .txt document and use our trustworthy Linux to give us the answer, let’s see:

4.1) Export the Packets Dissection by clicking `File > Export Packets Dissection > as Plain Text` and name the file with a name of your choosing;

<img width="710" height="683" alt="5 3" src="https://github.com/user-attachments/assets/ddc38ba3-8481-417b-81ab-cee1daa4ed9e" />

4.2) open a Terminal in the folder you have saved the file

4.3) use the command `grep "username" (filename) | uniq`

<img width="1499" height="931" alt="4 1 username discovered" src="https://github.com/user-attachments/assets/1069ac27-ae5e-46aa-81bc-b62f35b178f4" />

This command will search for lines containing the word “**username**” and **remove any consecutive duplicate lines from the result** (command `uniq`).

### 5.  What is the “clientName” of the attacker’s machine?

To answer this question we will use once again Wireshark’s search function, specifically targeting the `clientName`.

<img width="1898" height="858" alt="6" src="https://github.com/user-attachments/assets/80dc1532-cbfe-45c3-a142-c39d6c00aecf" />

The answer is: t3m0-virtual-ma.

**RDP** stands for **Remote Desktop Protocol** and is a Microsoft-developed protocol that **allows users to remotely connect to another computer over a network**.

### 6. When did the user last successfully log in via SSH, and who was it?

We now inspect out other file, called auth.log. We use the convenient **search/find** tool to discover all the information we need: successful accesses.

_But what to look for, right?_ — Simple, for the lines in the log where the voice ‘**accepted password**’ is present, as it means the attacker has successfully logged into the target server.

<img width="1903" height="781" alt="6 2" src="https://github.com/user-attachments/assets/f0b45f9c-893e-423f-9963-6267e793e722" />

### 7. How many unsuccessful SSH connection attempts were made by the attacker?

We tackle this question using the same approach of the previous one, using the filter ‘**failed password**’.

<img width="1920" height="880" alt="7  failed password attempts" src="https://github.com/user-attachments/assets/9b80734f-0596-4bd9-9579-44b0d547074f" />

### 8. What technique is used to gain access?

Is quite clear that we are in front of a Brute Force attack, given the dynmics and the huge amount of login requests received from the server.

We head to the **Mitre Att&ck **website (_https://attack.mitre.org/_) and we use the search bar to find our ‘**Brute Force**” technique.

_From the Mitre Att&ck site:_
_Adversaries with no prior knowledge of legitimate credentials within the system or environment may guess passwords to attempt access to accounts. Without knowledge of the password for an account, an adversary may opt to systematically guess the password using a repetitive or iterative mechanism. An adversary may guess login credentials without prior knowledge of system or environment passwords during an operation by using a list of common passwords. Password guessing may or may not take into account the target’s policies on password complexity or use policies that may lock accounts out after a number of failed attempts._

_Guessing passwords can be a risky option because it could cause numerous authentication failures and account lockouts, depending on the organization’s login failure policies._

------------------------

## CONCLUSIONS

This lab served as a highly practical deep dive into the anatomy of a brute force attack — from initial enumeration all the way to credential compromise. By analyzing both network traffic and authentication logs, I was able to retrace the attacker’s steps, identify the methods used, and correlate network-level activity with system-level breaches. The integration of Wireshark with log analysis really emphasized how crucial it is to have visibility across multiple layers of a system when responding to security incidents.

One of the main technical difficulties I encountered during the lab was parsing and filtering through a large volume of HTTP POST requests efficiently. It was time-consuming to manually trace individual packets and correlate responses — especially when looking for successful login indicators. Additionally, extracting and counting unique usernames from the exported traffic required solid command-line skills and familiarity with tools like grep and uniq. It was a good reminder that knowing how to manipulate raw data quickly in a terminal is just as essential as understanding the theory.

To learn more efficiently, I’d recommend a two-pronged approach:

* Master your tools: Spend time learning advanced Wireshark filtering, log analysis techniques, and command-line utilities. These skills save a huge amount of time when navigating large datasets.
* Document your process: Writing down each step as you go helps with both clarity and retention. It also builds a personal playbook you can reuse in future challenges or real-world scenarios.

From a defensive standpoint, the lab clearly highlighted how vulnerable systems can be when basic protections are missing. To mitigate brute force threats:

**INDIVIDUALS** should:

Use strong, unique passwords and enable multi-factor authentication wherever possible.
Monitor for unauthorized access attempts on personal accounts and devices.
Regularly update passwords and avoid reusing credentials across platforms.

**ENTERPRISES** must:

- Implement account lockout policies, rate limiting, and IP blacklisting for repeated failed login attempts.
- Deploy intrusion detection/prevention systems (IDS/IPS) to flag unusual login behavior.
- Regularly audit auth logs, enforce password complexity rules, and educate users about credential security.
- Consider adding behavioral analytics or zero trust architecture to make brute-force-based intrusions
  significantly harder.

In the end, this lab was not only a technical challenge but also a realistic simulation of how a seemingly simple vector like brute force can lead to full system compromise. It reinforces that proactive defense, user education, and continuous monitoring are vital in today’s security operations.

------------------------

I hope you have found this Write-Up insightful and make sure to follow me on all the platform to stay up-to-date with my latest analysis, write-ups and be part of my journey into the world of Cybersecurity!

### Link: https://linktr.ee/atlas.protect


