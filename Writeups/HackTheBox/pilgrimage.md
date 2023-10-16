# Prigrimage
* Difficulty: Easy
* Tags: Enumaration, Images, recent CVEs and POC

## Some after thoughts:
* I personally didn't enjoy this room so much because it was more about finding the CVEs and using POC provided.
* Sometimes in pentesting, we gonna go down in rabbitholes. Realizing that and change approach is pretty important.
* When doing priviledge escalation, also look at running process by root.
* Dirsearch is pretty good actually. Picked up git-dumper as a new tool.

## Recon & Enum
* The website provided gives you the ability to upload a file, then it said it would shrunk it for you, then it would give you a unique hex id with png/jpeg mime type of the shrunken version.
* Website running some UI libraries, but we know it's php (From Wappalyzer)
* Nmap  reveals port 22 and 80, nothing special
* Dirsearch reveals .git repo, which is always nice

## Finding a way in
### Struggling at first
* Reading some git files revealed by git-dumper, I know that there's a bulletproof.php file that isn't shown on the web. The image compression used magick (ImageMagick), in which the binary can be pulled by 'wget' from the web. We can try to run it and examine the versions and interworking.
* I tried putting php scripts into the image and try to persist the payload through compression. Found a really nice guide on it here [here.](https://www.synacktiv.com/publications/persistent-php-payloads-in-pngs-how-to-inject-php-code-in-an-image-and-keep-it-there.html)
* Although the payload persist, it doesn't get processed by the php compiler, so that didn't work out.
* A git file discovered reveal users root and emily, so I tried bruteforcing emily's ssh credential using rockyou, but that also didn't work out.
### Going in the right direction
* I used git-dumper (realizing the need of it through some online hints) to pull the git content and examine it. so bulletproof.php is an implementation of safe file upload, the original image was stored but the path was unlinked, so there's no way to access it. The code was using an exec() function, so I tried command injection with space and "$IFS" but doesn't get any results.
* Search up exploits related to ImageMagick, found a POC implementation [here](https://github.com/duc-nt/CVE-2022-44268-ImageMagick-Arbitrary-File-Read-PoC). Basically, it can be used to read arbitrary files on the system, as long as the ImageMagick binary has permission.
* There's a database file so I prepped the malicious image, upload it, wget the shrunken version, and decrypt the exposed content. I discovered user emily's password this way.
* Finally used the leaked credentials to ssh into the target machine and pwn **user.txt**

## Privilege escalation
* As always, get **linpeas.sh** onto the system and let it go ham, we read the output.
* Nothing much, but there's this process running a susssy named file malwarescan.sh. Examining what it does, we see that it runs binwalk on every image in the directory storing the shrunken images.
* Again, searching up we found a [recent binwalk CVE](https://portswigger.net/daily-swig/serious-security-hole-plugged-in-infosec-tool-binwalk?&web_view=true). It's a path traversal vuln that can be leverage to RCE. We used the [RCE POC](https://www.exploit-db.com/exploits/51249) the generated a malicious image, put it in the shrunk folder, set up a listener on our end, and grabbed the root shell.

