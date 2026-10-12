# SocialPilot AI Frontend

The SocialPilot AI frontend is a React and TypeScript application built with Vite. It includes a presentation landing page and a project workspace for importing project material, reviewing analysis and sources, generating LinkedIn drafts and case studies, editing artifacts, and exporting case studies.

The landing page also contains illustrative product concepts for a developer community and wallet/payment experience. These panels are presentation content, not connected social publishing or blockchain features.

## Features

- Product landing page with entry to the project workspace.
- Import modal for public GitHub repository URLs or pasted Markdown.
- Project overview with analysis action and project profile.
- Sources screen for reviewing ingested project material.
- Content Studio for generating and editing drafts across three LinkedIn angles.
- Case study generation, editing, and Markdown/PDF export.
- Claim review UI for displaying generated content's evidence metadata.
- API client with request error handling and file download helpers.

The frontend depends on the FastAPI backend for project persistence, analysis, generation, and export. See [`Backend/README.md`](../../Backend/README.md) for backend setup and endpoint details.

## Technology

- React 19
- TypeScript 6
- Vite 8
- Tailwind CSS 4 with the Vite plugin
- Lucide React icons

## Requirements

- Node.js and npm compatible with the Vite version in `package.json`.
- The SocialPilot FastAPI backend running at `http://localhost:8000` for the workspace features.

## Run locally

From this directory (`Frontend/socilaPilot`):

```powershell
npm install
npm run dev
```

Open the local address printed by Vite, usually `http://localhost:5173`.

Start the backend separately from the repository's `Backend` directory:

```powershell
Set-Location ..\..\Backend
uvicorn api.main:app --reload --host 127.0.0.1 --port 8000
```

The frontend development server proxies `/projects`, `/export`, and `/health` to `http://localhost:8000`, as configured in `vite.config.ts`.

The API client also reads `VITE_API_URL`. If needed, create `.env.local` in this frontend directory:

```dotenv
VITE_API_URL=http://localhost:8000
```

Only use a browser-accessible API URL here. Vite exposes `VITE_` variables in client-side code; never place credentials or secrets in them. For a separate production API origin, configure appropriate backend CORS and deployment proxy settings; the current backend does not enable CORS by default.

## User workflow

1. Select **Open workspace** on the landing page.
2. Import a public GitHub repository or paste Markdown project notes.
3. Select the project and run analysis.
4. Review the generated profile and imported sources.
5. Open Content Studio and choose technical lesson, engineering decision, or personal reflection.
6. Add optional user context and generate a draft.
7. Review the draft's claim evidence, make edits, and save.
8. Generate or edit a case study and export it as Markdown or PDF.

An LLM service must be configured on the backend for analysis and generation. PDF export additionally requires the Typst executable on the backend host; Markdown export does not.

## Routes and screens

This frontend uses application state to switch workspace sections rather than URL-based section routes.

| Screen | Purpose |
| --- | --- |
| Landing page | Present SocialPilot AI and open the workspace. |
| Overview | Select/import projects, inspect profile, and trigger analysis. |
| Content studio | Generate, review, edit, and save LinkedIn drafts. |
| Case studies | Generate, edit, save, and export a case study. |
| Sources | Inspect imported source documents and run/re-run analysis. |

## Source layout

```text
src/
├── App.tsx                 # Workspace state and API actions
├── components/             # Shared shell, import modal, evidence UI
├── landing/                # Landing page sections
├── pages/                  # Landing and workspace screens
├── services/api.ts         # API requests and export/download helpers
├── types.ts                # Frontend API and UI types
├── App.css
└── index.css
```

## Available scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite development server. |
| `npm run build` | Run TypeScript project checks and create a production build in `dist/`. |
| `npm run lint` | Run ESLint. |
| `npm run preview` | Preview the production build locally. |

Run checks from this directory:

```powershell
npm run lint
npm run build
```

## Frontend/backend contract

The frontend's API client calls these backend routes:

- `GET /projects`
- `POST /projects/import`
- `GET /projects/{project_id}`
- `POST /projects/{project_id}/analyze`
- `POST /projects/{project_id}/linkedin/{angle}`
- `POST /projects/{project_id}/case-study`
- `PUT /projects/{project_id}/artifacts/{artifact_type}`
- `POST /export/markdown`
- `POST /export/pdf`

Treat the backend's FastAPI OpenAPI documentation at `http://localhost:8000/docs` as the definitive request/response contract.

## Current scope and limitations

- The current frontend is a single-user project workspace; the backend does not currently provide authentication or project ownership.
- Landing-page social/community and wallet concepts are visual/illustrative and do not publish to external social networks or submit blockchain transactions.
- Import supports public GitHub repositories and pasted Markdown through the API; private GitHub access and local directory upload are not implemented in the UI.
- Evidence labels assist human review and should not be treated as factual guarantees.
- PDF generation depends on Typst being installed on the backend host.

## License

This project is licensed under Apache License 2.0. See the root [`LICENSE`](../../LICENSE).
