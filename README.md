# Pasamayo

A minimal KV password manager in a single HTML file. No install, no server, no dependencies.

🌐 **[pasamayo.app](https://pasamayo.app)** — download, donate, or open the app directly in your browser.

## How to Use

1. Visit [pasamayo.app](https://pasamayo.app) and click **Download free**
2. Open `pasamayo.html` in any browser
3. Click **"Create new vault"**, set a master password → saves a `.pasamayo` file to your disk
4. Next time: open the app, click **"Open vault"**, pick your `.pasamayo` file, enter master password

## Features

- **Single file** — one HTML file, works offline, runs anywhere
- **File-based vault** — your data lives in a `.pasamayo` file on your disk, not in the browser
- **AES-256-GCM** encryption with PBKDF2 + SHA-512 key derivation
- **Randomised iteration count** — unique per vault, stored in the file header
- **HMAC integrity check** — tamper detection before decryption is attempted
- **Payload padding** — file size reveals nothing about entry count or password lengths
- **Rolling backup** — last 5 vault snapshots stored inside the encrypted file
- **Password generator** — CSPRNG-based, configurable charset, guaranteed character sets
- **Fuzzy search** — filter by service name or username as you type
- **Month filter** — calendar picker to filter entries by creation or update month
- **Paginated list** — 10 entries per page, works with search and date filter
- **Assistant** — offline rule-based helper for recovery, security questions, and weak password detection
- **XSS protection** — all user input sanitized before rendering
- **Zero dependencies** — no frameworks, no build step, no server
- **Portable** — copy `.pasamayo` to any machine, open with any browser
- **Open source** — AGPL-3.0, read every line of code yourself

## Data Model

Each entry is a key-value record indexed by service name:

```
service (key) → { username, password, createdAt, updatedAt }
```

The entire vault is encrypted as a single blob — no partial access possible.

## Navigation

Everything is 1–2 steps:

| Action | Steps |
|--------|-------|
| Open vault | Type password → Open vault → pick file |
| Create vault | Type password → Create new vault |
| Add entry | Click + Add → fill fields → Enter |
| Edit entry | Click row → edit inline → Update |
| Copy password | Click "Copy pw" |
| Copy username | Click "Copy user" |
| Delete entry | Click ✕ → type service name to confirm |
| Search | Type in search box — filters instantly |
| Filter by month | Click 📅 → pick month from calendar |
| Generate password | Click ⚙️ Generate → configure → Use this → |
| Restore backup | Click 🕓 Restore → pick snapshot |
| Ask assistant | Click 🤖 → type your question |
| Lock | Click 🔒 Lock |

## Assistant

Click the 🤖 button (bottom-right, always visible) to open the assistant. Runs entirely offline — no data leaves your device.

It can help with:
- **Recovery** — forgot master password, corrupted vault, can't open file
- **Security questions** — how encryption works, what protects your data
- **Weak passwords** — scans the open vault for short or reused passwords
- **Vault health audit** — full report: weak, reused, undated entries and available snapshots
- **Feature navigation** — clickable links that open the relevant panel directly

## Password Generator

- Uses `crypto.getRandomValues` — same CSPRNG as the rest of the app
- Fisher-Yates shuffle eliminates positional bias
- Configurable: length (8–64), uppercase, lowercase, numbers, symbols
- Option to exclude ambiguous characters (0, O, 1, l, I)
- Guarantees at least one character from each selected set
- Click generated password to copy — or "Use this →" to fill the add form directly

## Vault Backups

Before every save, the current vault state is pushed into a history stack of up to 5 snapshots. Snapshots are stored inside the encrypted payload — same security as the vault itself. To restore: click "🕓 Restore from backup", pick a snapshot, confirm.

## Security Model

- All crypto uses the browser's native [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- Passwords are never stored in plaintext
- Nothing is stored in the browser — no localStorage, no cookies, no cache
- No network requests — works fully offline
- The master password cannot be recovered if lost

### Hardening Against Brute Force & AI-Assisted Attacks

| Protection | Detail |
|---|---|
| PBKDF2 randomised iterations + SHA-512 | Iteration count is randomised per vault and stored in the file — attacker cannot pre-optimize their cracking setup without reading it first; SHA-512 is slower on GPUs where most cracking runs |
| HMAC integrity check | Wrong password fails before decryption is attempted — no decryption oracle is exposed |
| Constant-time comparison | HMAC bytes compared without short-circuiting — prevents timing attacks that infer how many bytes matched |
| Payload padding (1KB blocks) | Encrypted file size reveals nothing about entry count or password lengths |
| Independent key derivation | Encryption key and HMAC key use separate salts — compromising one does not help with the other |
| XSS sanitization | All user-controlled data escaped before rendering — prevents script injection via vault entries |

### Vault File Layout

```
iter(4) | salt(16) | hSalt(16) | iv(12) | mac(32) | ciphertext
```

- `iter` — random PBKDF2 iteration count, unique per vault
- `salt` — PBKDF2 salt for the AES-256-GCM encryption key
- `hSalt` — PBKDF2 salt for the HMAC-SHA-256 integrity key
- `iv` — AES-GCM initialisation vector
- `mac` — HMAC over the ciphertext, verified before any decryption attempt
- `ciphertext` — AES-256-GCM encrypted, padded vault payload

## Compatibility

| Browser | Open | Save |
|---------|------|------|
| Chrome / Edge | Native file picker, writes back in place | Silent in-place update |
| Firefox / Safari | File picker | Downloads updated file |

## Project Structure

```
/                       ← landing page (pasamayo.app)
/app/index.html         ← the password manager
/app/test.html          ← test harness
/manifest.json          ← PWA manifest
/sw.js                  ← service worker (offline support)
/icons/                 ← app icons
```

## Files

| File | Purpose |
|------|---------|
| `app/index.html` | The app — open this in any browser |
| `app/test.html` | Test harness — open in browser to run all tests |
| `*.pasamayo` | Your encrypted vault — keep this safe |
| `README.md` | This file |
| `COMMERCIAL.md` | Commercial licensing terms |
| `SECURITY.md` | Security policy and vulnerability reporting |

## Testing

Serve locally and open `http://localhost:8080/app/test.html`, then click **▶ Run all tests**.

```bash
python3 -m http.server 8080
```

Covers 55 tests across 10 suites:
- Crypto encrypt / decrypt
- Padding and unpadding
- Constant-time comparison
- Vault structure and history
- Password generator
- XSS sanitization
- Fuzzy search
- Date filtering
- Assistant routing
- Assistant password analysis and vault audit

## Contributing

Security vulnerabilities: see [SECURITY.md](SECURITY.md).

All other contributions: open a pull request against `main`.

## License

Free for personal and open source use under [AGPL-3.0](LICENSE).
Commercial use requires a license — see [COMMERCIAL.md](COMMERCIAL.md).
