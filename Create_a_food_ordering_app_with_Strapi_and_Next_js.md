
# Create a food ordering app with Strapi and Next.js 6/7

![ordering-app](<https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(5)_6219e06f2b.png>)

_This tutorial is part of the « Cooking a Deliveroo clone with Next.js (React), GraphQL, Strapi and Stripe » tutorial series._
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3)
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4)
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5)
  - 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6) - **Current**
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7)

_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).

## **💵 Order and Checkout**

You must start starving... I am sure you want to be able to order!

![Order](https://d2zv2ciw0ln4h1.cloudfront.net/uploads/ezgif.com-optimize--5-.gif_3a4fd426d6.gif)

**Define Content Type**
You need to store the orders in the database, so a new Content Type will be created in Strapi.
Same process as usual:

- Navigate to the Content-Type Builder (http://localhost:1337/admin/plugins/content-type-builder).
- Click on `+ Create new collection type`.
- Set `order` as a name.
- Click on `Add New Field` and create the followings fields:
  - `address` with type `Text`.
  - `city` with type `Text`.
  - `dishes` with type `JSON`.
  - `amount` with type `Number` (big integer).
- Click on Finish then Save.
  ![Order Collection Type](https://paper-attachments.dropboxusercontent.com/s_380F58B14C9160F5FCBF5FB540473B78C40D968760D27913DB2FAC0F1D889120_1663419599070_Content-Type+Builder+-+Google+Chrome+2022-09-17+13-49-46.gif)

**Allow access**
To create new orders from the client, you are going to hit the `create` endpoint of the `order` API. To allow access, navigate to the Roles section ([http://localhost:1337/admin/plugins/users-permissions](http://localhost:1337/admin/settings/users-permissions/roles)), select the `authenticated` role, tick the `order/create` checkbox, and save.

![Order roles permission](/static/img/pixel.gif)

**Stripe setup**
In this section, you will need Stripe API keys. To get them, [create a Stripe account](https://dashboard.stripe.com/register) or [log in to Stripe](https://dashboard.stripe.com/login) then navigate to [https://dashboard.stripe.com/account/apikeys](https://dashboard.stripe.com/account/apikeys).
**Add logic**
If you have already used Stripe, you probably know the credit card information does not go through your backend server. Instead, the credit card information is sent to the Stripe API (ideally using their SDK). Then, your front end receives a token that can be used to charge credit cards. The `id` must be sent to your backend which will create the Stripe charge.
Not passing the credit card information through your server relieves you the responsibility to meet complicated data handling compliance, and is just far easier than worrying about securely storing sensitive data.
Install the `stripe` package in the **strapiapp** directory:

    npm i stripe --save

To integrate the Stripe logic, you need to update the `create` charge endpoint in our Strapi API. To do so, edit `strapiapp/src/api/order/controllers/order.js` and replace its content with:

> Make sure to insert your stripe secret key (sk\_) at the top where it instructs.

_Path:_ `*/strapiapp/src/api/order/controllers/order.js*`

    "use strict";
    const stripe = require("stripe")(
      "sk_test_randomStrings"
    );
    /**
     *  order controller
     */
    const { createCoreController } = require("@strapi/strapi").factories;
    module.exports = createCoreController("api::order.order", ({ strapi }) => ({
      async create(ctx) {
        const { address, amount, dishes, token, city, state } = ctx.request.body;
        try {
          // Charge the customer
          const charge = await stripe.charges.create({
            amount: amount * 100,
            currency: "usd",
            description: `Order ${new Date()} by ${ctx.state.user._id}`,
            source: token,
          });
          // Create the order
          const order = await strapi.service("api::order.order").create({
            data: {
              Amount: amount * 100,
              Address: address,
              Dishes: dishes,
              City: city,
              State: state,
            },
          });
          return order;
        } catch (err) {
          // return 500 error
          console.log("errorrr", err);
          ctx.response.status = 500;
          return {
            error: { message: "There was a problem creating the charge" },
            address,
            amount,
            dishes,
            token,
            city,
            state,
          };
        }
      },
    }));

_Note: in a real-world example, the amount should be checked on the backend side and the list of dishes related to the command should be stored in a more specific Content Type called_ `*orderDetail*`_._
Do not forget to restart the Strapi server.
To interact with the Stripe API, the [react-stripe-js](https://github.com/stripe/react-stripe-js) library will be used, which will give Elements components to style the credit card form and submit the information properly to Stripe.
**Checkout page**
Now install the stripe UI elements for the frontend. Open the frontend directory in the terminal and run the following command:

    npm install @stripe/react-stripe-js @stripe/stripe-js --legacy-peer-deps

Next, you are going to create the checkout form and card section component to capture the credit card info and pass it to Stripe using the react-stripe-elements package:
Create a folder named **checkout** in the components directory and create the file named **CheckoutForm.js.**
Once done, add the following code:
_Path:_ `*/frontend/frontend/components/checkout/CheckoutForm.js*`

    /* /components/Checkout/CheckoutForm.js */

    import React, { useState, useContext } from 'react';
    import { FormGroup, Label, Input } from 'reactstrap';
    import fetch from 'isomorphic-fetch';
    import Cookies from 'js-cookie';
    import { CardElement, useStripe, useElements } from '@stripe/react-stripe-js';
    import CardSection from './CardSection';
    import AppContext from '../../context/AppContext';
    import { useRouter } from 'next/router';
    function CheckoutForm() {
      const [data, setData] = useState({
        address: '',
        city: '',
        state: '',
        stripe_id: '',
      });
      const [error, setError] = useState('');
      const stripe = useStripe();
      const router = useRouter();
      const elements = useElements();
      const appContext = useContext(AppContext);
      function onChange(e) {
        // set the key = to the name property equal to the value typed
        const updateItem = (data[e.target.name] = e.target.value);
        // update the state data object
        setData({ ...data, updateItem });
        console.log(data);
      }
      async function submitOrder() {
        // event.preventDefault();
        // // Use elements.getElement to get a reference to the mounted Element.
        const cardElement = elements.getElement(CardElement);
        // // Pass the Element directly to other Stripe.js methods:
        // // e.g. createToken - https://stripe.com/docs/js/tokens_sources/create_token?type=cardElement
        // get token back from stripe to process credit card
        const token = await stripe.createToken(cardElement);
        const userToken = Cookies.get('token');
        try {
          const response = await fetch(`${
              process.env.STRAPI_URL || 'http://127.0.0.1:1337'
            }/api/orders`, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              Authorization: `Bearer ${userToken}`,
            },
            body: JSON.stringify({
              amount: Number(
                Math.round(appContext.cart.total + 'e2') + 'e-2'
              ),
              dishes: appContext.cart.items,
              address: data.address,
              city: data.city,
              state: data.state,
              token: token.token.id,
            }),
          });
          if (response.status === 200) {
            alert('Transaction Successful, continue your shopping');
            router.push('/');
          } else {
            alert(response.statusText);
          }
        } catch (error) {
          console.log('error', error);
        }
        // OTHER stripe methods you can use depending on app
        // // or createPaymentMethod - https://stripe.com/docs/js/payment_intents/create_payment_method
        // stripe.createPaymentMethod({
        //   type: "card",
        //   card: cardElement,
        // });
        // // or confirmCardPayment - https://stripe.com/docs/js/payment_intents/confirm_card_payment
        // stripe.confirmCardPayment(paymentIntentClientSecret, {
        //   payment_method: {
        //     card: cardElement,
        //   },
        // });
      }
      return (
        <div className="paper">
          <h5>Your information:</h5>
          <hr />
          <FormGroup style={{ display: 'flex' }}>
            <div style={{ flex: '0.90', marginRight: 10 }}>
              <Label>Address</Label>
              <Input name="address" onChange={onChange} />
            </div>
          </FormGroup>
          <FormGroup style={{ display: 'flex' }}>
            <div style={{ flex: '0.65', marginRight: '6%' }}>
              <Label>City</Label>
              <Input name="city" onChange={onChange} />
            </div>
            <div style={{ flex: '0.25', marginRight: 0 }}>
              <Label>State</Label>
              <Input name="state" onChange={onChange} />
            </div>
          </FormGroup>
          <CardSection
            data={data}
            stripeError={error}
            submitOrder={submitOrder}
          />
          <style jsx global>
            {`
              .paper {
                border: 1px solid lightgray;
                box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.2),
                  0px 1px 1px 0px rgba(0, 0, 0, 0.14),
                  0px 2px 1px -1px rgba(0, 0, 0, 0.12);
                height: 550px;
                padding: 30px;
                background: #fff;
                border-radius: 6px;
                margin-top: 90px;
              }
              .form-half {
                flex: 0.5;
              }
              * {
                box-sizing: border-box;
              }
              body,
              html {
                background-color: #f6f9fc;
                font-size: 18px;
                font-family: Helvetica Neue, Helvetica, Arial,
                  sans-serif;
              }
              h1 {
                color: #32325d;
                font-weight: 400;
                line-height: 50px;
                font-size: 40px;
                margin: 20px 0;
                padding: 0;
              }
              .Checkout {
                margin: 0 auto;
                max-width: 800px;
                box-sizing: border-box;
                padding: 0 5px;
              }
              label {
                color: #6b7c93;
                font-weight: 300;
                letter-spacing: 0.025em;
              }
              button {
                white-space: nowrap;
                border: 0;
                outline: 0;
                display: inline-block;
                height: 40px;
                line-height: 40px;
                padding: 0 14px;
                box-shadow: 0 4px 6px rgba(50, 50, 93, 0.11),
                  0 1px 3px rgba(0, 0, 0, 0.08);
                color: #fff;
                border-radius: 4px;
                font-size: 15px;
                font-weight: 600;
                text-transform: uppercase;
                letter-spacing: 0.025em;
                background-color: #6772e5;
                text-decoration: none;
                -webkit-transition: all 150ms ease;
                transition: all 150ms ease;
                margin-top: 10px;
              }
              form {
                margin-bottom: 40px;
                padding-bottom: 40px;
                border-bottom: 3px solid #e6ebf1;
              }
              button:hover {
                color: #fff;
                cursor: pointer;
                background-color: #7795f8;
                transform: translateY(-1px);
                box-shadow: 0 7px 14px rgba(50, 50, 93, 0.1),
                  0 3px 6px rgba(0, 0, 0, 0.08);
              }
              input,
              .StripeElement {
                display: block;
                background-color: #f8f9fa !important;
                margin: 10px 0 20px 0;
                max-width: 500px;
                padding: 10px 14px;
                font-size: 1em;
                font-family: 'Source Code Pro', monospace;
                box-shadow: rgba(50, 50, 93, 0.14902) 0px 1px 3px,
                  rgba(0, 0, 0, 0.0196078) 0px 1px 0px;
                border: 0;
                outline: 0;
                border-radius: 4px;
                background: white;
              }
              input::placeholder {
                color: #aab7c4;
              }
              input:focus,
              .StripeElement--focus {
                box-shadow: rgba(50, 50, 93, 0.109804) 0px 4px 6px,
                  rgba(0, 0, 0, 0.0784314) 0px 1px 3px;
                -webkit-transition: all 150ms ease;
                transition: all 150ms ease;
              }
              .StripeElement.IdealBankElement,
              .StripeElement.PaymentRequestButton {
                padding: 0;
              }
            `}
          </style>
        </div>
      );
    }
    export default CheckoutForm;

Now create a `CardSection.js` file to use the React Elements in, this will house the input boxes that will capture the CC information.

    $ touch CardSection.js

_Path:_ `*/frontend/frontend/components/checkout/CardSection.js*`

    /* components/Checkout/CardSection.js */

    import React from 'react';
    import { CardElement } from '@stripe/react-stripe-js';
    function CardSection(props) {
      return (
        <div>
          <div>
            <label htmlFor="card-element">Credit or debit card</label>
            <div>
              <fieldset style={{ border: 'none' }}>
                <div className="form-row">
                  <div id="card-element" style={{ width: '100%' }}>
                    <CardElement
                      options={{
                        style: {
                          width: '100%',
                          base: { fontSize: '18px' },
                        },
                      }}
                    />
                  </div>
                  <br />
                  <div className="order-button-wrapper">
                    <button onClick={props.submitOrder}>
                      Confirm order
                    </button>
                  </div>
                  {props.stripeError ? (
                    <div>{props.stripeError.toString()}</div>
                  ) : null}
                  <div id="card-errors" role="alert" />
                </div>
              </fieldset>
            </div>
          </div>
          <style jsx>
            {`
              .order-button-wrapper {
                display: flex;
                width: 100%;
                align-items: flex-end;
                justify-content: flex-end;
              }
            `}
          </style>
        </div>
      );
    }
    export default CardSection;

Once you’ve added the above code into the **CardSection.js** file, create the checkout page to bring it all together and input your **Public Key** where requested.
Create a new page: `pages/checkout.js/`,
_Path:_ `*/frontend/frontend/pages/checkout.js*`

    /* pages/checkout.js */

    import React, { useContext } from 'react';
    import { Row, Col } from 'reactstrap';
    import { loadStripe } from '@stripe/stripe-js';
    import { Elements } from '@stripe/react-stripe-js';
    import InjectedCheckoutForm from '../components/checkout/CheckoutForm';
    import AppContext from '../context/AppContext';
    import Cart from '../components/cart/';
    function Checkout() {
      // get app context
      const appContext = useContext(AppContext);
      // isAuthenticated is passed to the cart component to display order button
      const { isAuthenticated } = appContext;
      // load stripe to inject into elements components
      const stripePromise = loadStripe(
        'pk_test_randomStrings'
      );
      return (
        <Row>
          <Col
            style={{ paddingRight: 0 }}
            sm={{ size: 3, order: 1, offset: 2 }}
          >
            <h1 style={{ margin: 20 }}>Checkout</h1>
            <Cart isAuthenticated={isAuthenticated} />
          </Col>
          <Col style={{ paddingLeft: 5 }} sm={{ size: 6, order: 2 }}>
            <Elements stripe={stripePromise}>
              <InjectedCheckoutForm />
            </Elements>
          </Col>
        </Row>
      );
      // }
    }
    export default Checkout;

---

Now if you select a dish and click **order**, you should see:

![Checkout Page](https://paper-attachments.dropboxusercontent.com/s_380F58B14C9160F5FCBF5FB540473B78C40D968760D27913DB2FAC0F1D889120_1663688074049_image.png)

Now if you submit your order, you should see the order under the Strapi dashboard as follows:

![Strapi Order Content Manager](https://paper-attachments.dropboxusercontent.com/s_380F58B14C9160F5FCBF5FB540473B78C40D968760D27913DB2FAC0F1D889120_1663688652342_image.png)

**Explanations 🕵️**

---

**Note: explanations of code samples only, do not change your code to match this as you should already have this code this is simply a snippet**
The stripe library will be initialized by the `loadStripe('your key')` function/argument.
This will allow the stripe library to load the elements components.
This is what is rendering the card, zip and CVV on a single line.
Stripe will automatically detect which components are generating the CC information and what information to send to receive the token.
This submitOrder function will first make the call to Stripe with the CC information and receive back the Token if the CC check passed. If the token is received we next call the Strapi SDK to create the order passing in the appropriate extra order information and token id.
This is what creates the order in Stripe and creates the DB entry in Strapi. If successful you should see your Stripe test balances increase by the amount of the test order.

    async function submitOrder() {
        // event.preventDefault();

        // // Use elements.getElement to get a reference to the mounted Element.
        const cardElement = elements.getElement(CardElement);

        // // Pass the Element directly to other Stripe.js methods:
        // // e.g. createToken - https://stripe.com/docs/js/tokens_sources/create_token?type=cardElement
        // get token back from stripe to process credit card
        const token = await stripe.createToken(cardElement);
        const userToken = Cookies.get("token");
        const response = await fetch(`${process.env.STRAPI_URL}/api/orders`, {
          method: "POST",
          headers: userToken && { Authorization: `Bearer ${userToken}` },
          body: JSON.stringify({
            amount: Number(Math.round(appContext.cart.total + "e2") + "e-2"),
            dishes: appContext.cart.items,
            address: data.address,
            city: data.city,
            state: data.state,
            token: token.token.id,
          }),
        });

        if (!response.ok) {
          setError(response.statusText);
        }

        // OTHER stripe methods you can use depending on app
        // // or createPaymentMethod - https://stripe.com/docs/js/payment_intents/create_payment_method
        // stripe.createPaymentMethod({
        //   type: "card",
        //   card: cardElement,
        // });

        // // or confirmCardPayment - https://stripe.com/docs/js/payment_intents/confirm_card_payment
        // stripe.confirmCardPayment(paymentIntentClientSecret, {
        //   payment_method: {
        //     card: cardElement,
        //   },
        // });
      }

You are now able to let users submit their orders.
Bon appétit!
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
Once you’ve created the GitHub repo, open your terminal in the **strapiapp** directory and add the following lines of code. The code below will push the source code of your Strapi application to GitHub.

> Ensure you replace the repository link with the link to your repo.

Path: /backend/strapiapp

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

1.  Open your terminal in the **strapiapp** directory and run the following command that will install `pg-connection-string` and `pg`. The `pg-connection-string` contains functions for dealing with a PostgresSQL connection string.
    Path: /backend/strapiapp/
    npm install pg-connection-string pg
2.  Open the **Backend** folder, click on **strapiapp**, and select **database.js** from the **config** folder as shown below.
    ![database.js](https://paper-attachments.dropboxusercontent.com/s_6C0ECA10829B93D62998B06C40062587254EF184B701BBBA0364E3885E4704C3_1661481066701_image.png)

3.  Replace the following code into the **database.js** file to configure the PostgreSQL database.
    Path: /backend/strapiapp/config/server.js
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
    > Path: /backend/strapiapp/config/server.js
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
    > You can use your env values located in /backend/strapiapp/.env

Your Config Vars should be similar to the one shown below.

![Config Vars](https://paper-attachments.dropboxusercontent.com/s_AEDB67562F263740E493E7EB2EBD41033276C02EA96821D4AD94686E605FD0E5_1663805499696_image.png)

6. Lastly, update your GitHub by running the following command in your **strapiapp** terminal.
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
