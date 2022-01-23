# Base-project set up instructions ✨

## Create local repo
1. `mkdir project-name && cd project-name`
2. `git init`
3. `git remote add origin [GIT URL]`
4. `git branch -M main`
5. `git push -u origin main`

## Clone from GitHub
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
7. Add `node_modules` and `.env` to .gitignore file
8. Change `test` script to `cypress open` in `package.json`
9. Add a `start` script as `node -r dotenv/config server.js` 
10. Add a `dev` script as `nodemon -r dotenv/config server.js` 


## Set up express server
- Make server.js file on root. 
- Add port ``` const PORT = 3333; ```
- Listen to port 
``` 
server.listen(PORT, () => { 
  console.log(`listening on http://localhost:${PORT}`);
})
 ```



