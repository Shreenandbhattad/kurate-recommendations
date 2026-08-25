# Google Apps Script — Shared Replies Setup

## Quick Start (5 min)

1. Go to **sheets.google.com** — create a blank spreadsheet, name it **Kurate Recommendations Replies**
2. Click **Extensions → Apps Script**
3. Delete the existing code, paste everything below
4. Click **Deploy → New deployment**
   - Type: **Web App**
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Click **Deploy** → copy the Web App URL (looks like `https://script.google.com/macros/s/ABC.../exec`)
6. In `index.html` find `var REPLIES_URL = '';` and paste your URL inside the quotes

---

## Apps Script Code

```javascript
var SHEET = 'Replies';

function doGet(e) {
  // NOTE: Running this from the editor will fail (no e object) — that's normal.
  // Test by visiting the deployed URL in your browser instead.
  if (!e || !e.parameter) return respond([], '');

  var action   = e.parameter.action;
  var key      = e.parameter.key;
  var callback = e.parameter.callback || ''; // JSONP support

  if (action === 'get' || !action) {
    var rows = getSheet().getDataRange().getValues();
    var out  = [];
    for (var i = 1; i < rows.length; i++) {
      if (rows[i][0] === key && rows[i][5] !== 'deleted') {
        out.push({
          id:     rows[i][1],
          name:   rows[i][2],
          text:   rows[i][3],
          ts:     rows[i][4],
          edited: rows[i][6] === true
        });
      }
    }
    return respond(out, callback);
  }

  // Return ALL replies grouped by key — used for page-load prefetch
  if (action === 'getall') {
    var rows = getSheet().getDataRange().getValues();
    var out  = {};
    for (var i = 1; i < rows.length; i++) {
      if (rows[i][5] !== 'deleted') {
        var k = rows[i][0];
        if (!out[k]) out[k] = [];
        out[k].push({
          id:     rows[i][1],
          name:   rows[i][2],
          text:   rows[i][3],
          ts:     rows[i][4],
          edited: rows[i][6] === true
        });
      }
    }
    return respond(out, callback);
  }

  return respond({ error: 'unknown action' }, callback);
}

function doPost(e) {
  if (!e || !e.parameter) return respond({ error: 'no request' }, '');
  var p      = e.parameter;
  var action = p.action;
  var sheet  = getSheet();

  if (action === 'add') {
    sheet.appendRow([p.key, p.id, p.name, p.text, Number(p.ts), '', false]);
    return respond({ ok: true }, '');
  }

  if (action === 'edit') {
    var rows = sheet.getDataRange().getValues();
    for (var i = 1; i < rows.length; i++) {
      if (rows[i][1] === p.id) {
        sheet.getRange(i + 1, 4).setValue(p.text);
        sheet.getRange(i + 1, 7).setValue(true);
        return respond({ ok: true }, '');
      }
    }
  }

  if (action === 'delete') {
    var rows = sheet.getDataRange().getValues();
    for (var i = 1; i < rows.length; i++) {
      if (rows[i][1] === p.id) {
        sheet.getRange(i + 1, 6).setValue('deleted');
        return respond({ ok: true }, '');
      }
    }
  }

  return respond({ error: 'unknown action' }, '');
}

function getSheet() {
  var ss    = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(SHEET);
  if (!sheet) {
    sheet = ss.insertSheet(SHEET);
    sheet.appendRow(['key','id','name','text','timestamp','deleted','edited']);
    sheet.setFrozenRows(1);
  }
  return sheet;
}

function respond(obj, callback) {
  var json = JSON.stringify(obj);
  // If a JSONP callback name is provided, wrap the response
  if (callback) {
    return ContentService
      .createTextOutput(callback + '(' + json + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  return ContentService
    .createTextOutput(json)
    .setMimeType(ContentService.MimeType.JSON);
}
```

---

## Sheet columns (auto-created)

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| key | id | name | text | timestamp | deleted | edited |

---

## Troubleshooting

**Nothing appearing in the sheet?**
- Make sure you deployed as a NEW deployment (not "Manage deployments → Edit")
- Make sure "Who has access" is set to **Anyone** (not "Anyone with Google account")
- After changing any code, always re-deploy as a new deployment and copy the new URL

**CORS error in console?**
- This code uses form-encoded POST which avoids CORS preflight — should not happen
- If it does, re-check that the deployment is set to "Anyone" not "Anyone with Google account"

**Replies load on refresh but not in real time?**
- Normal — replies load from localStorage cache instantly, and sync to Sheets on every add/edit/delete
