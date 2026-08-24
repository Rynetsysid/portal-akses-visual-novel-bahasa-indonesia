# Rynet Portal Resource Access System Documentation

## 1. Summary & System Context
The **Rynet Portal Resource Access System** is a digital resource rights management and access verification platform integrated between a **Frontend (GitHub Pages)** and a **Backend (Google Apps Script / GAS)** with a **Google Sheets** relational database structure.

The system handles VIP access token/license key verification, download rate limiting, access ticket expiration tracking, and storage node security protection.

---

## 2. Database Schema & Structure (Google Sheets)

### A. License & User Access Tables
1. **`Master_Kode_Premium`**
   - **Function:** Stores primary VIP/Premium access code records.
   - **Columns:** `Kode Akses` (Access Code), `Masa_Aktif` (Active Period), `Email Terikat` (Linked Email), `Status Kode` (UNUSED / ACTIVE / EXPIRED / BLOCKED), `Tanggal Pertama Klaim` (First Claim Date), `Tanggal Expired Tiket` (Ticket Expiration Date), `Catatan WA` (WA Notes).

2. **`Master_Game`**
   - **Function:** Catalog of all available digital assets and archives.
   - **Columns:** `ID Game`, `Kategori Arsip` (Archive Category), `Nama Game`, `Link Google Drive`, `Node Storage`, `Status Game` (Active / Inactive), `Akses Level` (Access Level), `Password Zip`.

3. **`Premium_Add_Akses`**
   - **Function:** Top-up / extension tokens to prolong active premium duration.
   - **Columns:** `Token Akses`, `Masa Aktif (Hari)` (Duration in Days), `Kondisi` (UNUSED / USED), `Email Pengklaim` (Claimer Email), `Waktu Klaim` (Claim Timestamp), `Catatan Admin`.

4. **`Gift_Voucher_Akses_Premium`**
   - **Function:** Bonus or special voucher keys for claiming access tickets.
   - **Columns:** `Kode Akses`, `Email Terikat`, `Masa Aktif`, `Klaim Akses`, `Timestamp`, `Expired`, `Kondisi`, `Catatan`.

---

### B. Logs, Audit & Security Tables
1. **`Log_Akses`**
   - **Function:** Access request log for digital asset download link claims.
   - **Columns:** `Timestamp`, `Tipe Akses`, `Kode Akses`, `Email User`, `ID Game`, `Node Storage`, `IP Public`, `Negara` (Country), `Tanggal Expired Akses Game`, `Tanggal Boleh Klaim`, `Status`, `Cabut Paksa (Admin Only)`.

2. **`Log_User_Akses`**
   - **Function:** 24-hour user query hit rate limit tracking.
   - **Columns:** `Timestamp`, `Email User`, `IP User`, `Status Akses`, `Hit Pengecekan (24Jam)`.

3. **`Log_Token` & `Log_IP_Block_Token`**
   - **Function:** Session token management and IP abuse mitigation logs.

4. **`Node_Registry`**
   - **Function:** Storage node worker server registration and status tracking.

5. **`Blacklist` & `IP_Blocked`**
   - **Function:** Security enforcement blocking malicious IP addresses or email accounts.

6. **`Log_System_Debug` & `Log_Device_Users`**
   - **Function:** Internal backend error tracing and user browser/device diagnostics.

---

## 3. High-Level Operational Workflow

1. **License Verification:**
   - User submits their Email & Access Code via the Portal interface.
   - The backend validates code status against `Master_Kode_Premium`. Upon first activation, the expiration timestamp is computed based on the assigned `Masa_Aktif`.

2. **Access Rights & Rate Limiting:**
   - The system checks `Log_User_Akses` and `Log_Akses` to verify that the user's 24-hour claim/check limits comply with system policies.

3. **Resource Provisioning:**
   - Upon successful verification, the system securely routes the user to the verified storage node using temporary tokenized access protection.
