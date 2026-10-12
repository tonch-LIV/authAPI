# authAPI

Implementing RBAC using an ACL ///

## Changelog

- imported starter files from class repo; ***`\Code-401-PDX\class-08\lab\starter-code`***;
  - *`api-server/src/`*.
  - *`api-server/index.js`*.
  - *`api-server/.env.sample`* -> *`.env.example`*.
  - *`auth-server/src/auth/`*.
- **`package-json`**;
  - installed authen and SQLite dependencies; `base-64 bcrypt jsonwebtoken sqlite3`.
  - configd startup and test cmds; `scripts`.
  - updated `license`.
- **`package-lock.json`**;
  - recorded installed dependecy versions.
- **`src/server.js`**;
  - removed wildcard (`*`), middleware with no path Express 5 compatability; `notFoundHandler`.
  - registered CORS, included URL-encoded body parsing, and authen routes; `cors`, `express.urlencoded`, `authRoutes`.
  - return HTTP server from startup; `start`. (creates Express app -> register middleware/routes -> exports app).
- **`src/models/index.js`**;
  - one DB connection for food, clotes, and users; `sequelize`.
  - configd in-memory SQLite storage for test and temp fallback; `storage`.
  - Exported user model; `users`.
  - cleanup of `environment`; `testOrProduction`.
- **`src/auth/models/index.js`**;
  - reused shared DB connection and user model; `db`, `users`.
  - updated `required` to `allowNull`, and `token`.
  - added `model.prototype.toJSON`.
  - updated `model.authenticateBasic` and `model.authenticateToken`.
- **`index.js`**;
  - updated to load config before importing app modules; `dotenv`.
  - synch registered models before listening, using async/await; `startServer`.
  - logged startup failures and set failing exit code.
- **`/home/tonch_liv/projects/courses/401/bearerAuth`**; imported files that validate authen scheme, reject malformed headers, and attach authen DB user to update existing copies;
- **`src/auth/middleware/basic.js`**, **`src/auth/middleware/bearer.js`**;
  - reused header validation from bearerAuth and forwarded authentication errors with status 401.
- **`src/auth/middleware/acl.js`**;
  - enforced capabilities and returned status 403 for insufficient permissions.
- **`src/auth/error-handlers/500.js`**;
  -logged full errors and returned JSON with the supplied status.
- **`src/auth/routes.js`**;
  - forwarded error objects and caught user-list database failures.
- **`src/auth/models/users`**
  - enforced non-null fields and required configured signing secret.
  - excluded password hashes from responses; `toJSON`.
  - validated credentials and awaited token user lookup; `authenticateBasic`, `authenticateToken`.