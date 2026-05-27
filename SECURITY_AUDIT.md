# Security Audit Report

**Date:** 2026-05-27
**Auditor:** Claude Code (claude-sonnet-4-6)
**Verdict: CLEAN — No malicious code detected**

---

## Scope

Full source code review of all JS, HTML, and manifest files for malicious code and serious security vulnerabilities.

---

## Manifest Permissions

| Permission | Purpose | Verdict |
|---|---|---|
| `tabs` | Read/track open tabs (core feature) | Appropriate |
| `storage` + `unlimitedStorage` | Save session data to IndexedDB | Appropriate |
| `history` | Used once to remove the extension's own popup URL from history on close (`background.js:563`) | Appropriate |
| `chrome://favicon/*` | Display tab favicons in the UI | Appropriate |
| `contextMenus` | "Add link to space" right-click menu | Appropriate |

No `host_permissions`, no `<all_urls>`, no `content_scripts`, no `web_accessible_resources` exposed to web pages.

---

## Network Calls

Zero outbound network requests in any JS file. No `fetch()`, `XMLHttpRequest`, `WebSocket`, or `navigator.sendBeacon()` calls exist anywhere in the codebase. All data stays local.

The only external URLs in the project are in HTML files:
- `popup.html` and `spaces.html` load **Font Awesome 4.3.0** from `maxcdn.bootstrapcdn.com` and **Open Sans** from `fonts.googleapis.com`

These are standard CDN resources used for display only. No user data is transmitted. Note: these CDNs can observe when the extension UI is opened (minor privacy consideration, not malicious).

---

## Malicious Pattern Checklist

| Pattern | Result |
|---|---|
| `eval()` / `Function()` dynamic execution | None found |
| `atob` / `btoa` / `fromCharCode` obfuscation | None found |
| Hardcoded external API endpoints | None found |
| Credential or password harvesting | None found |
| Keyloggers | None found |
| Cryptomining | None found |
| Data exfiltration | None found |
| `chrome.cookies` access | None found |
| `chrome.debugger` access | None found |
| `chrome.webRequest` interception | None found |
| Minified or obfuscated code | None found |

---

## Message Handling

`background.js` (lines 92–361) has a `chrome.runtime.onMessage` listener. Findings:

- No `onMessageExternal` listener — external websites cannot communicate with the background
- All ID parameters passed via messages are sanitized through `_cleanParameter()` (lines 362–373), which parses them as integers
- No message handler triggers any network call or accesses sensitive data
- Risk: low

---

## innerHTML Usage

Some `innerHTML` assignments use user-typed space names and tab titles sourced from Chrome's own `chrome.tabs` API. Since there are no content scripts injecting into web pages and no remote data sources, any XSS impact is limited to a user affecting their own session only. Not a practical vulnerability.

---

## Third-Party Library

`js/db.js` is the open-source IndexedDB wrapper "db.js" by Aaron Powell (MIT, v0.9.2). It only interacts with local browser storage — no network activity.

---

## localStorage Usage

`spacesService.js` (lines 229, 238) uses `localStorage` only to store and retrieve the extension's own version number (`spacesVersion`). Benign.

---

## Summary

The codebase is a straightforward tab/session manager. All user data stays local (IndexedDB + localStorage). There are no outbound connections to third-party servers, no credential access, no obfuscated or dynamically executed code, and no content scripts running on web pages. The extension operates entirely within Chrome's own APIs and its own local storage.
