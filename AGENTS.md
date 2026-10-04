# Frontend fork guidance

This is the Jellyfin Web fork, paired with the sibling `../media-server` backend.
See [local development](../media-server/DEVELOPMENT.md) and
[feature development](../media-server/docs/FEATURE_DEVELOPMENT.md) in that checkout.
For a standalone clone, the same guides are linked from this repository's README.

## Architecture

- `src/index.jsx`: initialization and plugin loading.
- `src/RootApp.tsx`, `RootAppRouter.tsx`: providers and hash routing.
- `src/apps/modern`, `legacy`, `dashboard`, `wizard`: application areas.
- `src/apps/legacy/controllers/playback/video/`: shared legacy video view/controller,
  also reused by the modern video page.
- `src/plugins/syncPlay/`: group updates, playback synchronization, and client requests.
- `src/plugins/htmlVideoPlayer/`: HTML video playback implementation.
- `src/hooks/useApi.tsx`: SDK and legacy API client context.
- `src/styles/` and `src/strings/en-us.json`: shared styles and source-language labels.

The UI mixes React/TypeScript with legacy JavaScript/HTML. Trace the active route and
view before changing controls. Frontend plugins are not server plugins.

## Implementation conventions

Follow `CONTRIBUTING.md` and existing formatting, with these fork-specific clarifications:

- Use Conventional Commits for this fork, overriding the upstream guide's preference
  against them. Example: `feat(syncplay): add reaction picker`.
- New source files should be TypeScript; new pages should use the existing React page
  patterns. Avoid broad legacy rewrites when extending existing behavior.
- Prefer the Jellyfin TypeScript SDK. A fork-only endpoint absent from the generated
  SDK can use a small adapter through the existing authenticated API client; the reaction
  request in `src/plugins/syncPlay/core/Controller.js` is an existing example.
- Add labels to `en-us.json`; avoid editing translated dictionaries by hand.
- Check TV/legacy browser compatibility, keyboard/touch access, reduced motion, and
  cleanup of event listeners, timers, and overlays for lifecycle-sensitive UI changes.
- Keep stable IDs on the wire; ignore unknown IDs and updates for another active group.

## Build and checks

Use Node >=24 and npm >=11 (`package.json` is authoritative).
`npm ci` installs locked dependencies; `npm start` runs Webpack;
`npm run build:production` builds assets. Vite config is for Vitest, not the app dev server.

Run checks appropriate to the change:

- `npm run build:check`: TypeScript checking.
- `npm run lint -- <changed-source-files>`: targeted ESLint.
- `npx stylelint <changed-style-files>`: targeted stylesheet checks.
- `npm test -- <test-file>`: focused Vitest; `npm test` for the complete suite.
- Build and manually exercise the player for legacy UI changes that lack test harnesses.

Test shared player changes in relevant layouts and with the modified backend. Record
actual results and unrun checks. Preserve unrelated edits and stage files explicitly.
Do not commit `node_modules`, `dist`, coverage, credentials, or runtime data.
