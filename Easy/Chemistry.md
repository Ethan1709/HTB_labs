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
