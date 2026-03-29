  

## Development & Git

  

  

- **Manual Git:** No autocommits/autopushes. Provide the message; I commit.

  

  

- **Local Dev:** Use Windows. Default to a random port (not 3000).

  

  

## Tech Stack

  

  

- Default: Node.js (ES6), Fastify with `{ trustProxy: true }`

  

  

- **Config:** Use `dotenv`. Load via: `path: ./config/.env.${process.env.APP_ENV || 'local'}`.

  

  

- **Process:** Managed by PM2 via `ecosystem.config.cjs` on VPS

  

  

- **No Builds:** App runs from source on VPS; no transpilation/bundling unless asked.

  

  

- **Health:** App must have a GET /health endpoint that returns a JSON object containing the current ISO timestamp and the app version read from a `VERSION` file at the project root. Example response: `{ "timestamp": "2026-01-01T00:00:00.000Z", "version": "1.0.0" }`. The VERSION file is read once at startup (changing it requires a restart). Bump it manually before each deploy to confirm the deployment landed by hitting /health after the deploy.

  

- **VERSION file:** Every project must have a `VERSION` file at the project root containing a plain version string (e.g. `1.0.0`). Increment it (or use a date/label like `2026-02-25-hotfix`) before each deploy. It is committed to git. The health route reads it at startup via `readFileSync`.

  

- **Graceful Shutdown:** Handle SIGTERM and SIGINT signals. When received, call app.close() to allow Fastify to finish processing existing requests before the process exits.

  

  

  

## Routing & Paths (CRITICAL)

  

  

- **Environment:** Local runs at `/`. VPS runs behind Nginx at `/subpath/`.

  

  

- **Rule:** Never use root-relative paths (`/assets/...`).

  

  

- **Rule:** Always use relative paths (`./assets/...`) or `./` prefix for all UI links, CSS, JS, and API fetches.

  

  

- **Static Files:** Serve source folders directly using @fastify/static (no build step).

  

  

- **Nginx Note:** Nginx handles the subpath strip via `proxy_pass http://127.0.0.1:PORT/;`. Code should remain agnostic of the specific subpath name.

  

  

  

## Documentation & Context (New)

  

  

- **Agent Handoff:** Every project must include a README.md that is updated with project arhitecture, decisions, api paths, etc. Meant for humans to understand.

  

  

- **Inline Context:** Use JSDoc or clear comments to explain the intent of logic, ensuring another AI agent or developer can pick up the context immediately.

  

  

  

## Context Handoff File

  

  

  

Maintain a `CONTEXT.md` file at the project root that documents the chat history, decisions, and current state. This file is read by other AI agents to understand the project without re-reading the full conversation.

  

  

  

### CONTEXT.md Format

  

  

  

Update `CONTEXT.md` after each significant decision or change. Use this structure:

  

  

```markdown

  

  

# Project Context

  

  

  

## Overview

  

  

[1-2 sentence description of what this project does]

  

  

  

## Current Status

  

  

[Current state: what's been built, what's in progress, blockers if any]

  

  

  

## Key Decisions

  

  

- **Decision**: [What was decided]

  

  

  - **Rationale**: [Why]

  

  

  - **Date/Chat**: [When and where it was decided]

  

  

  

## Architecture

  

  

[Diagram or description of how components fit together]

  

  

  

## Recent Changes

  

  

- [What changed and why]

  

  

- [Format: "Changed X because Y, see commit/file Z"]

  

  

  

## Next Steps

  

  

- [ ] Task 1

  

  

- [ ] Task 2

  

  

  

## Notes for Next Agent

  

  

- [Anything important for handoff]

  

  

- [Known issues]

  

  

- [Assumptions made]

  

  

  

## File Structure

  

  

\`\`\`

  

  

project-root/

  

  

├── server.js

  

  

├── package.json

  

  

├── CONTEXT.md (this file)

  

  

├── config/

  

  

│   ├── .env.local

  

  

│   └── .env.vps

  

  

└── public/

  

  

    └── index.html

  

  

\`\`\`

  

  

  

## Environment Setup

  

  

[How to run locally, what env vars are needed]

  

  

  

## Related Docs

  

  

- README.md: Setup and usage instructions

  

  

- server.js: Code comments explain key logic

  

  

```

  

  

  

### Guidelines

  

  

  

- Keep it concise—focus on decisions and state, not a transcript

  

  

- Update it whenever you make architectural changes, add features, or hit a blocker

  

  

- Include links to relevant files, commits, or previous chats if available

  

  

- Make it scannable: use short sentences, bullets, and sections

  

  

- Assume the next agent knows general programming but not this project's specifics

  

  

## UI Architecture

  

  

- **Prefer static SPA over templates**: Always serve UI as plain HTML + vanilla JS from a `public/` directory using `@fastify/static`. Never use server-side template engines (e.g., Eta, `@fastify/view`).

  

  

- **API pattern**: All data goes through JSON API routes (e.g., `/api/users`, `/api/environments`). The HTML page fetches data via `fetch()` and renders it client-side.

  

  

- **Browser-redirect routes**: Routes that receive form POSTs from external services (e.g., OAuth/MFA callbacks) may still do server-side `302` redirects to static HTML pages with data encoded in query params (e.g., `?claims=<b64>`).

  

  

- **Dependencies**: Use `@fastify/static` for serving `public/`. Remove `eta` and `@fastify/view` from all projects.

  

  

  

## If Forced to Use Eta Templates with @fastify/view because user specifically asked for it

  

  

- **No global layout**: Do NOT configure a global `layout` in `@fastify/view` options. On Windows, Eta resolves layout paths using OS path separators which causes lookup failures. Instead, make each template a self-contained full HTML page (no layout inheritance).

  

  

- **No native JS array methods inline**: Eta templates do not support `.map()`, `.filter()`, `.forEach()` directly on arrays. Assign to a local variable first or use a `for` loop with `<% for (const x of it.items) { %>`.

  

  

- **Use `<%~` for unescaped HTML, `<%=` for escaped**: When rendering HTML strings from the server, use `<%~ val %>`. Use `<%= val %>` only for plain text that should be HTML-escaped.

  

  

- **Always resolve views path with `__dirname`**: Register with `root: resolve(__dirname, 'views')` — do not use relative strings like `'./views'`.

  

## Sample nginx configuration on vps

  

  

```

  

  

location /myapp/ {

  

  

  proxy_pass http://127.0.0.1:5001/;

  

  

  proxy_set_header X-Real-IP $remote_addr;

  

  

  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

  

  

}

  

  

```

  

  

  

## Sample .env.vps file

  

  

```env

  

  

PORT=5002

  

  

ISSUER=https://test.scrambleid.com/oidc-server

  

  

```

  

  

APP_ENV must never appear in .env.* files. It is set externally (by PM2's

  

  

ecosystem.config.cjs or the shell) and is read before dotenv.config() is called to select which .env file to load. Putting it inside a .env file either duplicates it uselessly or risks silently overriding the correct externally-set value if dotenv ever loads early.

  

  

  

## Sample content of ecosystem.config.cjs

  

  

```module.exports = {

  

  

  apps : [{

  

  

    name   : "myapp",

  

  

    script : "./server.js",

  

  

    env_local: {

  

  

      APP_ENV: "local"

  

  

    },

  

  

    env_vps: {

  

  

      APP_ENV: "vps"

  

  

    }

  

  

  }]

  

  

}

  

  

```