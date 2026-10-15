# Meta Geo Stalking

**Category:** Sensitive Data Exposure  
**Difficulty:** ⭐⭐  
**Assignee:** Emmanuel Oshike  
**Status:** [x] Completed  

## Objective
Determine the answer to John's security question by looking at an upload of his.

## Tools Used
* Web browser
* ExifTool (or any online EXIF data viewer)
* Google Maps

## Analysis & Approach
When users take photos with modern smartphones, the image file often contains hidden EXIF (Exchangeable Image File Format) metadata. This metadata can include the exact GPS coordinates of where the photo was taken. If a web application does not strip this metadata during the upload process, it exposes the user's location to anyone who downloads the image. By downloading John's uploaded photo from the Photo Wall, extracting the GPS coordinates, and mapping them, we can discover his favorite hiking location and use it to reset his password via his security question.

## Steps to Reproduce
1. Navigate to the **Photo Wall** (`/#/photo-wall`) in the Juice Shop.
2. Locate the hiking photo uploaded by the user "John" (or `john@juice-sh.op`).
3. Download the image to your local machine.
4. Inspect the image's EXIF data. You can do this using the `exiftool` command in Kali Linux or by uploading the image to an online EXIF viewer.
5. Locate the `GPS Latitude` and `GPS Longitude` in the metadata.
6. Input those coordinates into Google Maps. The map will pinpoint a location inside the **Daniel Boone National Forest**.
7. Navigate to the **Forgot Password** page (`/#/forgot-password`).
8. Enter John's email address: `john@juice-sh.op`.
9. The security question "What is your favorite hiking destination?" will appear. Enter `Daniel Boone National Forest` as the answer.
10. Set a new password and submit the form. 
11. The password resets successfully, and the green success banner for the challenge drops down.

## Proof of Concept (PoC)

**Payload Used:**
```text
Daniel Boone National Forest
```

**Request/Response Snippet:**
```http
POST /rest/user/reset-password HTTP/1.1
Host: localhost:42000
Content-Type: application/json

{"email":"john@juice-sh.op","answer":"Daniel Boone National Forest","new":"hacked123","repeat":"hacked123"}

HTTP/1.1 200 OK
Content-Type: application/json

{"status":"success","data":{"id":3,"email":"john@juice-sh.op", ...}}
```

![Screenshot of successful exploitation](../../assets/images/meta-geo-stalking-success.png)

## Root Cause & Remediation

**Why did this happen?**
The application fails to sanitize or strip EXIF metadata from image files when users upload them to the Photo Wall. This allows sensitive geolocation data to remain embedded in the publicly accessible asset. Furthermore, relying on public or easily discoverable OSINT (Open Source Intelligence) data for account recovery questions is a flawed authentication mechanism.

**How to fix it:**
* Implement an image processing library on the backend (such as Sharp for Node.js) to strip all EXIF metadata from images immediately upon upload before saving them to the server.
* Deprecate the use of easily guessable or researchable security questions for password resets, favoring secure alternatives like email-based reset links, SMS OTP, or authenticator apps.
