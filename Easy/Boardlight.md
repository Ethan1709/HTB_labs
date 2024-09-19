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
Did a directory enumeration but nothing interesting was found.
Then I did a subdomain enumeration using ffuf and found one:

<img width="1093" alt="Screenshot 2024-09-19 at 11 19 46" src="https://github.com/user-attachments/assets/5117fb04-5ab7-40bc-87ca-06af05840706">

It's called crm, and it's a good thing we found our domain board.htb earlier otherwise we couldn't find it while enumerating.
After adding the subdomain to the hosts, we arrive on a Dolibarr web app:

<img width="1171" alt="Screenshot 2024-09-19 at 11 20 36" src="https://github.com/user-attachments/assets/d58a6f81-24dd-4c7b-bc3c-e9a50544c0e9">

There is also the version available, let's make a quick search to see if it's vulnerable.
And yay it is ! https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253
For the exploit beeing able to work, we first need to authenticate as a regular user. But we don't have credentials.
I tried the basic creds admin/admin and it worked, I was able to connect as the user admin. So now we have everything for the exploit to work.

<img width="890" alt="Screenshot 2024-09-19 at 12 33 22" src="https://github.com/user-attachments/assets/bff53998-24a1-4f27-b39a-40ff08c30789">

And in another terminal:

<img width="737" alt="Screenshot 2024-09-19 at 12 33 53" src="https://github.com/user-attachments/assets/7d673dfc-d396-4813-8e7b-26b579eeb3df">

The reverse shell worked perfectly. Now we can see that there is a user called larissa in this server. Let's escalate to her.
In this situation, you have to look for the config files, as there is a high chance that the password is stored in.

<img width="492" alt="Screenshot 2024-09-19 at 11 36 32" src="https://github.com/user-attachments/assets/7e736e15-a692-4203-80ce-2f222a680257">

I indeed found it in the conf/conf.php file.
Let's if that's larissa's password.

<img width="672" alt="Screenshot 2024-09-19 at 11 36 55" src="https://github.com/user-attachments/assets/d3c9c037-a07c-413a-bb03-c8e8d8e5151f">

Yayy ! Grab your flag and piece of cupcake now, and enjoy. First step done.

## Root privilege escalation

Now let's become root. I did the sudo -l first but we have no sudo rights.
Then searched for the SUID files:

<img width="863" alt="Screenshot 2024-09-19 at 12 04 10" src="https://github.com/user-attachments/assets/101080bb-dce7-40c1-bc5f-9ef7a96e2d92">

Interesting ! We have some uncommon SUID files here. I searched on google about these and apparently they are a way to escalate to root.
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253
Let's copy past the exploit and run it to see if it works:

<img width="576" alt="Screenshot 2024-09-19 at 12 03 32" src="https://github.com/user-attachments/assets/f2cdf3a7-9081-4d10-9914-8b07ed7835b6">

Rooted ! Nice box, the tricky part from my point of view was the initial foothold, inspect source code first ! Always !





