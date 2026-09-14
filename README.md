# דרור — Dror

An AI documentation assistant for Israeli psychologists, built around a Hebrew, right-to-left, mobile-first interface.

[Product video & case study](https://sagi-argaman-portfolio.vercel.app/#dror) · [Hosted app](https://dror-b44-6f3cb1e4.base44.app) · [Checks](https://github.com/argamandev/Dror/actions/workflows/ci.yml)

Dror brings session notes, recordings, and supporting documents into a patient workspace. The assistant helps a psychologist retrieve context and prepare session summaries and clinical letters for review. This repository contains the React frontend and the Base44 backend configuration for the product's Base44 implementation.

<a href="https://sagi-argaman-portfolio.vercel.app/#dror"><img src="docs/images/dror-preview.jpg" width="240" alt="Dror Hebrew patient-context chat, captured from the product walkthrough"></a>

## Product workflow

- **Find context:** ask questions about a patient's previous sessions and uploaded documents.
- **Capture a session:** type notes, dictate, or record audio with Hebrew transcription.
- **Prepare documentation:** generate a structured session summary or a formal letter from selected sessions, then review and edit it.
- **Work in Hebrew:** use a responsive RTL interface designed around a persistent chat entry point and patient records.

The hosted app requires authentication and a working Base44 backend. The portfolio video is the quickest way to inspect the experience without an account. Use synthetic records when exploring this implementation; it is not presented as a clinically certified deployment.

## Engineering overview

| Area | Implementation | Start here |
| --- | --- | --- |
| UI | React 18, TypeScript, Vite, Tailwind CSS | [`src/screens`](src/screens), [`src/state`](src/state) |
| Integration boundary | Base44 SDK access grouped under `src/api` | [`base44Client.ts`](src/api/base44Client.ts), [`data.ts`](src/api/data.ts) |
| Assistant | Managed Base44 agent with record-reading and entry-creation tools; subscription transport with polling fallback | [`dror.jsonc`](base44/agents/dror.jsonc), [`ai.ts`](src/api/ai.ts) |
| Draft generation | Authenticated Deno functions returning structured summary/letter output | [`summarize`](base44/functions/summarize/entry.ts), [`document`](base44/functions/document/entry.ts) |
| Speech | Browser speech recognition with server transcription fallback using ElevenLabs | [`stt`](base44/functions/stt/entry.ts), [`src/hooks`](src/hooks) |
| Persistence | Five entity schemas with ownership-based access rules: patients, entries, chats, documents, preferences | [`base44/entities`](base44/entities) |

Base44 supplies hosted authentication, persistence, agent execution, and backend functions. This is not a standalone offline backend; the repository makes the application's schemas, functions, agent instructions, and frontend available for inspection.

## Run and verify locally

Use Node.js **22.21.1** (pinned in [`.nvmrc`](.nvmrc)) and npm.

```bash
git clone https://github.com/argamandev/Dror.git
cd Dror
npm ci
npm run check
npm run dev
```

`npm run check` runs unit tests, frontend TypeScript checking, and a production build. These checks need no Base44 credentials or API keys. `npm run dev` starts Vite; the application itself connects to the existing hosted Base44 app configured in [`src/api/base44Client.ts`](src/api/base44Client.ts), so authenticated features depend on that service. Base44 CLI login is not browser login and is not required for the local checks.

| Command | Purpose |
| --- | --- |
| `npm test` | Vitest unit suite |
| `npm run typecheck` | Type-check `src` without emitting files |
| `npm run build` | Build the frontend into `dist` |
| `npm run preview` | Preview the built frontend locally |

The current suite has **152 tests across 13 files**, covering formatting, chat scope, document guards, preferences, microphone-error handling, recording/dictation decisions, and UI helpers. These are unit checks: they do not establish live backend isolation, model output quality, or full browser recording compatibility. GitHub Actions runs the same `npm run check` command on pushes and pull requests. Deno backend functions are outside the frontend TypeScript configuration.

### Use your own Base44 backend

Create/link a separate Base44 app using the Base44 CLI, then update the public `appId` in [`base44Client.ts`](src/api/base44Client.ts) to match it. The checked-in app identifier is not an API secret. Deploy the schemas, agent, auth configuration, functions, and frontend to your own app; server transcription also requires an `ELEVENLABS_API_KEY` platform secret. Never put that key in frontend code.

[`scripts/seed.ts`](scripts/seed.ts) and [`scripts/verify-rls.ts`](scripts/verify-rls.ts) are optional authenticated backend tools. They are not part of CI; seeding writes records. See [backend setup](docs/backend-setup.md) for the explicit commands and scope.

## Current limitations and next steps

- **Draft enforcement:** the agent is instructed to create drafts, but that value is not enforced by a server-side guard. The Entry schema defaults `is_draft` to `false`. Server-side enforcement is a remaining improvement, not a guarantee of this implementation.
- **Long inputs:** the summary function limits source notes/transcripts to 6,000 characters and uses bounded history excerpts. Long input handling needs a visible limit or a chunking strategy.
- **Data handling:** audio may reach the browser's speech service or ElevenLabs, and AI processing uses Base44 integrations. Vendor retention terms and clinical deployment requirements need separate review.
- **Isolation evidence:** schemas express ownership rules. The [historical build log](docs/context/build-log.md) records a manual second-account check; this is not a continuous integration test or an independent security audit.

## Project context

Built by **Sagi Argaman**, with Claude as an AI coding collaborator. The Base44 implementation originated during the Base44 Dev Build-Off; [historical notes](docs/context/build-off-notes.md) preserve that context, and the [build log](docs/context/build-log.md) records implementation decisions and verification at the time. The current README describes the repository as it stands rather than the competition submission.
