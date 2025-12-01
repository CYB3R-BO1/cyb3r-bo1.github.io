---
title: "TryHackMe Advent of Cyber 2025 - 1"
date: 2025-12-01 23:00:00 +0530
categories: [WalkThrough, THM, AoC2025]
tags: [walkthrough, cybersecurity, tryhackme, aoc]
excerpt: "A walkthrough of THM's Advent of Cyber 2025"
---

![Advent of Cyber](/assets/img/posts/THM/AoC2025.png)

#  Linux CLI - Shells Bells

>TryHackMe Advent of Cyber 2025 Kick-Off (Day 1)

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