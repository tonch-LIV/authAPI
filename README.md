# **authAPI**

Implementing RBAC ///  
An Express API combining PostgreSQL-backed CRUD operations with Basic authentication, JWT bearer authentication, and role-based access control.  

V1 routes provide  unauthenticated **CRUD** access; V2 routes require a valid bearer token, and additional capabilities (role permissions) required for modification (**C**reating, **U**pdating, and **D**eleting).

## Setup

The project requires Node.js, npm, and PostgreSQL.  
Local development was verified using Node.js 20.20.2.

From the project directory, install the dependencies recorded in the lockfile:

```bash
npm ci
```

On Ubuntu, start PostgreSQL and create the assignment database if it does not already exist:

```bash
sudo service postgresql start
sudo -u postgres createdb auth-api
```

Create the local environment file if it does not already exist:

```bash
cp .env.example .env
```

Edit `.env`:

```dotenv
NODE_ENV=development
PORT=3001
DATABASE_URL=postgres://postgres:YOUR_PASSWORD@localhost:5432/auth-api
SECRET=YOUR_GENERATED_SECRET
```

Use your existing PostgreSQL credentials.  
URL-encode special characters in the password portion of `DATABASE_URL`.

Generate a signing secret:

```bash
openssl rand -hex 32
```

Copy the generated value into `SECRET`. Keep `.env` private; it is excluded from Git.

Start the application:

```bash
npm start
```

Startup loads environment configuration, synchronizes the registered database models, and then starts listening.  
Normal startup requires `DATABASE_URL` and `SECRET`.

The default local address is `http://localhost:3001`.

## API Reference

### Authentication

| Method | Route | Requirement | Successful response |
|---|---|---|---|
| POST | `/signup` | JSON username and password; optional lab role | 201; `{ user, token }` |
| POST | `/signin` | Basic authentication credentials | 200; `{ user, token }` |
| GET | `/secret` | Valid bearer token | 200; welcome message |
| GET | `/users` | Valid bearer token and delete capability | 200; username array |

User responses include the *ID*, *username*, *role*, and *capabilities*.  
**Password hashes remain internal**.

For this course lab, signup accepts a role so each permission level can be demonstrated.  
Public production signup would require controlled role assignment.

### Collections

The supported collection names are `food` and `clothes`.  
Other names, including `users` and `db`, are rejected by the API collection selector.

Replace `:version` with `v1` or `v2`,  
`:model` with a collection name, and `:id` with a positive record ID.

| Method | Route | V2 requirement | Successful response |
|---|---|---|---|
| GET | `/api/:version/:model` | Bearer token | 200; record array |
| GET | `/api/:version/:model/:id` | Bearer token | 200; record object |
| POST | `/api/:version/:model` | Bearer token and create capability | 201; created record |
| PUT | `/api/:version/:model/:id` | Bearer token and update capability | 200; updated record |
| DELETE | `/api/:version/:model/:id` | Bearer token and delete capability | 200; `{}` |

V1 requires no authentication.  
Both versions use the same database records and CRUD handlers.

Food fields:

```json
{
  "name": "apple",
  "calories": 95,
  "type": "fruit"
}
```

Food type is defined as `fruit`, `vegetable`, or `protein`.

Clothes fields:

```json
{
  "name": "shirt",
  "color": "blue",
  "size": "M"
}
```

All listed fields are required when creating a record.

## Permissions

| Role | Read | Create | Update | Delete |
|---|---|---|---|---|
| user | Yes | No | No | No |
| writer | Yes | Yes | No | No |
| editor | Yes | Yes | Yes | No |
| admin | Yes | Yes | Yes | Yes |

V2 GET routes require authentication only. Every defined role has read capability.

### Example Authentication Flow

Implies two terminal/shells;  
One to run the server, the other to run the `curl` commands.  

- Create a writer account:

```bash
curl -i -H 'Content-Type: application/json' \
  -d '{"username":"example-writer","password":"example-password","role":"writer"}' \
  http://localhost:3001/signup
```

- Sign in:

```bash
curl -i -u example-writer:example-password \
  -X POST http://localhost:3001/signin
```

- Copy the returned token into a temporary shell variable:

```bash
LAB_TOKEN='paste-returned-token-here'
```

- Create food through the protected API:

```bash
curl -i \
  -H "Authorization: Bearer $LAB_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"name":"apple","calories":95,"type":"fruit"}' \
  http://localhost:3001/api/v2/food
```

### Error Responses

Errors use JSON with `status` and `message` fields:

```json
{
  "status": 403,
  "message": "Access Denied"
}
```

| Status | Meaning |
|---|---|
| 400 | Invalid record ID, missing required fields, or duplicate username |
| 401 | Missing or invalid authentication |
| 403 | Authenticated user lacks the required capability |
| 404 | Unknown route, unsupported collection, or missing record |
| 500 | Unexpected server or database failure |

## Testing

- Run the suite with coverage:

```bash
npm test -- --runInBand
```

- Run tests in watch mode:

```bash
npm run test:watch
```

Tests set their own signing secret and use an isolated in-memory SQLite database.  
Each test starts with fresh tables and accounts; PostgreSQL configuration is not required for the automated suite.

The suite covers signup/signin, safe user responses, authentication failures, all four roles, both collections, V1/V2 CRUD, denied writes, missing records, database failures, and server startup behavior.

Latest verified results:

- 64 tests passed.
- Statement coverage: 100%.
- Branch coverage: 100%.
- Function coverage: 100%.
- Line coverage: 100%.

Coverage includes all JavaScript files under `src/`.  
Root `index.js` is outside that coverage scope.

Local PostgreSQL persistence was checked separately by creating a record, restarting the application, and retrieving the record again.

## Architecture and Diagrams

The application separates startup, request handling, authentication, and database operations:

- `index.js`; loads configuration, synchronizes models, and starts the server.
- `src/server.js`; creates the Express application and registers middleware and routers.
- `src/auth/`; authenticates users and checks capabilities.
- `src/routes/`; selects supported collections and handles V1/V2 CRUD requests.
- `src/models/`; registers models on one connection and performs database operations.
- `src/error-handlers/`; formats error responses.

### Authentication and API Structure

```mermaid
classDiagram
    class ExpressApplication {
        authenticationRoutes
        v1Router
        v2Router
    }

    class UserModel {
        username
        passwordHash
        role
        token
        capabilities
        authenticateBasic()
        authenticateToken()
        toJSON()
    }

    class AuthenticationMiddleware {
        basic()
        bearer()
    }

    class AccessControl {
        checkCapability()
    }

    class CRUDHandlers {
        getAll()
        getOne()
        create()
        update()
        remove()
    }

    class DataCollection {
        model
        get()
        create()
        update()
        delete()
    }

    class SharedDatabase {
        Users
        Food
        Clothes
    }

    ExpressApplication --> AuthenticationMiddleware
    AuthenticationMiddleware --> UserModel
    ExpressApplication --> AccessControl
    AccessControl --> UserModel : reads capabilities
    ExpressApplication --> CRUDHandlers
    CRUDHandlers --> DataCollection
    DataCollection --> SharedDatabase
    UserModel --> SharedDatabase
```

### Web Request/Response Cycle

Example: creating a food record through V2.

```mermaid
sequenceDiagram
    participant Client
    participant API as Express API
    participant Auth as Bearer Middleware
    participant ACL as Access Control
    participant Handler as CRUD Handler
    participant Collection as Data Collection
    participant DB as Database
    participant Error as Error Handler

    Client->>API: POST /api/v2/food with bearer token and JSON
    API->>Auth: Validate authentication
    Auth->>DB: Find user after verifying JWT
    DB-->>Auth: User or no matching user

    alt Authentication fails
        Auth->>Error: Authentication error
        Error-->>Client: 401 JSON response
    else Authentication succeeds
        Auth->>ACL: Request with authenticated user
        alt User lacks create capability
            ACL->>Error: Access denied
            Error-->>Client: 403 JSON response
        else User has create capability
            ACL->>Handler: Continue request
            Handler->>Collection: create(request body)
            Collection->>DB: Insert food record
            alt Insert succeeds
                DB-->>Collection: Created record
                Collection-->>Handler: Created record
                Handler-->>Client: 201 JSON response
            else Insert fails
                DB-->>Collection: Database error
                Collection-->>Handler: Rejected operation
                Handler->>Error: Forward error
                Error-->>Client: 400 validation or 500 unexpected error
            end
        end
    end
```

Record lookup, update, and deletion return 404 when the requested record does not exist.

## Submission Links

- Deployed API: optional; not included in this submission.
- [Merged pull request from dev to main](https://github.com/tonch-LIV/authAPI/pull/1).

## AI Assistance

Codex provided integration guidance, suggested code changes for authen, shared DB registration, V1/V2 routing, CRUD error handling, and tests. Also help with documentation and mermaid diagram drafts, and the Jest/Supertest suite and configuration. Changes were made manually and verification ran locally.

The work reused the class assignment starter code and my previous API, Basic authentication, and Bearer authentication projects.

Example of prompts used include;

- “Can you review the recent changes and determine what couldve caused this to happen? when it was passing before...” - investigated and corrected the user-list database-failure test.
- “I made the changes to [], feel free to review, consider the scope, and offer revisions and/or suggestions on changes where needed.... 
- "let's go over the remaining coverage gaps" - generated config and error-formatting tests.

Code contributions; `src/auth/`, `src/routes/`, `src/models/`, `src/server.js`, `src/error-handlers/500.js`, `index.js`.

Test contribution; `__tests__/server.test.js`, `__tests__/configuration.test.js`, and `jest.config.js`.

Documentation contributions; `README.md`.

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
  - preserved authentication lookup while simulating user-list database failure; `users.findAll`.
- **`.gitignore`**;
  - excluded generated coverage reports; `coverage/`.
  - added local SQLite databases to list to prevent future accidental adds.
- **`.env.example`**;
  - documented development environment, port, database URL, and signing-secret placeholders.
- **`README.md`**;
  - expanded setup, API examples, permissions, errors, and verified test results.
  - added architecture and request-cycle diagrams; Mermaid.
  - recorded pending deployment and submission artifacts.
- **`__tests__/configuration.test.js`**;
  - `jest.isolateModules()` reloads configuration without disturbing the existing integration suite.
  - configuration and error-handler tests that cover `SECRET`,
    - missing `DATABASE_URL` outside tests,
    - selection of PostgreSQL config mocks Sequelize, checking chosen URL w/o connecting to a DB.
    - string-error formatting,