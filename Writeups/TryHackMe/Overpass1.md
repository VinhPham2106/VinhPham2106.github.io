# Overpass level 1
* Difficulty: Easy
* Broken Authentication, Domain attack, cryptography

## Context
* We are given a website advertising a password management tools created by Computer Science students. There's an login panel to the admin portal. You can also download the binaries and source code for the password manager.

## Broken Authentication and Initial Foothold
* We used dirsearch to enumerate directories and interesting files. We found this really interesting **login.js** file for the login panel:
```
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}
```
* We managed to fool this check by accessing the admin panel with the "Cookie: SessionToken=asdfsfadffd" header in the request.
* Once inside, we found that a private key file is provided for user james. It is mentioned that the key is password protected. We used sshtojohn to get the hash, the use john to crack the password using the rockyou file.
* We have to change the permission of the key file to 700 then proceed to use that to log in to a james'ssh sessions on the target machine. Got user.txt

## Privilege Escalation
* Again and again we run linpeas.sh and analyze it's output. The exploit I used is that the /etc/hosts file is editable
* I found in the crontab file that the server curl a bashscript on the overpass.thm domain and run it using root account. Knowing this, I point the overpass.thm to the IP of my fake hosted domain, let it curl a script that send the content of root.txt to my nc listener, and grab the flag. Honestly could have done a shell, but why the extra hassle?
