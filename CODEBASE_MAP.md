---
lastIndexed: 2026-06-04
repository: prawnsolo/MailToOutlook.live
branch: master
refreshReminder: 7 days
---

# MailToOutlook.live — Codebase Map

Chrome extension that intercepts `mailto:` links and opens them in Microsoft Outlook Web Application (OWA) / Office 365 webmail instead of the default email client.

## Extension Overview

**Name:** MailtOWA: Mailto O365 Webmail / OutlookWebApp  
**Version:** 1.1  
**Manifest Version:** 2 (Manifest V2)  
**Purpose:** Route mailto: links to Outlook Web App rather than default email client

## Manifest & Permissions

**manifest.json** (v2 structure)
- **Permissions:** `storage` (chrome.storage.sync for user preferences)
- **Background Script:** `background.js` (non-persistent)
- **Content Scripts:** `mailtowa.js` (runs on all_urls, at document_end)
- **Browser Action:** Icon + popup (`popup.html`)
- **Options Page:** `popup.html`
- **Icons:** 16x16, 32x32, 48x48, 128x128 PNG (in /images)

## File Inventory

```
MailToOutlook.live/
├── manifest.json              # Extension metadata, permissions, scripts
├── background.js              # Initialize default preference (owaIntercept: "on")
├── mailtowa.js               # Content script: intercept mailto: clicks
├── popup.html                # Settings UI (radio options for OWA routing)
├── popup.js                  # Popup controller (load/save sync storage)
├── options.html              # Options page (references popup.html)
├── README.md                 # User-facing documentation
├── Mailtowa.crx             # Packaged extension binary
└── images/                   # Icons (mail16.png, mail32.png, mail48.png, mail128.png)
```

## How Mailto Interception Works

1. **User clicks a mailto: link** on any webpage
2. **mailtowa.js content script** (running on all pages) intercepts the click via `document.addEventListener("click", handleLink)`
3. **handleLink()** checks:
   - Is it an `<a>` tag?
   - Does href start with `mailto:`?
   - Call `chrome.storage.sync.get('owaIntercept')` to fetch preference
4. **Based on preference:**
   - `"on"` → opens `https://outlook.office.com/owa/?path=/mail/action/compose&to=...`
   - `"on_live"` → opens `https://outlook.live.com/owa/?path=/mail/action/compose&to=...`
   - `"off"` → no interception; allows default browser behavior
5. **Parses mailto URI:**
   - Extracts recipient from `mailto:email@example.com?subject=...&body=...`
   - Replaces `?` with `&` to append to OWA URL
   - Builds: `to={email}&subject={subject}&body={body}&cc=...&bcc=...`
6. **Opens new tab:** `window.open(outlookString, '_blank')`
7. **Prevents default:** `e.preventDefault()`, `e.stopImmediatePropagation()`

## Chrome APIs Used

- **chrome.runtime.onInstalled** — Initialize extension on first install
- **chrome.storage.sync** — Persistent user preference (sync across Chrome profiles)
  - Key: `owaIntercept` (values: `"on"`, `"on_live"`, `"off"`)
- **chrome.runtime.getURL()** — Not directly used; standard for asset loading
- **Content Script Lifecycle** — DOM click interception, event handling

## Core Logic Flow

### Background Script (background.js)
```javascript
chrome.runtime.onInstalled.addListener(() => {
  chrome.storage.sync.set({owaIntercept: "on"});
});
```
Sets default preference on installation.

### Content Script (mailtowa.js)
```javascript
const handleLink = (e) => {
  const target = e.target;
  if (target.tagName.toLowerCase() === 'a' && target.href.startsWith('mailto:')) {
    chrome.storage.sync.get('owaIntercept', (data) => {
      if (String(data.owaIntercept).startsWith("on")) {
        const mailtoLink = target.href.substr(7).replace('?', '&');
        const urlString = data.owaIntercept.length === 2 ? "office" : "live";
        const outlookString = `https://outlook.${urlString}.com/owa/?path=/mail/action/compose&to=${mailtoLink}`;
        window.open(outlookString, '_blank');
      }
    });
    e.preventDefault();
  }
};
document.addEventListener("click", handleLink, false);
```

### Popup / Options (popup.js)
- Radio buttons control `owaIntercept` preference
- On selection, updates storage and visual state (font-weight)
- Reads preference on load and reflects current selection

## CodeGraph Coverage

**Status:** 3 files indexed, 10 nodes, 7 edges
- **Parsed files:** manifest.json, background.js, mailtowa.js
- **Nodes by kind:** 6 constants, 3 files, 1 function
- **Language:** JavaScript (3 files)
- **Index size:** 0.14 MB (SQLite WAL backend)
- **Coverage:** ~100% of functional code (HTML/CSS not parsed)

### Indexable Assets
- `handleLink()` function (mailtowa.js)
- Storage constants and event listeners
- No external dependencies or imports

## Key Design Notes

- **Manifest V2:** Uses deprecated Chrome Extension manifest (V2 phased out; consider migration to V3 for future compatibility)
- **Storage Model:** Synced preferences allow consistency across user's Chrome devices
- **URL Encoding:** Simple parameter extraction; assumes well-formed mailto: URIs
- **No Network Requests:** All functionality local; no external APIs
- **Minimal Footprint:** ~9 KB extension; icon set provided
- **i❤️.ws Creator:** Author Jon Roig; project hosted on GitHub and Chrome Web Store

## Next Steps for Maintenance

1. Consider Manifest V3 migration (Manifest V2 deprecated as of Jan 2025)
2. Add URL encoding for special characters in subject/body
3. Implement error handling for malformed mailto: links
4. Add keyboard shortcut alternative to click interception
5. Test with mailto: links containing special characters, very long bodies
