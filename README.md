## What is ZEIT?

ZEIT is a hosting service that provides us with a command line interface (cli) that we will use to manage our deployments. There are many different options for hosting live projects. ZEIT stands out from the rest due to its ease of use and automated processes. This guide will help you get a React project hosted on ZEIT so that it can be accessed by anyone at any time on the web.

### Step 1: Globablly Install ZEIT's deployment CLI: 'now'

`npm i -g now`

### Step 2: Login to ZEIT

`now login`  

The terminal will ask for your email, then send a verification email to the address provided. This will also create an account if you don't have one. Open up your inbox and click **Verify**.

### Step 3: Make sure your server is setup to run off the build process

Before we deploy our project, we should make sure that our project is working on our local machine. Simply run your server in the terminal and debug any errors you may have.

Up until now we have been using the React Dev Server to view and run our app with `npm start`.  We are going to add a production build to our project and enable express to serve up those files.  This will do things such as removing extra white-space, comments, and minify our code so that it's as fast to download as possible.

Tell create-react-app to use webpack to create a build folder with your latest code.

`npm run build`

Make sure your dev server isn't running for the front end.  Start your backend server with:

`nodemon`

In your browser, check that you can go to http://localhost:3030 (or whichever port you told your backend to run on.) Use this code to point express to the build folder.

```
app.use( express.static( `${__dirname}/../build` ) );
```

If you are using browser history, you'll need this to make sure your index.html file is being given on the other routes.

Towards the *end* of your server file make sure you have this (this needs to run after you've setup all your other endpoints).

```
const path = require('path')
app.get('*', (req, res)=>{
  res.sendFile(path.join(__dirname, '../build/index.html'));
})
```
Check that your project is working on the localhost port that your server is using. Fix any errors if it isn't (look in the node terminal).

## Step 4: Ensure you've setup a .env file, and set node to use it for your project.

All the configuration that you need different between deployed and local should go into this file, as well as any keys that you want to be secret.  

The connection string to your database, the REACT_APP_ reference for Auth0, as well as your Auth0 credentials are most likely in here.  You may have additional variables set if you want to make other changes between a local and production build.  

*Make sure the .gitignore contains the .env and .env.prod files*

Example .env file

```
REACT_APP_LOGIN="http://localhost:3030/api/auth/login"
REACT_APP_LOGOUT="http://localhost:3030/api/auth/logout"

DOMAIN="brack.auth0.com"
ID="46NxlCzM0XDE7T2upOn2jlgvoS"
SECRET="0xbTbFK2y3DIMp2TdOgK1MKQ2vH2WRg2rv6jVrMhSX0T39e5_Kd4lmsFz"
SUCCESS_REDIRECT="http://localhost:3030/"
FAILURE_REDIRECT="http://localhost:3030/api/auth/login"

CONNECTION_STRING="postgres://vuigx:k8Io23cePdUorndJAB2ijk_u0r4@stampy.db.elephantsql.com:5432/vuigx"
NODE_ENV=development
```

Copy your `.env` file and rename it `.env.prod` Change any values that you need to change (The REACT_APP_LOGIN info will certainly need to change, depending on your project you may or may not want to change the DB - and if you've used something like Stripe that needs a real key outside of development, that would change as well.)

Example .env.prod
```
REACT_APP_LOGIN="/api/auth/login"
REACT_APP_LOGOUT="/api/auth/logout"

DOMAIN="brack.auth0.com"
ID="7yq4hbufvtq32uirf7v4w"
SECRET="aszxvuigvagsfbawroyzx8gvaiger8xtv87gsfxzv"
SUCCESS_REDIRECT="/"
FAILURE_REDIRECT="/api/auth/login"

CONNECTION_STRING="postgres://sdfsdrtfg:98yhuisdfiybriuhfg@stampy.db.elephantsql.com:5432/sdfsdrtfg"
NODE_ENV=production
```

Then we are going to check out main server file.
It should contain the following at the beginning of your code (It can go anywhere before you first try to use the process.env variables, but the first line makes sure that it is always available to you);

`require('dotenv').config();`

If you did not have this already in your project, you will need to install dotenv.

`npm i dotenv`

Double check that your server is still working with this new configuration setting.

### Step 5: Creating the Scripts Now Needs to run

Under the scripts section, we want to add a new script called 'now-start'. This script is going to tell ZEIT what file it should start for the backend - in this case `node server/index.js`, though `node server/server.js` may be how your project is setup.

We are also going to add a "deploy" script where we will configure our deployment.

`now --public --dotenv=.env.prod -d`

now ---- This is the ZEIT's command to make a deployment.

 --public ---- This is to acknowledge we are making a public deployment so the source code and logs are publicly available.

 --dotenv=.env.prod ---- This is telling our deployment what configuration we want to use for our environment variables.

 -d This is telling it to use debug mode.  You'll get better error messages in your logs.  After your deployment is working properly you may choose to disable this.

### Step 6: Add an alias for our deployment to use.

`now` lets us set up an alias for our deployments.  This will be the domain that we can give out to people.  On the free tier you will be automatically subdomained to .now.sh  

In the package.json we are going to add a new property called "now" and set it to an object with an "alias" property that we want to be our subdomian.  In this example "devmountainproject"

The bottom of our package.json should look more or else like this.
```
"scripts": {
  "start": "react-scripts start",
  "build": "react-scripts build",
  "test": "react-scripts test --env=jsdom",
  "eject": "react-scripts eject",
  "now-start": "node server/index.js",
  "deploy": "now --public --dotenv=.env.prod -d"
},
"now":{
  "alias":"devmountainproject"
},
"proxy":"http://localhost:3030"
```

### Step 7: Try the deployment

We should now be able to do `npm run deploy`. This will upload our project to ZEIT, and set up a server.  ZEIT will tell you the random address that they currently have pointing to the new server (This process can take quite a while).

If we run `now alias` it will apply the alias address we setup in our package.json.

### Step 8: Managing ZEIT instances.

You can only have 3 instances running at once on the free tier of ZEIT. Unfortunately, they will fall aslep automatically after 3-5 hours. If you have multiple projects on ZEIT, you may need to terminate some of your previous deployments. The command `now ls` will list out the instances that are running on your account.  
```
 url                                          inst #    state                 age
 devmountainproject-jvcagclcqa.now.sh              0    READY                 14d
 devmountainproject-hhentmkpty.now.sh              0    FROZEN                27d
 devmountainproject-arjdfpbxgk.now.sh              0    FROZEN                27d
 devmountainproject-xmjqiajpmb.now.sh              0    FROZEN                27d
 devmountainproject-kdwehzgvxp.now.sh              0    FROZEN                27d
 ```
 
 You can remove an instance to free up another instance by running `now rm devmountainproject-jvcagclcqa.now.sh` (use the url provided by the now ls command).

### Updating your hosted project

ZEIT does not let you edit a server once it is created.  To make changes to your project, perform the desired alterations to your code.  Run `npm run deploy`.  After it's created the server, go to the address that they've provided, make sure that it is working as you expect, then run `now alias`. This will change where your server is pointing, so all new traffic will go to the new server.
