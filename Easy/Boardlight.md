# HTB lab Boardlight Writeup

## User privilege escalation

Today I'm going to make a writeup about the boardlight machine on Hackhthebox.
To begin let's make a quick scan to see which ports are open:

<img width="651" alt="Screenshot 2024-09-19 at 10 43 51" src="https://github.com/user-attachments/assets/797b5269-b6d1-465d-aa4c-bdfdcdf96112">

We have the classic 22 and 80 ports.
Let's have more info now:

<img width="966" alt="Screenshot 2024-09-19 at 10 45 39" src="https://github.com/user-attachments/assets/ec2b0ebd-a8a6-4500-81fb-e3d1919744f4">

Okay so not much more interesting info. We can notice that we don't need to edit our hosts to access the website.
I explored the website but nothing was really intersting besides that in the source code:

<img width="526" alt="Screenshot 2024-09-19 at 11 19 30" src="https://github.com/user-attachments/assets/150fd18f-63b5-4843-80c0-e945cfb3ec96">

It gives me a hint that I maybe need to edit my hosts.
