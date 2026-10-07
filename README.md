# authAPI

Implementing RBAC using an ACL ///

## Changelog

- imported starter files from class repo; `\Code-401-PDX\class-08\lab\starter-code`;
  - `api-server/src/`.
  - `api-server/index.js`.
  - `api-server/.env.sample` -> `.env.example`.
  - `auth-server/src/auth/`.
- `package-json`;
  - installed authen and SQLite dependencies; `base-64 bcrypt jsonwebtoken sqlite3`.
  - configd startup and test cmds; `scripts`.
  - updated `license`.
- `package-lock.json`;
  - recorded installed dependecy versions.
- `src/server.js`;
  - removed wildcard (`*`), middleware with no path Express 5 compatability; `notFoundHandler`.
  - registered CORS, included URL-encoded body parsing, and authen routes; `cors`, `express.urlencoded`, `authRoutes`.
  - return HTTP server from startup; `start`. (creates Express app -> register middleware/routes -> exports app).
- `src/models/index.js`;
  - one DB connection for food, clotes, and users; `sequelize`.
  - configd in-memory SQLite storage for test and temp fallback; `storage`.
  - Exported user model; `users`.
  - cleanup of `environment`; `testOrProduction`.
- `src/auth/models/index.js`;
  - reused shared DB connection and user model; `db`, `users`.
- `index.js`;
  - updated to load config before importing app modules; `dotenv`.
  - synch registered models before listening, using async/await; `startServer`.
  - logged startup failures and set failing exit code.