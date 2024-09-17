# HTB lab Permx Writeup

## User privilege escalation

Hello everyone and welcome to my first ever writeup ! Hope you will enjoy it !

Today we are going to take control of the PermX machine.
- So first things first, let's see which ports are open:

<img width="661" alt="Screenshot 2024-09-17 at 00 46 29" src="https://github.com/user-attachments/assets/bb7da2ea-9902-41a4-9f8c-56e1141fffe1">

As we can see we have just the usual 80 and 22 (ssh) ports.

- Now let's have more info about these:
<img width="1079" alt="Screenshot 2024-09-17 at 01 03 59" src="https://github.com/user-attachments/assets/0c54fe2e-8586-4985-81fd-eab0dabaa0fe">

Let's see what the website looks like, but first we need to edit the /etc/hosts to match the ip address to the domain permx.htb.
After exploring, it looks like there is nothing interesting around.

I decided to enumerate the directories with gobuster but I found nothing intetesting either.
Then I decided to try my luck with a subdomain enumeration, let's see if we can get something interesting:
 - I used this command:
   
<img width="1082" alt="Screenshot 2024-09-17 at 18 23 02" src="https://github.com/user-attachments/assets/31ba57c6-a221-4c65-ae80-f93a7d71e018">

 - And found two subdomains ! lms and www:
   
<img width="1031" alt="Screenshot 2024-09-17 at 18 24 35" src="https://github.com/user-attachments/assets/5b13ebcb-c5e1-4d53-83d1-d1480a4a3721">

Let's add them in the hosts file. After doing it, I checked the www.permx.htb website but it was in fact the same as the original one.
BUT the lms.permx.htb one looks really interesting:

<img width="1483" alt="Screenshot 2024-09-17 at 18 28 22" src="https://github.com/user-attachments/assets/ebe8304b-0886-4f4b-85d8-cb1e4e59de98">

It's a website that is running under Chamilo LMS, with just a basic authentication system.
Before enumerating this subdomain, I search if there was any Chamilo exploit available (I couldn't retrieve the Chamilo version on the website).
I found an interesting one on Github (https://github.com/Ziad-Sakr/Chamilo-CVE-2023-4220-Exploit): 

<img width="849" alt="Screenshot 2024-09-17 at 18 34 13" src="https://github.com/user-attachments/assets/2f1f70be-80c7-4b52-a495-7ec53d906914">

Apparently Chamilo LMS is vulnerable to RCE if we upload a big file.

I cloned the repo locally so I have already the exploit prepared.
Before executing the exploit, we need to prepare also a reverse shell file. I used the Pentestmonkey php reverse shell on rev shell generator.
Now let's see how it works:

<img width="796" alt="Screenshot 2024-09-17 at 18 39 55" src="https://github.com/user-attachments/assets/11b18447-527d-4845-96ba-49fc4bf673a9">


Okay so it seems we just need to our reverse shell file, the host link (here lms.permx.htb) and our listening port that we open in another terminal. Let's do it !

<img width="1020" alt="Screenshot 2024-09-17 at 18 46 13" src="https://github.com/user-attachments/assets/90f2e97b-95c3-4dc4-b2a8-b7a839c68fff">

Yay it worked !! now lets upgrade the shell to have a terminal fully functional.

After doing that, we are connected as www-data

<img width="523" alt="Screenshot 2024-09-17 at 18 48 34" src="https://github.com/user-attachments/assets/2f04544d-df6c-4aa0-b951-7e85bee054c5">

I checked the /home directory and found the user mtz. 

<img width="273" alt="Screenshot 2024-09-17 at 18 50 55" src="https://github.com/user-attachments/assets/579d7acc-f367-4789-aa7d-a9d7002e7746">

Let's find a way to escalate to this user.
So when you arrive in this kind of situation, the good practise is to search on the internet where the configuration of the application is stored. Here it's chamilo and found that:

<img width="664" alt="Screenshot 2024-09-17 at 18 55 41" src="https://github.com/user-attachments/assets/8b75ea22-5e1f-4da2-9425-ea089e302e48">

So let's navigate there.

<img width="693" alt="Screenshot 2024-09-17 at 18 56 39" src="https://github.com/user-attachments/assets/ca3b4e31-892d-4e45-9daa-032e58279c7c">

After reviewing the config files to see if I can get any credentials, I found some credentials for the databse connection in configuration.php:

<img width="686" alt="Screenshot 2024-09-17 at 18 59 29" src="https://github.com/user-attachments/assets/22b6e104-d13c-4fd0-8643-11519d3194ac">

It says these are the credentials of the database for the chamilo user, but as people like to reuse the same passwords, I tried to see if this password worked to escalate to the user mtz:

<img width="506" alt="Screenshot 2024-09-17 at 19 01 54" src="https://github.com/user-attachments/assets/414c39ee-6701-4117-93a4-aa5d486baca9">

Double Yayy !! Now we can get our user.txt flag, we deserved it guys ;)

## Root privilege Escalation

First I always like to to sudo -l, as it is often the way to escalate our privileges using a local binary:

<img width="933" alt="Screenshot 2024-09-17 at 19 06 46" src="https://github.com/user-attachments/assets/6ffcba7e-91c0-4887-ad0b-d656130f05b2">

Apparently we can indeed execute the file /opt/acl.sh with the sudo permission.
Let's analyse first what the file does:

<img width="677" alt="Screenshot 2024-09-17 at 19 12 24" src="https://github.com/user-attachments/assets/dfc5c5c6-b545-4383-85cc-e829159b3ecc">

Okay so it takes as first param a user, second param the permissions we want to give to the file and third param the file we want to change the permissions of. But here is the trick, we can only change the permissions of any files located in /home/mtz directory. But there is no interesting file to change the perms under mtz (there is just the flag). So I thought about symlinks. We could use a symlink to a sensitive file that only root has access to, and change its permissions because we can use sudo (so we would have the root privileges doing so).
I wanna change the perms of the /etc/passwd file, so I can write my own password for root.
Let's try it !

First i generate a password using openssl:

<img width="419" alt="Screenshot 2024-09-17 at 19 27 12" src="https://github.com/user-attachments/assets/6c749558-5b46-4fc6-8ff9-9543ff9e0172">

This is the one we will use.
Then I made a symlink to the /etc/passwd:

<img width="402" alt="Screenshot 2024-09-17 at 19 21 50" src="https://github.com/user-attachments/assets/db4eeb75-2bf3-4914-a7d8-c0ab142e72c6">

Now let's try the command:

<img width="551" alt="Screenshot 2024-09-17 at 19 28 32" src="https://github.com/user-attachments/assets/bd9b7737-2a4c-476c-a7a4-3f8de9c6b1b5">

Let's see if we can modify it:

<img width="860" alt="Screenshot 2024-09-17 at 19 30 16" src="https://github.com/user-attachments/assets/473b728e-eca9-456e-a2e8-6f1b142fd213">

We just remove the 'x' and paste our password generated which is simply 'pass'.
Now I should be able to be root.

<img width="295" alt="Screenshot 2024-09-17 at 19 32 36" src="https://github.com/user-attachments/assets/ca1204dc-70fe-4aae-87da-98d814798939">

Yayy we made it, we are root ! Just get your flag and grab a nice coffee.


This was a really nice and fun box, liked the escalation part that was new for me.
See ya in another write up :D
