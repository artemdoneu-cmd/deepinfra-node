# GitHub Copilot Instructions for deepinfra-node 🔧

## Quick summary (what this repo is)
- This package is a TypeScript client library that wraps DeepInfra inference APIs and exposes typed model classes (e.g., `TextGeneration`, `Embeddings`, `TextToImage`, `Sdxl`). See `src/lib/models/base/*.ts` and `src/clients/deep-infra.ts`.
- Requests are sent via a small `DeepInfraClient` wrapper over `axios` with built-in retry/backoff (`src/clients/deep-infra.ts`, `src/lib/types/common/client-config.ts`, and `src/lib/constants/client.ts`).

---

## Architecture & data flow 💡
- Top-level exports live in `src/index.ts`, re-exporting model classes from `src/lib/models/base/index.ts`.
- Each model class extends `BaseModel` (or `CogBaseModel`) which:
  - resolves an endpoint from `modelName` or a full URL using `URLUtils.isValidUrl` (`src/lib/utils/url.ts`),
  - pulls `authToken` from the constructor arg or `DEEPINFRA_API_KEY` env var (`src/lib/models/base/base-model.ts`),
  - constructs a `DeepInfraClient` for HTTP requests.
- Binary inputs (images/audio) use `ReadStreamUtils` to accept `Buffer`, file path, base64 data-URL, or remote URL and convert to streams (`src/lib/utils/read-stream.ts`).
- `FormDataUtils.prepareFormData` turns request objects into `form-data` and accepts a `blobKeys` list for binary fields (`src/lib/utils/form-data.ts`).

---

## Principal patterns & conventions ✅
- Model usage: `const model = new TextGeneration(modelName, apiKey); const out = await model.generate(body);` (examples: `README.MD`).
- `modelName` may be either a model identifier (then `ROOT_URL + modelName` is used) or a full URL (detected via `URLUtils`).
- Error handling: models often log Axios error details and then throw a generic Error — keep logging behavior when adding new models for parity.
- Tests mock `axios` globally (see `test/base/*.spec.ts`): `jest.mock('axios', () => ({ create: jest.fn(() => ({ post: postMock }) }) ))`. Follow this pattern for unit tests.
- Path aliasing: code uses `@/*` path aliasing (`tsconfig.json`) — keep imports consistent with `@/...`.

---

## Developer workflows & commands 🔧
- Build: `npm run build` (runs `tsc` then `tsc-alias` to fix path aliases in `dist`).
- Test: `npm test` (Jest configured in `jest.config.cjs`, test files in `test/**/*.spec.ts`). Tests commonly mock `axios` to avoid network calls.
- Lint & format: `npm run lint` and `npm run prettier` (prettier also formats `README.MD`).
- Docs: `npm run build-docs` (typedoc) and `npm run deploy-docs` to push to GH Pages.
- Commit conventions: `commitizen` is configured (`cz-conventional-changelog`), and `prepare: "husky"` is present for git hooks — use conventional commit messages.

---

## What to watch for / helpful tips ⚠️
- When adding tests that would perform network requests, prefer mocking `axios` as in existing tests to keep unit tests hermetic and fast.
- If adding a model that accepts files, reuse `ReadStreamUtils.getReadStream` and `FormDataUtils.prepareFormData` to support Buffer/path/base64/URL inputs.
- Keep `ClientConfig` defaults in sync with `src/lib/constants/client.ts` (retry/backoff values). Changes there affect runtime retry behavior.
- When adding new top-level exports, update `src/lib/models/base/index.ts` so `src/index.ts` continues to re-export everything.

---

## Examples to reference in code reviews 🔎
- HTTP client & retry behavior: `src/clients/deep-infra.ts`
- Endpoint resolution and environment key: `src/lib/models/base/base-model.ts`
- File/binary handling: `src/lib/utils/read-stream.ts` and `src/lib/utils/form-data.ts`
- Test mocking pattern: `test/base/text-generation.spec.ts`
- Public usage examples: `README.MD` (copy small snippets into tests/examples if needed)

---

If anything above is unclear or you want more detail on a specific area (tests, adding a new model class, multipart uploads, or CI hooks), tell me which section to expand and I will iterate. ✅
