---
title: picoGym Reverse Engineering - Hard writeups
date: 2025-07-3 11:00:00 +0530
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

## **Medium**

### **vault-door-8**

![vault-door-8](/assets/img/posts/picoGym-rev/vault-door-8.png)

**Hint:** 
- Clean up the source code so that you can read it and understand what is going on.
- Draw a diagram to illustrate which bits are being switched in the scramble() method, then figure out a sequence of bit switches to undo it. You should be able to reuse the switchBits() method as is.

**Explanation:**

We are given a java file - _VaultDoor8.java_ and here are it's contents:

```java
// These pesky special agents keep reverse engineering our source code and then
// breaking into our secret vaults. THIS will teach those sneaky sneaks a
// lesson.
//
// -Minion #0891
import java.util.*; import javax.crypto.Cipher; import javax.crypto.spec.SecretKeySpec;
import java.security.*; class VaultDoor8 {public static void main(String args[]) {
Scanner b = new Scanner(System.in); 
System.out.print("Enter vault password: ");
String c = b.next(); String f = c.substring(8,c.length()-1); 
VaultDoor8 a = new VaultDoor8(); 
if (a.checkPassword(f)) {
    System.out.println("Access granted."); 
} else {System.out.println("Access denied!"); } } public char[] scramble(String password) {/* Scramble a password by transposing pairs of bits. */
char[] a = password.toCharArray(); for (int b=0; b<a.length; b++) {char c = a[b]; c = switchBits(c,1,2); c = switchBits(c,0,3); /* c = switchBits(c,14,3); c = switchBits(c, 2, 0); */ c = switchBits(c,5,6); c = switchBits(c,4,7);
c = switchBits(c,0,1); /* d = switchBits(d, 4, 5); e = switchBits(e, 5, 6); */ c = switchBits(c,3,4); c = switchBits(c,2,5); c = switchBits(c,6,7); a[b] = c; } return a;
} public char switchBits(char c, int p1, int p2) {/* Move the bit in position p1 to position p2, and move the bit
that was in position p2 to position p1. Precondition: p1 < p2 */ char mask1 = (char)(1 << p1);
char mask2 = (char)(1 << p2); /* char mask3 = (char)(1<<p1<<p2); mask1++; mask1--; */ char bit1 = (char)(c & mask1); char bit2 = (char)(c & mask2); /* System.out.println("bit1 " + Integer.toBinaryString(bit1));
System.out.println("bit2 " + Integer.toBinaryString(bit2)); */ char rest = (char)(c & ~(mask1 | mask2)); char shift = (char)(p2 - p1); char result = (char)((bit1<<shift) | (bit2>>shift) | rest); return result;
} public boolean checkPassword(String password) {char[] scrambled = scramble(password); char[] expected = {
0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0, 0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xE0, 0x95, 0xF1, 0xE1, 0xE0, 0xA4, 0xC0, 0x94, 0xA4 }; return Arrays.equals(scrambled, expected); } }
```

Looks like the code isn't arranged properly though it works well as java does not follow indentation rules. After cleaning up, here is how it looks:

```java
// These pesky special agents keep reverse engineering our source code and then
// breaking into our secret vaults. THIS will teach those sneaky sneaks a
// lesson.
//
// -Minion #0891
import java.util.*; 
import javax.crypto.Cipher; 
import javax.crypto.spec.SecretKeySpec;
import java.security.*; 

class VaultDoor8 {
    public static void main(String args[]) {
        Scanner b = new Scanner(System.in); 
        System.out.print("Enter vault password: ");
        String c = b.next(); 
        String f = c.substring(8,c.length()-1); 
        VaultDoor8 a = new VaultDoor8(); 
        if (a.checkPassword(f)) {
            System.out.println("Access granted."); 
        } else {
            System.out.println("Access denied!"); 
        } 
    } 

    public char[] scramble(String password) {
        /* Scramble a password by transposing pairs of bits. */
        char[] a = password.toCharArray(); 
        for (int b=0; b<a.length; b++) {
            char c = a[b]; 
            c = switchBits(c,1,2); 
            c = switchBits(c,0,3); 
            /* c = switchBits(c,14,3); 
            c = switchBits(c, 2, 0); */ 
            c = switchBits(c,5,6); 
            c = switchBits(c,4,7);
            c = switchBits(c,0,1); 
            /* d = switchBits(d, 4, 5); 
            e = switchBits(e, 5, 6); */ 
            c = switchBits(c,3,4); 
            c = switchBits(c,2,5); 
            c = switchBits(c,6,7); 
            a[b] = c; 
        } 
        return a;`
    } 

    public char switchBits(char c, int p1, int p2) {
        /* Move the bit in position p1 to position p2, and move the bit
        that was in position p2 to position p1. Precondition: p1 < p2 */ 
        char mask1 = (char)(1 << p1);
        char mask2 = (char)(1 << p2); 
        /* char mask3 = (char)(1<<p1<<p2); 
        mask1++; mask1--; */ 
        char bit1 = (char)(c & mask1); 
        char bit2 = (char)(c & mask2); 
        /* System.out.println("bit1 " + Integer.toBinaryString(bit1));
        System.out.println("bit2 " + Integer.toBinaryString(bit2)); */ 
        char rest = (char)(c & ~(mask1 | mask2)); 
        char shift = (char)(p2 - p1); 
        char result = (char)((bit1<<shift) | (bit2>>shift) | rest); 
        return result;
    } 

    public boolean checkPassword(String password) {
        char[] scrambled = scramble(password); 
        char[] expected = {0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0, 0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xE0, 0x95, 0xF1, 0xE1, 0xE0, 0xA4, 0xC0, 0x94, 0xA4 }; 
        return Arrays.equals(scrambled, expected); 
    } 
}
```

Now after looking at the source code we can understand how it works. This is the work flow:

- Gets input from the user
- Divides it into three parts: first part consists of 8 characters, we can tell it's `picoCTF{}` and the second parts consists of rest if the string except the last character which is supposed to be `}`. So the inner contents of the flag is stored in the variable _f_.
- string _f_ is being checked using the _checkPassword_ method, the _checkPassword_ method scrambles f using the _scramble_ method.
- In the _scramble_ method, each character of the string is modified/encrypted using the _switchBits_ method.
- Then the returned modified/encrypted character array of the string is compared with predefined bytes of another character array _expected_. If it is equal it says "Access Granted".

To find the value of the string f, we have to reverse the process and modify the source code. Here the reversed work flow.

- Take the character array expected and 

**Flag:** `picoCTF{num3r1cal_c0ntr0l_XXXXXXXX}` 