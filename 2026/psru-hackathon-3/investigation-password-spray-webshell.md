# Investigation — Password Spray & Web Shell Upload

**CTF:** PSRU Cyber Hackathon #3  
**Category:** Investigation / DFIR  
**Flags:** `/portal/api/login.php` · `5.16.20.1` · `4` · `/portal/api/payment_upload.php` · `1764514959_redaxe-backdoor.php` · `cp 1764514959_redaxe-backdoor.php dashboard.php`

## Description

A multi-part investigation challenge following an attacker's full kill chain — from password spray to web shell upload to post-exploitation.

---

## Part 1 — Target URL of the Password Spray

**Flag format:** `/../../index.php`

### Solution

Searched for failed login attempts:

```
http.response.status_code: 401 AND http.request.method: POST
```

Checked `url.path` on the results.

**Flag:** `/portal/api/login.php`

---

## Part 2 — Attacker's IP Address

### Solution

Narrowed the previous query to the login endpoint:

```
http.response.status_code: 401 AND http.request.method: POST AND url.path: "/portal/api/login.php"
```

Checked `source.ip`.

**Flag:** `5.16.20.1`

---

## Part 3 — Number of Successful Logins

**Flag format:** Number

### Solution

Switched to HTTP 200 responses from the same attacker IP:

```
http.response.status_code: 200 AND http.request.method: POST AND url.path: "/portal/api/login.php" AND source.ip: "5.16.20.1"
```

Got 4 matching documents.

**Flag:** `4`

---

## Part 4 — File Upload Vulnerability URL

**Flag format:** `/../../index.php`

### Solution

Looked for upload activity from the attacker's IP:

```
source.ip: "5.16.20.1" AND url.path: *upload*
```

Found the vulnerable endpoint in `url.path`.

**Flag:** `/portal/api/payment_upload.php`

---

## Part 5 — Web Shell Filename

**Flag format:** `filename.pdf`

### Solution

The answer came from the previous challenge's results — the full upload path was visible in the response:

```
/portal/uploads/slips/1764514959_redaxe-backdoor.php
```

**Flag:** `1764514959_redaxe-backdoor.php`

---

## Part 6 — Command Executed via Web Shell

**Flag format:** `ls -la`

### Solution

Searched for activity involving the uploaded web shell:

```
"1764514959_redaxe-backdoor.php"
```

Checked `process.command_line` on the results.

**Flag:** `cp 1764514959_redaxe-backdoor.php dashboard.php`

The attacker copied the web shell to `dashboard.php` — a more inconspicuous filename — to maintain persistence while making the original upload less obvious.

---

## Full Attack Timeline

1. Attacker performs password spray against `/portal/api/login.php`
2. Achieves 4 successful logins from IP `5.16.20.1`
3. Discovers file upload vulnerability at `/portal/api/payment_upload.php`
4. Uploads web shell, saved as `1764514959_redaxe-backdoor.php`
5. Copies shell to `dashboard.php` to blend in and maintain access

## Takeaways

- Password spray leaves a clear trail of 401s from a single IP — easy to pivot from there
- Attackers commonly rename uploaded web shells to legitimate-sounding filenames for persistence
- Following the attacker IP through each stage of the kill chain (auth → upload → execution) is the most efficient investigation approach
