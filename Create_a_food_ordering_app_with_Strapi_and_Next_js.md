
🚀 In the next (and last) section, you will learn how to **deploy your Strapi app on Heroku** and **your frontend app on** [**NOW**](https://zeit.co/now#get-started): [https://strapi.io/blog/nextjs-react-hooks-strapi-deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy).

# Create a food ordering app with Strapi and Next.js 7/7

![ordering-app](<https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(6)_d012e1aa4d.png>)

_This tutorial is part of the « Cooking a Deliveroo clone with Next.js (_[_React_](https://strapi.io/integrations/react-cms)_), GraphQL, Strapi and Stripe » tutorial series._
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3)
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4)
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5)
- 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6)
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7) - **Current**

_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).

## **🚀 Bonus: deploy**

At this point, you only need to deploy our API and the web app.
Strapi can be hosted on any major provider offering node deployments (AWS, Heroku, DO). Read further about the [deployment of Strapi](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/deployment.html)
**Backend**

## **Prerequisites**

- [Nodejs v16+](https://nodejs.org/en/download/) installed
- A [Heroku account](https://signup.heroku.com/)
- A [Netlify account](https://app.netlify.com/signup)
- [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) Installed
- [Netlify CLI](https://docs.netlify.com/cli/get-started/) Installed

## Goal

At the end of this guide, you should be able to:

- Deploy your Strapi app to Heroku
- Push your NextJS application to Vercel
- Connect your Strapi project to the Postgres add-on database provided by Heroku.

## Pushing your Strapi app to Github

There are several ways to deploy your application to Heroku which includes Heroku CLI, Docker, and GitHub. This article makes use of the GitHub method to deploy to Heroku.

> Feel free to use any [method](https://devcenter.heroku.com/categories/deployment) of your choosing.

To get started, ensure you have a GitHub account.
Create a GitHub repository. You can decide to name it anything, but this tutorial names it **strapi-app**.
Once you’ve created the GitHub repo, open your terminal in the **backend** directory and add the following lines of code. The code below will push the source code of your Strapi application to GitHub.

> Ensure you replace the repository link with the link to your repo.

Path: /backend/backend

    git init
    git add .
    git commit -m "food ordering app"
    git branch -M main
    git remote add origin https://github.com/yourUsername/strapi-app.git
    git push -u origin main

The output below shows the pushed source code to GitHub.

![GitHub Strapi-app](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663714713757_image.png)

## Connecting your GitHub to Heroku

For Heroku to have access to your GitHub code, it is important that you log in to your GitHub account in the same browser that your Heroku account is logged in.

1. Now, head on to your [Heroku dashboard](https://dashboard.heroku.com/new-app?org=personal-apps) to create a new Heroku application.
   ![Heroku new app](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663715776350_image.png)

2. Give your app a unique name that hasn’t been chosen, leave everything else with its default option and click on **create app**. Heroku will inform you if the provided name has been taken. The name of the app for this tutorial is **strapi-food-ordering-app,** ensure you use a different name.

![Strapi-food-ordering-app](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663716619750_image.png)

3. Click on [Connect to GitHub](https://dashboard.heroku.com/apps/strapi-food-ordering-app/deploy/github) to use the GitHub method. Then, authorize Heroku to access your GitHub repository.

![Heroku Authorize](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663717165965_image.png)

4. Now, enter the name of the GitHub repository, **strapi-app**, and click search. You should get an output similar to the one shown below.
   ![Heroku’s GitHub repo search](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663717578372_image.png)

5. Click on **Connect**, scroll down and **Enable Automatic Deploys**. When this is enabled, Heroku will automatically sync your GitHub repository to the Heroku application.

![Enable Automatic Deploy](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663717851824_image.png)

6. Lastly, scroll to the bottom and click deploy.

## Adding a database to Heroku

This tutorial has been using a local database to store all contents. Now that the Strapi app is deployed, you need to add an accessible database.
To do this, you need to use Heroku Postgres Addon as the PostgreSQL database.

1. Click on the resources tab and search for Heroku Postgres in the **Add-ons** sections.
   ![Heroku Postgres Add-ons](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663770623399_Heroku+Postgres.jpg)

2. Click on **Heroku Postgres** and you should be brought to output similar to the one shown below.

![Heroku Postgres](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663771234270_image.png)

3. Click on **Submit Order Form** and \***\*go to the **Settings\*\* tab.
   ![Settings tab](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663771739671_Settings+Tab.jpg)

4. Click on **Reveal Config Vars.**
   ![Reveal Config Vars](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663772001651_Reveal+config+vars.jpg)

Once you click on **Reveal Config Vars**, you will be able to get the database URL.

## Setting up Postgres in Strapi

In this section, you will instruct Strapi to use the Heroku Postgres database as its database.

1.  Open your terminal in the **backend** directory and run the following command that will install `pg-connection-string` and `pg`. The `pg-connection-string` contains functions for dealing with a PostgresSQL connection string.
    Path: /backend/backend/
    npm install pg-connection-string pg
2.  Open the **Backend** folder, click on **backend**, and select **database.js** from the **config** folder as shown below.
    ![database.js](https://paper-attachments.dropboxusercontent.com/s_6C0ECA10829B93D62998B06C40062587254EF184B701BBBA0364E3885E4704C3_1661481066701_image.png)

3.  Replace the following code into the **database.js** file to configure the PostgreSQL database.
    Path: /backend/backend/config/server.js
    const parse = require("pg-connection-string").parse;
    const config = parse(process.env.DATABASE_URL);
    module.exports = ({ env }) => ({
    connection: {
    client: "postgres",
    connection: {
    host: config.host,
    port: config.port,
    database: config.database,
    user: config.user,
    password: config.password,
    ssl: {
    rejectUnauthorized: false,
    },
    },
    debug: false,
    },
    });
4.  Open your **server.js** file located inside the **config** folder and add the following code.
    > Replace the URL with your own Heroku URL.
    > Path: /backend/backend/config/server.js
    > module.exports = ({ env }) => ({
    > url: env("https://strapi-food-ordering-app.herokuapp.com/"),
    > proxy: true,
    > app: {
        keys: env.array("APP_KEYS", ["testKey1", "testKey2"]),
    },
    });
5.  Open the **Settings** tab in your Heroku’s dashboard, Click on **Reveal Config Vars** and add the following Configs
    ADMIN_JWT_SECRET : Onwo9oDUUf294KJ8W25IbQ==
    API_TOKEN_SALT : Onwo9oDUUf294KJ8W25IbQ==
    APP_KEYS : s4RAEbsJAcvpmMgGdWkmMQ==,/yh3FHR2u3SHyaSH8v1IZw==,5TcP4sY0KOl5t7a3ycuBwg==,thCs474mxnaSVY1+fk3QtA==
    JWT_SECRET : S8n42b7XulDz0A2z+/xeZQ==
    > You can use your env values located in /backend/backend/.env

Your Config Vars should be similar to the one shown below.

![Config Vars](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663805499696_image.png)

6. Lastly, update your GitHub by running the following command in your **backend** terminal.
   git add .
   git commit -m "update"
   git push -u origin main

Heroku will automatically sync the updated files with the deployed ones.
Head on to your Heroku’s URL and view your application.

![Deployed Strapi Application](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663805801261_image.png)

> Feel free to comment on any error encountered in the comment sections.

## Deploying the NextJS app to Vercel

1. Push your NextJS code to a GitHub repository using the following code.
   git init
   git add .
   git commit -m "food-ordering-app"
   git branch -M main
   git remote add origin https://github.com/yourUsername/yourRepoName
   git push -u origin main
2. [Login](https://vercel.com/login) to your Vercel account or [Signup](https://vercel.com/signup) if you don’t have an account.
3. Once logged in, Create a [new project](https://vercel.com/new) and Add a GitHub account to Vercel.
   ![Vercel New](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663808106089_vercel+new.jpg)

4. Next, install Vercel in your GitHub account.
   ![Vercel Install](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663808447128_Vercel+Install.jpg)

5. Now that you have installed Vercel, you can now access your GitHub repository from Vercel, as shown below. Import the NextJS application repository.
   ![Vercel Import](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663808760147_vercel+import.jpg)

6. Choose a unique name for the NextJS application and hit Deploy.

![Vercel Deploy](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663808921169_image.png)

## Edits for the NextJS application

1. Open the **Layout.js** file and add the `async` option to the **stripe** script tag
   > This should resolve the synchronous error you might encounter
   <script src="https://js.stripe.com/v3" async />
2. Go to the project’s setting and add environment variable in Vercel, as shown below.
   ![](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663837432259_strapi-food-ordering-app++Overview++Vercel+-+Google+Chrome+2022-09-22+03-55-22.gif)

3. Lastly replace the following code with one in **next.config.js.**
   /\*_ @type {import('next').NextConfig} _/
   const nextConfig = {
   reactStrictMode: true,
   swcMinify: true,
   };
   module.exports = {
   nextConfig,
   env: {
   STRAPI_URL: process.env.STRAPI_URL,
   },
   };

> Ensure that the STRAPI_URL environmental variable does not have a forward slash at the end of the URL.

## **Conclusion**

Huge congrats, you successfully achieved this tutorial. I hope you enjoyed it!
_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).
_Still hungry?_
Feel free to add additional features, adapt these projects to your own needs and give your feedback in the comments section.
_Share your meal!_
Did you enjoy this tutorial? Share it around you!

```

```
