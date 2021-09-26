+++
author = "Jeff Chang"
title = "Deploy Node Application to Heroku"
date = "2021-09-26"
description = "Heroku makes DevOps easy !!! Whether you're building a simple prototype or a business-critical product, Heroku's fully-managed platform gives you the simplest path to delivering your desire project"
tags = [
    "javascript", "devops"
]
categories = [
  "DevOps","NodeJs","Javascript"
]
metakeywords = "devops, nodejs, javascript, deploy node application to heroku"
image = "cover.jpg"
+++

# Sections
* [Prerequisite](#prerequisite)
* [Create a simple node app](#node)
* [Upload code to Github repository](#upload-github)
* [Deploy in Heroku with Github repository](#upload-heroku)
* [Configure environment variable](#env)

## Prerequisite<a name="prerequisite"></a>
- Make sure you have created an account in [heroku](https://signup.heroku.com/login) and [github](https://github.com/signup)

## Create a simple node app<a name="node"></a>
{{< highlight js>}}
const express = require('express');
const app = express();

app.get("/", (req, res) => {
    res.send("Server is Running")
})

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`App running on port ${port}...`);
});
{{< /highlight >}}

## Create a simple node app<a name="upload-github"></a>
- Login into Github Account
- Create New Repository
    * ![create repository](create_repository.jpg)
- Create `.gitignore` file to ignore files such as  'node_modules, .env and etc'
- Upload code into Github Repository via Git bash / Terminal
    *   {{< highlight js>}}
        git init
        git add .
        git commit -m "< commit message >"
        git branch -M master
        git remote add origin < your respository link >
        git push -u origin master
        {{< /highlight >}}


## Deploy in Heroku with Github repository<a name="upload-heroku"></a>
- Login into Heroku Account
- **New** -> **Create New App** -> **Enter project name and region** 
    * ![create project](new_project.jpg)

- Select Github as Deployment Method
    * ![connect project](connect_github.jpg)
    * ![select project](select_project.jpg)

- Setup Automate Deployment
    * Select the branch we wanted to trigger deployment when it's **updated / merged**
    * ![automate deployment](automate_deploy.JPG)

After the deployment is finished, you shall see your project in `https://<your project name>.herokuapp.com`

## Configure environment variable<a name="env"></a>
- Kindly select your project under dashboard project listing, then navigate to setting tab and select **Reveal Config Var**. 
    * ![reveal config var](setting.jpg)
- Now you're able to enter the environment variable inside here such as JWT Secret key, Access token and etc
    * ![setup config var](setting_2.jpg)