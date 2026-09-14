# Backend setup

Local unit tests and frontend builds do not require the Base44 CLI. These steps are for maintainers who want to operate a separate backend; they create or change remote resources.

```bash
npx base44 login
npx base44 link --create --name dror-development
```

Use the new app identifier in `src/api/base44Client.ts` before building. Confirm the linked app is your intended development app. The local link file is ignored by Git.

```bash
npm ci
npm run check
npx base44 deploy
```

Configure `ELEVENLABS_API_KEY` as a server-side platform secret for the `stt` function. Authentication and the other AI integrations depend on your Base44 app configuration and account availability. The frontend cannot run these integrations offline.

Optional synthetic data and scoped-read checks (authenticated CLI session required):

```bash
# Bash
cat scripts/seed.ts | npx base44 exec
cat scripts/verify-rls.ts | npx base44 exec
```

```powershell
# PowerShell
Get-Content -Raw scripts/seed.ts | npx base44 exec
Get-Content -Raw scripts/verify-rls.ts | npx base44 exec
```

The seed script writes records under the logged-in account. The verification script checks one account's visible records and ownership; it does not by itself demonstrate cross-account isolation. Test with a second account in your development app before relying on access rules.
