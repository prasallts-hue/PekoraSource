# CANDY backend preparation plan

This repository is being evaluated as a possible backend for the CANDY frontend. This document records the preparation work still required. No production configuration or live service is assumed.

## Components identified

- `Roblox/Roblox.Website`: ASP.NET Core website/API targeting .NET 8.
- `frontend`: Next.js 12 / React 17 web frontend.
- `game-server`: Node.js/TypeScript service that manages RCC processes and rendering jobs.
- `game-renderer`: TypeScript rendering service.
- `AssetValidationServiceV2`: Go validation service.
- `RCCService`: legacy Windows RCC executables, libraries, and content.
- `game-arbiter`: required by the source notes but not included.

## Preparation order

1. Finish the security review and verify or remove untrusted binaries.
2. Replace the legacy `silrev.biz` RCC BaseUrl with the CANDY HTTPS domain.
3. Create `*.example.json` configuration templates for the website, frontend, and game-server. Keep real credentials outside Git.
4. Replace hard-coded Windows paths in startup scripts with repository-relative paths.
5. Document the database (PostgreSQL), cache (Redis), CDN/storage, CAPTCHA, Discord OAuth/bot, authorization secrets, and game-server settings.
6. Decide how the missing game-arbiter will be implemented or supplied.
7. Define the API contract used by CANDY login, registration, logout, and current-user/session checks.
8. Integrate the CANDY frontend and configure CORS, cookies, CSRF, HTTPS, and reverse-proxy rules.
9. Build and test each component separately in an isolated environment.
10. Deploy only after binary verification and end-to-end tests pass.

## Secrets and configuration

Never commit real values for database credentials, cookies, bot tokens, OAuth client secrets, authorization strings, signing keys, or CAPTCHA private keys. Use environment variables or deployment secret storage. Example files must contain obvious placeholders only.

## Current status

The repository is not ready for a safe production deployment. See [SECURITY-AUDIT-PRELIMINARY.md](./SECURITY-AUDIT-PRELIMINARY.md) for the preliminary findings and limitations.
