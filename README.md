# authAPI

Implementing RBAC using an ACL ///

## AI Assistance

Codex provided integration guidance, suggested code changes, and the Jest/Supertest suite and configuration. Changes were made manually and verification ran locally.

Actual prompt that continued the test-writing task: “ok let us continue”.

Test contribution; `__tests__/server.test.js`, `jest.config.js`.

## Changelog

- imported starter files from class repo; ***`\Code-401-PDX\class-08\lab\starter-code`***;
  - *`api-server/src/`*.
  - *`api-server/index.js`*.
  - *`api-server/.env.sample`* -> *`.env.example`*.
  - *`auth-server/src/auth/`*.
- **`package.json`**;
  - installed authen and SQLite dependencies; `base-64 bcrypt jsonwebtoken sqlite3`.
  - configd startup and test cmds; `scripts`.
  - updated `license`.
- **`package-lock.json`**;
  - recorded installed dependecy versions.
- **`src/server.js`**;
  - removed wildcard (`*`), middleware with no path Express 5 compatability; `notFoundHandler`.
  - registered CORS, included URL-encoded body parsing, and authen routes; `cors`, `express.urlencoded`, `authRoutes`.
  - return HTTP server from startup; `start`. (creates Express app -> register middleware/routes -> exports app).
  - registered the protected API alongside V1; `/api/v2`, `v2Routes`.
- **`src/models/index.js`**;
  - one DB connection for food, clotes, and users; `sequelize`.
  - isolated tests with in-memory SQLite and required PostgreSQL configuration outside tests; `storage`, `DATABASE_URL`.
  - Exported user model; `users`.
  - cleanup of `environment`; `testOrProduction`.
- **`src/auth/models/index.js`**;
  - reused shared DB connection and user model; `db`, `users`.
- **`src/auth/models/users.js`**;
  - enforced non-null fields and required configured signing secret; `allowNull`, `SECRET`.
  - generated signed tokens through the getter and removed the unused setter; `token`.
  - excluded password hashes from responses; `toJSON`.
  - validated credentials and awaited token user lookup; `authenticateBasic`, `authenticateToken`.
- **`index.js`**;
  - updated to load config before importing app modules; `dotenv`.
  - synch registered models before listening, using async/await; `startServer`.
  - logged startup failures and set failing exit code.
- **`/home/tonch_liv/projects/courses/401/bearerAuth`**; imported files that validate authen scheme, reject malformed headers, and attach authen DB user to update existing copies;
- **`src/auth/middleware/basic.js`**, **`src/auth/middleware/bearer.js`**;
  - reused header validation from bearerAuth and forwarded authentication errors with status 401.
- **`src/auth/middleware/acl.js`**;
  - enforced capabilities and returned status 403 for insufficient permissions.
- **`src/error-handlers/500.js`**;
  - logged full errors and returned JSON with the supplied status.
  - mapped Sequelize validation and duplicate-record errors to status 400.
- **`src/auth/routes.js`**;
  - forwarded error objects and caught user-list database failures.
- **`src/models/data-collection.js`**;
  - replaced promise chaining with async/await and handled missing records; `update`.
  - returned an empty object after deletion; `delete`.
- **`src/models/food/model.js`**, **`src/models/clothes/model.js`**;
  - enforced required fields; `allowNull`.
- **`src/routes/model.js`**;
  - restricted API collections to food and clothes; `collections`.
  - attached the selected collection to the request and rejected unknown models; `req.model`.
- **`src/routes/handlers.js`**;
  - shared CRUD handlers, validated IDs, and forwarded errors.
  - returned status 404 for missing records.
- **`src/routes/v1.js`**, **`src/routes/v2.js`**
  - shared CRUD behavior and protected V2 with bearer authentication and capabilities.
- **`jest.config.js`**;
  - configured Node test environment and coverage collection; includes source files even when tests never execute them.
- **`__tests__/server.test.js`**;
  - added integration, `beforeEach` creates fresh DB and accounts (4) for each test, verifies HTTP responses and stored data; 
  - added authentication, V1/V2 CRUD, role permission, and error-response checks.
- **`.gitignore`**;
  - excluded generated coverage reports; `coverage/`.
- **`.env.example`**;
  - documented development environment, port, database URL, and signing-secret placeholders.