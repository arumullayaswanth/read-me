# website-contact-form-to-google-sheets.md

# 📄 Contact Form → Google Sheets Integration Documentation

## 1. Overview

This document explains how to connect a **website contact form** (Name, Email, Mobile, Message) so that all submitted data is **automatically stored in Google Sheets** using **Google Apps Script**.

---

## 2. Technologies Used

* HTML & JavaScript (Website Form)
* Google Sheets
* Google Apps Script (Web App)
* Fetch API (POST request)

---

## 3. Form Fields

The contact form contains the following fields:

* Name
* Email
* Mobile Number
* Message

---

## 4. Google Sheet Setup

### 4.1 Create Google Sheet

1. Open Google Sheets
2. Create a new sheet
3. Add the following headers in **Row 1**:

| Column | Header  |
| ------ | ------- |
| A      | Name    |
| B      | Email   |
| C      | Mobile  |
| D      | Message |
| E      | Date    |

4. Save the sheet (any name is fine)

---

## 5. Google Apps Script Setup

### 5.1 Open Apps Script

1. Open the Google Sheet
2. Click **Extensions → Apps Script**
3. Delete existing code

### 5.2 Add Script Code

Paste the following code:

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  var data = JSON.parse(e.postData.contents);

  sheet.appendRow([
    data.name,
    data.email,
    data.mobile,
    data.message,
    new Date()
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ result: "success" }))
    .setMimeType(ContentService.MimeType.JSON);
}

```

4. Save the script

---

## 6. Deploy Apps Script as Web App

### 6.1 Deployment Steps

1. Click **Deploy → New deployment**
2. Select **Web app**
3. Set the following:

   * Execute as: **Me**
   * Who has access: **Anyone**
4. Click **Deploy**
5. Grant permissions
6. Copy the **Web App URL**

Example:

```
https://script.google.com/macros/s/AKfycbwT91QEMLzR0b0uQCCoH8FUUNOlX2C-oY0f0vmrIjgVOJb6M3YmYse03Ra2JW6SXSS7Og/exec

```

---

## 7. Website Form Integration

### 7.1 HTML Form Code

```html
<form id="contactForm">
  <input type="text" name="name" placeholder="Your Name" required>
  <input type="email" name="email" placeholder="Your Email" required>
  <input type="text" name="mobile" placeholder="Your Mobile Number" required>
  <textarea name="message" placeholder="Your Message"></textarea>
  <button type="submit">Send Message</button>
</form>
```

---

### 7.2 Update Your Website Form Code

```html
<script>
document.getElementById("contactForm").addEventListener("submit", function(e) {
  e.preventDefault();

  const data = {
    name: e.target.name.value,
    email: e.target.email.value,
    mobile: e.target.mobile.value,
    message: e.target.message.value
  };

  fetch("https://script.google.com/macros/s/AKfycbwT91QEMLzR0b0uQCCoH8FUUNOlX2C-oY0f0vmrIjgVOJb6M3YmYse03Ra2JW6SXSS7Og/exec", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(data)
  })
  .then(res => res.json())
  .then(() => {
    alert("Message sent successfully!");
    e.target.reset();
  })
  .catch(() => alert("Submission failed"));
});
</script>

```

🔹 Replace `YOUR_WEB_APP_URL` with the deployed Apps Script URL.

---

## 8. Data Flow

1. User submits the form
2. JavaScript sends data via POST request
3. Google Apps Script receives the data
4. Data is appended to Google Sheets
5. Date & time are automatically recorded

---

## 9. Testing Procedure

1. Open the website
2. Fill all form fields
3. Click **Send Message**
4. Verify a new row appears in Google Sheets

---

## 10. Common Issues & Fixes

| Issue                      | Solution                      |
| -------------------------- | ----------------------------- |
| No data in sheet           | Check Web App URL             |
| Permission error           | Redeploy and allow access     |
| Empty fields               | Check input `name` attributes |
| Script changes not working | Redeploy web app              |

---

## 11. Security Notes

* Web App is publicly accessible
* Recommended to add CAPTCHA for production
* Avoid exposing sensitive data

---

## 12. Conclusion

This integration provides a **free, reliable, and real-time solution** to store website form submissions in Google Sheets without third-party services.

---

If you want, I can also:

* Convert this into **PDF**
* Create a **Word (.docx) format**
* Write **client-friendly documentation**
* Add **screenshots references**

Just tell me 👍
