# PrivacyFileShare — Workflow Diagrams

> **How to use this file:**
> Paste any Mermaid block into Claude, Gemini, ChatGPT, or GitHub and ask to render it.
> Paste PlantUML blocks into https://www.plantuml.com/plantuml/uml/

---

## 1. File Sending Workflow (Mobile App)

```mermaid
flowchart TD
    START([User Opens App]) --> HOME[Home Screen]
    HOME --> SEND_BTN[Tap Send Button]
    SEND_BTN --> AUTH_CHECK{Is User Authenticated?}
    
    AUTH_CHECK -- No --> GUEST_CHECK{Continue as Guest?}
    GUEST_CHECK -- Yes --> AUTO_PROFILE[Auto-Generate Guest Profile]
    GUEST_CHECK -- No --> LOGIN[Go to Login / Signup]
    LOGIN --> AUTH_CHECK
    AUTO_PROFILE --> SEND_SCREEN
    AUTH_CHECK -- Yes --> SEND_SCREEN[Open Send Screen]

    SEND_SCREEN --> SELECT[Select Files — max 5]
    SELECT --> FILE_TYPE{File Type?}

    FILE_TYPE -- Image --> COMPRESS_CHECK{Compression\nEnabled in Settings?}
    COMPRESS_CHECK -- Yes --> COMPRESS[Compress Image\nreact-native-image-resizer]
    COMPRESS_CHECK -- No --> VALIDATE
    COMPRESS --> VALIDATE

    FILE_TYPE -- Document / Video / Other --> VALIDATE[Validate File Size & Count]

    VALIDATE --> SIZE_CHECK{Any file too large\nor count > 5?}
    SIZE_CHECK -- Yes --> ERROR_UI[Show Error Toast]
    ERROR_UI --> SELECT

    SIZE_CHECK -- No --> UPLOAD_START[Begin Upload to\nSupabase Storage]
    UPLOAD_START --> PROGRESS[Show Upload Progress Bar]
    PROGRESS --> UPLOAD_DONE{Upload\nSuccessful?}

    UPLOAD_DONE -- No --> RETRY_CHECK{Retry?}
    RETRY_CHECK -- Yes --> UPLOAD_START
    RETRY_CHECK -- No --> FAIL[Show Failure Toast]

    UPLOAD_DONE -- Yes --> GEN_QR[Generate 8-char Transfer Code]
    GEN_QR --> SHOW_QR[Display QR Code on Screen]
    SHOW_QR --> WAIT[Wait for Receiver to Scan]

    WAIT --> SCANNED{QR Scanned by\nReceiver?}
    SCANNED -- Yes --> TRANSFER_OK[Transfer Complete]
    TRANSFER_OK --> HISTORY[Save to Transfer History]
    HISTORY --> HAPTIC[Haptic Feedback — Success]
    HAPTIC --> DONE([End])

    SCANNED -- Timeout --> EXPIRE[QR Code Expired]
    EXPIRE --> DONE

    style START fill:#4CAF50,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style ERROR_UI fill:#f44336,color:#fff
    style FAIL fill:#f44336,color:#fff
    style EXPIRE fill:#FF9800,color:#fff
    style TRANSFER_OK fill:#2196F3,color:#fff
```

---

## 2. File Receiving Workflow (Mobile App)

```mermaid
flowchart TD
    START([User Opens App]) --> HOME[Home Screen]
    HOME --> RECV_BTN[Tap Receive Button]
    RECV_BTN --> RECV_SCREEN[Open Receive Screen]

    RECV_SCREEN --> OPTION{How to receive?}

    OPTION -- Show My QR --> DISPLAY_QR[Display Personal QR Code\n8-char alphanumeric]
    DISPLAY_QR --> SENDER_SCANS[Sender Scans QR]
    SENDER_SCANS --> SENDER_UPLOADS[Sender Uploads Files\nto Supabase Storage]
    SENDER_UPLOADS --> NOTIFY[Push Notification via FCM]
    NOTIFY --> REALTIME[Supabase Realtime Event\nINSERT on files table]
    REALTIME --> TOAST[Show Toast — New Files Received!]
    TOAST --> DL_PROMPT[Prompt: Download Files?]
    DL_PROMPT -- Yes --> DOWNLOAD

    OPTION -- Scan QR --> CAMERA[Open Camera — CameraKit]
    CAMERA --> SCAN[Scan Sender QR Code]
    SCAN --> DECODE[Decode Transfer Code from QR]
    DECODE --> FETCH[Fetch File Records from Supabase]
    FETCH --> SIGNED_URL[Get Signed Download URLs]
    SIGNED_URL --> DOWNLOAD

    DOWNLOAD[Download Files\nreact-native-blob-util] --> PROGRESS[Show Download Progress]
    PROGRESS --> DL_DONE{Download\nComplete?}

    DL_DONE -- No --> DL_RETRY{Retry?}
    DL_RETRY -- Yes --> DOWNLOAD
    DL_RETRY -- No --> DL_FAIL[Show Error Toast]

    DL_DONE -- Yes --> SAVE_DEVICE[Save to Device Storage\n/Download or custom path]
    SAVE_DEVICE --> HIST[Log to Transfer History]
    HIST --> HAPTIC[Haptic Feedback — Success]
    HAPTIC --> DONE([End])

    style START fill:#4CAF50,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style DL_FAIL fill:#f44336,color:#fff
    style TOAST fill:#2196F3,color:#fff
    style NOTIFY fill:#9C27B0,color:#fff
```

---

## 3. Web QR Login Workflow (pfs-web)

```mermaid
flowchart TD
    START([User Visits pfs-web]) --> LANDING[Login Page]
    LANDING --> METHOD{Login Method?}

    METHOD -- QR Login --> GENERATE[API: Create Web Session\n/api/web-sessions/create]
    GENERATE --> SESSION_DB[Insert web_sessions row\nis_active=false, expires_at=+5min]
    SESSION_DB --> QR_DISPLAY[Display QR Code\n— encodes session token]

    QR_DISPLAY --> POLL[Browser Polls for\nSession Activation]
    POLL --> MOBILE[User Opens PFS Mobile App]
    MOBILE --> SCAN_WEB[Scan Web QR Code]
    SCAN_WEB --> VERIFY[Mobile API Call:\nVerify + Activate Session]
    VERIFY --> UPDATE_SESSION[Update web_sessions:\nis_active=true]
    UPDATE_SESSION --> REALTIME[Supabase Realtime Event\n— web_sessions UPDATE]
    REALTIME --> BROWSER_AUTH[Browser Detects Activation]
    BROWSER_AUTH --> SET_COOKIE[Set Auth Cookie / Store Token]
    SET_COOKIE --> DASHBOARD[Redirect to Dashboard /home]

    METHOD -- Email Login --> EMAIL_FORM[Enter Email Address]
    EMAIL_FORM --> SEND_MAGIC[API: Create Email Session\n/api/web-sessions/create-email]
    SEND_MAGIC --> MAGIC_LINK[Supabase Sends Magic Link Email]
    MAGIC_LINK --> USER_CLICK[User Clicks Link in Email]
    USER_CLICK --> VERIFY_TOKEN[Verify Token]
    VERIFY_TOKEN --> DASHBOARD

    POLL --> TIMEOUT{Session Expired\n5 min?}
    TIMEOUT -- Yes --> EXPIRE[Show: QR Expired — Refresh]
    EXPIRE --> GENERATE

    DASHBOARD --> AUTHENTICATED([Dashboard Ready])

    style START fill:#4CAF50,color:#fff
    style AUTHENTICATED fill:#4CAF50,color:#fff
    style EXPIRE fill:#FF9800,color:#fff
    style REALTIME fill:#9C27B0,color:#fff
    style BROWSER_AUTH fill:#2196F3,color:#fff
```

---

## 4. User Registration & Onboarding Workflow (Mobile App)

```mermaid
flowchart TD
    INSTALL([App First Install]) --> ONBOARD[Onboarding Screen\nFeatures Introduction]
    ONBOARD --> ONBOARD_DONE[User Completes Onboarding]
    ONBOARD_DONE --> onboardingStore[Save onboarding=true\nto AsyncStorage via onboardingStore]
    onboardingStore --> AUTH_CHOICE{Create Account\nor Guest?}

    AUTH_CHOICE -- Guest --> GUEST_GEN[Auto-Generate Username\ne.g. PFS_User_X8K2]
    GUEST_GEN --> GUEST_PROFILE[Save Guest Profile\nauthStore — AsyncStorage]
    GUEST_PROFILE --> HOME[Home Screen — Guest Mode]

    AUTH_CHOICE -- Register --> SIGNUP[Sign Up Screen]
    SIGNUP --> INPUT[Enter Email + Password]
    INPUT --> VALIDATE[Validate Input]
    VALIDATE --> INVALID{Valid?}
    INVALID -- No --> INPUT_ERR[Show Inline Errors]
    INPUT_ERR --> INPUT

    INVALID -- Yes --> SUPABASE_AUTH[Supabase Auth: signUp]
    SUPABASE_AUTH --> CONFIRM{Email\nConfirmation\nRequired?}

    CONFIRM -- Yes --> VERIFY_EMAIL[User Checks Email & Verifies]
    VERIFY_EMAIL --> LOGIN_SCREEN[Login Screen]
    CONFIRM -- No --> AUTO_LOGIN[Auto Sign In]
    AUTO_LOGIN --> PROFILE_CREATE

    LOGIN_SCREEN --> LOGIN_INPUT[Enter Credentials]
    LOGIN_INPUT --> SUPABASE_LOGIN[Supabase Auth: signInWithPassword]
    SUPABASE_LOGIN --> LOGIN_OK{Success?}
    LOGIN_OK -- No --> LOGIN_ERR[Show Error Toast]
    LOGIN_ERR --> LOGIN_INPUT
    LOGIN_OK -- Yes --> PROFILE_CREATE

    PROFILE_CREATE[Create/Fetch Profile\nfrom profiles table] --> AVATAR[Choose Avatar — 8 Options]
    AVATAR --> USERNAME[Set Username]
    USERNAME --> PERSIST[Persist to authStore\n+ AsyncStorage]
    PERSIST --> HOME[Home Screen — Registered]

    HOME --> DONE([App Ready to Use])

    style INSTALL fill:#4CAF50,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style INPUT_ERR fill:#f44336,color:#fff
    style LOGIN_ERR fill:#f44336,color:#fff
```

---

## 5. Real-Time File Sync Workflow (Mobile ↔ Supabase ↔ Web)

```mermaid
sequenceDiagram
    actor MobileSender as Mobile Sender
    participant SupaStorage as Supabase Storage
    participant SupaDB as Supabase Database
    participant Realtime as Supabase Realtime
    actor MobileReceiver as Mobile Receiver
    actor WebDashboard as Web Dashboard (pfs-web)

    MobileSender->>SupaStorage: Upload file(s) — multipart upload
    SupaStorage-->>MobileSender: File URL + path returned

    MobileSender->>SupaDB: INSERT into files table\n{sender_id, receiver_id, file_path, status}
    SupaDB-->>Realtime: Trigger INSERT event on files channel
    
    par Notify Mobile Receiver
        Realtime-->>MobileReceiver: Realtime INSERT event
        MobileReceiver->>MobileReceiver: Show Toast Notification
        MobileReceiver->>SupaDB: Request signed URL
        SupaDB-->>MobileReceiver: Signed URL (60s expiry)
        MobileReceiver->>SupaStorage: Download file via signed URL
        SupaStorage-->>MobileReceiver: File data stream
        MobileReceiver->>MobileReceiver: Save to device storage
    and Notify Web Dashboard
        Realtime-->>WebDashboard: Realtime INSERT event\nuseRealtimeFiles hook
        WebDashboard->>WebDashboard: Update files list in UI
        WebDashboard->>WebDashboard: Show Sonner toast notification
        WebDashboard->>WebDashboard: User clicks Download
        WebDashboard->>SupaDB: Request signed URL\n/api/files/signed-url
        SupaDB-->>WebDashboard: Signed URL
        WebDashboard->>SupaStorage: Download file
    end

    MobileSender->>SupaDB: UPDATE files — status=completed
    SupaDB-->>Realtime: UPDATE event broadcast
```

---

## 6. Auto-Delete Workflow (Background / System)

```mermaid
flowchart TD
    SCHED([Supabase Cron / Scheduled Function]) --> CHECK[Check files table for\nexpired records]
    CHECK --> AGED{Files older than\nuser's auto-delete\nsetting?}
    AGED -- No --> WAIT[Wait for next scheduled run]
    WAIT --> SCHED
    AGED -- Yes --> LIST[List expired file records]
    LIST --> DEL_STORAGE[Delete from Supabase Storage\n— remove file bytes]
    DEL_STORAGE --> DEL_DB[Delete from files table]
    DEL_DB --> REALTIME[Supabase Realtime:\nDELETE event broadcast]
    REALTIME --> MOBILE_UPDATE[Mobile: Remove from History UI]
    REALTIME --> WEB_UPDATE[Web: Remove from Dashboard UI]
    MOBILE_UPDATE --> DONE([Cleanup Complete])
    WEB_UPDATE --> DONE

    style SCHED fill:#9C27B0,color:#fff
    style DONE fill:#4CAF50,color:#fff
```

---

## 7. Linked Device (Web Session) Management Workflow (Mobile)

```mermaid
flowchart TD
    START([User on Mobile App]) --> SETTINGS[Open Settings Screen]
    SETTINGS --> LINKED[Tap Manage Linked Devices]
    LINKED --> FETCH[Fetch active web_sessions\nfor current user_id]
    FETCH --> LIST[Display List of Active Sessions\n— token, created_at, device_info]

    LIST --> ACTION{User Action?}

    ACTION -- Revoke Session --> CONFIRM_REVOKE{Confirm\nRevoke?}
    CONFIRM_REVOKE -- No --> LIST
    CONFIRM_REVOKE -- Yes --> UPDATE_DB[Update web_sessions:\nis_active=false]
    UPDATE_DB --> REALTIME[Supabase Realtime:\nUPDATE event on web_sessions]
    REALTIME --> WEB_LOGOUT[Web Dashboard detects\nsession deactivated]
    WEB_LOGOUT --> WEB_REDIRECT[Web: Redirect to Login Page]
    UPDATE_DB --> REFRESH[Refresh Linked Devices List]
    REFRESH --> LIST

    ACTION -- Revoke All --> REVOKE_ALL[Set all sessions:\nis_active=false]
    REVOKE_ALL --> REALTIME

    ACTION -- Back --> SETTINGS

    style START fill:#4CAF50,color:#fff
    style WEB_LOGOUT fill:#f44336,color:#fff
    style REALTIME fill:#9C27B0,color:#fff
```

---

## 8. Contact Form Workflow (pfs-official-site)

```mermaid
flowchart TD
    START([User on Marketing Site]) --> CONTACT_PAGE[Navigate to Contact Page]
    CONTACT_PAGE --> FORM[Fill in Contact Form\n— name, email, message]
    FORM --> SUBMIT[Submit Form]
    SUBMIT --> VALIDATE{Client-side\nValidation OK?}
    VALIDATE -- No --> ERRORS[Show Field Errors]
    ERRORS --> FORM
    VALIDATE -- Yes --> API_CALL[POST /api/send-email]
    API_CALL --> RESEND[Resend API:\nsend email to privacyfileshare@gmail.com]
    RESEND --> RESULT{Email Sent?}
    RESULT -- Yes --> SUCCESS[Show Success Message\n— Thank you! We'll be in touch]
    RESULT -- No --> API_ERROR[Show Error Toast\n— Please try again]
    API_ERROR --> FORM
    SUCCESS --> DONE([End])

    style START fill:#4CAF50,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style ERRORS fill:#f44336,color:#fff
    style API_ERROR fill:#f44336,color:#fff
    style SUCCESS fill:#2196F3,color:#fff
```

---

## Workflow Summary Table

| # | Workflow | Actors Involved | Key Systems |
|---|---|---|---|
| 1 | **File Sending** | Guest / Registered User | Mobile App, Supabase Storage |
| 2 | **File Receiving** | Guest / Registered User | Mobile App, FCM, Supabase Realtime |
| 3 | **Web QR Login** | Registered User, Mobile App | pfs-web, Supabase Realtime |
| 4 | **Registration & Onboarding** | New User | Mobile App, Supabase Auth |
| 5 | **Real-Time File Sync** | Sender, Receiver, Web User | Supabase Realtime, Storage |
| 6 | **Auto-Delete** | Supabase System | Supabase Cron, Realtime |
| 7 | **Linked Device Management** | Registered User | Mobile App, Supabase |
| 8 | **Contact Form** | Visitor | Official Site, Resend API |
