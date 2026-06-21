# Lead Form Setup — EventSetupSG

The quote form is live on all three landing pages (Booth Setup, LED Screen
Rental, Event Logistics). It currently works immediately: it fires the Google
Ads conversion and opens WhatsApp with the lead's details pre-filled.

To ALSO record every lead in a Google Sheet and email you a copy, do the
5-minute setup below. Until you do, the form still works (WhatsApp + conversion)
— it just won't write to a Sheet or email yet.

---

## Step 1 — Create the Google Sheet

1. Go to https://sheets.new
2. Name it e.g. "EventSetupSG Leads"
3. In row 1, add these headers (column A–F):
   `Time | Name | Phone | Service | Details | Page`

## Step 2 — Add the Apps Script

1. In the Sheet: **Extensions → Apps Script**
2. Delete any code there and paste the script below
3. Click **Save**

```javascript
// EventSetupSG — lead handler
var NOTIFY_TO  = "Sang@kvell.com.sg";
var NOTIFY_BCC = "Jiajunnn05@gmail.com";

function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    sheet.appendRow([
      data.time || new Date(),
      data.name || "",
      data.phone || "",
      data.service || "",
      data.details || "",
      data.page || ""
    ]);

    var body =
      "New lead from eventsetupsg.com\n\n" +
      "Name: " + (data.name || "") + "\n" +
      "Phone: " + (data.phone || "") + "\n" +
      "Service: " + (data.service || "") + "\n" +
      "Details: " + (data.details || "") + "\n" +
      "Page: " + (data.page || "") + "\n" +
      "Time: " + (data.time || "");

    MailApp.sendEmail({
      to: NOTIFY_TO,
      bcc: NOTIFY_BCC,
      subject: "New Lead — " + (data.service || "Enquiry") + " (" + (data.name || "") + ")",
      body: body
    });

    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

## Step 3 — Deploy as a Web App

1. Click **Deploy → New deployment**
2. Click the gear ⚙ next to "Select type" → choose **Web app**
3. Settings:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. Click **Deploy**
5. Authorise when prompted (it'll warn it's unverified — that's normal for your
   own script; click Advanced → Go to project → Allow)
6. **Copy the Web app URL** (looks like
   `https://script.google.com/macros/s/AKfy.../exec`)

## Step 4 — Paste the URL into the site

Send me the Web app URL and I'll drop it into all three pages (replacing the
`REPLACE_WITH_YOUR_APPS_SCRIPT_URL` placeholder) and push it live. After that,
every form submission lands in your Sheet AND emails Sang (BCC to your Gmail).

---

## Google Ads conversion action

In Google Ads → **Goals → Conversions → + New conversion action → Website**,
create a **"Submit lead form"** action (value ~50 SGD). The form already calls
`trackConversion('form')` which fires your existing gtag event, so once the
conversion action exists and is linked, it'll start counting form submissions.
