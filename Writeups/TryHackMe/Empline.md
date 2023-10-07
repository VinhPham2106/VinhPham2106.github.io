# Empline

* Difficulty: Medium

* Topic: Enumeration, Virtual Hosts, exploitDB, File Upload, Linux Capabilities

## Initial Recon
We started with our nmap scan:
```
$ nmap -A [target IP]
```
We got the following result:

<img width="910" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/f3292650-0d66-459a-8210-bc2db2a393aa">

Apart from the typical 22 and 80, there's mysql at 3306. I didn't there's a way to extract data from the mysql server. So I'll just proceed with viewing the webpage.

I usually jump directly to the source code to see if there's any thing interesting, because that's what these easy/medium boxes tend to do. And I was rewarded with finding a hostname:

<img width="842" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/d4fbe650-a7ad-4afe-9ae7-d90c1238bce8">

Quickly add that along with the room's IP address in the /etc/hosts file. Now let's go to that url.

<img width="904" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/d119c2cc-ca14-4dbb-a4ec-20f4e6a94ac7">

## Initial foothold
I played with a bit of the functionalities, like viewing job description, applying, upload resume, messing with parameters. No clear good attack vectors. So I searched up Opencats on exploitDB:
```
$ searchploit opencats
```
Found an exploit that gave a shell. Here's the execution:

<img width="901" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/e440c254-faa2-4f70-8a2d-e754524438fb">

The thing is that I was stucked in that same directory, and the errors were not reflected. Well I know there's user george, ubuntu and root, but that's about it. So I had to change approach. 
Run some directory discovery using give us the endpoint **upload**. We can see the upload for the shell of the exploit we just used. 

<img width="910" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/6e574e9b-9e7c-4777-8ece-cc84aa188170">

Well then, just use the shell we had to wget a rev shell and pop it on the web. 

<img width="698" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/4e05bc0c-d91f-47d5-810f-411ac60aeff9">
Then boom we get shell

## The final blow
After some manual searching, even wget linpeas.sh to check but nothing. Gotta consult some help from the forum, and found out about linux capabilities. Ran this command to get linux capabilities of files
```
$ getcap / 2>/dev/null
```
Found that the ruby binary has chown+ep. Well if so, then how about we change the ownership of /root and /home/george to www-data. Well I consulted ChatGPT for the command:
```
$ ruby -e ‘File.chown(33, 33, [any path here])’
```

Well then just cat the user.txt and root.txt files and complete the room. LMAO

## Final words
I checked out some online writeups and found some pretty nice one with no shortcuts like mine. Their explanation and approaches made a lot of sense, and there's a lot to learn from
* [Check this out](https://medium.com/@gabrielrostron/thm-empline-writeup-6730af3b4f1e): Using hex manipulation for file upload and LFI, also use ruby to create new root user, pretty dope
* [And this aswell](https://allfun.blog/writeups/tryhackme/empline/): Stumbling with searchsploit, find configs for mysql server and use that for george.



