# Create a food ordering app with Strapi and Next.js 7/7

![ordering-app](https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(6)_d012e1aa4d.png)


*This tutorial is part of the « Cooking a Deliveroo clone with Next.js (*[*React*](https://strapi.io/integrations/react-cms)*), GraphQL, Strapi and Stripe » tutorial series.*
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3)
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4)
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5)
- 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6)
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7) - **Current**

**Note:** the source code is available on GitHub [here](https://github.com/divofred/food-ordering-app)

# 🚀 Bonus: deploy

At this point, you only need to deploy our API and the web app.
Strapi can be hosted on any major provider offering node deployments (Strapi Cloud, AWS, Heroku, DO, Railway, Render). Read further about the [deployment of Strapi](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/deployment.html).

# Prerequisites
- [Nodejs v16+](https://nodejs.org/en/download/) installed
- A [GitHub account](https://github.com/signup)
- A [Vercel account](https://vercel.com/signup)
# Deploying the Backend

In this article, we will deploy our backend Strapi source code to Render. [Render](https://render.com/) is s a fully-managed cloud platform where you can host static sites, backend APIs, databases, cron jobs, and all your other apps in one place. It is a fully-managed cloud platform where you can host static sites, backend APIs, databases, cron jobs, and all your other apps in one place.

## Getting Started

Let’s install the following packages that will help us to easily destructure our PostgreSQL database’s URL to obtain the PostgreSQL connection details.

    npm i pg pg-connection-string

Next, open the **database.js** file in the **config** folder and replace the code in it with the one below.

    const path = require("path");
    
    const parse = require("pg-connection-string").parse;
    const config = parse(process.env.DATABASE_URL);//Getting the URL from the Environment
    
    module.exports = ({ env }) => {
      const client = env("DATABASE_CLIENT", "postgres");
    
      const connections = {
        mysql: {
          connection: {
            connectionString: env("DATABASE_URL"),
            host: env("DATABASE_HOST", "127.0.0.1"),
            port: env.int("DATABASE_PORT", 5432),
            database: env("DATABASE_NAME", "strapi"),
            user: env("DATABASE_USERNAME", "strapi"),
            password: env("DATABASE_PASSWORD", "strapi"),
            ssl: env.bool("DATABASE_SSL", false) && {
              key: env("DATABASE_SSL_KEY", undefined),
              cert: env("DATABASE_SSL_CERT", undefined),
              ca: env("DATABASE_SSL_CA", undefined),
              capath: env("DATABASE_SSL_CAPATH", undefined),
              cipher: env("DATABASE_SSL_CIPHER", undefined),
              rejectUnauthorized: env.bool(
                "DATABASE_SSL_REJECT_UNAUTHORIZED",
                true
              ),
            },
          },
          pool: {
            min: env.int("DATABASE_POOL_MIN", 2),
            max: env.int("DATABASE_POOL_MAX", 10),
          },
        },
        postgres: {
          connection: {
            connectionString: env("DATABASE_URL"),//DESTRUCTURING THE URL
            host: config.host,
            port: config.port,
            database: config.database,
            user: config.user,
            password: config.password,
            ssl: env.bool("DATABASE_SSL", false) && {
              key: env("DATABASE_SSL_KEY", undefined),
              cert: env("DATABASE_SSL_CERT", undefined),
              ca: env("DATABASE_SSL_CA", undefined),
              capath: env("DATABASE_SSL_CAPATH", undefined),
              cipher: env(
                "DATABASE_SSL_CIPHER",
                "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256"
              ),
              rejectUnauthorized: env.bool(
                "DATABASE_SSL_REJECT_UNAUTHORIZED",
                true
              ),
            },
            schema: env("DATABASE_SCHEMA", "public"),
          },
          pool: {
            min: env.int("DATABASE_POOL_MIN", 2),
            max: env.int("DATABASE_POOL_MAX", 10),
          },
        },
        sqlite: {
          connection: {
            filename: path.join(
              __dirname,
              "..",
              env("DATABASE_FILENAME", "data.db")
            ),
          },
          useNullAsDefault: true,
        },
      };
    
      return {
        connection: {
          client,
          ...connections[client],
          acquireConnectionTimeout: env.int("DATABASE_CONNECTION_TIMEOUT", 60000),
        },
      };
    };

Now, create the following folders: **env** and **production,** and a file: **plugins.js**, in the **config** folder.

    config
     ┣ env
     ┃ ┗ production
     ┃ ┃ ┗ plugins.js

Add the following to the **plugins.js** file that will help config the graphql plugin.

    module.exports = {
      graphql: {
        config: {
          endpoint: "/graphql",
          shadowCRUD: true,
          playgroundAlways: true,
          depthLimit: 10,
          amountLimit: 100,
          apolloServer: {
            tracing: false,
            introspection: true,
          },
        },
      },
    };
> Source: https://github.com/strapi/strapi/issues/9105#issuecomment-1080073060

To deploy our Strapi’s source code to Render, we need to deploy the application to GitHub.

    git init
    git add .
    git commit -m "first commit"
    git branch -M main
    git remote add <repository url>
    git push -u origin main
## Setting up PostgreSQL

In this section, we will create a PostgreSQL database using the  Render services.

1. Navigate to [Render’s signup](https://dashboard.render.com/register) page to create an account if you don’t have one. You can choose to signup with your Google, GitHub or GitLab account.
2. Once the installation is successful, you’d be redirected to your [dashboard](https://dashboard.render.com/). Click on **New** and select **PostgreSQL.**
![Selecting PostgreSQL](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683316936470_image.png)

3. You’d be redirected to a screen where you’d be prompted to provide the name of the database and the user.
![name and user](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683317648824_image.png)



4. Lastly, select your desired plan and hit the **Create Database** button. This tutorial makes use of the free tier as seen in the output below.
![Create Database](/static/img/pixel.gif)


Following all these steps will successfully create our PostgreSQL database. Render has two kinds of PostgreSQL URLs, Internal and External Database URLs. Internal URL is used when the application that is using the Postgres database is hosted in Render. External URL is used when the application is hosted somewhere else and this requires more configuration. Since we are hosting our Strapi application on Render, we would make use of the Internal URL.
Scroll to the **Connections** section and copy the I**nternal Database URL**.

![Internal Database URL](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683320529898_image.png)

## Deploying Strapi
1. Once the database has been created successfully, select **New** and click on **Web Services.**
![New web service](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683320761950_image.png)

2. Next, you’d be prompted to connect your GitHub account. Once you’re connected, select **All repository or** specify the Strapi repository.
![Selecting repository](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683280942906_image.png)

3. Search for the Strapi repository and click **Connect**.
![GitHub repository](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683284094785_image.png)

4. Next, give the project a name and set the Root Directory to the folder that contains Strapi’s source code.
![Name and Root Directory](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683284523830_image.png)

5. Set the **Build Command** to `npm install && npm run build` and the **Start Command** to `npm start`
![Build and Start command](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683284678396_image.png)

6. This tutorial selects the free tier in the **Select Plan** section. You can choose to select another plan according to your taste.
7. Scroll to the bottom, click on **Advanced,** and let’s add the following to the environment variable. Paste the Internal Database URL you copied from the previous section.
    DATABASE_URL: Internal Database URL
    APP_KEYS: s4RAEbsJAcvpmMgGdWkmMQ==,/yh3FHR2u3SHyaSH8v1IZw==,5TcP4sY0KOl5t7a3ycuBwg==,thCs474mxnaSVY1+fk3QtA==
    JWT_SECRET: S8n42b7XulDz0A2z+/xeZQ==
    API_TOKEN_SALT: Onwo9oDUUf294KJ8W25IbQ==
    ADMIN_JWT_SECRET: ZOnwo9oDUUf294KJ8W25IbQ==
![Environmental Variable](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683321608422_image.png)

8. Lastly, scroll down and click **Create Web Service** and watch the build logs for any errors.
![logs](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683323344515_image.png)

> Feel free to comment in the comment section on any errors encountered while deploying the code.
## Setting up Strapi Admin

Once your Strapi project is live, you’d be provided with a URL that will be used to access your Strapi admin panel. This URL can be found at the top of the page.

![Strapi URL](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683323785289_strapi+url.png)


Copy out the URL and append `/admin` to access your admin dashboard. If you are not registered, you’d be prompted to create an admin user.

![](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683323920949_image.png)


In your Strapi’s admin dashboard, click on the following to configure permissions for the various content types.

    ┣ Settings
    ┃ ┣ Roles
    ┃ ┃ ┗ public
![Permissions](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683324612652_dashboard.png)


Select the specific CRUD operations you need for the various content types and hit **Save.**

![](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683324892709_content+type.png)

# Deploying the Frontend

In this tutorial, we will make use of the [Vercel](https://vercel.com/) hosting platform to deploy our NextJS code.

1. Head to your [Vercel dashboard](https://vercel.com/dashboard), select **Add New** and click on **Project.**
![Vercel’s dashboard](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683325641495_image.png)

2. Search for the repository and hit **import.**
![Import](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683325863893_image.png)

3. Under **Root Directory**, click on **Edit** and select the **Frontend** folder.
![Frontend folder](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683325983786_image.png)

4. Click on **Environment Variables** and add the deployed Strapi URL.
![](https://paper-attachments.dropboxusercontent.com/s_E9F590A058ABF42FBC974F087DB3AA257EB223FF42B5C24AABC3AECAC0572749_1683326143860_image.png)

5. Lastly, hit Deploy and look at the various logs for any errors.
# Conclusion

Huge congrats, you have successfully achieved this tutorial. I hope you enjoyed it!
**Note:** the source code is available on GitHub [here](https://github.com/divofred/food-ordering-app)
*Still hungry?*
Feel free to add additional features, adapt these projects to your own needs and give your feedback in the comments section.
*Share your meal!*
Did you enjoy this tutorial? Share it around you!

