---
title: picoGym Reverse Engineering - Easy writeups
date: 2025-05-24 19:00:00 +0530
categories: [CTF, PicoCTF]
tags: [ctf, writeups, cybersecurity, reverse-engineering]     # TAG names should always be lowercase
description: This post consists of writeups of the challenges CYB3R_BO1 had solved in PicoCTF practice.
---

# **picoCTF**

![picoCTF](/assets/img/posts/picoGym-Forensics/picoCTF.jpg)

## **About picoCTF:** 

picoCTF gamifies learning hacking with capture-the-flag puzzles created by trusted computer security and privacy experts at Carnegie Mellon University.

Using picoCTF practice platform, participants learn to overcome sets of challenges from six domains of cybersecurity including general skills, cryptography, web exploitation, forensics, etc. The challenges are all set up with the intent of being hacked, making it an excellent, legal way to get hands-on experience.

picoGym is a noncompetitive practice space where you can explore and solve challenges from previously released picoCTF competitions, find fresh never before revealed challenges, and build a knowledge base of cybersecurity skills in a safe environment.

Whether you are a cybersecurity professional, competitive hacker or new to CTFs you will find interesting challenges in the picoGym that you can solve at your own pace. Team picoCTF will regularly update this challenge repository so visit the picoGym often.

This is their Official webpage - [https://picoctf.org/](https://picoctf.org/)

This is their url for picoGym - [https://play.picoctf.org/practice](https://play.picoctf.org/practice)

## **Easy**

### **vault-door-training**

![vault-door-training](/assets/img/posts/picoGym-rev/vault-door-training.png)

**Explanation:**

So they have given us a java file - VaultDoorTraining.java with source code:

```java
import java.util.*;

class VaultDoorTraining {
    public static void main(String args[]) {
        VaultDoorTraining vaultDoor = new VaultDoorTraining();
        Scanner scanner = new Scanner(System.in); 
        System.out.print("Enter vault password: ");
        String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
	}
   }

    // The password is below. Is it safe to put the password in the source code?
    // What if somebody stole our source code? Then they would know what our
    // password is. Hmm... I will think of some ways to improve the security
    // on the other doors.
    //
    // -Minion #9567
    public boolean checkPassword(String password) {
        return password.equals("w4rm1ng_Up_w1tH_jAv4_eec0716b713");
    }
}
```

From the given source code we can understand the working mechansim. The main function is called, then Prints the string "Enter the vault password", then asks for input from the user. After taking the input, it slices the input to collect a substring. Checks if the substring is valid or not using the checkPassword method. From the source code we can get the flag. To check if it is valid or not, use the below command to run the java file.

```bash
javac VaultDoorTraining.java
```
```bash
java VaultDoorTraining
```

When prompted for password, enter the complete string, which is - picoCTf{ + w4rm1ng_Up_w1tH_jAv4_eec0716b713 + }, the access will be granted

I got the flag!

**Flag:** `picoCTF{w4rm1ng_Up_w1tH_jAv4_xxxxxxxxxx}`

### 