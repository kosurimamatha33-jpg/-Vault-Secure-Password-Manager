# 🔐 Vault — Secure Password Manager

A full-stack password manager built with real cryptography. Your passwords are encrypted in the browser before they ever leave your device — the server stores only ciphertext it cannot read.


---

## ✨ Features

- 🔑 **Master password login** — one password to unlock everything
- 🔒 **AES-256-GCM encryption** — every field (website, username, password) is encrypted individually in the browser before being sent to the server
- 🧪 **PBKDF2 key derivation** — 100,000 iterations of SHA-256 to make brute-forcing slow
- 🛡️ **Zero-knowledge architecture** — the server never sees your master password or any plaintext data
- 💥 **Tamper detection** — GCM authentication tags catch any modification to stored ciphertext
- 🎲 **Strong password generator** — cryptographically random passwords via `window.crypto`
- 📊 **Password strength meter** — real-time feedback as you type
- 🔍 **Search & filter** — instantly search entries by website or username
- ✏️ **Edit entries** — update saved passwords without deleting and re-adding
- 📋 **One-click copy** — copy passwords to clipboard without revealing them
- 🔐 **Rate limiting** — auth endpoints limited to 20 requests per 15 minutes
- 🪖 **Security headers** — powered by Helmet.js
- 🔄 **Auto-lock on reload** — derived key is never persisted anywhere, requiring re-authentication on page reload

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        BROWSER                              │
│                                                             │
│  Master Password ──► PBKDF2 (100k iterations) ──► AES Key  │
│                                     │                       │
│  Plaintext Entry ──► AES-256-GCM ──► Ciphertext + IV       │
│                                     │                       │
│                              fetch() to API                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      BACKEND (Node.js)                      │
│                                                             │
│  POST /api/vault  ◄── receives ciphertext only             │
│  Stores in SQLite ◄── server has no decryption key         │
│                                                             │
│  POST /api/auth/login ◄── bcrypt verifies master password  │
│  Returns JWT + KDF salt ◄── client re-derives key locally  │
└─────────────────────────────────────────────────────────────┘
```

### Why this is secure

The master password is sent to the server **once** at login — only to verify identity via bcrypt (12 rounds). It is never stored or logged. The actual **encryption key** is derived independently in the browser using PBKDF2 and never transmitted. The server is just a sync point for ciphertext it has no way to decrypt.

---

## 🗂️ Project Structure

```
password-manager/
├── backend/
│   ├── server.js               # Express app, security middleware
│   ├── package.json
│   ├── .env.example            # Environment variable template
│   ├── .gitignore
│   ├── db/
│   │   └── database.js         # SQLite schema (auto-created on first run)
│   ├── middleware/
│   │   └── auth.js             # JWT verification middleware
│   └── routes/
│       ├── auth.js             # POST /api/auth/register, /login
│       └── vault.js            # GET/POST/PUT/DELETE /api/vault
└── frontend/
    ├── index.html              # App structure — auth screen + vault UI
    ├── style.css               # Design system, responsive layout
    ├── crypto.js               # All Web Crypto API logic (AES-256-GCM, PBKDF2)
    ├── api.js                  # Fetch wrapper for backend communication
    └── script.js               # UI logic, form handling, rendering
```

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript |
| Encryption | Web Crypto API (AES-256-GCM + PBKDF2) |
| Backend | Node.js + Express |
| Database | SQLite via `better-sqlite3` |
| Auth | JWT (jsonwebtoken) + bcrypt |
| Security | Helmet.js, express-rate-limit, CORS |

---

## 🚀 Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v18 or higher
- Python 3 (for serving the frontend — comes pre-installed on most systems)

### 1. Clone the repository

```bash
git clone https://github.com/kosurimamatha33-jpg/-Vault-Secure-Password-Manager.git
cd vault-password-manager
```

### 2. Install backend dependencies

```bash
cd backend
npm install
```

> **Windows users:** If `better-sqlite3` fails to install, you need C++ build tools. Run `npm install -g windows-build-tools` as administrator, or install "Desktop development with C++" via Visual Studio Installer.
>
> **Mac users:** If you see build errors, run `xcode-select --install` first.

### 3. Create your environment file

```bash
cp .env.example .env
```

On Windows (PowerShell):
```powershell
copy .env.example .env
```

### 4. Generate a JWT secret

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

Copy the output and paste it into `backend/.env`:

```env
PORT=3001
FRONTEND_ORIGIN=http://localhost:5500
JWT_SECRET=your_generated_secret_here
```

### 5. Start the backend

```bash
npm start
```

You should see:
```
Password manager backend running on http://localhost:3001
```

The SQLite database (`backend/db/vault.db`) is created automatically on first run — no setup needed.

### 6. Serve the frontend

Open a **second terminal** and run:

```bash
cd frontend
python -m http.server 5500
```

Or on some systems:

```bash
python3 -m http.server 5500
```

**Alternative:** Install the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) VS Code extension, right-click `frontend/index.html`, and choose **Open with Live Server**.

### 7. Open in browser

Visit `http://localhost:5500`

---

## 🖥️ Running in VS Code

Open two terminals side by side (**Terminal → New Terminal**, then click **+**):

| Terminal | Folder | Command |
|---|---|---|
| **Terminal 1** | `backend/` | `npm start` |
| **Terminal 2** | `frontend/` | `python -m http.server 5500` |

Both must be running simultaneously for the app to work.

---

## 📖 Usage

1. **Create a vault** — click "Create one" on the login screen and choose a strong master password
2. **Add entries** — fill in the website, username, and password (or click "Generate strong password")
3. **Copy a password** — click "Copy" on any entry; the password goes to your clipboard without being shown on screen
4. **Reveal a password** — click "Show" to temporarily display it in plaintext
5. **Edit an entry** — click "Edit", make your changes, click "Update entry"
6. **Search** — type in the search box to filter entries by website or username
7. **Lock your vault** — click "Lock vault" in the top-right; the in-memory key is wiped immediately

> ⚠️ **Your master password cannot be recovered.** It is never stored anywhere in reversible form. If you forget it, your vault data cannot be decrypted. This is intentional — the same tradeoff real password managers make.

---

## 🔒 Security Details

| Mechanism | Implementation |
|---|---|
| Vault encryption | AES-256-GCM (authenticated encryption) |
| Key derivation | PBKDF2-SHA256, 100,000 iterations, per-user random salt |
| Password hashing | bcrypt, 12 salt rounds |
| Session tokens | JWT, 2-hour expiry |
| Auth rate limiting | 20 requests / 15 minutes per IP |
| HTTP security headers | Helmet.js defaults |
| CORS | Locked to configured frontend origin |
| Input validation | Enforced on both client and server |
| Ownership checks | Every vault operation verifies the entry belongs to the requesting user |

---

## ⚠️ Limitations

This is a portfolio/learning project demonstrating the correct cryptographic architecture. Before using it to store real credentials:

- Deploy with **HTTPS** — never HTTP in production, as the master password is transmitted over the network
- Add **refresh token rotation** for longer sessions without reducing security
- Implement **account lockout** after repeated failed login attempts
- Consider a proper **security audit** before storing sensitive data

---

## 📄 License

MIT — free to use, modify, and distribute.

---

## 🙏 Acknowledgements

Cryptographic design inspired by [Bitwarden's security whitepaper](https://bitwarden.com/images/resources/security-white-paper-download.pdf) and the [Web Crypto API specification](https://www.w3.org/TR/WebCryptoAPI/).
