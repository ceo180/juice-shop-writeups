# Deprecated Interface

**Category:** Security Misconfiguration  
**Difficulty:** ⭐⭐  
**Assignee:** Emmanuel Oshike  
**Status:** [x] Completed  

## Objective
Use a deprecated B2B interface that was not properly shut down.

## Tools Used
* Web browser Developer Tools (F12)
* Burp Suite (or Postman / cURL)

## Analysis & Approach
When web applications are updated, older API versions are often abandoned but not properly removed from the server (a flaw known as Improper Asset Management). In the OWASP Juice Shop, an old Business-to-Business (B2B) XML order interface was "deprecated" but left actively running on the backend. Because the frontend no longer links to it, it is functionally invisible to standard users. However, by searching the client-side JavaScript source code, we can discover the hidden endpoint and send a manual request to trigger it.

## Steps to Reproduce
1. Open your browser and navigate to the Juice Shop.
2. Open Developer Tools (press `F12` or right-click and select **Inspect**) and navigate to the **Debugger** or **Sources** tab.
3. Search the main JavaScript bundle (e.g., `main.js`) for the keyword `b2b`.
4. You will locate a string referencing a deprecated API route: `/b2b/v2/orders`.
5. Open Burp Suite (or a similar API testing tool) and intercept a random request to the application.
6. Send the intercepted request to the Repeater module.
7. Change the HTTP method to `POST` and modify the endpoint path to `/b2b/v2/orders`.
8. Change the `Content-Type` header to `application/xml` and provide a basic XML body.
9. Send the request. The server processes the deprecated route, triggering the green success banner for the challenge.

## Proof of Concept (PoC)

**Payload Used:**
```xml
<order>
    <item>Apple Juice</item>
</order>
```

**Request/Response Snippet:**
```http
POST /b2b/v2/orders HTTP/1.1
Host: localhost:42000
Content-Type: application/xml
Authorization: Bearer BearereyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNzIiwiZGF0YSI6eyJpZCI6MjMsInVzZXJuYW1lIjoiIiwiZW1haWwiOiJvc2hpa2VlY0BnbWFpbC5jb20iLCJwYXNzd29yZCI6IjI1ZjllNzk0MzIzYjQ1Mzg4NWY1MTgxZjFiNjI0ZDBiIiwicm9sZSI6ImN1c3RvbWVyIiwiZGVsdXhlVG9rZW4iOiIiLCJsYXN0TG9naW5JcCI6IjAuMC4wLjAiLCJwcm9maWxlSW1hZ2UiOiIvYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0LnN2ZyIsInRvdHBTZWNyZXQiOiIiLCJpc0FjdGl2ZSI6dHJ1ZSwiY3JlYXRlZEF0IjoiMjAyNi0xMC0xNSAxNjozNzo1Ni42NzUgKzAwOjAwIiwidXBkYXRlZEF0IjoiMjAyNi0xMC0xNSAxNjozNzo1Ni42NzUgKzAwOjAwIiwiZGVsZXRlZEF0IjpudWxsfSwiaWF0IjoxNzkyMDgyMjgyfQ.VFc-vtwkUI28x9qlmCOkc1dVaaZCp3aXhaaLF26jioxOHj0YAfFTYPhsE-8u7irpj3ijy-4Cwqpn5tediMDLANNOPh6YIzj8bCwCB9lv8zvNiQOHhPLZKoMcHcOgYGD0uG72vl-K65P21VghLDnjFTZsYIfsvAjo-oN_tXjMM_w

<order>
    <item>Apple Juice</item>
</order>

HTTP/1.1 200 OK
Content-Type: application/json

{"status":"success","data": "..."}
```

![Screenshot of successful exploitation](../../assets/images/deprecated-interface-success.png)

## Root Cause & Remediation

**Why did this happen?**
The development team created a new order interface but failed to decommission the old `/b2b/v2/orders` endpoint. They relied on "security through obscurity" by simply removing the UI buttons that pointed to it, leaving the actual backend logic active and vulnerable to discovery via source code analysis.

**How to fix it:**
* Properly decommission and remove deprecated code from production environments. 
* Implement strict API lifecycle management. If an older version of an API (e.g., `v2`) must be kept active temporarily for backward compatibility, it should be protected by the exact same authentication, authorization, and rate-limiting controls as the newest version.
