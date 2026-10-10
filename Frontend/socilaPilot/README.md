# SocialPilot AI

React + TypeScript + Vite presentation website and project-to-content workspace.

## Run locally

Start the FastAPI backend from `Backend/` first:

```bash
uvicorn api.main:app --reload
```

Then start the frontend from this directory:

```bash
npm install
npm run dev
```

Vite proxies `/projects`, `/export`, and `/health` to `http://localhost:8000`. To use a different backend URL in the frontend, set `VITE_API_URL` in the frontend environment.

## Workflow

The landing page presents the SocialPilot AI idea, its proposed MVP, technical approach, developer community, and blockchain payments vision. The proposed payment model includes direct creator tips and wallet payments for developer products or services; the marketplace and wallet screens are illustrative, not live. Use **Open workspace** to switch into the project studio.

The workspace flow is: import a public GitHub repository or paste a README, analyze the indexed sources, generate and edit LinkedIn drafts or a case study, review claim evidence, save edits, and export a case study as Markdown or PDF.

Landing-page community and wallet panels are illustrative product mockups, not live features. The MVP is under development. The UI uses the endpoints documented by the FastAPI app in `Backend/api/main.py`.

## Source layout

- `src/App.tsx` coordinates the workspace state and connects pages to API actions.
- `src/pages/HomePage.tsx` is the presentation website entry point.
- `src/landing/` contains the landing page and its independently maintained sections.
- `src/pages/` contains the Overview, Content Studio, Case Study, and Sources screens.
- `src/components/` contains shared layout, import dialog, headings, and evidence UI.
- `src/services/api.ts` contains backend requests and export/download helpers.
- `src/types.ts` defines the shared API and UI types.
- `src/App.css` and `src/index.css` contain the application and global styles.
