# Base-project set up instructions ✨

## Create local repo
1. `mkdir project-name && cd project-name`
2. `git init`
3. `git remote add origin [GIT URL]`
4. `git branch -M main`
5. `git push -u origin main`

## Clone from GitHub (if not creating locally)
1. `git clone [GIT URL]`
2. `cd project-name`

## Set up project
1. `npm init -y`
2. `npm install express`
3. `npm install -D nodemon`
3. `npm install -D cypress`
3. `npm install -D dotenv`
4. `touch .gitignore`
5. `touch .env`
6. `code .`
7. Add `node_modules` and `.env` to .gitignore file (`echo "node_modules\n.env\n" >> .gitignore`)
8. Change `test` script to `cypress open` in `package.json`
9. Add a `start` script as `node server.js` 
10. Add a `dev` script as `nodemon -r dotenv/config server.js` 
11. Run cypress to get files `npm run test`

## Set up express server
Make server.js file on root. 
```js
// Import express
const express = require("express");

// Add to cypress JSON file to not have to write out full URL
{
  "chromeWebSecurity": false,
  "baseUrl": "http://localhost:3333"
}

// Port to listen on
const PORT = 3333;

// Create server
const server = express();

// Set up routes
server.get("/", (request, response) => { 
  const html = `
  <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
     <h1>hello world</h1>
    </body>
    </html>`;
    
  response.send(html)
});

// Run server
server.listen(PORT, () => { 
  console.log(`listening on http://localhost:${PORT}`);
})
```

## Set up cypress test 
```js
// Add test.js file to cypress/intergration

// Wrap linked tests in a describe
describe("homepage tests", () => {
  it("can find homepage", () => {
    cy.visit("/");
  });
  it("can find title on home page", () => {
    cy.visit("/");
    cy.get("h1").contains("hello world");
  });
});

```
- run server `npm run dev`
- run cypress test `npm run test`

## Link CSS file 
Add `public` file on root and add `index.css`
```js
// add static handler to have access to all files inside public
const staticHandler = express.static("public");

// pass into .use
server.use(staticHandler);
```
Add `<link rel="stylesheet" type="text/css" href="./index.css">` to head of html.

## Database file set-up 
1. Create `database` folder on root
2. Add `connection.js` file
```js
// Inside file add
const { Pool } = require("pg");

// DB URL should either be read from .env in development
// or set as part of production deployment (e.g. on Heroku)
if (!process.env.DATABASE_URL) {
  console.log(process.env.DATABASE_URL);
  throw new Error("Missing DATABASE_URL env var");
}
console.log(process.env.DATABASE_URL);

// Connect to the database
// and create a pool of available connections to support simultaneous requests
const db = new Pool({
  connectionString: process.env.DATABASE_URL,
});

// export the pool object so we can query the DB in other files
module.exports = db;
```
4. Add `init.sql` file.
```sql 
BEGIN;

DROP TABLE IF EXISTS users, user_facts CASCADE;

-- Create tables and define their columns

CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  first_name VARCHAR(100) NOT NULL,
  cohort VARCHAR(6)

);

CREATE TABLE user_facts (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  facts TEXT
);

-- Insert some example data for us to test with

INSERT INTO users (first_name, cohort) VALUES
  ('Oli', 'FAC23'),
  ('Milly', 'FAC23'),
  ('Miah', 'FAC23'),
  ('Adam', 'FAC23'),
  ('Anna', 'FAC22')
;

INSERT INTO user_facts (facts, user_id) VALUES
  ('I am a secret millionaire', 1),
  ('I know kung fu', 2),
  ('I have never seen Friends', 3),
  ('I cannot ride a bike', 4),
  ('I still have my xmas tree up!', 5)
;

COMMIT;

```
6. Add `model.js` (use later to modularise)

## Create database locally
1. `psql`
2. `psql -c "CREATE USER mydbuser WITH PASSWORD pass123"`
3. `psql -c "CREATE DATABASE mydb WITH OWNER mydbuser"`
4. Add DB_URL to .env file 
Eg. `DATABASE_URL='postgres://mydbuser:pass123@localhost:5432/mydb'`

## Set up shell for create_db and repopulate_db for others to use.
1. Create `scripts` folder on the root with `create_db` and  `repopulate_db` files
```
// Add to create_db
# #! /bin/sh

# # stop script when an error occurs
# set -e

# # name will be whatever the first argument is when called
# # e.g. ./scripts/create_db mydb => DB_NAME='mydb' (defaults to "example")
DB_NAME=${1:-example}

# psql -q -c "CREATE USER ${DB_NAME}user SUPERUSER PASSWORD 'pass123'"
# echo "Created Postgres user '${DB_NAME}user'"

# psql -q -c "CREATE DATABASE ${DB_NAME} WITH OWNER ${DB_NAME}user"
# echo "Created Postgres database '${DB_NAME}'"

DB_URL="postgres://${DB_NAME}user:pass123@localhost:5432/${DB_NAME}"
echo "DATABASE_URL='${DB_URL}'" > .env
echo "Created .env containing DATABASE_URL"
```
RUN`./scripts/create_db mydb` 
```
// Add to repopulate_db
#! /bin/sh

# stop script when an error occurs
set -e

# import .env so we can access DB URL
. .env

psql $DATABASE_URL -q -f "./database/init.sql"
echo "Populated database tables"
```
RUN `./scripts/populate_db`

!ISSUE WITH PERMISSIONS ---> Add `chmod +x` prefix to the script commands.

## Automatic repopulate db for testing 
1. Add to cypress/pluggins/index.js file
```
// const build = require("../../database/build.js");
const { execFileSync } = require("child_process");

module.exports = (on, config) => {
  on("task", {
    resetDb: () => {
      return execFileSync("./scripts/populate_db");
    },
  });
};
```
2. Add to cypress/integration/test.js file
```
beforeEach(() => {
  cy.task("resetDb");
});
```

## Sanitization
When you get request.body of input, ensure sanitized to prevent cross-site scripting.
```js
function addFact(request, response) {
  const firstName = request.body.first_name.replace(/</g, "&lt;");
  const cohort = request.body.cohort.replace(/</g, "&lt;");
  const facts = request.body.facts.replace(/</g, "&lt;");
})
 ```

## Middleware 
To chain information when using post requests
```js
// set up body parser
const bodyParser = express.urlencoded();

// pass body paser in as an argument to get access to the request.body.
server.post("/", bodyParser, home.addFact);
```

## Deploy to Heroku
1. Create account
2. Create new app (*lowercase name and europe region*)
3. *Connect to github* - search for you repo
4. Add `Procfile` to root of project with `web: node server.js` inside.
5. Edit port `const PORT = process.env.PORT || 3000` 
6. If issues connecting to DB add `psql [insert heroku db URL] -f database/init.sql`
