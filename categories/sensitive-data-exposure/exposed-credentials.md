# Exposed Credentials

**Category:** Sensitive Data Exposure  
**Difficulty:** ⭐⭐  
**Assignee:** Emmanuel Oshike  
**Status:** [x] Completed  

## Objective
A developer was careless with hardcoding unused, but still valid credentials for a testing account on the client-side.

## Tools Used
* Web browser Developer Tools (F12)

## Analysis & Approach
Developers sometimes temporarily hardcode credentials into the frontend source code during the testing phase (e.g., to bypass login screens quickly). If they forget to remove these credentials before compiling and deploying to production, anyone who inspects the client-side JavaScript can extract them. By searching the main application bundle for test email domains, we discovered a hardcoded set of testing credentials and used them to authenticate.

## Steps to Reproduce
1. Navigate to the Juice Shop in your browser.
2. Open Developer Tools (press `F12` or right-click and select **Inspect**) and go to the **Debugger** or **Sources** tab.
3. Open the `main.js` file.
4. Use the search function (`Ctrl+F` or `Cmd+F`) inside the file to search for the string `@juice-sh.op`.
5. Locate the variables where the test credentials were inadvertently left behind: `testingUsername="testing@juice-sh.op"` and `testingPassword="IamUsedForTesting"`.
6. Navigate to the Juice Shop Login page (`/#/login`).
7. Enter `testing@juice-sh.op` as the email and `IamUsedForTesting` as the password.
8. Click **Log in**. The authentication succeeds, and the green success banner for the challenge drops.

## Proof of Concept (PoC)

**Payload / Credentials Extracted:**
```text
Email: testing@juice-sh.op
Password: IamUsedForTesting
```

**Request/Response Snippet:**
```http
POST /rest/user/login HTTP/1.1
Host: localhost:42000
Content-Type: application/json

{"email":"testing@juice-sh.op","password":"IamUsedForTesting"}

HTTP/1.1 200 OK
Content-Type: application/json

{"status":"success","data":{"token":"eyJhbG...","id":<id>,"email":"testing@juice-sh.op"}}
```

![Screenshot of successful exploitation](../../assets/images/exposed-credentials-success.jpg)

## Root Cause & Remediation

**Why did this happen?**
The frontend code was deployed to production with testing artifacts left intact. The developers failed to implement a secure secrets management pipeline, leaving sensitive authentication data in the client-readable bundle.

**How to fix it:**
* Never hardcode credentials, API keys, or secrets into client-side code, as everything sent to the browser is public.
* Implement pre-commit hooks and static application security testing (SAST) in the CI/CD pipeline to automatically scan for and block commits containing hardcoded secrets.
* Use environment variables injected securely at runtime for necessary configuration data.
