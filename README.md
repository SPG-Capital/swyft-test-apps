# Swyft Finance — Stage 2 Technical Project

Welcome to Stage 2. This is a **paid take-home project worth ₹5,000 INR**, sent via Wise once you submit. The code you write is yours to keep.

## What We're Looking For

We're hiring an engineer to build **desktop apps with Electron** for Windows and macOS. This project is a chance to show us how you build one: a secure, well-structured Electron app that talks to a cloud backend and gets the important details exactly right.

## Choose Your Project

Choose **ONE** of these two projects:

| Option | Project | Description |
|--------|---------|-------------|
| A | [Quoting Calculator](./quoting-tool/) | Build a multi-lender loan quoting engine with precise financial calculations |
| B | [Bank Statement Tool](./bank-statement-tool/) | Build a bank statement analysis and visualisation tool |

## Tech Stack

Both projects use the same stack. A Google Cloud project is provisioned for you.

- **App:** An **Electron desktop app** that runs on **Windows or macOS**, written in **TypeScript**, with **React** for the UI. Use any build tooling you like (Electron Forge, electron-vite, electron-builder, etc.).
- **Backend API:** A small API you deploy to **Google Cloud Run**. The desktop app talks **only** to this API — never directly to the database or to storage.
- **Database:** Cloud SQL for PostgreSQL
- **Sign-in:** Google sign-in via Google Identity Platform. Sign-in must open in the user's **normal web browser** (not inside an Electron window) and return the user to the app when done.
- **File Storage:** Cloud Storage, accessed through your API (for example, using signed URLs).
- **Distribution:** An installer for **Windows or macOS** (whichever you use), attached to a **GitHub Release**. Unsigned builds are fine.

## Desktop App Requirements (Both Projects)

These apply to whichever project you choose.

### Must Have

- [ ] **Secure Electron setup:** context isolation on, Node integration off in the UI, sandbox on. The UI only reaches the rest of the app through a small, clearly defined preload bridge.
- [ ] **Clear structure:** a clean split between the main process, the preload bridge and the React UI.
- [ ] **No secrets in the installer:** no database passwords, service account keys or other secrets shipped inside the app. Anything secret lives in your backend.
- [ ] **Remembered sign-in:** the user's session is stored using the operating system's secure storage (for example, Electron's `safeStorage`), survives an app restart, and is fully cleared on sign-out.
- [ ] **Signed-out state:** until the user signs in, the app shows only the sign-in screen.
- [ ] **Backend checks every request:** your API confirms who the user is on every request and only returns their own data.
- [ ] **An installer:** a working installer for **Windows or macOS** — whichever you use. Tell us which one.
- [ ] **Offline behaviour:** when there's no internet connection, the app shows a clear message instead of a blank screen or a crash.
- [ ] **Resizable window:** the app works well from a small laptop window up to full screen.

### Nice to Have

- [ ] Installers for **both** Windows and macOS, built automatically with GitHub Actions
- [ ] Automatic updates
- [ ] Native app menus (File, Edit, etc.) with keyboard shortcuts
- [ ] Remembering window size and position between launches

## Evaluation Criteria

Both projects are evaluated on:

1. **Code Quality** (25%)
   - Clean, readable, maintainable code
   - Appropriate abstractions
   - Error handling
   - Testing approach

2. **Technical Accuracy** (25%)
   - Calculations or parsing match expected outputs exactly
   - Edge cases handled correctly
   - Data integrity maintained

3. **Desktop App Quality** (20%)
   - Secure Electron setup and clear process structure
   - Sign-in, secure token storage and backend communication
   - A working installer for Windows or macOS

4. **User Experience** (15%)
   - Intuitive interface
   - Feels like a proper desktop app
   - Helpful error messages
   - Performance

5. **Problem Solving & Communication** (15%)
   - Approach to ambiguous requirements
   - Documentation of assumptions
   - Communication during the project

## Timeline

All deadlines are **11:59 pm India Standard Time (IST)** on the date shown. These dates are fixed.

| Milestone | Date |
|-----------|------|
| Project starts | Sat 26 Sep, 11:59 pm IST |
| **Code due** | **Tue 29 Sep**, 11:59 pm IST |
| **Video due** | **Wed 30 Sep**, 11:59 pm IST |
| You hear back by | Fri 9 Oct |

This is a short window — three days — so scope your build accordingly. **A focused, well-tested, working app beats a broad, half-finished one.** Get the core working and packaged first, then add features.

- **Check-ins:** Daily async updates by email are encouraged
- **Questions:** Ask anytime via email

## Submission

1. Push code to a **private** GitHub repository
2. Add both `SauraPG72` and `s2dmad` as collaborators
3. Deploy your backend API to your provisioned **Google Cloud** project
4. Publish a **GitHub Release** containing your installer (Windows or macOS), and send us the link
5. Include **tests**
6. Include a README covering:
   - How to run the app locally and how to build the installers
   - Architecture decisions
   - Which operating system your installer is for
   - Known limitations
7. Submit a **10-minute video**, face on camera, by **Wed 30 Sep, 11:59 pm IST**, walking through your code and the decisions behind it:
   - Your database schema design decisions
   - Application architecture, including how the main process, preload bridge and UI communicate
   - Security: how sign-in works, where the session is stored, how your API protects each user's data, and how you kept secrets out of the app
   - How the app is packaged into an installer
   - Testing approach (E2E, unit tests)

## Getting Started

1. Read both project briefs carefully
2. Choose ONE project
3. Notify us which project you've chosen
4. Begin development — we suggest getting a signed-in "hello world" app building into an installer early, before the main features

## Contacts

- Saura — Founder — saura@spgcapital.com.au
- Sid — Engineer — sid@spgcapital.com.au

If anything here is unclear, email either of us.

Good luck!
