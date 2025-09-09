---
title: picoGym Reverse Engineering - Medium writeups
date: 2025-07-25 11:00:00 +0530
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

### **speeds and feeds**

![speeds and feeds](/assets/img/posts/picoGym-rev/speeds_and_feeds.png)

**Hint:** What language does a CNC machine use?

**Explanation:**

We are given an nc command to connect to a specific server: `nc mercury.picoctf.net 20301`. After connecting the server gives us a set of lines like this:

```plaintext
G17 G21 G40 G90 G64 P0.003 F50
G0Z0.1
G0Z0.1
G0X0.8276Y3.8621
G1Z0.1
G1X0.8276Y-1.9310
G0Z0.1
G0X1.1034Y3.8621
G1Z0.1
G1X1.1034Y-1.9310
G0Z0.1
G0X1.1034Y3.0345
G1Z0.1
G1X1.6552Y3.5862
G1X2.2069Y3.8621
G1X2.7586Y3.8621
```

I included only a couple of lines as the nc returns more than 1000 lines. It isn't some normal text, after searching in the intenet using the hint and the lines. I found out it was G-code.

G-code (short for Geometric Code) is the programming language used to control CNC machines (Computer Numerical Control), 3D printers, and other automated tools like laser cutters and mills. It tells the machine exactly how to move, where to move, how fast, and what actions to take (like cutting, extruding, or drilling).

So in simple words, the codes draws something, so to see what it is let's try to plot the coordinates using matploitlib. Here is the code:

```python
import matplotlib.pyplot as plt
import re

with open("input.gcode", "r") as f:
    gcode_lines = f.readlines()

pen_down = False
current_pos = [0.0, 0.0]
lines = []

for line in gcode_lines:
    line = line.strip()
    if not line:
        continue

    if "Z" in line:
        match = re.search(r'Z([-0-9.]+)', line)
        if match:
            z = float(match.group(1))
            pen_down = z <= 0.1 

    if "X" in line and "Y" in line:
        x_match = re.search(r'X([-0-9.]+)', line)
        y_match = re.search(r'Y([-0-9.]+)', line)
        if x_match and y_match:
            x = float(x_match.group(1))
            y = float(y_match.group(1))
            if pen_down:
                lines.append(([current_pos[0], x], [current_pos[1], y]))
            current_pos = [x, y]


for x_vals, y_vals in lines:
    plt.plot(x_vals, y_vals, 'k')  

plt.axis('equal')
plt.title("G-code Drawing from input.gcode")
plt.show()
```

The python code reads the Gcode from a file named input.gcode line by line. Then it checks for Z-axis and later it checks for X and Y axis and based on the input values, it starts drawing/plotting using matplotlib library. First we need to save the lines of Gcode we got from the server to a file named input.gcode.

```bash
nc mercury.picoctf.net 20301 > input.gcode
```

This saves the ouput that we recieved from the server into the file. Now after running the python file we can see this:

![output drawing](/assets/img/posts/picoGym-rev/saf-1.png)

We can see the flag.

**Flag:** `picoCTF{num3r1cal_c0ntr0l_XXXXXXXX}` 

### **file-run1**

![file-run1](/assets/img/posts/picoGym-Rev/file-run1.png)

**Hint:**
- To run the program at all, you must make it executable (i.e. $ chmod +x run)
- Try running it by adding a '.' in front of the path to the file (i.e. $ ./run)

**Explanation:**

Though this is a medium level challege, it is far more easier then other medium rev challeges.

We are given a file named: _run_. After checking it's details using file command it returned:

```bash
run: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a468f05064d2f1ffa0047ba7295de4995176da15, for GNU/Linux 3.2.0, not stripped
```

So it is a normal ELF file, let's try giving it permissions and running the ELF executable.

```bash
chmod +x run
```


```bash
./run
```

After running the ELF executable, it printed the flag.

**Flag:** `picoCTF{U51N6_Y0Ur_F1r57_F113_XXXXXXXX}`

### **file-run2**

![file-run2](/assets/img/posts/picoGym-Rev/file-run2.png)

**Hint:** Try running it and add the phrase "Hello!" with a space in front (i.e. "./run Hello!")

**Explanation:**

We are given a file named: _run_, after checking it's file content there is nothing suspectable as it a normal ELF 64-bit LSB pie executable. Let's try giving it permissions and running it.

```bash
chmod +x run
```


```bash
./run
```

This will print the output: "Run this file with only one argument." So it tells us to add an single argument when running. When we give some random argument it will return: "Won't you say 'Hello!' to me first?", from this output and the description there is a chance that "Hello!" is the required argument. So let's give that argument.

```bash
./run Hello!
```

This will return the solution flag of this challenge.

**Flag**: `picoCTF{F1r57_4rgum3n7_f65ed63e}`

### **crackme-py**

![crackme-py](/assets/img/posts/picoGym-Rev/crackme-py.png)

**Explanation:**

We are given a python file named: _crackme.py_. Here are the contents of the file:

```python
# Hiding this really important number in an obscure piece of code is brilliant!
# AND it's encrypted!
# We want our biggest client to know his information is safe with us.
bezos_cc_secret = "A:4@r%uL`M-^M0c0AbcM-MFE0g4dd`_cgN"

# Reference alphabet
alphabet = "!\"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ"+ \
            "[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~"



def decode_secret(secret):
    """ROT47 decode

    NOTE: encode and decode are the same operation in the ROT cipher family.
    """

    # Encryption key
    rotate_const = 47

    # Storage for decoded secret
    decoded = ""

    # decode loop
    for c in secret:
        index = alphabet.find(c)
        original_index = (index + rotate_const) % len(alphabet)
        decoded = decoded + alphabet[original_index]

    print(decoded)



def choose_greatest():
    """Echo the largest of the two numbers given by the user to the program

    Warning: this function was written quickly and needs proper error handling
    """

    user_value_1 = input("What's your first number? ")
    user_value_2 = input("What's your second number? ")
    greatest_value = user_value_1 # need a value to return if 1 & 2 are equal

    if user_value_1 > user_value_2:
        greatest_value = user_value_1
    elif user_value_1 < user_value_2:
        greatest_value = user_value_2

    print( "The number with largest positive magnitude is "
        + str(greatest_value) )



choose_greatest()
```

From this file we can notice that there is an encrypted variable called - *bezos_cc_secret*, it must be the flag. There are two functions in the source code: *decode_secret* and *choose_greatest*, but only **one function is being called**.

So if we mofify the source code so that the code calls the *decode_secre*t function with the *bezos_cc_secret* as an argument, we can get the flag.

As the *choose_greatest* function is useless, replace the line with the below line:

```python
decode_secret(bezos_cc_secret)
```

This will return the decoded *bezos_cc_secret* which is the solution flag.

**Flag:** `picoCTF{1|\/|_4_p34|\|ut_XXXXXXXX}`

### **vault-door-1**

![vault-door-1](/assets/img/posts/picoGym-Rev/vault-door-1.png)

**Hint:** Look up the charAt() method online.

**Explanation:**

We are given a java file: _VaultDoor1.java_. Here are it's contents:

```java
import java.util.*;

class VaultDoor1 {
    public static void main(String args[]) {
        VaultDoor1 vaultDoor = new VaultDoor1();
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

    // I came up with a more secure way to check the password without putting
    // the password itself in the source code. I think this is going to be
    // UNHACKABLE!! I hope Dr. Evil agrees...
    //
    // -Minion #8728
    public boolean checkPassword(String password) {
        return password.length() == 32 &&
               password.charAt(0)  == 'd' &&
               password.charAt(29) == '9' &&
               password.charAt(4)  == 'r' &&
               password.charAt(2)  == '5' &&
               password.charAt(23) == 'r' &&
               password.charAt(3)  == 'c' &&
               password.charAt(17) == '4' &&
               password.charAt(1)  == '3' &&
               password.charAt(7)  == 'b' &&
               password.charAt(10) == '_' &&
               password.charAt(5)  == '4' &&
               password.charAt(9)  == '3' &&
               password.charAt(11) == 't' &&
               password.charAt(15) == 'c' &&
               password.charAt(8)  == 'l' &&
               password.charAt(12) == 'H' &&
               password.charAt(20) == 'c' &&
               password.charAt(14) == '_' &&
               password.charAt(6)  == 'm' &&
               password.charAt(24) == '5' &&
               password.charAt(18) == 'r' &&
               password.charAt(13) == '3' &&
               password.charAt(19) == '4' &&
               password.charAt(21) == 'T' &&
               password.charAt(16) == 'H' &&
               password.charAt(27) == '5' &&
               password.charAt(30) == '2' &&
               password.charAt(25) == '_' &&
               password.charAt(22) == '3' &&
               password.charAt(28) == '0' &&
               password.charAt(26) == '7' &&
               password.charAt(31) == 'e';
    }
}
```

From the source code we can understand that, the java when comiled and ran asks for input. The input is supposed to be the solution flag. Then it slices the `picoCTF{`part and the last chaaracter part which is normally `}`and the rest is stored in _input_ variable, then compares the characters of the input variables based on index in **random** order.

So our task is to reverse the process and find the right _input_ string. Based on the values and index in checkPassword function, we can decode the _input_ variable. Aftering arranging the values based on the index positions this is the value of _input_ variable.

```plaintext
d35cr4mbl3_tH3_cH4r4cT3r5_75092e
```

Now if we combine it with the flag format we get:

```plaintext
picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_XXXXXX}
```

Now ket's check if this is the right flag or not by using the java file provided.

```bash
javac VaultDoor1.java
```

```bash
java VaultDoor1.java
```

When entered the flag, this will give us "Access Granted", which means it is the right flag.

**Flag:** `picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_XXXXXX}`

### **vault-door-3**

![vault-door-3](/assets/img/posts/picoGym-Rev/vault-door-3.png)

**Hint:** Make a table that contains each value of the loop variables and the corresponding buffer index that it writes to.

**Explanation:**

We are given a java file named: _VaultDoor3.java_. Here are it's contents: 

```java
import java.util.*;

class VaultDoor3 {
    public static void main(String args[]) {
        VaultDoor3 vaultDoor = new VaultDoor3();
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

    // Our security monitoring team has noticed some intrusions on some of the
    // less secure doors. Dr. Evil has asked me specifically to build a stronger
    // vault door to protect his Doomsday plans. I just *know* this door will
    // keep all of those nosy agents out of our business. Mwa ha!
    //
    // -Minion #2671
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        char[] buffer = new char[32];
        int i;
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i);
        }
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm18g947_u_4_m9r54f");
    }
}
```

From the source code we can understand that, the java when comiled and ran asks for input. The input is supposed to be the solution flag. Then it slices the `"picoCTF{"`part and the last chaaracter part which is normally `"}"`and the rest is stored in _input_ variable, then **jumbles** the characters of the _input_ variable and stores it in a buffer and compares the characters with the jumbled string.

So our task is to reverse the process and find the right input string. Based on the values and index in checkPassword function, we can decode the input variable. I used the below script to reverse it else it is going to take a lot of time doing it manually. Here is the python script:

```python
target = list("jU5t_a_sna_3lpm18g947_u_4_m9r54f")

password = [''] * 32

for i in range(8):
    password[i] = target[i]

for i in range(8, 16):
    password[23 - i] = target[i]

for i in range(16, 32, 2):
    password[46 - i] = target[i]

for i in range(17, 32, 2):
    password[i] = target[i]

print(''.join(password))
```

This will return the value of the _input_ string:

```plaintext
jU5t_a_s1mpl3_an4gr4m_4_u_xxxxxx
```

Now combining it with `picoCTF{` and `}`, may give us a valid flag, you can check it by compiling and running the given java file.

```bash
javac VaultDoor1.java
```

```bash
java VaultDoor1.java
```

When entered the flag, this will give us "Access Granted", which means it is the right flag.

**Flag:** `picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_xxxxxx}`

### **vault-door-4**

![vault-door-4](/assets/img/posts/picoGym-Rev/vault-door-4.png)

**Hint:**
- Use a search engine to find an "ASCII table".
- You will also need to know the difference between octal, decimal, and hexadecimal numbers.


**Explanation:**

We are given a java file: _VaultDoor4.java_. Here are it's contents:

```java
import java.util.*;

class VaultDoor4 {
    public static void main(String args[]) {
        VaultDoor4 vaultDoor = new VaultDoor4();
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

    // I made myself dizzy converting all of these numbers into different bases,
    // so I just *know* that this vault will be impenetrable. This will make Dr.
    // Evil like me better than all of the other minions--especially Minion
    // #5620--I just know it!
    //
    //  .:::.   .:::.
    // :::::::.:::::::
    // :::::::::::::::
    // ':::::::::::::'
    //   ':::::::::'
    //     ':::::'
    //       ':'
    // -Minion #7781
    public boolean checkPassword(String password) {
        byte[] passBytes = password.getBytes();
        byte[] myBytes = {
            106 , 85  , 53  , 116 , 95  , 52  , 95  , 98  ,
            0x55, 0x6e, 0x43, 0x68, 0x5f, 0x30, 0x66, 0x5f,
            0142, 0131, 0164, 063 , 0163, 0137, 0143, 061 ,
            '9' , '4' , 'f' , '7' , '4' , '5' , '8' , 'e' ,
        };
        for (int i=0; i<32; i++) {
            if (passBytes[i] != myBytes[i]) {
                return false;
            }
        }
        return true;
    }
}
```

From the source code we can understand that, the java when comiled and ran asks for input. The input is supposed to be the solution flag. Then it slices the `"picoCTF{"`part and the last chaaracter part which is normally `"}"`and the rest is stored in input variable, then compares few caracters of input variable in **Decimal** format, few others with **Hexadecimal** format, few others with **Octal** format and the rest with **ASCII** characters.

So our task is to reverse the process and find the right _input_ string. Based on the character balues of the given **decimal**, **hexadecimal** and **octal**, then we can decode the _input_ variable. After decoding all the formats to character format this is the value of _input_ string:

```plaintext
jU5t_4_bUnCh_0f_bYt3s_xxxxxxxxxx
```

Now combining it with `picoCTF{` and `}`, may give us a valid flag, you can check it by compiling and running the given java file.

```bash
javac VaultDoor1.java
```

```bash
java VaultDoor1.java
```

When entered the flag, this will give us "Access Granted", which means it is the right flag.

**Flag:** `picoCTF{jU5t_4_bUnCh_0f_bYt3s_xxxxxxxxxx}`

### **vault-door-5**

![vault-door-5](/assets/img/posts/picoGym-Rev/vault-door-5.png)

**Hint:**
- You may find an encoder/decoder tool helpful, such as https://encoding.tools/
- Read the wikipedia articles on URL encoding and base 64 encoding to understand how they work and what the results look like.

**Explanation:**

We are given a java file: _VaultDoor5.java_. Here are it's contents:

```java
import java.net.URLDecoder;
import java.util.*;

class VaultDoor5 {
    public static void main(String args[]) {
        VaultDoor5 vaultDoor = new VaultDoor5();
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

    // Minion #7781 used base 8 and base 16, but this is base 64, which is
    // like... eight times stronger, right? Riiigghtt? Well that's what my twin
    // brother Minion #2415 says, anyway.
    //
    // -Minion #2414
    public String base64Encode(byte[] input) {
        return Base64.getEncoder().encodeToString(input);
    }

    // URL encoding is meant for web pages, so any double agent spies who steal
    // our source code will think this is a web site or something, defintely not
    // vault door! Oh wait, should I have not said that in a source code
    // comment?
    //
    // -Minion #2415
    public String urlEncode(byte[] input) {
        StringBuffer buf = new StringBuffer();
        for (int i=0; i<input.length; i++) {
            buf.append(String.format("%%%2x", input[i]));
        }
        return buf.toString();
    }

    public boolean checkPassword(String password) {
        String urlEncoded = urlEncode(password.getBytes());
        String base64Encoded = base64Encode(urlEncoded.getBytes());
        String expected = "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
                        + "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
                        + "JTM0JTVmJTM4JTM0JTY2JTY0JTM1JTMwJTM5JTM1";
        return base64Encoded.equals(expected);
    }
}
```

After looking at the code, we can understand that when we run this file, it asks for an input which is supposed to be the solution flag. Then the program processess the input by taking the substring _input_ variable by slicing `picoCTF{`and `}`. Now it validates the _input_ by **url encoding** the input variable then **base64 encoding** the url encoded string and compares it with three pieces of base64 string. If it matches then it returns Access Granted telling that's the right flag.

```python
import base64
import urllib.parse

decoded = base64.b64decode(
    "JTYzJTMwJTZlJTc2JTMzJTcyJTc0JTMxJTZlJTY3JTVm"
    "JTY2JTcyJTMwJTZkJTVmJTYyJTYxJTM1JTY1JTVmJTM2"
    "JTM0JTVmJTM4JTM0JTY2JTY0JTM1JTMwJTM5JTM1"
)


s = decoded.decode()
print(urllib.parse.unquote(s))
```

This will return the value of the input variable and by combining it in the format: `picoCTF{` + output + `}`, we will get the flag. We can test whether the flag is right or wrong.

```bash
javac VaultDoor1.java
```

```bash
java VaultDoor1.java
```

When entered the flag, this will give us "Access Granted", which means it is the right flag.

**Flag:** `picoCTF{c0nv3rt1ng_fr0m_ba5e_64_XXXXXXXX}`

### **vault-door-6**

![vault-door-5](/assets/img/posts/picoGym-Rev/vault-door-6.png)

**Hint:** If X ^ Y = Z, then Z ^ Y = X. Write a program that decrypts the flag based on this fact.

**Explanation:**

We are given a java file: _VaultDoor5.java_. Here are it's contents:

```java
import java.util.*;

class VaultDoor6 {
    public static void main(String args[]) {
        VaultDoor6 vaultDoor = new VaultDoor6();
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

    // Dr. Evil gave me a book called Applied Cryptography by Bruce Schneier,
    // and I learned this really cool encryption system. This will be the
    // strongest vault door in Dr. Evil's entire evil volcano compound for sure!
    // Well, I didn't exactly read the *whole* book, but I'm sure there's
    // nothing important in the last 750 pages.
    //
    // -Minion #3091
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        byte[] passBytes = password.getBytes();
        byte[] myBytes = {
            0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
            0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
            0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
            0xa , 0x6c, 0x60, 0x37, 0x30, 0x60, 0x31, 0x36,
        };
        for (int i=0; i<32; i++) {
            if (((passBytes[i] ^ 0x55) - myBytes[i]) != 0) {
                return false;
            }
        }
        return true;
    }
}
```

From this java source code we can undestand that the java file asks the user for an input then validates whether it is right or not. The input is supposed to be the solution flag. THe code removes the `picoCTF{` and `}` parts of the flag and stored in a _input_ variable and valiadtes it using _checkPassword_ method. Coming to the validation process, what we can understand from the source code is the _input_ variable is converted to Byte format then the bytes are **XOR**'ed and subtracted with a pre-defined array of bytes. So it must mean the bytes of the password after encrypting it with _0x55_ using XOR should match the bytes of the array, else it going to return false.

The Reverse Methodology: 

- As the hint said "X ^ Y = Z, then Z ^ Y = X", we can use myBytes[i] ^ 0x55. This will give us the passBytes.
- Convert the bytes to ascii string, we will get the flag.

we can use the below python script:

```python        
my_bytes = [
    0x3b, 0x65, 0x21, 0x0a, 0x38, 0x00, 0x36, 0x1d,
    0x0a, 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0x0a,
    0x21, 0x1d, 0x61, 0x3b, 0x0a, 0x2d, 0x65, 0x27,
    0x0a, 0x6c, 0x60, 0x37, 0x30, 0x60, 0x31, 0x36,
]

password = ''.join([chr(b ^ 0x55) for b in my_bytes])
print(password)
```

This code will return the decrypted value which is the value of the input variable. Now combining it with `picoCTF{` at the start and `}` at the end will give us the flag.

**Flag:** `picoCTF{n0t_mUcH_h4rD3r_tH4n_x0r_95be5dc}`


