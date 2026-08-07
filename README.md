# Visual Novel Archive Link Access Portal

Welcome to the official repository for the Visual Novel Archive Link Access Portal. This platform serves as a secure, encrypted gateway for managing the distribution of Visual Novel archive links safely, systematically, and protected from bot tracking.

---

## System Overview

The portal is built using a lightweight client-side architecture fully integrated with Google Apps Script (GAS) as its server-side backend. This system ensures that all users accessing the archive files have successfully completed a valid verification process from the main blog post.

---

## Key Services and Features

### 1. Access Tiering System
* Free Access Tier:
  * Claim limit of 1 game per day.
  * Short-term session duration.
  * Requires completing the WCaptcha verification on the blog post.
* Premium Access (VIP Tier):
  * Higher claim quota of 3 games per day.
  * Long-term access session.
  * Requires a valid Premium Access Code.

### 2. Integrated Payment and Support
* Integrated Payment Gateways: Supports instant payments via Saweria and ShopeePay (equipped with a One-Click Copy Number feature).
* WhatsApp Confirmation and Support: Direct communication links (wa.link) to the Admin ("Mimin") for payment confirmations or technical issue reporting.

### 3. Transparency and User Experience (UI/UX)
* Real-time Statistics: Displays Total Registered Users and Free Quota Used Today.
* Interactive Countdown Timer: Displays remaining active session time for the verification token in real time.
* Quick Copy Password: One-click feature to quickly copy the .zip archive extraction password (pass_rynet_key).

---

## Security Measures

This portal implements a layered defense (Defense-in-Depth) mechanism to prevent link piracy, scraping, and automated bot attacks:

* Session-Based Token Verification: Users must possess a valid token issued directly by the Google Apps Script backend.
* Instant URL Sanitization (history.replaceState): The ?token= parameter in the address bar is immediately removed upon successful validation to prevent token link sharing.
* Memory Session Isolation (sessionStorage): Verification state is safely stored in browser memory. Page refreshes preserve the session, but copying the link to another tab or browser results in an automatic Access Denied.
* Anti-Bot Protection (Honeypot Trap): Features hidden trap fields to catch automated scrapers or spammers.
* Submit Duration Analysis (submitDuration): Measures form completion speed to detect instant executions by automated bot scripts.
* Storage Link Protection: Original Google Drive URLs are held temporarily in JavaScript memory and are immediately cleared (rawDriveUrl = "") once the link is opened.
* Geolocation and IP Tracking: Records public IP addresses and country codes to prevent quota abuse (rate-limit abuse).
* Post-Download Protection: All .zip archives are password-protected for an extra layer of security.

---

## Integration Architecture

```text
[ Blogspot Post ] --( WCaptcha )--> [ Google Apps Script (GAS) ]
                                                | (Issues Token)
                                                v
[ Google Drive Storage ] <--( Access Link )-- [ GitHub Pages Portal ]
```
