---
title: "TryHackMe Advent of Cyber 2025"
date: 2025-12-03 11:30:00 +0530
categories: [WalkThrough, THM, AoC2025]
tags: [walkthrough, cybersecurity, tryhackme, aoc]
excerpt: "A walkthrough of THM's Advent of Cyber 2025"
---

![Advent of Cyber](/assets/img/posts/THM/AoC2025.png)

# Linux CLI - Shells Bells

> TryHackMe Advent of Cyber 2025 Kick-Off (Day 1)
> 

> Advent of Cyber 2025 - Day 2 (Social Engineering)
> 

## Task 1: Introduction

- Connect to the challenge machine. No additional actions required.

## Task 2: Introduction

- Which CLI command would you use to list a directory?
    
    `ls`
    
- What flag did you see inside of the McSkidy's guide?
    
    `THM{learning-linux-cli}`
    
    Commands Used: 
    
    ```bash
    cd Guides
    ls -la
    cat .guides.txt
    ```
    
- Which command helped you filter the logs for failed logins?
    
    `grep`
    
- What flag did you see inside the Eggstrike script?
    
    `THM{sir-carrotbane-attacks}`
    
    Commands Used: 
    
    ```bash
    cat /home/socmas/2025/eggstrike.sh
    ```
    
- Which command would you run to switch to the root user?
    
    `sudo su`
    
- Finally, what flag did Sir Carrotbane leave in the root bash history?
    
    `THM{until-we-meet-again}`
    
    Commands used:
    
    ```bash
    sudo su
    cd /root/
    cat .bash_history
    ```
    

# Phishing - Merry Clickmas

> Phishing for Passwords! (Advent of Cyber Day 02)
> 

> Advent of Cyber 2025 - Day 2 (Social Engineering)
> 

## Task 1: Introduction

- Start the Attack Box and VM. That’s the objective

## Task 2: Phishing Exercise for TBFC

- What is the password used to access the TBFC portal?
    
    `unranked-wisdom-anthem`
    
    Commands Used: 
    
    ```bash
    $ cd ~/Rooms/Aoc2025/Day02
    $ ./server.py
    ```
    
    ```bash
    $ setoolkit
    > 1
    > 5
    > 1
    > factory@wareville.thm
    > 2
    > updates@flyingdeer.thm
    > Flying Deer
    > [ENTER] #username
    > [ENTER] #password
    > [MACHINE_IP]
    > [ENTER] #port-number
    > no
    > n
    > n
    > Shipping Schedule Changes
    > [ENTER] 
    > Dear elves, 
    : Kindly note that there have been significant changes to the shipping schedules due to increased shipping orders.
    : Please confirm the new schedule by visiting [http://CONNECTION_IP:8000]
    : Best regards,
    : Flying Deer
    : END
    ```
    
- Browse to `http://MACHINE_IP` from within the AttackBox and try to access the mailbox of the `factory` user to see if the previously harvested `admin` password has been reused on the email portal. What is the total number of toys expected for delivery?
    
    `1984000`
    
    Actions Used: 
    
    ```
    (open mozilla firefox and search [http://MACHINE_IP]).
    Login with the Username and Password.
    Open the previous mail and search for the answer.
    ```