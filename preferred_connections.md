# Connection Registry

List your Boomi connections here so the agent can find and re-use them.

Format is free-form — describe each connection and include whatever helps the agent identify and select it: a platform link, component IDs, operations, endpoints, or notes about when to use it. The agent reads this file and matches entries by context to the connectors it needs.

## API Gateway listener testing (test)

Use this whenever building or testing WSS / API Service listeners against the Scandlines **test** API Gateway. Do not put secrets in this file; they live in `.env`.

**Runtime**
- Test atom `apiType` is **advanced**. Publish REST with an **API Service Component** wrapping a WSS Listen process. Bare `/ws/simple/...` is not served on the Gateway host (404 context-path).
- Shared Web Server default URL is an internal atom IP; always test via Gateway `SERVER_BASE_URL`, not that IP.

**`.env` (dual-header)**
- `SERVER_AUTH_TYPE=bearer`
- `SERVER_BEARER_TOKEN` = JWT / Bearer token (not the Gateway API key, not `BOOMI_API_TOKEN`)
- `SERVER_API_KEY` = Gateway **API Key**, sent as `X-API-KEY`
- Leave `SERVER_USERNAME` / `SERVER_TOKEN` empty
- `boomi-wss-test.sh` sends both headers when those vars are set. Banner should show `auth=bearer x-api-key=set`.

**`SERVER_BASE_URL`**
- Use `https://api-test.scandlines.com/ws` with **no trailing slash**.
- That value already includes the `/ws` context. Pass `--path` **without** another `/ws` prefix.
- Wrong: `--path /ws/rest/test/hello/` → `https://api-test.scandlines.com/ws/ws/rest/...` or `.../ws//ws/...`
- Right: `--path rest/test/hello/` → `https://api-test.scandlines.com/ws/rest/test/hello/`

**Smoke test (verified 2026-09-18)**
- `bash <skill-path>/scripts/boomi-wss-test.sh --path rest/test/hello/ --method GET`
- Expect HTTP 200 body like `Hello World` plus a timestamp.
- POST on the same path is **404** (GET-only). Do not treat that as a deploy failure.
- HTTP 401 `{"message":"Unauthorized"}` means Bearer and/or API key were rejected. Stop and ask; do not retry the same credentials (platform lockout risk).

**Windows / WSL**
- `C:\Windows\System32\bash.exe` is WSL. Skill scripts live at `/mnt/c/Users/dlersjan/.cursor/skills/boomi-integration/scripts/`.
- Run them from `/mnt/c/Users/dlersjan/Projects/<project>` so `.env` loads.
- `C:/Users/...` paths fail in this bash (`No such file`).

**Template workspace**
- This directory **is** the personal template. Do not run `/freshies` here (source and cwd are the same). Scaffold into a new empty project folder instead.

**Out of scope**
- Gateway policies, plans, tokens, and the developer portal stay in the API Gateway GUI. This skill builds/deploys the API Service Component and tests it with `.env` credentials.

## Example formats

### Salesforce connection - production org
https://platform.boomi.com/AtomSphere.html#build;accountId=YOUR_ACCOUNT_ID;components=CONNECTION_COMPONENT_ID

### RevOps API
Use this when posting billing events from internal pipelines.
REST Client Connection (sender): CONNECTION_COMPONENT_ID
API Service component (listener): API_SERVICE_COMPONENT_ID
charge-event endpoint: /ws/rest/revops/charge-event
refund-event endpoint: /ws/rest/revops/refund-event

### PostgreSQL - customer orders database
Connection: CONNECTION_COMPONENT_ID
get-orders operation: OPERATION_COMPONENT_ID
submit-order operation: OPERATION_COMPONENT_ID
