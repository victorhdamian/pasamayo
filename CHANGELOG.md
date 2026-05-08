# Changelog

All notable changes to Pasamayo are documented here.

## [v0.0.1] — 2026-05-08

### Added
- AES-256-GCM encryption with PBKDF2 + SHA-512 key derivation
- Randomised PBKDF2 iteration count per vault
- HMAC integrity check with constant-time comparison
- Payload padding to 1KB blocks
- Rolling backup — last 5 snapshots stored inside encrypted file
- Password generator — CSPRNG-based with Fisher-Yates shuffle
- Fuzzy search by service name and username
- Month filter calendar picker
- Paginated list — 10 entries per page
- Inline edit with createdAt / updatedAt timestamps
- Delete confirmation — requires typing service name
- Offline assistant — recovery, weak password detection, vault audit
- XSS sanitization on all user-controlled data
- File System Access API for in-place vault updates (Chrome/Edge)
- Download fallback for Firefox/Safari
- PWA support — installable from browser
- AGPL-3.0 + commercial dual license
- Landing page at pasamayo.app
- Anti-phishing headers via Cloudflare
- SPF + DMARC + CAA DNS protection
