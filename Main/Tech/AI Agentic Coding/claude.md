

\## Development \& Git



\- \*\*Manual Git:\*\* No autocommits/autopushes. Provide the message; I commit.



\- \*\*Local Dev:\*\* Use Windows. Default to a random port (not 3000).





\## Tech Stack



\- Default: Node.js (ES6), Fastify with `{ trustProxy: true }`



\- \*\*Config:\*\* Use `dotenv`. Load via: `path: ./config/.env.${process.env.APP\_ENV || 'local'}`.



\- \*\*Process:\*\* Managed by PM2 via `ecosystem.config.cjs` on VPS



\- \*\*No Builds:\*\* App runs from source on VPS; no transpilation/bundling unless asked.



\- \*\*Health:\*\* App must have a GET /health endpoint that returns a JSON object containing the current ISO timestamp and the app version read from a `VERSION` file at the project root. Example response: `{ "timestamp": "2026-01-01T00:00:00.000Z", "version": "1.0.0" }`. The VERSION file is read once at startup (changing it requires a restart). Bump it manually before each deploy to confirm the deployment landed by hitting /health after the deploy.



\- \*\*VERSION file:\*\* Every project must have a `VERSION` file at the project root containing a plain version string (e.g. `1.0.0`). Increment it (or use a date/label like `2026-02-25-hotfix`) before each deploy. It is committed to git. The health route reads it at startup via `readFileSync`.



\- \*\*Graceful Shutdown:\*\* Handle SIGTERM and SIGINT signals. When received, call app.close() to allow Fastify to finish processing existing requests before the process exits.





\## Routing \& Paths (CRITICAL)



All apps run behind Nginx at a subpath on VPS. \*\*Every path in HTML, CSS, and JS must be relative.\*\*



\- \*\*Environment:\*\* Local runs at `http://localhost:PORT/`. VPS runs behind Nginx at `https://host/subpath/`.

\- \*\*How Nginx works:\*\* `proxy\_pass http://127.0.0.1:PORT/;` strips the subpath. The app always sees `/`. But the browser URL includes the subpath, so root-relative paths skip the subpath and break.



\### Mandatory — use relative paths everywhere



| Context | Correct | Wrong |

|---|---|---|

| CSS/JS in HTML | `href="./style.css"` | `href="/style.css"` |

| Fetch calls | `fetch('./api/data')` | `fetch('/api/data')` |

| Navigation links | `<a href="./page">` | `<a href="/page">` |

| Server redirects | `reply.redirect('./login')` | `reply.redirect('/login')` |

| Nested pages going up | `href="../shared.css"`, `fetch('../api/data')` | `href="/shared.css"` |



\### Forbidden



\- No root-relative paths starting with `/` in HTML, JS, or CSS (except full CDN URLs like `https://...`)

\- No `<base>` tag (dynamic or static)

\- No `getAppBase()` / `getAppApiUrl()` helpers that parse `window.location`

\- No `rewriteUrl` / `stripProxyPrefix` hacks on the server

\- No hardcoded subpath names anywhere in code

\- No double-mounting routes at both `/` and a `BASE\_PATH`



\### Correct server setup



```js

// This is all you need — no rewriteUrl, no prefix tricks

await fastify.register(fastifyStatic, {

&#x20; root: join(\_\_dirname, 'public'),

&#x20; prefix: '/'

});

```



\### Static Files

\- Serve source folders directly using @fastify/static (no build step).



\### Nginx Note

\- Nginx handles the subpath strip via `proxy\_pass http://127.0.0.1:PORT/;`. Code must remain agnostic of the specific subpath name.





\## Documentation \& Context



\- \*\*Agent Handoff:\*\* Every project must include a README.md that is updated with project architecture, decisions, api paths, etc. Meant for humans to understand.



\- \*\*Inline Context:\*\* Use JSDoc or clear comments to explain the intent of logic, ensuring another AI agent or developer can pick up the context immediately.





\## Context Handoff File



Maintain a `CONTEXT.md` file at the project root that documents the chat history, decisions, and current state. This file is read by other AI agents to understand the project without re-reading the full conversation.



\### CONTEXT.md Format



Update `CONTEXT.md` after each significant decision or change. Use this structure:



```markdown

\# Project Context



\## Overview

\[1-2 sentence description of what this project does]



\## Current Status

\[Current state: what's been built, what's in progress, blockers if any]



\## Key Decisions

\- \*\*Decision\*\*: \[What was decided]

&#x20; - \*\*Rationale\*\*: \[Why]

&#x20; - \*\*Date/Chat\*\*: \[When and where it was decided]



\## Architecture

\[Diagram or description of how components fit together]



\## Recent Changes

\- \[What changed and why]



\## Next Steps

\- \[ ] Task 1



\## Notes for Next Agent

\- \[Anything important for handoff]



\## File Structure

```

project-root/

├── server.js

├── package.json

├── CONTEXT.md (this file)

├── config/

│   ├── .env.local

│   └── .env.vps

└── public/

&#x20;   └── index.html

```



\## Environment Setup

\[How to run locally, what env vars are needed]

```



\### Guidelines



\- Keep it concise—focus on decisions and state, not a transcript

\- Update it whenever you make architectural changes, add features, or hit a blocker

\- Assume the next agent knows general programming but not this project's specifics





\## Sample nginx configuration on vps



Both the `location` and `proxy\_pass` MUST have trailing slashes. The `location /myapp/` trailing slash ensures the browser URL always ends with `/` (so relative paths like `./api/data` resolve correctly). The `proxy\_pass` trailing slash tells nginx to strip the prefix before forwarding.



```

\# Redirect /myapp to /myapp/ so relative paths resolve correctly

location = /myapp {

&#x20;   return 301 /myapp/;

}



location /myapp/ {

&#x20; proxy\_pass http://127.0.0.1:5001/;

&#x20; proxy\_set\_header X-Real-IP $remote\_addr;

&#x20; proxy\_set\_header X-Forwarded-For $proxy\_add\_x\_forwarded\_for;

}

```





\## Sample .env.vps file



```env

PORT=5002

ISSUER=https://test.scrambleid.com/oidc-server

```



APP\_ENV must never appear in .env.\* files. It is set externally (by PM2's ecosystem.config.cjs or the shell) and is read before dotenv.config() is called to select which .env file to load.





\## Sample content of ecosystem.config.cjs



```js

module.exports = {

&#x20; apps : \[{

&#x20;   name   : "myapp",

&#x20;   script : "./server.js",

&#x20;   env\_local: { APP\_ENV: "local" },

&#x20;   env\_vps: { APP\_ENV: "vps" }

&#x20; }]

}

```



\## UI Architecture



\- \*\*Prefer static SPA over templates\*\*: Always serve UI as plain HTML + vanilla JS from a `public/` directory using `@fastify/static`. Never use server-side template engines (e.g., Eta, `@fastify/view`).



\- \*\*API pattern\*\*: All data goes through JSON API routes. The HTML page fetches data via `fetch()` and renders it client-side.



\- \*\*Browser-redirect routes\*\*: Routes that receive form POSTs from external services (e.g., OAuth/MFA callbacks) may still do server-side `302` redirects to static HTML pages with data encoded in query params (e.g., `?claims=<b64>`).



\- \*\*Dependencies\*\*: Use `@fastify/static` for serving `public/`. Remove `eta` and `@fastify/view` from all projects.





\## If Forced to Use Eta Templates with @fastify/view because user specifically asked for it



\- \*\*No global layout\*\*: Do NOT configure a global `layout` in `@fastify/view` options.

\- \*\*No native JS array methods inline\*\*: Use a `for` loop with `<% for (const x of it.items) { %>`.

\- \*\*Use `<%\~` for unescaped HTML, `<%=` for escaped\*\*.

\- \*\*Always resolve views path with `\_\_dirname`\*\*: `root: resolve(\_\_dirname, 'views')`.

\- \*\*`@fastify/formbody` must be v8+ for Fastify 5\*\*.



