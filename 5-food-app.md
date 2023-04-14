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

That exactly what we are going to do. We going to add the following card component and allow users add items to their cart.
![Cart](./images-project/cart.gif)

First things first, let's update our context provider to allow us to save our items in the cart.

## **React Context**

To keep track of the items added to the cart across pages you will use the React Context API.

This will prevent having to prop drill the items multiple levels deep. The context will allow you to manage the state of items that will be re-used on the checkout page.

The only thing React Context will not take care of for you is preserving items through a page refresh, for that, we will save out cart items to a cookie.

You can also save it to a DB and restore them, but that is not want we are going to do here today.

Update the code in your `AuthContext.js` file found in our `context` folder with the code below.

```jsx
import { useState, createContext, useContext, useEffect } from "react";
import Cookie from "js-cookie";
import { gql } from "@apollo/client";
import { client } from "@/pages/_app.js";

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [cart, setCart] = useState(
    JSON.parse(Cookie.get("cart") || "{}") || {
      items: [],
      total: 0,
    }
  );

  useEffect(() => {
    const fetchData = async () => {
      const userData = await getUser();
      setUser(userData);
    };
    fetchData();
  }, []);

  useEffect(() => {
    Cookie.set("cart", JSON.stringify(cart));
  }, [cart]);

  const addItem = (item) => {
    let newItem = cart.items.find((i) => i.id === item.id);
    if (!newItem) {
      const newItem = {
        quantity: 1,
        ...item,
      };
      setCart((prevCart) => ({
        items: [...prevCart.items, newItem],
        total: prevCart.total + item.attributes.price,
      }));
    } else {
      setCart((prevCart) => ({
        items: prevCart.items.map((i) =>
          i.id === newItem.id ? { ...i, quantity: i.quantity + 1 } : i
        ),
        total: prevCart.total + item.attributes.price,
      }));
    }
  };

  const removeItem = (item) => {
    let newItem = cart.items.find((i) => i.id === item.id);
    if (newItem.quantity > 1) {
      setCart((prevCart) => ({
        items: prevCart.items.map((i) =>
          i.id === newItem.id ? { ...i, quantity: i.quantity - 1 } : i
        ),
        total: prevCart.total - item.attributes.price,
      }));
    } else {
      setCart((prevCart) => ({
        items: prevCart.items.filter((i) => i.id !== item.id),
        total: prevCart.total - item.attributes.price,
      }));
    }
  };

  return (
    <AuthContext.Provider value={{ user, setUser, cart, addItem, removeItem }}>
      {children}
    </AuthContext.Provider>
  );
};

const getUser = async () => {
  const token = Cookie.get("token");
  if (!token) return null;
  const { data } = await client.query({
    query: gql`
      query {
        me {
          id
          email
          username
        }
      }
    `,
    context: {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    },
  });
  return data.me;
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined)
    throw new Error("useAuth must be used within an AuthProvider");
  return context;
};
```

This will allow us to save and remove our cart to state via `addItem` and `removeItem` function.

Now let's create our **cart** component.

## **Cart Component**

Create a new file in the `components` folder named `Cart.jsx` and add the following code.

_Path:_ `frontend/components/Cart.jsx`

```jsx
import { useAuth } from "@/context/AuthContext";
import Link from "next/link";

function CartItem({ data }) {
  const { addItem, removeItem } = useAuth();
  const { quantity, attributes } = data;
  return (
    <div className="p-6 flex flex-wrap justify-between border-b border-blueGray-800">
      <div className="w-2/4">
        <div className="flex flex-col h-full">
          <h6 className="font-bold text-white mb-1">{attributes.name}</h6>
          <span className="block pb-4 mb-auto font-medium text-gray-400">
            {quantity} x ${attributes.price.toFixed(2)}
          </span>
        </div>
      </div>
      <div className="w-1/4">
        <div className="flex flex-col items-end h-full">
          <div className="flex justify-between">
            <button
              className="mr-2 inline-block mb-auto font-medium text-sm text-gray-400 hover:text-gray-200"
              onClick={() => removeItem(data)}
            >
              Remove
            </button>
            <button
              className="inline-block mb-auto font-medium text-sm text-gray-400 hover:text-gray-200"
              onClick={() => addItem(data)}
            >
              Add
            </button>
          </div>
          <span className="block mt-2 text-sm font-bold text-white">
            ${(attributes.price * quantity).toFixed(2)}
          </span>
        </div>
      </div>
    </div>
  );
}

export default function Cart() {
  const { user, cart } = useAuth();
  const total = cart.total.toFixed(2);
  const displayTotal = Math.abs(total);

  return (
    <section className="relative ">
      <div className="rounded-2xl bg-gray-800">
        <div className="max-w-lg pt-6 pb-8 px-8 mx-auto bg-blueGray-900">
          <div className="flex mb-10 items-center justify-between">
            <h6 className="font-bold text-2xl text-white mb-0">Your Cart</h6>
          </div>

          <div>
            {cart.items
              ? cart.items.map((item, index) => {
                  if (item.quantity > 0) {
                    return <CartItem key={index} data={item} />;
                  }
                })
              : null}
          </div>
          <div className="p-6">
            <div className="flex mb-6 content-center justify-between">
              <span className="font-bold text-white">Order total</span>
              <span className="text-sm font-bold text-white">
                ${displayTotal}
              </span>
            </div>
            <Link
              href={user ? "/pay" : "/login"}
              className="inline-block w-full px-6 py-3 text-center font-bold text-white bg-green-500 hover:bg-green-600 transition duration-200 rounded-full"
            >
              {user ? "Continue To Pay" : "Login to Order"}
            </Link>
          </div>
        </div>
      </div>
    </section>
  );
}
```

## **Putting Things Together**

Now let's connect all these things together.

Update the `/pages/restaurant/[id].js` file to use our newly created **Cart** component.

Path: `/frontend/pages/restaurant/[id].js`

```jsx
import { gql, useQuery } from "@apollo/client";
import { useRouter } from "next/router";
import { useAuth } from "@/context/AuthContext";

import Image from "next/image";
import Cart from "@/components/Cart";

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

function DishCard({ data }) {
  const { addItem } = useAuth();

  return (
    <div className="w-3/4 container mx-auto">
      <div className="grid grid-cols-2 gap-4">
        <div className="h-full bg-gray-100 rounded-2xl">
          <Image
            className="w-full rounded-2xl"
            height={300}
            width={300}
            src={`${process.env.STRAPI_URL || "http://localhost:1337"}${
              data.attributes.image.data.attributes.url
            }`}
            alt=""
          />
          <div className="p-8">
            <div className="group inline-block mb-4" href="#">
              <h3 className="font-heading text-xl text-gray-900 hover:text-gray-700 group-hover:underline font-black">
                {data.attributes.name}
              </h3>
              <h2>{data.attributes.price}</h2>
            </div>
            <p className="text-sm text-gray-500 font-bold">
              {data.attributes.description}
            </p>
            <div className="flex flex-wrap md:justify-center -m-2">
              <div className="w-full md:w-auto p-2 my-6">
                <button
                  className="block w-full px-12 py-3.5 text-lg text-center text-white font-bold bg-gray-900 hover:bg-gray-800 focus:ring-4 focus:ring-gray-600 rounded-full"
                  onClick={() => addItem(data)}
                >
                  + Add to Cart
                </button>
              </div>
            </div>
          </div>
        </div>
        <div>
          <Cart />
        </div>
      </div>
    </div>
  );
}

export default function Restaurant() {
  const router = useRouter();
  const { loading, error, data } = useQuery(GET_RESTAURANT_DISHES, {
    variables: { id: router.query.id },
  });

  if (error) return "Error Loading Dishes";
  if (loading) return <h1>Loading ...</h1>;
  if (data.restaurant.data.attributes.dishes.data.length) {
    const { restaurant } = data;

    return (
      <div>
        <h1 className="text-2xl text-green-600">
          {restaurant.data.attributes.name}
        </h1>
        <div className="py-16 px-8 bg-white rounded-3xl">
          <div className="max-w-7xl mx-auto">
            <div className="flex flex-wrap -m-4 mb-6">
              {restaurant.data.attributes.dishes.data.map((res) => {
                return <DishCard key={res.id} data={res} />;
              })}
            </div>
          </div>
        </div>
      </div>
    );
  } else {
    return <h1>No Dishes Found</h1>;
  }
}
```

Now if you refresh the page you should see the Cart component to the right of the dishes.

Your Layout header should also update with the username of the logged in user and show a logout button if you are logged-in.

Good job, let's finish the last step for ordering the food!

💵 In the next section, you will learn how to set up **Stripe for checkout and create orders**: [https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6](https://strapi.io/blog/nextjs-react-hooks-strapi-checkout-6).
