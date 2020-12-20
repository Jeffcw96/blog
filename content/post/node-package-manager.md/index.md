+++
author = "Jeff Chang"
title = "Node Package Manager"
date = "2020-12-20"
description = "Node Package Manager (NPM) is a package manager for the JavaScript programming language.It makes us easily install modules or packages in our system. It also makes it easier for developer to share their code. For example, after we cloned someone respository from Github, we can always type `npm install` and `npm start` to see the application in the development"
tags = [
    "javascript"
]
categories = [
    "Javascript"
]
image = "cover.jpg"
+++

### Prerequisites:
* Before everything started. Please make sure you have installed [Node.js](https://nodejs.org/en/download/) on your system.

Through out this article, we will know more about:
* [Package.json](#package-json)
* [Install Module](#install)
    * [Dev Dependency VS Regular Dependency](#dev-vs-regular)
* [Update Module](#update)
* [Remove Module](#remove)
* [Summary](#summary)


## Package.json<a name="package-json"></a>
Package.json file is the file that contains all info of your project such as it will 
* listed out all the dependency *(both Regular and Dev)*
* It will warn us if some of the packages are deprecated
* It can easily create NPM script by entering the command `npm init`

#### Check NPM version
`npm --version`

#### Create a Package.json file
1. `npm init`
2. `package name: (npm)` **:** *Type Your Package Name Here*
3. `version (1.0.0)` **:** *Normally we will skip for this*
4. `description` **:** *Descriptions to describe your project*
5. `entry point (index.js)` **:** *This is the main javascript file. Sometimes people like to name it main.js, server.js and etc*
6. `test command` **:** *We don't have any test command*
7. `git respository` **:** *Put the respository if any*
8. `keywords` **:** *Put the keywords if any*
9.  `author` **:** *Put your name as the author of this project*
10. `license: (ISC)` **:** *You may put the license. Usually will be MIT*

If you want to create a package.json without going through all these steps, `npm init -y` or `npm init --yes` will allow you to create a NPM script with default value. <br/>
Then you should see a file created call **package.json**
{{< highlight json >}}
{
  "name": "npm-testing",
  "version": "1.0.0",
  "description": "This is the demo of NPM script",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Jeff",
  "license": "MIT"
}
{{< /highlight >}}

### Set, Get & DELETE default value
Some keys such as **author** and **license** will be quite similar all the time. This case we can then give them a default value so that it skip for asking the value when we trying to create this NPM script

#### SET
* `npm set init-author-name < Your Name Here >`
* `npm set init-license < Default License >`

#### GET
* `npm get init-author-name`
* `npm get init-license`

#### DELETE
* `npm config delete init-author-name`
* `npm config delete init-license`

## Install Package<a name="install"></a>
Note: Installed modules are added as a dependency by default, so the --save option is no longer needed. <br/>

> For example: <br/>
> From `npm install < module name > --save` to `npm install < module name >`
>
> or
>
> From `npm i < module name > --save` to `npm i < module name >`

### Install with version
`npm install <package>@<version>` allow us to install the module with specific version.<br/>
For example, the latest version of [Express](https://www.npmjs.com/package/express) is 4.16.4 but if we wanted to install the older version of this. We can enter the command <br/>`npm install express@4.16.4`

### Dev Dependency VS Regular Dependency<a name="dev-vs-regular"></a>
By default, if we never added the command `-D` isuch as `npm i <module> -D` command, it will be auto save in our regular dependency. <br/>
One main different between Dev Dependency VS Regular Dependency is that whatever inside the Dev Dependencies are not going to use in the production while whatever inside Regular Dependencies will be used in the production. <br/>

One good example on this is when we try to create a server in Nodejs with Express framework. Normally we will install [Express](https://www.npmjs.com/package/express) in our regular dependency and install [Nodemon](https://www.npmjs.com/package/nodemon) *(Update the result without restarting the server)* in our dev dependency. <br/>

After you had installed the modules, there will be some changes in our **package.json** file which will then added `dependencies` and `devDependencies` object. 
{{< highlight json >}}
{
  "name": "npm-testing",
  "version": "1.0.0",
  "description": "This is the demo of NPM script",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "Jeff",
  "license": "MIT",
  "dependencies": {
    "express": "^4.16.4"
  },
  "devDependencies": {
    "nodemon": "^2.0.6"
  }
}
{{< /highlight >}}
This is very useful as when someone cloned our project and try to experience it. They will enter the command `npm install` and this command will install the necessary modules by looking the value from these dependencies.

### Update Module<a name="update"></a>
Update Module is pretty straight forward. Let's take the Express module as example. <br/>
`npm update express` will update the Express version from **4.16.4** to the latest version **4.17.1** (up to 20/12/2020) <br/>

Or the other way to update all modules to the latest version is by entering the command `npx npm-update-all`

### Remove Module<a name="remove"></a>
Remove/ Uninstall module is also very straigh forward. <br/>
`npm remove <module>` or `npm rm <module>` will do the job for both regular and dev dependencies

### Summary<a name="summary"></a>

 Actions | Commands
--------|--------------------
Create NPM script <small>(package.json)</small>| npm init <small>(enter desire value)</small>
Create NPM script in faster way                | npm init -y <small>(auto filled in with default value)</small>
Set Default value in NPM script                | npm set init-< field name > "< value >"
Get Default value in NPM script                | npm get init-< field name >
Delete Default value in NPM script             | npm delete init-< field name >
Check NPM version                              | npm --version
Install Module                                 | npm install < module name > **OR** npm i < module name >
Install Module with specific version           | npm install < module name >@version
Install Module in DevDependencies              | npm install < module name > -D
Update Module                                  | npm update < module name >
Update All Modules                             | npx npm-update-all
Remove Module                                  | npm remove < module name > or npm rm < module >





