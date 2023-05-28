In this blog post, we'll explore the seamless integration of JWT authentication with React and react-router. We'll also learn how to handle public routes, secure authenticated routes, and utilize the axios library to make API requests with the authentication token.

## Create a React Project
The following command will create a react project for us.

```
npm create vite@latest react-auth-demo
```

We will choose `React` as our framework and `JavaScript` as a variant.
To begin working on our project, we need to ensure that all dependencies are properly installed. We can achieve this by running

```
npm install
```

within our project directory. Once the installation process is complete, we can launch our React project using the command

```
npm run dev
```

Let's take these necessary steps to get our React project up and running smoothly.

## Installing Dependencies for React-Router v6 and Axios

Before we proceed, let's ensure we have the necessary dependencies installed for our project. We'll start by installing react-router v6, which will handle routing within our React application. Additionally, we'll install Axios, a powerful library used for making API requests. By following these steps, we'll be equipped with the tools needed to implement seamless routing and perform efficient API communication. Let's begin by installing these dependencies.

```
npm install react-router-dom axios
```

**Creating the AuthProvider and AuthContext in React**
With our project set up and dependencies installed, we're ready to take the next step in implementing JWT authentication. In this section, we'll create an `AuthProvider` component and an associated `AuthContext`. These will enable us to store and share authentication-related data and functions throughout our application

In the following code snippet, we'll create the `authProvider.js` file located at `src > provider > authProvider.js`. Let's explore the implementation of the `AuthProvider` and `AuthContext`.

1.  Import the necessary modules and packages:
    - axios is imported from the "axios" package to handle API requests.
    - createContext, useContext, useEffect, useMemo, and useState are imported from the "react" library.
    ```js
    import axios from "axios";
    import {
      createContext,
      useContext,
      useEffect,
      useMemo,
      useState,
    } from "react";
    ```

2.  Create an authentication context using createContext():
    - createContext() creates an empty context object that will be used to share the authentication state and functions between components.
    ```js
    const AuthContext = createContext();
    ```

3.  Create the AuthProvider component:
    - This component serves as the provider for the authentication context.
    - It receives children as a prop, which represents the child components that will have access to the authentication context.
    ```js
    const AuthProvider = ({ children }) => {
      // Component content goes here
    };
    ```

4.  Define the `token` state using `useState()`:
    - `token` represents the authentication token.
    - `localStorage.getItem("token")` retrieves the token value from the local storage if it exists.
    ```js
    const [token, setToken_] = useState(localStorage.getItem("token"));
    ```

5.  Create the `setToken` function to update the authentication token:
    - This function is used to set the new token value.
    - It updates the `token` state using `setToken_()` and stores the token value in the local storage using `localStorage.setItem()`.
    ```js
    const setToken = (newToken) => {
      setToken_(newToken);
    };
    ```

6.  Use `useEffect()` to set the default authorization header in axios and stores the token value in the local storage using `localStorage.setItem()`:
    - This effect runs whenever the `token` value changes.
    - If the `token` exists, it sets the authorization header in axios and localStorage.
    - If the `token` is null or undefined, it removes the authorization header from axios and localStorage.
    ```js
    useEffect(() => {
      if (token) {
        axios.defaults.headers.common["Authorization"] = "Bearer " + token;
        localStorage.setItem('token',token);
      } else {
        delete axios.defaults.headers.common["Authorization"];
        localStorage.removeItem('token')
      }
    }, [token]);
    ```

7.  Create the memoized context value using useMemo():
    - The context value includes the token and setToken function.
    - The token value is used as a dependency for memoization.
    ```js
    const contextValue = useMemo(
      () => ({
        token,
        setToken,
      }),
      [token]
    );
    ```

8.  Provide the authentication context to the child components:
    - Wrap the children components with the AuthContext.Provider.
    - Pass the contextValue as the value prop of the provider.
    ```js
    return (
      <AuthContext.Provider value={contextValue}>
        {children}
      </AuthContext.Provider>
    );
    ```

9.  Export the useAuth hook for accessing the authentication context:
    - useAuth is a custom hook that can be used in components to access the authentication context.
    ```js
    export const useAuth = () => {
      return useContext(AuthContext);
    };
    ```

10. Export the AuthProvider component as the default export:
    - This allows other files to import and use the AuthProvider component as needed.
    ```js
    export default AuthProvider;
    ```

**Complete Code:**

```js
import axios from "axios";
import { createContext, useContext, useEffect, useMemo, useState } from "react";

const AuthContext = createContext();

const AuthProvider = ({ children }) => {
  // State to hold the authentication token
  const [token, setToken_] = useState(localStorage.getItem("token"));

  // Function to set the authentication token
  const setToken = (newToken) => {
    setToken_(newToken);
  };

  useEffect(() => {
    if (token) {
      axios.defaults.headers.common["Authorization"] = "Bearer " + token;
      localStorage.setItem('token',token);
    } else {
      delete axios.defaults.headers.common["Authorization"];
      localStorage.removeItem('token')
    }
  }, [token]);

  // Memoized value of the authentication context
  const contextValue = useMemo(
    () => ({
      token,
      setToken,
    }),
    [token]
  );

  // Provide the authentication context to the children components
  return (
    <AuthContext.Provider value={contextValue}>{children}</AuthContext.Provider>
  );
};

export const useAuth = () => {
  return useContext(AuthContext);
};

export default AuthProvider;
```

In summary, this code sets up the authentication context using React's context API. It provides the authentication token and the setToken function to child components through the context. It also ensures that the default authorization header in axios is updated with the authentication token whenever it changes.

## Creating Routes in React for JWT Authentication

To organize our routes effectively, we'll create a dedicated `src > routes` folder. Inside this folder, we'll create an `index.jsx` file, which will serve as the entry point for defining our application's routes. By structuring our routes in a separate folder, we can maintain a clear and manageable routing structure. Let's proceed with creating our routes and explore how we can incorporate JWT authentication into our React application.

**Creating a ProtectedRoute Component for Authenticated Routes**

In order to secure our authenticated routes and prevent unauthorized access, we'll create a dedicated component called `ProtectedRoute`. This component will serve as a wrapper for our authenticated routes, ensuring that only authenticated users can access them. By implementing this component, we can easily enforce authentication requirements and provide a seamless user experience. Let's create the `ProtectedRoute.jsx` file located at `src > routes > ProtectedRoute.jsx` and enhance the security of our application.

1.  We start by importing the necessary dependencies from the react-router-dom library:
    ```js
    import { Navigate, Outlet } from "react-router-dom";
    import { useAuth } from "../provider/authProvider";
    ```

2.  We define the ProtectedRoute component, which will serve as a wrapper for our authenticated routes:
    ```js
    export const ProtectedRoute = () => {
      const { token } = useAuth();

      // Check if the user is authenticated
      if (!token) {
        // If not authenticated, redirect to the login page
        return <Navigate to="/login" />;
      }

      // If authenticated, render the child routes
      return <Outlet />;
    };
    ```

3.  Inside the ProtectedRoute component, we access the token from the useAuth custom hook provided by the AuthContext. This hook allows us to retrieve the authentication token stored in the context.

4.  We then check if the token exists. If the user is not authenticated (token is falsy or null), we use the Navigate component from react-router-dom to redirect the user to the login page ("/login").

5.  If the user is authenticated, we render the child routes using the Outlet component. The Outlet component acts as a placeholder that displays the child components defined in the parent route.

In summary, the ProtectedRoute component serves as a guard for authenticated routes. If the user is not authenticated, they will be redirected to the login page. If the user is authenticated, the child routes defined within the ProtectedRoute component will be rendered using the Outlet component.

This code allows us to easily protect specific routes and control access based on the user's authentication status, providing a secure navigation experience in our React application.

**Deep dive into Routes**
Now that we have our `ProtectedRoute` component and authentication context in place, we can proceed with defining our routes. By differentiating between public routes, authenticated routes, and routes for non-authenticated users, we can effectively handle navigation and access control based on JWT authentication. In this code snippet, we'll dive into the `src > routes > index.jsx` file and explore how we can incorporate JWT authentication into our routing structure. Let's get started!

1.  Import necessary dependencies:
    - RouterProvider and createBrowserRouter are components imported from the react-router-dom library. They are used for configuring and providing the routing functionality.
    - useAuth is a custom hook imported from "../provider/authProvider". It allows us to access the authentication context.
    - ProtectedRoute is a component imported from "./ProtectedRoute". It serves as a wrapper for authenticated routes.
    ```js
    import { RouterProvider, createBrowserRouter } from "react-router-dom";
    import { useAuth } from "../provider/authProvider";
    import { ProtectedRoute } from "./ProtectedRoute";
    ```

2.  Define the Routes component:
    - This functional component acts as the entry point for configuring the application routes.
    ```js
    const Routes = () => {
      const { token } = useAuth();
      // Route configurations go here
    };
    ```

3.  Access the authentication token using the useAuth hook:
    - The useAuth hook is called to retrieve the token value from the authentication context. It allows us to access the authentication token within the Routes component.
    ```js
    const { token } = useAuth();
    ```

4.  Define routes accessible to all users:
    - The routesForPublic array contains route objects that can be accessed by all users. Each route object consists of a path and an element.
    - The path property specifies the URL path for the route, and the element property contains the JSX element/component to render.
    ```js
    const routesForPublic = [
      {
        path: "/service",
        element: <div>Service Page</div>,
      },
      {
        path: "/about-us",
        element: <div>About Us</div>,
      },
    ];
    ```

5.  Define routes accessible only to authenticated users:
    - The routesForAuthenticatedOnly array contains route objects that can be accessed only by authenticated users. It includes a protected root route ("/") wrapped in the ProtectedRoute component and additional child routes defined using the children property.
    ```js
    const routesForAuthenticatedOnly = [
      {
        path: "/",
        element: <ProtectedRoute />,
        children: [
          {
            path: "/",
            element: <div>User Home Page</div>,
          },
          {
            path: "/profile",
            element: <div>User Profile</div>,
          },
          {
            path: "/logout",
            element: <div>Logout</div>,
          },
        ],
      },
    ];
    ```

6.  Define routes accessible only to non-authenticated users:
    - The routesForNotAuthenticatedOnly array contains route objects that are accessible only to non-authenticated users. It includes a login route ("/login").
    ```js
    const routesForNotAuthenticatedOnly = [
      {
        path: "/",
        element: <div>Home Page</div>,
      },
      {
        path: "/login",
        element: <div>Login</div>,
      },
    ];
    ```

7.  Combine and conditionally include routes based on authentication status:
    - The createBrowserRouter function is used to create the router configuration. It takes an array of routes as its argument.
    - The spread operator (...) is used to merge the route arrays into a single array.
    - The conditional expression (!token ? routesForNotAuthenticatedOnly : []) checks if the user is authenticated (token exists). If not, it includes the routesForNotAuthenticatedOnly array; otherwise, it includes an empty array.
    ```js
    const router = createBrowserRouter([
      ...routesForPublic,
      ...(!token ? routesForNotAuthenticatedOnly : []),
      ...routesForAuthenticatedOnly,
    ]);
    ```

8.  Provide the router configuration using RouterProvider:
    - The RouterProvider component wraps the router configuration, making it available for the entire application.
    ```js
    return <RouterProvider router={router} />;
    ```

**Complete Code:**
```js
import { RouterProvider, createBrowserRouter } from "react-router-dom";
import { useAuth } from "../provider/authProvider";
import { ProtectedRoute } from "./ProtectedRoute";

const Routes = () => {
  const { token } = useAuth();

  // Define public routes accessible to all users
  const routesForPublic = [
    {
      path: "/service",
      element: <div>Service Page</div>,
    },
    {
      path: "/about-us",
      element: <div>About Us</div>,
    },
  ];

  // Define routes accessible only to authenticated users
  const routesForAuthenticatedOnly = [
    {
      path: "/",
      element: <ProtectedRoute />, // Wrap the component in ProtectedRoute
      children: [
        {
          path: "/",
          element: <div>User Home Page</div>,
        },
        {
          path: "/profile",
          element: <div>User Profile</div>,
        },
        {
          path: "/logout",
          element: <div>Logout</div>,
        },
      ],
    },
  ];

  // Define routes accessible only to non-authenticated users
  const routesForNotAuthenticatedOnly = [
    {
      path: "/",
      element: <div>Home Page</div>,
    },
    {
      path: "/login",
      element: <div>Login</div>,
    },
  ];

  // Combine and conditionally include routes based on authentication status
  const router = createBrowserRouter([
    ...routesForPublic,
    ...(!token ? routesForNotAuthenticatedOnly : []),
    ...routesForAuthenticatedOnly,
  ]);

  // Provide the router configuration using RouterProvider
  return <RouterProvider router={router} />;
};

export default Routes;
```

## Final Integration

Now that we have our `AuthContext`, `AuthProvider` and `Routes` ready, let's integrate them together in `App.jsx`.

1.  Import the necessary components and files:
    - `AuthProvider` is a component imported from `"./provider/authProvider"`. It provides the authentication context to the application.
    - `Routes` is a component imported from `"./routes"`. It defines the application routes.
    ```js
    import AuthProvider from "./provider/authProvider";
    import Routes from "./routes";
    ```

2.  Wrap the `Routes` component with the `AuthProvider` component: 
    - The `AuthProvider` component is used to provide the authentication context to the application. It wraps around the `Routes` component to make the authentication context available to all components within the `Routes` component tree.
    ```js
    return (
      <AuthProvider>
        <Routes />
      </AuthProvider>
    );
    ```

**Complete Code:**
```js
import AuthProvider from "./provider/authProvider";
import Routes from "./routes";

function App() {
  return (
    <AuthProvider>
      <Routes />
    </AuthProvider>
  );
}

export default App;
```

## Now that everythings is in place, its time to implement `Login` and `Logout`.

Let's create a file for Login Page `src > pages > Login.jsx`
**Login Page**
```js
const Login = () => {
  const { setToken } = useAuth();
  const navigate = useNavigate();

  const handleLogin = () => {
    setToken("this is a test token");
    navigate("/", { replace: true });
  };

  setTimeout(() => {
    handleLogin();
  }, 3 * 1000);

  return <>Login Page</>;
};

export default Login;
```

- The Login component is a functional component that represents the login page.
- It imports the setToken function from the authentication context using the useAuth hook.
- The navigate function from the react-router-dom library is imported to handle navigation.
- Inside the component, there is a handleLogin function that sets a test token using the setToken function from the context  and navigates to the home page ("/") with the replace option set to true.
- A setTimeout function is used to simulate a delay of 3 seconds before calling the handleLogin function.
- The component returns the JSX for the login page, which in this case is a placeholder text.

Now we will create a file for Logout Page `src > pages > Logout.jsx`
**Logout Page**
```js
import { useNavigate } from "react-router-dom";
import { useAuth } from "../provider/authProvider";

const Logout = () => {
  const { setToken } = useAuth();
  const navigate = useNavigate();

  const handleLogout = () => {
    setToken();
    navigate("/", { replace: true });
  };

  setTimeout(() => {
    handleLogout();
  }, 3 * 1000);

  return <>Logout Page</>;
};

export default Logout;
```

- In Logout component, we call `setToken` function with no arguments, with is equal to `setToken(null)`.

Now, we will replace the existing Login and Logout components in the Routes component with the updated versions.

```js
const routesForNotAuthenticatedOnly = [
  {
    path: "/",
    element: <div>Home Page</div>,
  },
  {
    path: "/login",
    element: <Login />,
  },
];
```

In the `routesForNotAuthenticatedOnly` array, the `element` property of `"/login"` is set to `<Login />`, which means that the `Login` component will be rendered when the user visits the "/login" path.

```js
const routesForAuthenticatedOnly = [
  {
    path: "/",
    element: <ProtectedRoute />, // Wrap the component in ProtectedRoute
    children: [
      {
        path: "/",
        element: <div>User Home Page</div>,
      },
      {
        path: "/profile",
        element: <div>User Profile</div>,
      },
      {
        path: "/logout",
        element: <Logout />,
      },
    ],
  },
];
```

In the `routesForAuthenticatedOnly` array, the `element` property of `"/logout"` is set to `<Logout />`, which means that the Logout component will be rendered when the user visits the "/logout" path.

**Test Flow**

1.  When you first visit the root page `/`, you will see the "Home Page" from the `routesForNotAuthenticatedOnly` array.
2.  If you navigate to `/login`, after a delay of 3 seconds, the login process will be simulated. It will set a test token using the `setToken` function from the authentication context, and then you will be redirected to the root page `/` using the `navigate` function from the `react-router-dom` library. After the redirect, you will see the "User Home Page" from the `routesForAuthenticatedOnly` array.
3.  If you then visit `/logout`, after a delay of 3 seconds, the logout process will be simulated. It will clear the authentication token by calling the `setToken` function without any argument, and then you will be redirected to the root page `/` again. Since you are now logged out, we will see the "Home Page" from the `routesForNotAuthenticatedOnly` array.


This flow demonstrates the login and logout processes, where the user transitions between authenticated and non-authenticated states and the corresponding routes are displayed accordingly.