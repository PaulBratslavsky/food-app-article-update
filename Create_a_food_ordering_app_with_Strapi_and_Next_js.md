
# Create a food ordering app with Strapi and Next.js 3/7

![ordering-app](<https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(2)_619cc259ca.png>)

_This tutorial is part of the « Cooking a Deliveroo clone with_ [_Next.js_](https://strapi.io/integrations/nextjs-cms) _(React), GraphQL, Strapi and Stripe » tutorial series._
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3) - **Current**
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4)
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5)
- 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6)
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7)

_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).

## **🍔 Dishes list**

Congratulations, you successfully displayed the list of restaurants!
**Define Content Type**
Every restaurant sells dishes, which also must be stored in the database. Now a new Content Type is needed named `dish`. Create a new [Content Type](http://localhost:1337/admin/plugins/content-type-builder) the same way with the following attributes:

- `name` with type `Text`.
- `description` with type `Text (Long Text)`.
- `image` with type `Media (Single media)`.
- `price` with type `Number` (Decimal).
- `restaurant` with type `Relation`: this one is a bit more specific. Our purpose here is to tell Strapi that every dish can be related to a restaurant. To do so, create a one-to-many relation, as below.
  ![Dish Content Type](https://paper-attachments.dropboxusercontent.com/s_4BDD8E4420361B9C84C1B38752FA95BC0ECC7AC4EF3FFBCD44137BCE47B65F9D_1662411349776_Content-Type+Builder+-+Google+Chrome+2022-09-05+21-37-55.gif)

Relations in Strapi are shown below.

![Strapi relation](https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Screen-Shot-2018-11-07-at-17.10.39.png_cc14a24c8c.png)

Here is the final result:

![Dishes fields](https://paper-attachments.dropboxusercontent.com/s_4BDD8E4420361B9C84C1B38752FA95BC0ECC7AC4EF3FFBCD44137BCE47B65F9D_1662411667219_image.png)

> Don’t forget to set up [Roles and Permission](http://localhost:1337/admin/settings/users-permissions/roles) for the **Dishes** Content type

**Add some entries**
Then, add some dishes from the Content Manager: [http://localhost:1337/admin/plugins/content-manager/dish](http://localhost:1337/admin/content-manager/collectionType/api::dish.dish?page=1&pageSize=10&sort=id:ASC). Make sure they all have an image and are linked to a restaurant. As shown below

![Creating a Dish](https://paper-attachments.dropboxusercontent.com/s_4BDD8E4420361B9C84C1B38752FA95BC0ECC7AC4EF3FFBCD44137BCE47B65F9D_1662814338191_Content+Manager+-+Google+Chrome+2022-09-10+13-48-17.gif)

**Display dishes**
A new route called `/restaurant` will be used to display the dishes for a particular restaurant using its `id`.
Now create a new folder in the pages folder named **restaurant** and create a file named **[id].js**. This file named **[id].js** is responsible for retrieving the restaurant’s **id** from the URL and using this **id** to get the dishes for that restaurant.
Path: `/frontend/frontend/pages/restaurant/[id].js`

    /* /pages/restaurant/[id].js */
    import { useRouter } from 'next/router';
    import { gql, useQuery } from '@apollo/client';
    import {
      Button,
      Card,
      CardBody,
      CardImg,
      CardText,
      CardTitle,
      Col,
      Row,
    } from 'reactstrap';
    const GET_RESTAURANT_DISHES = gql`
      query ($id: ID!) {
        restaurant(id: $id) {
          data {
            id
            attributes {
              name
              dishes {
                data {
                  id
                  attributes {
                    name
                    description
                    price
                    image {
                      data {
                        attributes {
                          url
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    `;
    function Restaurants(props) {
      const router = useRouter();
      const { loading, error, data } = useQuery(GET_RESTAURANT_DISHES, {
        variables: { id: router.query.id },
      });
      if (error) return 'Error Loading Dishes';
      if (loading) return <h1>Loading ...</h1>;
      if (data.restaurant.data.attributes.dishes.data.length) {
        const { restaurant } = data;
        return (
          <>
            <h1>{restaurant.data.attributes.name}</h1>
            <Row>
              {restaurant.data.attributes.dishes.data.map(res => (
                <Col xs="6" sm="4" style={{ padding: 0 }} key={res.id}>
                  <Card style={{ margin: '0 10px' }}>
                    <CardImg
                      top={true}
                      style={{ height: 250 }}
                      src={`${
                          process.env.STRAPI_URL ||
                          'http://localhost:1337'
                        }${
                          res.attributes.image.data.attributes
                            .url
                        }`}
                    />
                    <CardBody>
                      <CardTitle tag="h5">
                        {res.attributes.name}
                      </CardTitle>
                      <CardText>
                        {res.attributes.description}
                      </CardText>
                    </CardBody>
                    <div className="card-footer">
                      <Button outline color="primary">
                        + Add To Cart
                      </Button>
                      <style jsx>
                        {`
                          a {
                            color: white;
                          }
                          a:link {
                            text-decoration: none;
                            color: white;
                          }
                          .container-fluid {
                            margin-bottom: 30px;
                          }
                          .btn-outline-primary {
                            color: #007bff !important;
                          }
                          a:hover {
                            color: white !important;
                          }
                        `}
                      </style>
                    </div>
                  </Card>
                </Col>
              ))}
            </Row>
          </>
        );
      }
      return <h1>Add Dishes</h1>;
    }
    export default Restaurants;

Once you’ve added the lines of code and hit save, head on to [localhost:3000](http://localhost:3000) to try out the application. You should be able to view the dishes attached to specific restaurants as seen in the GIF below.

![Viewing dishes](https://paper-attachments.dropboxusercontent.com/s_4BDD8E4420361B9C84C1B38752FA95BC0ECC7AC4EF3FFBCD44137BCE47B65F9D_1662815423182_Welcome+to+Nextjs+-+Google+Chrome+2022-09-10+14-08-07.gif)

🔐 In the next section, you will learn how to **authenticate users in your app** (register, logout & login): [https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4)

# Create a food ordering app with Strapi and Next.js 4/7

![ordering-app](<https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(3)_ac01b5c04b.png>)

_This tutorial is part of the « Cooking a Deliveroo clone with Next.js (React), GraphQL, Strapi and Stripe » tutorial series._
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3)
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4) - **Current**
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5)
- 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6)
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7)

_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).

## **🔐 Authentication**

For authentication calls, a POST request will be made to the respective register/login endpoints to register new users and login to existing users. Strapi will return a JWT token and a user object, the former can be used to verify transactions on the server while the user object will display the username in the header bar.
The Strapi documentation on authentication can be found here: [https://strapi.io/documentation/3.0.0-beta.x/guides/auth-request.html#authenticated-request](https://strapi.io/documentation/3.0.0-beta.x/guides/auth-request.html#authenticated-request)
Authentication with Next requires some additional consideration outside of a normal client-side authentication system because you have to be mindful of whether the code is being rendered on the client or the server. Because of the different constructs between a server and a client, client-only code should be prevented from running on the server.
One thing to keep in mind is that cookies are sent to the server in the request headers, so using something like next-cookies to universally retrieve the cookie value would work well. I'm not taking this approach in the tutorial, I will use the componentDidMount lifecycle inside the \_app.js file to grab my cookie. componentDidMount only fires client-side ensuring that we will have access to the cookie.
A simple check for the window object will prevent client-only code from being executed on the server.
This code would only run on the client:

    if (typeof window === "undefined") {
      return;
    }

Open the **frontend** directory in your terminal and install the following packages:

    npm install axios js-cookie isomorphic-fetch --legacy-peer-deps

Strapi's built-in JWT authentication will be used in this tutorial. This will allow you to easily register, manage, login, and check users' status in our application.
Your backend admin panel will provide a GUI for user management out of the box to view, edit, activate, and delete users if needed.
**User Content-Type in Strapi:**

![User Content Manager](https://paper-attachments.dropboxusercontent.com/s_B4CAABFA4FA6C43D3A9FFBE9FB6EFF14DEA70F2D8449F45D6ED2A4A504F387EF_1662847685448_image.png)

![Creating a user](https://paper-attachments.dropboxusercontent.com/s_B4CAABFA4FA6C43D3A9FFBE9FB6EFF14DEA70F2D8449F45D6ED2A4A504F387EF_1662847613788_image.png)

The premise of the JWT system is, a POST request will be sent to `http://localhost:1337/api/auth/local/register` with a username, email, and password to register a new user. This will return a user object and a JWT token in the user object will be stored in a cookie on the user's browser.
The same thing for logging in as a user, a POST to `http://localhost:1337/api/auth/local` with an email/password will return the same user object and JWT token if successful.
Strapi will also expose a `http://localhost:1337/api/users/me` route that a GET request will be sent to, passing a JWT as an authorization header. This will return the user object for user verification. The user object will be placed in a global context to share throughout the application.
Now, create a file named **auth.js** in the **lib** folder. This file will be charged with the responsibility of managing common authentication methods.
The `auth.js` file will create helper functions to log in, register and log out users. This is used to sync logouts across multiple logged-in tabs. However, it's not used in the tutorial, only provided for example if needed. A HOC is a component that returns a component, this allows you to increase the reusability of the components across your application, a common code pattern in React.
Read more here about [HOCs](https://reactjs.org/docs/higher-order-components.html)
`/lib/auth.js`

    /* /lib/auth.js */

    import { useEffect } from "react";
    import Router from "next/router";
    import Cookie from "js-cookie";
    import axios from "axios";

    const API_URL = process.env.STRAPI_URL || "http://localhost:1337";

    //register a new user
    export const registerUser = (username, email, password) => {
      //prevent the function from being run on the server
      if (typeof window === "undefined") {
        return;
      }
      return new Promise((resolve, reject) => {
        axios
          .post(`${API_URL}/auth/local/register`, { username, email, password })
          .then((res) => {
            //set token response from Strapi for server validation
            Cookie.set("token", res.data.jwt);

            //resolve the promise to set loading to false in SignUp form
            resolve(res);
            //redirect back to the home page for restaurant selection
            Router.push("/");
          })
          .catch((error) => {
            //reject the promise and pass the error object back to the form
            reject(error);
          });
      });
    };

    export const login = (identifier, password) => {
      //prevent the function from being run on the server
      if (typeof window === "undefined") {
        return;
      }

      return new Promise((resolve, reject) => {
        axios
          .post(`${API_URL}/auth/local/`, { identifier, password })
          .then((res) => {
            //set token response from Strapi for server validation
            Cookie.set("token", res.data.jwt);

            //resolve the promise to set loading to false in SignUp form
            resolve(res);
            //redirect back to the home page for restaurant selection
            Router.push("/");
          })
          .catch((error) => {
            //reject the promise and pass the error object back to the form
            reject(error);
          });
      });
    };

    export const logout = () => {
      //remove token and user cookie
      Cookie.remove("token");
      delete window.__user;
      // sync logout between multiple windows
      window.localStorage.setItem("logout", Date.now());
      //redirect to the home page
      Router.push("/");
    };

    //Higher Order Component to wrap our pages and logout simultaneously logged-in tabs
    // THIS IS NOT USED in the tutorial, only provided if you wanted to implement
    export const withAuthSync = (Component) => {
      const Wrapper = (props) => {
        const syncLogout = (event) => {
          if (event.key === "logout") {
            Router.push("/login");
          }
        };

        useEffect(() => {
          window.addEventListener("storage", syncLogout);

          return () => {
            window.removeEventListener("storage", syncLogout);
            window.localStorage.removeItem("logout");
          };
        }, []);

        return <Component {...props} />;
      };

      if (Component.getInitialProps) {
        Wrapper.getInitialProps = Component.getInitialProps;
      }

      return Wrapper;
    };

**Authentication**
For authentication, the JWT token returned from Strapi will be used for authentication.
Most of the time, progressive web apps store a JSON Web Token (JWT) in the local storage. That works pretty well if you don't have server-side rendering (tokens are also stored as a cookie).
Since [Next.js](https://strapi.io/integrations/nextjs-cms) renders code on the server, you will need to store the token returned from Strapi as a cookie in the browser, since localStorage is not accessible on the server. This allows the client to set the cookie with a package like **js-cookie** using the code, **Cookie.set(cookie)**.
Our token management will happen client-side only, however, your application could be developed differently in the real world.
**React Context**
To store the user object, you will need to create a global context state inside of the application. The context in React allows you to prevent prop-drilling multiple levels down and lets you grab and use the context state locally from a component.
This is a powerful construct in React, and definitely worth reading more info here: https://reactjs.org/docs/context.html.
Start by creating a folder named **context** in the **frontend** directory, then create a file named **AppContext.js** and add the following lines of code:
`path: /frontend/frontend/context/AppContext.js`

    /* /context/AppContext.js */

    import React from 'react';
    // create auth context with default value
    // set backup default for isAuthenticated if none is provided in Provider
    const AppContext = React.createContext({ isAuthenticated: false });
    export default AppContext;

Now, add state to your **\_app.js** file to share across the application.
You could also locate this in any container component in your application.
`path: /pages/_app.js`

    /* _app.js */
    import React from 'react';
    import App from 'next/app';
    import Head from 'next/head';
    import Cookie from 'js-cookie';
    import fetch from 'isomorphic-fetch';
    import Layout from '../components/Layout';
    import AppContext from '../context/AppContext';
    import withData from '../lib/apollo';
    class MyApp extends App {
      state = {
        user: null,
      };
      componentDidMount() {
        // grab the token value from the cookie
        const token = Cookie.get('token');
        if (token) {
          // authenticate the token on the server and place set user object
          fetch(
            `${
              process.env.STRAPI_URL || `http://localhost:1337`
            }/api/users/me`,
            {
              headers: {
                Authorization: `Bearer ${token}`,
              },
            }
          ).then(async res => {
            // if res comes back not valid, token is not valid
            // delete the token and log the user out on client
            if (!res.ok) {
              Cookie.remove('token');
              this.setState({ user: null });
              return null;
            }
            const user = await res.json();
            this.setUser(user);
          });
        }
      }
      setUser = user => {
        this.setState({ user });
      };
      render() {
        const { Component, pageProps } = this.props;
        return (
          <AppContext.Provider
            value={{
              user: this.state.user,
              isAuthenticated: !!this.state.user,
              setUser: this.setUser,
            }}
          >
            <Head>
              <link
                rel="stylesheet"
                href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
                integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
                crossOrigin="anonymous"
              />
            </Head>
            <Layout>
              <Component {...pageProps} />
            </Layout>
          </AppContext.Provider>
        );
      }
    }
    export default withData({ ssr: true })(MyApp);

Then update your header bar as well to display our username and a logout button if a user is signed in:
`/path/components/Layout.js`

    /* /components/Layout.js */
    import React, { useContext } from 'react';
    import Head from 'next/head';
    import Link from 'next/link';
    import { Container, Nav, NavItem } from 'reactstrap';
    import { logout } from '../lib/auth';
    import AppContext from '../context/AppContext';
    const Layout = props => {
      const title = 'Welcome to Nextjs';
      const { user, setUser } = useContext(AppContext);
      return (
        <div>
          <Head>
            <title>{title}</title>
            <meta charSet="utf-8" />
            <meta
              name="viewport"
              content="initial-scale=1.0, width=device-width"
            />
            <link
              rel="stylesheet"
              href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
              integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
              crossOrigin="anonymous"
            />
            <script src="https://js.stripe.com/v3" />
          </Head>
          <header>
            <style jsx>
              {`
                a {
                  color: white;
                }
                h5 {
                  color: white;
                  padding-top: 11px;
                }
              `}
            </style>
            <Nav className="navbar navbar-dark bg-dark">
              <NavItem>
                <Link href="/">
                  <a className="navbar-brand">Home</a>
                </Link>
              </NavItem>
              <NavItem className="ml-auto">
                {user ? (
                  <h5>{user.username}</h5>
                ) : (
                  <Link href="/register">
                    <a className="nav-link"> Sign up</a>
                  </Link>
                )}
              </NavItem>
              <NavItem>
                {user ? (
                  <Link href="/">
                    <a
                      className="nav-link"
                      onClick={() => {
                        logout();
                        setUser(null);
                      }}
                    >
                      Logout
                    </a>
                  </Link>
                ) : (
                  <Link href="/login">
                    <a className="nav-link">Sign in</a>
                  </Link>
                )}
              </NavItem>
            </Nav>
          </header>
          <Container>{props.children}</Container>
        </div>
      );
    };
    export default Layout;

**Register**
To register a user you will pass a username, email, and password through a POST request to the route `/api/auth/local/register`. This will register a user in Strapi and log the user in. Inside the signup page, you will call the registerUser function created inside of the auth.js file to register the user, then set the JWT cookie inside the browser.
Request body options for auth:

    NewUsers-PermissionsUser{
    username*                 string
                          minLength: 3
    email*                    string
                          minLength: 6
    provider                  string
    password                  string
                          minLength: 6
    resetPasswordToken        string
    confirmed                 boolean
                          default:  false
    blocked                   boolean
                          default: false
    role        string
    }

Swagger docs for the register endpoint:

![swagger](https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Screen-Shot-2019-10-06-at-3.50.32-PM_f545678b60.png)

Path: `/frontend/pages/register.js`

    /* /pages/register.js */

    import React, { useState, useContext } from 'react';
    import {
      Container,
      Row,
      Col,
      Button,
      Form,
      FormGroup,
      Label,
      Input,
    } from 'reactstrap';
    import { registerUser } from '../lib/auth';
    import AppContext from '../context/AppContext';
    const Register = () => {
      const [data, setData] = useState({ email: '', username: '', password: '' });
      const [loading, setLoading] = useState(false);
      const [error, setError] = useState({});
      const appContext = useContext(AppContext);
      return (
        <Container>
          <Row>
            <Col sm="12" md={{ size: 5, offset: 3 }}>
              <div className="paper">
                <div className="header">
                  <img
                    src="https://super-static-assets.s3.amazonaws.com/e7c0f16c-8bd3-4c76-8075-4c86f986e1b2/images/4ddec037-40d0-4919-ab34-adbd9f23c6a1.svg"
                    style={{ width: '75%' }}
                  />
                </div>
                <section className="wrapper">
                  {Object.entries(error).length !== 0 &&
                    error.constructor === Object && (
                      <div
                        key={error.error.name}
                        style={{ marginBottom: 10 }}
                      >
                        <small style={{ color: 'red' }}>
                          {error.error.message}
                        </small>
                      </div>
                    )}
                  <Form>
                    <fieldset disabled={loading}>
                      <FormGroup>
                        <Label>Username:</Label>
                        <Input
                          disabled={loading}
                          onChange={e =>
                            setData({
                              ...data,
                              username: e.target.value,
                            })
                          }
                          value={data.username}
                          type="text"
                          name="username"
                          style={{
                            height: 50,
                            fontSize: '1.2em',
                          }}
                        />
                      </FormGroup>
                      <FormGroup>
                        <Label>Email:</Label>
                        <Input
                          onChange={e =>
                            setData({
                              ...data,
                              email: e.target.value,
                            })
                          }
                          value={data.email}
                          type="email"
                          name="email"
                          style={{
                            height: 50,
                            fontSize: '1.2em',
                          }}
                        />
                      </FormGroup>
                      <FormGroup style={{ marginBottom: 30 }}>
                        <Label>Password:</Label>
                        <Input
                          onChange={e =>
                            setData({
                              ...data,
                              password: e.target.value,
                            })
                          }
                          value={data.password}
                          type="password"
                          name="password"
                          style={{
                            height: 50,
                            fontSize: '1.2em',
                          }}
                        />
                      </FormGroup>
                      <FormGroup>
                        <span>
                          <a href="">
                            <small>Forgot Password?</small>
                          </a>
                        </span>
                        <Button
                          style={{
                            float: 'right',
                            width: 120,
                          }}
                          color="primary"
                          disabled={loading}
                          onClick={() => {
                            setLoading(true);
                            registerUser(
                              data.username,
                              data.email,
                              data.password
                            )
                              .then(res => {
                                // set authed user in global context object
                                appContext.setUser(
                                  res.data.user
                                );
                                setLoading(false);
                              })
                              .catch(error => {
                                setError(
                                  error.response.data
                                );
                                setLoading(false);
                              });
                          }}
                        >
                          {loading ? 'Loading..' : 'Submit'}
                        </Button>
                      </FormGroup>
                    </fieldset>
                  </Form>
                </section>
              </div>
            </Col>
          </Row>
          <style jsx>
            {`
              .paper {
                border: 1px solid lightgray;
                box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.2),
                  0px 1px 1px 0px rgba(0, 0, 0, 0.14),
                  0px 2px 1px -1px rgba(0, 0, 0, 0.12);
                border-radius: 6px;
                margin-top: 90px;
              }
              .notification {
                color: #ab003c;
              }
              .header {
                width: 100%;
                height: 120px;
                background-color: #2196f3;
                margin-bottom: 30px;
                border-radius-top: 6px;
              }
              .wrapper {
                padding: 10px 30px 20px 30px !important;
              }
              a {
                color: blue !important;
              }
              img {
                margin: 15px 30px 10px 50px;
              }
            `}
          </style>
        </Container>
      );
    };
    export default Register;

You should get an output similar to the one shown below;

![Register route](https://paper-attachments.dropboxusercontent.com/s_B4CAABFA4FA6C43D3A9FFBE9FB6EFF14DEA70F2D8449F45D6ED2A4A504F387EF_1663080897279_image.png)

**Login**
Similar to the sign-up page, the sign-in page will use a token to log the user in and set the cookie.
Path: `/frontend/frontend/pages/login.js`

    /* /pages/login.js */

    import React, { useState, useEffect, useContext } from 'react';
    import { useRouter } from 'next/router';
    import {
      Container,
      Row,
      Col,
      Button,
      Form,
      FormGroup,
      Label,
      Input,
    } from 'reactstrap';
    import { login } from '../lib/auth';
    import AppContext from '../context/AppContext';
    function Login(props) {
      const [data, updateData] = useState({ identifier: '', password: '' });
      const [loading, setLoading] = useState(false);
      const [error, setError] = useState(false);
      const router = useRouter();
      const appContext = useContext(AppContext);
      useEffect(() => {
        if (appContext.isAuthenticated) {
          router.push('/'); // redirect if you're already logged in
        }
      }, []);
      function onChange(event) {
        updateData({ ...data, [event.target.name]: event.target.value });
      }
      return (
        <Container>
          <Row>
            <Col sm="12" md={{ size: 5, offset: 3 }}>
              <div className="paper">
                <div className="header">
                  <img
                    src="https://super-static-assets.s3.amazonaws.com/e7c0f16c-8bd3-4c76-8075-4c86f986e1b2/images/4ddec037-40d0-4919-ab34-adbd9f23c6a1.svg"
                    style={{ width: '75%' }}
                  />
                </div>
                <section className="wrapper">
                  {Object.entries(error).length !== 0 &&
                    error.constructor === Object && (
                      <div
                        key={error.error.name}
                        style={{ marginBottom: 10 }}
                      >
                        <small style={{ color: 'red' }}>
                          {error.error.message}
                        </small>
                      </div>
                    )}
                  <Form>
                    <fieldset disabled={loading}>
                      <FormGroup>
                        <Label>Email:</Label>
                        <Input
                          onChange={event => onChange(event)}
                          name="identifier"
                          style={{
                            height: 50,
                            fontSize: '1.2em',
                          }}
                        />
                      </FormGroup>
                      <FormGroup style={{ marginBottom: 30 }}>
                        <Label>Password:</Label>
                        <Input
                          onChange={event => onChange(event)}
                          type="password"
                          name="password"
                          style={{
                            height: 50,
                            fontSize: '1.2em',
                          }}
                        />
                      </FormGroup>
                      <FormGroup>
                        <span>
                          <a href="">
                            <small>Forgot Password?</small>
                          </a>
                        </span>
                        <Button
                          style={{
                            float: 'right',
                            width: 120,
                          }}
                          color="primary"
                          onClick={() => {
                            setLoading(true);
                            login(
                              data.identifier,
                              data.password
                            )
                              .then(res => {
                                setLoading(false);
                                // set authed User in global context to update header/app state
                                appContext.setUser(
                                  res.data.user
                                );
                              })
                              .catch(error => {
                                setError(
                                  error.response.data
                                );
                                setLoading(false);
                              });
                          }}
                        >
                          {loading ? 'Loading... ' : 'Submit'}
                        </Button>
                      </FormGroup>
                    </fieldset>
                  </Form>
                </section>
              </div>
            </Col>
          </Row>
          <style jsx>
            {`
              .paper {
                border: 1px solid lightgray;
                box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.2),
                  0px 1px 1px 0px rgba(0, 0, 0, 0.14),
                  0px 2px 1px -1px rgba(0, 0, 0, 0.12);
                border-radius: 6px;
                margin-top: 90px;
              }
              .notification {
                color: #ab003c;
              }
              .header {
                width: 100%;
                height: 120px;
                background-color: #2196f3;
                margin-bottom: 30px;
                border-radius-top: 6px;
              }
              .wrapper {
                padding: 10px 30px 20px 30px !important;
              }
              a {
                color: blue !important;
              }
              img {
                margin: 15px 30px 10px 50px;
              }
            `}
          </style>
        </Container>
      );
    }
    export default Login;

You should get an output similar to the one shown below;

![Login route](https://paper-attachments.dropboxusercontent.com/s_B4CAABFA4FA6C43D3A9FFBE9FB6EFF14DEA70F2D8449F45D6ED2A4A504F387EF_1663081196323_image.png)

Your user registration, login, and logout should be set correctly!
🛒 In the next section, you will learn how to **create a full-featured shopping cart**: [https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5)

# Create a food ordering app with Strapi and Next.js 5/7

![ordering-app](<https://d2zv2ciw0ln4h1.cloudfront.net/uploads/Deliver_Clone_Next.js_(4)_ad8b2cd7ea.png>)

_This tutorial is part of the « Cooking a Deliveroo clone with Next.js (React), GraphQL, Strapi and Stripe » tutorial series._
**Table of contents**

- 🏗️ [Setup](https://strapi.io/blog/nextjs-react-hooks-strapi-food-app-1) (part 1)
- 🏠 [Restaurants](https://strapi.io/blog/nextjs-react-hooks-strapi-restaurants-2) (part 2)
- 🍔 [Dishes](https://strapi.io/blog/nextjs-react-hooks-strapi-dishes-3) (part 3)
- 🔐 [Authentication](https://strapi.io/blog/nextjs-react-hooks-strapi-auth-4) (part 4)
- 🛒 [Shopping Cart](https://strapi.io/blog/nextjs-react-hooks-strapi-shopping-cart-5) (part 5) - **Current**
- 💵 [Order and Checkout](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6) (part 6)
- 🚀 [Bonus: Deploy](https://strapi.io/blog/nextjs-react-hooks-strapi-deploy) (part 7)

_Note: the source code is available on GitHub for_ [_Frontend_](https://github.com/divofred/food-ordering-app) and [Backend](https://github.com/divofred/strapi-app).

## **🛒 Shopping Cart**

These dishes look so tasty! What if you could add some of them to a shopping cart?
Create a new folder in the **component** directory named, create a file named **index.js**, and add the following lines of codes to the **index.js** file
_Path:_ `*/frontend/frontend/components/cart/index.js*`

    /* components/cart/index.js */

    import React, { useContext } from 'react';
    import Link from 'next/link';
    import { useRouter } from 'next/router';
    import { Button, Card, CardBody, CardTitle, Badge } from 'reactstrap';
    import AppContext from '../../context/AppContext';
    function Cart() {
      const appContext = useContext(AppContext);
      const router = useRouter();
      const { cart, isAuthenticated } = appContext;
      return (
        <div>
          <Card style={{ padding: '10px 5px' }} className="cart">
            <CardTitle style={{ margin: 10 }}>Your Order:</CardTitle>
            <hr />
            <CardBody style={{ padding: 10 }}>
              <div style={{ marginBottom: 6 }}>
                <small>Items:</small>
              </div>
              <div>
                {cart.items
                  ? cart.items.map(item => {
                      if (item.quantity > 0) {
                        return (
                          <div
                            className="items-one"
                            style={{ marginBottom: 15 }}
                            key={Math.random()}
                          >
                            <div>
                              <span id="item-price">
                                &nbsp; $
                                {
                                  item.res.attributes
                                    .price
                                }
                              </span>
                              <span id="item-name">
                                &nbsp;{' '}
                                {
                                  item.res.attributes
                                    .name
                                }
                              </span>
                            </div>
                            <div>
                              <Button
                                style={{
                                  height: 25,
                                  padding: 0,
                                  width: 15,
                                  marginRight: 5,
                                  marginLeft: 10,
                                }}
                                onClick={() =>
                                  appContext.addItem(
                                    item
                                  )
                                }
                                color="link"
                              >
                                +
                              </Button>
                              <Button
                                style={{
                                  height: 25,
                                  padding: 0,
                                  width: 15,
                                  marginRight: 10,
                                }}
                                onClick={() =>
                                  appContext.removeItem(
                                    item
                                  )
                                }
                                color="link"
                              >
                                -
                              </Button>
                              <span
                                style={{
                                  marginLeft: 5,
                                }}
                                id="item-quantity"
                              >
                                {item.quantity}x
                              </span>
                            </div>
                          </div>
                        );
                      }
                    })
                  : null}
                {isAuthenticated ? (
                  cart.items.length > 0 ? (
                    <div>
                      <Badge
                        style={{ width: 200, padding: 10 }}
                        color="light"
                      >
                        <h5
                          style={{
                            fontWeight: 100,
                            color: 'gray',
                          }}
                        >
                          Total:
                        </h5>
                        <h3>
                          ${appContext.cart.total.toFixed(2)}
                        </h3>
                      </Badge>
                      <div
                        style={{
                          marginTop: 10,
                          marginRight: 10,
                        }}
                      >
                        <Link href="/checkout">
                          <Button
                            style={{ width: '100%' }}
                            color="primary"
                          >
                            <a>Order</a>
                          </Button>
                        </Link>
                      </div>
                    </div>
                  ) : (
                    <>
                      {router.pathname === '/checkout' && (
                        <small
                          style={{ color: 'blue' }}
                          onClick={() =>
                            window.history.back()
                          }
                        >
                          back to restaurant
                        </small>
                      )}
                    </>
                  )
                ) : (
                  <h5>Login to Order</h5>
                )}
              </div>
            </CardBody>
          </Card>
          <style jsx>{`
            #item-price {
              font-size: 1.3em;
              color: rgba(97, 97, 97, 1);
            }
            #item-quantity {
              font-size: 0.95em;
              padding-bottom: 4px;
              color: rgba(158, 158, 158, 1);
            }
            #item-name {
              font-size: 1.3em;
              color: rgba(97, 97, 97, 1);
            }
          `}</style>
        </div>
      );
    }
    export default Cart;

## **React Context**

To keep track of the items added to the cart across pages you will use the React Context API. This will prevent having to prop drill the items multiple levels deep. The context will allow you to manage the state of items that will be re-used on the checkout page. The only thing React Context will not take care of for you is preserving items through a page refresh, for that, you would want to save the items to a cookie or DB and restore them that way in a real-world application.
The items are currently saved to a cookie called items and restored if the user closes the browser/tab.
All items will be added to the \_app.js file to make it easy to manipulate/change. This could live in any container of your choosing, it would be a good idea to move it out if your functions/state become large.
Now you will need to make some changes to use the Context throughout the application and on the dishes page.
Update the `_app.js` and `/pages/restaurant/[id].js` files to use the AppProvider/Consumer components using the following lines of code:
Path: `/frontend/frontend/pages/_app.js`

    /* _app.js */
    import React from 'react';
    import App from 'next/app';
    import Head from 'next/head';
    import Cookie from 'js-cookie';
    import fetch from 'isomorphic-fetch';
    import Layout from '../components/Layout';
    import AppContext from '../context/AppContext';
    import withData from '../lib/apollo';
    class MyApp extends App {
      state = {
        user: null,
        cart: { items: [], total: 0 },
      };
      componentDidMount() {
        const token = Cookie.get('token');
        // restore cart from cookie, this could also be tracked in a db
        const cart = Cookie.get('cart');
        if (typeof cart === 'string' && cart !== 'undefined') {
          let sum = 0;
          JSON.parse(cart).forEach(item => {
            sum += item.res.attributes.price * item.quantity;
            this.setState({
              cart: {
                items: JSON.parse(cart),
                total: sum,
              },
            });
          });
        }
        if (token) {
          // authenticate the token on the server and place set user object
          fetch(`${
              process.env.STRAPI_URL || `http://localhost:1337`
            }/api/users/me`, {
            headers: {
              Authorization: `Bearer ${token}`,
            },
          }).then(async res => {
            // if res comes back not valid, token is not valid
            // delete the token and log the user out on client
            if (!res.ok) {
              Cookie.remove('token');
              this.setState({ user: null });
              return null;
            }
            const user = await res.json();
            this.setUser(user);
          });
        }
      }
      setUser = user => {
        this.setState({ user });
      };
      addItem = item => {
        let { items } = this.state.cart;
        //check for item already in cart
        //if not in cart, add item if item is found increase quantity ++
        let newItem = items.find(i => i.res.id === item.res.id);
        // if item is not new, add to cart, set quantity to 1
        if (!newItem) {
          //set quantity property to 1
          item.quantity = 1;
          this.setState(
            {
              cart: {
                items: [...items, item],
                total:
                  this.state.cart.total + item.res.attributes.price,
              },
            },
            () => Cookie.set('cart', JSON.stringify(this.state.cart.items))
          );
        } else {
          this.setState(
            {
              cart: {
                items: this.state.cart.items.map(item =>
                  item.res.id === newItem.res.id
                    ? Object.assign({}, item, {
                        quantity: item.quantity + 1,
                      })
                    : item
                ),
                total:
                  this.state.cart.total + item.res.attributes.price,
              },
            },
            () => Cookie.set('cart', JSON.stringify(this.state.cart.items))
          );
        }
      };
      removeItem = item => {
        let { items } = this.state.cart;
        //check for item already in cart
        //if not in cart, add item if item is found increase quantity ++
        let newItem = items.find(i => i.res.id === item.res.id);
        if (newItem.quantity > 1) {
          this.setState(
            {
              cart: {
                items: this.state.cart.items.map(item =>
                  item.res.id === newItem.res.id
                    ? Object.assign({}, item, {
                        quantity: item.quantity - 1,
                      })
                    : item
                ),
                total:
                  this.state.cart.total - item.res.attributes.price,
              },
            },
            () => Cookie.set('cart', JSON.stringify(this.state.cart.items))
          );
        } else {
          const items = [...this.state.cart.items];
          const index = items.findIndex(i => i.res.id === item.res.id);
          items.splice(index, 1);
          this.setState(
            {
              cart: {
                items: items,
                total:
                  this.state.cart.total - item.res.attributes.price,
              },
            },
            () => Cookie.set('cart', JSON.stringify(this.state.cart.items))
          );
        }
      };
      render() {
        const { Component, pageProps } = this.props;
        return (
          <AppContext.Provider
            value={{
              user: this.state.user,
              isAuthenticated: !!this.state.user,
              setUser: this.setUser,
              cart: this.state.cart,
              addItem: this.addItem,
              removeItem: this.removeItem,
            }}
          >
            <Head>
              <link
                rel="stylesheet"
                href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
                integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
                crossOrigin="anonymous"
              />
            </Head>
            <Layout>
              <Component {...pageProps} />
            </Layout>
          </AppContext.Provider>
        );
      }
    }
    export default withData({ ssr: true })(MyApp);

Path: `/frontend/pages/restaurant/[id].js`

    /* /pages/restaurant/[id].js */

    import { useContext } from 'react';
    import { useRouter } from 'next/router';
    import { gql, useQuery } from '@apollo/client';
    import Cart from '../../components/cart/';
    import AppContext from '../../context/AppContext';
    import {
      Button,
      Card,
      CardBody,
      CardImg,
      CardText,
      CardTitle,
      Col,
      Row,
    } from 'reactstrap';
    const GET_RESTAURANT_DISHES = gql`
      query ($id: ID!) {
        restaurant(id: $id) {
          data {
            id
            attributes {
              name
              dishes {
                data {
                  id
                  attributes {
                    name
                    description
                    price
                    image {
                      data {
                        attributes {
                          url
                        }
                      }
                    }
                  }
                }
              }
            }
          }
        }
      }
    `;
    function Restaurants(props) {
      const appContext = useContext(AppContext);
      const router = useRouter();
      const { loading, error, data } = useQuery(GET_RESTAURANT_DISHES, {
        variables: { id: router.query.id },
      });
      if (error) return 'Error Loading Dishes';
      if (loading) return <h1>Loading ...</h1>;
      if (data.restaurant.data.attributes.dishes.data.length) {
        const { restaurant } = data;
        return (
          <>
            <h1>{restaurant.data.attributes.name}</h1>
            <Row>
              {restaurant.data.attributes.dishes.data.map(res => {
                let ress = { res };
                return (
                  <Col
                    xs="6"
                    sm="4"
                    style={{ padding: 0 }}
                    key={Math.random()}
                  >
                    <Card style={{ margin: '0 10px' }}>
                      <CardImg
                        top={true}
                        style={{ height: 250 }}
                        src={`${
                          process.env.STRAPI_URL ||
                          'http://localhost:1337'
                        }${res.attributes.image.data.attributes.url}`}
                      />
                      <CardBody>
                        <CardTitle tag="h5">
                          {res.attributes.name}
                        </CardTitle>
                        <CardText>
                          {res.attributes.description}
                        </CardText>
                      </CardBody>
                      <div className="card-footer">
                        <Button
                          outline
                          color="primary"
                          onClick={() => {
                            appContext.addItem(ress);
                          }}
                        >
                          + Add To Cart
                        </Button>
                        <style jsx>
                          {`
                            a {
                              color: white;
                            }
                            a:link {
                              text-decoration: none;
                              color: white;
                            }
                            .container-fluid {
                              margin-bottom: 30px;
                            }
                            .btn-outline-primary {
                              color: #007bff !important;
                            }
                            a:hover {
                              color: white !important;
                            }
                          `}
                        </style>
                      </div>
                    </Card>
                  </Col>
                );
              })}
              <Col xs="3" style={{ padding: 0 }}>
                <div>
                  <Cart />
                </div>
              </Col>
            </Row>
          </>
        );
      }
      return <h1>Add Dishes</h1>;
    }
    export default Restaurants;

Now if you refresh the page you should see the Cart component to the right of the dishes.
Your Layout header should also update with the username of the logged in user and show a logout button if you are logged-in.
Note: In a real-world application, you should always verify a user/token on the server before processing any user-specific actions. While this may not be demonstrated for the sake of the tutorial, you should most certainly do this in a real-world application.
Good job, let's finish the last step for ordering the food!
💵 In the next section, you will learn how to set up **Stripe for checkout and create orders**: [https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6).

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
