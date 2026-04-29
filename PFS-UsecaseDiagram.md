# PrivacyFileShare — Use Case Diagram

> **How to use this file:**
> Paste any diagram block into Claude, Gemini, or ChatGPT and ask:
> *"Render this Mermaid diagram"* or *"Draw this PlantUML diagram"*
> GitHub, GitLab, and Notion also render Mermaid natively.

---

## 1. Full System — All Actors & Use Cases (Mermaid)

```mermaid
%%{init: {"theme": "default", "themeVariables": {"fontSize": "14px"}}}%%
graph TD

  %% ─── ACTORS ───────────────────────────────────────────────────────────────
  subgraph Actors
    GU([Guest User])
    RU([Registered User])
    PS([Print Shop / Business User])
    SYS([Supabase System])
  end

  %% ─── PFS OFFICIAL SITE ────────────────────────────────────────────────────
  subgraph "pfs-official-site  (Marketing Website)"
    UC_BROWSE[Browse Landing Page]
    UC_LEARN[Learn Features & Privacy Policy]
    UC_BLOG[Read Blog Posts]
    UC_CONTACT[Submit Contact Form]
    UC_DL[Download Android APK]
    UC_FAQ[View FAQ]
    UC_ABOUT[View About Page]
    UC_TOS[Read Terms of Service]
  end

  %% ─── MOBILE APP ───────────────────────────────────────────────────────────
  subgraph "PrivacyFileShare  (Android App — Core Product)"
    UC_ONBOARD[Complete Onboarding]
    UC_GUEST[Use as Guest — No Sign-Up]
    UC_SIGNUP[Sign Up with Email]
    UC_LOGIN[Log In]
    UC_LOGOUT[Log Out]

    UC_SEND[Send Files]
    UC_SELECT[Select Files — up to 5]
    UC_COMPRESS[Compress Images]
    UC_GENQR[Generate QR Code]
    UC_UPLOAD[Upload Files to Supabase]

    UC_RECV[Receive Files]
    UC_DISPQR[Display My QR Code]
    UC_SCAN[Scan Sender QR Code]
    UC_DOWNLOAD[Download Files to Device]

    UC_HIST[View Transfer History]
    UC_SEARCH[Search History]
    UC_FILEDET[View File Detail]
    UC_DELETE[Delete Transfer Record]

    UC_PROFILE[Edit Profile]
    UC_AVATAR[Choose Avatar]
    UC_USERNAME[Set Username]

    UC_SETTINGS[Manage Settings]
    UC_THEME[Toggle Dark / Light Theme]
    UC_AUTODEL[Configure Auto-Delete]
    UC_COMPRESS2[Set Compression Preference]

    UC_LINKED[Manage Linked Web Devices]
    UC_REVOKE[Revoke Web Session]

    UC_UPDATE[Check for App Updates]
    UC_OFFLINE[View Offline Banner]
  end

  %% ─── WEB DASHBOARD ────────────────────────────────────────────────────────
  subgraph "pfs-web  (Web Dashboard)"
    UC_QRLOGIN[Log In via QR Code]
    UC_EMAILLOGIN[Log In via Email]
    UC_MAGIC[Receive Magic Link]

    UC_WEBDASH[View Received Files Dashboard]
    UC_WEBDL[Download File from Web]
    UC_WEBDEL[Delete File from Web]
    UC_WEBNOTIF[Receive Real-Time Notification]

    UC_WEBHIST[View Transfer History — Web]
    UC_FILTERH[Filter / Search History]

    UC_MYQR[Display My QR Code — Web]
    UC_WEBSETT[Manage Web Settings]
    UC_DELACCT[Delete Account]
  end

  %% ─── RELATIONSHIPS ─────────────────────────────────────────────────────────

  %% Marketing site
  GU --> UC_BROWSE
  GU --> UC_LEARN
  GU --> UC_BLOG
  GU --> UC_CONTACT
  GU --> UC_DL
  GU --> UC_FAQ
  GU --> UC_ABOUT
  GU --> UC_TOS

  %% Mobile — Guest
  GU --> UC_ONBOARD
  GU --> UC_GUEST
  GU --> UC_SEND
  GU --> UC_RECV
  GU --> UC_SCAN
  GU --> UC_HIST

  %% Mobile — Registered
  RU --> UC_SIGNUP
  RU --> UC_LOGIN
  RU --> UC_LOGOUT
  RU --> UC_SEND
  RU --> UC_RECV
  RU --> UC_HIST
  RU --> UC_PROFILE
  RU --> UC_SETTINGS
  RU --> UC_LINKED
  RU --> UC_UPDATE

  %% Mobile — Send flow inclusions
  UC_SEND --> UC_SELECT
  UC_SEND --> UC_COMPRESS
  UC_SEND --> UC_GENQR
  UC_SEND --> UC_UPLOAD

  %% Mobile — Receive flow inclusions
  UC_RECV --> UC_DISPQR
  UC_RECV --> UC_SCAN
  UC_RECV --> UC_DOWNLOAD

  %% Mobile — History
  UC_HIST --> UC_SEARCH
  UC_HIST --> UC_FILEDET
  UC_HIST --> UC_DELETE

  %% Mobile — Profile
  UC_PROFILE --> UC_AVATAR
  UC_PROFILE --> UC_USERNAME

  %% Mobile — Settings
  UC_SETTINGS --> UC_THEME
  UC_SETTINGS --> UC_AUTODEL
  UC_SETTINGS --> UC_COMPRESS2

  %% Mobile — Linked devices
  UC_LINKED --> UC_REVOKE

  %% Web — Print Shop
  PS --> UC_QRLOGIN
  PS --> UC_EMAILLOGIN
  PS --> UC_WEBDASH
  PS --> UC_WEBDL
  PS --> UC_WEBHIST
  PS --> UC_MYQR

  RU --> UC_QRLOGIN
  RU --> UC_EMAILLOGIN
  RU --> UC_WEBDASH
  RU --> UC_WEBDL

  %% Web login inclusions
  UC_EMAILLOGIN --> UC_MAGIC

  %% Web dashboard inclusions
  UC_WEBDASH --> UC_WEBDEL
  UC_WEBDASH --> UC_WEBNOTIF

  %% Web history
  UC_WEBHIST --> UC_FILTERH

  %% Web settings
  RU --> UC_WEBSETT
  UC_WEBSETT --> UC_DELACCT

  %% System actor
  SYS --> UC_WEBNOTIF
  SYS --> UC_AUTODEL
  SYS --> UC_UPDATE
```

---

## 2. Mobile App — Detailed Use Cases (Mermaid)

```mermaid
%%{init: {"theme": "default"}}%%
graph LR

  GU([Guest User])
  RU([Registered User])

  subgraph "Send File Flow"
    S1[Open Send Screen]
    S2[Select Files — max 5]
    S3[Optionally Compress Images]
    S4[Upload to Supabase Storage]
    S5[Generate Transfer QR Code]
    S6[Receiver Scans QR]
    S7[Download Triggered on Receiver]
  end

  subgraph "Receive File Flow"
    R1[Open Receive Screen]
    R2[Display Personal QR Code]
    R3[Sender Scans QR]
    R4[Files Uploaded by Sender]
    R5[Notification Received]
    R6[Download Files to Device]
  end

  subgraph "Authentication"
    A1[Open App — Onboarding]
    A2[Choose: Guest or Register]
    A3[Guest: Auto Username Generated]
    A4[Register: Enter Email + Password]
    A5[Login: Enter Credentials]
    A6[Session Persisted via AsyncStorage]
  end

  GU --> S1
  GU --> R1
  GU --> A1
  RU --> S1
  RU --> R1
  RU --> A1

  S1 --> S2 --> S3 --> S4 --> S5 --> S6 --> S7
  R1 --> R2 --> R3 --> R4 --> R5 --> R6
  A1 --> A2 --> A3
  A2 --> A4 --> A5 --> A6
```

---

## 3. Web Dashboard — Use Cases (Mermaid)

```mermaid
%%{init: {"theme": "default"}}%%
graph LR

  RU([Registered User])
  PS([Print Shop Operator])
  MOB([Mobile App — triggers])

  subgraph "Web Authentication"
    W1[Visit pfs-web URL]
    W2[Choose QR Login]
    W3[Choose Email Login]
    W4[QR Code Displayed on Screen]
    W5[Scan QR with Mobile App]
    W6[Session Activated]
    W7[Receive Magic Link Email]
    W8[Click Link — Authenticated]
  end

  subgraph "Web Dashboard Actions"
    D1[View Received Files List]
    D2[Download File via Signed URL]
    D3[Delete File]
    D4[Receive Real-Time Toast Notification]
    D5[View Transfer History]
    D6[Filter History]
    D7[Display My QR Code]
    D8[Manage Settings]
    D9[Delete Account]
  end

  RU --> W1
  PS --> W1
  W1 --> W2
  W1 --> W3
  W2 --> W4 --> W5 --> W6
  W3 --> W7 --> W8
  W6 --> D1
  W8 --> D1

  MOB --> D4

  D1 --> D2
  D1 --> D3
  D1 --> D4
  D5 --> D6
  D8 --> D9
```

---

## 4. PlantUML Version — Full System Use Case

> Paste into https://www.plantuml.com/plantuml/uml/ or any PlantUML renderer.

```plantuml
@startuml PFS_UseCaseDiagram
left to right direction
skinparam packageStyle rectangle
skinparam actorStyle awesome
skinparam usecase {
  BackgroundColor LightBlue
  BorderColor DarkBlue
  ArrowColor Navy
}

actor "Guest User" as GU
actor "Registered User" as RU
actor "Print Shop / Business" as PS
actor "Supabase System" as SYS

' ─── Official Site ─────────────────────────────────────────────────────
rectangle "pfs-official-site (Marketing)" {
  usecase "Browse Landing Page" as UC_BROWSE
  usecase "Read Blog Posts" as UC_BLOG
  usecase "Submit Contact Form" as UC_CONTACT
  usecase "Download APK" as UC_DL
  usecase "View FAQ / Privacy / TOS" as UC_LEGAL
}

' ─── Mobile App ────────────────────────────────────────────────────────
rectangle "PrivacyFileShare (Android App)" {
  usecase "Complete Onboarding" as UC_ONBOARD
  usecase "Use as Guest" as UC_GUEST
  usecase "Sign Up / Log In" as UC_AUTH

  usecase "Send Files" as UC_SEND
  usecase "Select Files (≤5)" as UC_SELECT
  usecase "Compress Images" as UC_COMPRESS
  usecase "Generate Transfer QR" as UC_GENQR
  usecase "Upload to Supabase" as UC_UPLOAD

  usecase "Receive Files" as UC_RECV
  usecase "Display QR Code" as UC_DISPQR
  usecase "Scan QR Code" as UC_SCAN
  usecase "Download Files" as UC_DL2

  usecase "View Transfer History" as UC_HIST
  usecase "Search & Filter History" as UC_SEARCH
  usecase "View File Detail" as UC_FILEDET

  usecase "Manage Profile & Avatar" as UC_PROFILE
  usecase "Toggle Theme" as UC_THEME
  usecase "Configure Auto-Delete" as UC_AUTODEL
  usecase "Manage Linked Devices" as UC_LINKED
  usecase "Check for Updates" as UC_UPDATE
}

' ─── Web Dashboard ─────────────────────────────────────────────────────
rectangle "pfs-web (Desktop Dashboard)" {
  usecase "Log In via QR Code" as UC_QRLOGIN
  usecase "Log In via Magic Link" as UC_MAGIC
  usecase "View Received Files" as UC_WEBDASH
  usecase "Download File" as UC_WEBDL
  usecase "Delete File" as UC_WEBDEL
  usecase "Real-Time Notifications" as UC_NOTIF
  usecase "View History (Web)" as UC_WEBHIST
  usecase "Display My QR (Web)" as UC_MYQR
  usecase "Delete Account" as UC_DELACCT
}

' ─── ACTOR → USE CASE ──────────────────────────────────────────────────
GU --> UC_BROWSE
GU --> UC_BLOG
GU --> UC_CONTACT
GU --> UC_DL
GU --> UC_LEGAL
GU --> UC_ONBOARD
GU --> UC_GUEST
GU --> UC_SEND
GU --> UC_RECV

RU --> UC_AUTH
RU --> UC_SEND
RU --> UC_RECV
RU --> UC_HIST
RU --> UC_PROFILE
RU --> UC_LINKED
RU --> UC_UPDATE
RU --> UC_QRLOGIN
RU --> UC_MAGIC
RU --> UC_WEBDASH
RU --> UC_WEBHIST

PS --> UC_QRLOGIN
PS --> UC_MAGIC
PS --> UC_WEBDASH
PS --> UC_WEBDL
PS --> UC_MYQR

SYS --> UC_NOTIF
SYS --> UC_AUTODEL
SYS --> UC_UPDATE

' ─── INCLUSIONS ────────────────────────────────────────────────────────
UC_SEND ..> UC_SELECT : <<include>>
UC_SEND ..> UC_COMPRESS : <<include>>
UC_SEND ..> UC_GENQR : <<include>>
UC_SEND ..> UC_UPLOAD : <<include>>

UC_RECV ..> UC_DISPQR : <<include>>
UC_RECV ..> UC_SCAN : <<include>>
UC_RECV ..> UC_DL2 : <<include>>

UC_HIST ..> UC_SEARCH : <<include>>
UC_HIST ..> UC_FILEDET : <<include>>

UC_WEBDASH ..> UC_WEBDL : <<extend>>
UC_WEBDASH ..> UC_WEBDEL : <<extend>>
UC_WEBDASH ..> UC_NOTIF : <<include>>

@enduml
```

---

## Actor Summary Table

| Actor | System Involved | Primary Goals |
|---|---|---|
| **Guest User** | Official Site + Mobile | Browse info, download app, send/receive files without account |
| **Registered User** | Mobile + Web Dashboard | Full file transfer, history, profile, linked devices |
| **Print Shop / Business** | Web Dashboard | Receive files from customers via QR, download for printing |
| **Supabase System** | Backend (all apps) | Auto-delete expired files, send real-time events, auth tokens |

---

## Use Case Count by Module

| Module | Use Cases |
|---|---|
| pfs-official-site | 8 |
| PrivacyFileShare (Mobile) | 22 |
| pfs-web (Dashboard) | 10 |
| **Total** | **40** |
