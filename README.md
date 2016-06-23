![xBrowserSync](https://raw.githubusercontent.com/xBrowserSync/xbrowsersync.github.io/master/images/logo_150.png "xBrowserSync")

[xBrowserSync](http://xbrowsersync.org/) is a tool for syncing your bookmarks and other browser information to a secure and anonymous cloud service. For full details, see http://xbrowsersync.org.

This repository contains the source code for the REST service API that client applications communicate with. If you'd like to run your own xBrowserSync service on your [Node.js](https://nodejs.org/) web server, follow the installation steps below.

Once configured, you can begin syncing your browser data to your xBrowserSync service, and if you're feeling generous, [allow others to sync their data to your service](http://xbrowsersync.org/connect/) also!

# Prerequisites

- [Node.js](https://nodejs.org/)
- [mongoDB](https://www.mongodb.com/)

# Installation

## 1. Install xBrowserSync API package

CD into the source directory and install the package and dependencies using NPM:

	$ npm install

## 2. Set up mongoDB database

  1. Run the following commands in the mongo shell:
  
  (Replace `[password]` with a cleartext password of your choice)

  ```
  use xBrowserSync 
  db.createUser({ user: "xbrowsersyncdb", pwd: "[password]", roles: ["readWrite"] }) 
  db.createCollection("bookmarks") 
  db.bookmarks.createIndex({ "_id": 1, "secretHash": 1 }) 
  db.createCollection("newSyncsLog") 
  db.newSyncsLog.createIndex({ "ipAddress": 1, "syncCreated": 1 })
  ```

  2. Add the following environment variables to hold xBrowserSync DB account username and password:

    - `XBROWSERSYNC_DB_USER`
    - `XBROWSERSYNC_DB_PWD`

  On Windows, open a Command Prompt and type (replacing `[password]` with the password entered in the mongo shell):
  
  ```
  setx XBROWSERSYNC_DB_USER "xbrowsersyncdb"
  setx XBROWSERSYNC_DB_PWD "[password]"
  ```
  
  On Ubuntu/Debian Linux, open a terminal emulator and type:
  
  ```
  $ pico ~/.profile
  ```
  
  Add the lines (replacing `[password]` with the password entered in the mongo shell):
  
  ```
  export XBROWSERSYNC_DB_USER=xbrowsersyncdb
  export XBROWSERSYNC_DB_PWD=[password]
  ```
  
  Save and exit, then log out and back in again.
  
  3. (If exposing your service to the public) Add a Scheduled Task (Windows) or CRON job (Ubuntu/Linux) to clear stale sync data that has not been accessed in a while. The task should run daily with the following command:
   
  - Windows:
  
    ```
    mongod.exe xBrowserSync -u %XBROWSERSYNC_DB_USER% -p %XBROWSERSYNC_DB_PWD% --eval 'db.bookmarks.remove({ lastAccessed: { $lt: new Date((new Date).setDate((new Date()).getDate() - 14)) } })'
    ```
  
  - Ubuntu/Linux:
  
    ```
    mongo xBrowserSync -u $XBROWSERSYNC_DB_USER -p $XBROWSERSYNC_DB_PWD --eval 'db.bookmarks.remove({ lastAccessed: { $lt: new Date((new Date).setDate((new Date()).getDate() - 14)) } })'
    ```

## 3. Edit xBrowserSync service configuration

Open `config.js` in a text editor and update the following variables with your desired values:

- `db.host` The mongoDB server address to connect to, either a hostname, IP address, or UNIX domain socket.
- `server.host` Host name or IP address to use for Node.js server for accepting incoming connections.
- `server.port` Port to use for Node.js server for accepting incoming connections.

## 4. Run xBrowserSync service

    $ node api.js

# Bugs

If you find a bug in the course of running an xBrowserSync service, please report it [here](https://github.com/xBrowserSync/API/issues/).
