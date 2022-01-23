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

## Set up cypress test example. 
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

## Create database locally

## Automatic repopulate db for testing 

## set up creat db and repopulate db for other people to use if they download it. 






