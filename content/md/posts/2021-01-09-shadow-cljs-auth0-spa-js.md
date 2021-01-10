{:title "Auth0 SPA login flow with Shadow CLJS"
 :layout :post
 :tags  ["Clojure" "ClojureScript" "JS Interop" "Shadow-CLJS" "Auth0"]}


- [Initial setup](#sec-1)
- [First Iteration](#sec-2)
  - [Import dependencies](#sec-2-1)
  - [Create login button component](#sec-2-2)
  - [Create Auth0 Client](#sec-2-3)
  - [First Login](#sec-2-4)
  - [Handle Redirect](#sec-2-5)
  - [Handle Session Initializaion](#sec-2-6)
  - [Logout](#sec-2-7)
- [Second Iteration](#sec-3)
  - [Install Re-Frame and setup refrisk](#sec-3-1)
  - [Migrate from reagent atoms to re-frame.](#sec-3-2)


# Initial setup<a id="sec-1"></a>

Create a new Clojurescript application to start with.

```bash
npx create-cljs-app cljs-auth0-spa
```

Install the following dependencies.

-   reagent
-   auth0-spa-js

Auth0-spa-js from npm. Clojurescript packages are downloaded automatically on the startup if not yet downloaded, when they are listed in the dependencies of \`shadow-cljs.edn\`.

```bash
npm i @auth0/auth0-spa-js
```

```clojure
{:dependencies [[reagent "0.8.1"]]}
```

Now you are ready to start the development server by running \`npm start\`. This will run the shadow-cljs cli that takes care of installing the clojurescript dependencies.

# First Iteration<a id="sec-2"></a>

Build the login flow here with reagent atoms.

## TODO Import dependencies<a id="sec-2-1"></a>

Reagent and Re-Frame from their respective Clojure packages. NPM package "auth0/auth0-spa-js" can be imported by the package name. Read more about the npm package imports from [Shadow CLJS documentation](https://shadow-cljs.github.io/docs/UsersGuide.html#_using_npm_packages).

```clojure
(ns app.hello
  (:require [reagent.core :as r]
            ["@auth0/auth0-spa-js" :as auth0]))
```

## TODO Create login button component<a id="sec-2-2"></a>

-   Create login button
    -   use alert login as a placeholder action
-   Create home component
-   Mount login button

```clojure
(defn login-button []
   [:button
    {:on-click #(.alert js/window "LOGIN")} "Login"])

(defn home []
  [:<>
   [:h1 "Auth0 SPA CLJS"]
   [login-button]])

```

```clojure
(ns app.core
  "This namespace contains your application and is the entrypoint for 'yarn start'."
  (:require [reagent.core :as r]
            [app.home :as home]))

(defn ^:dev/after-load render
  "Render the toplevel component for this app."
  []
  (r/render [home/view] (.getElementById js/document "app")))

(defn ^:export main
  "Run application startup logic."
  []
  (render))

```

## TODO Create Auth0 Client<a id="sec-2-3"></a>

```clojurescript

(defonce auth0-client
  (auth0/Auth0Client.
   (clj->js {:client_id ""
             :domain ""
             :redirect_uri""
             })))

```

## TODO First Login<a id="sec-2-4"></a>

```clojure
(defn login [] (.loginWithRedirect auth0-client))
```

## TODO Handle Redirect<a id="sec-2-5"></a>

```clojure

(defn should-handle-redirect?
    "Check URL search parameters for code and state.
     Auth0 returns these on redirect after successfull login."
        [],,,)

(defn init-session []
  (if (should-handle-redirect?)
      (handle-redirect)
      (init-session)))

(defn on-load-window [](init-session)

 (.addEventListener js/window "load" on-load-window)
```

## TODO Handle Session Initializaion<a id="sec-2-6"></a>

## TODO Logout<a id="sec-2-7"></a>

# Second Iteration<a id="sec-3"></a>

Clean up the implementation. Install reframe and devtool refrisk.

## TODO Install Re-Frame and setup refrisk<a id="sec-3-1"></a>

```clojure
{:dependencies [[reagent "0.8.1"]
                ;; add re-frame
                [re-frame "1.1.2"]]}
```

## TODO Migrate from reagent atoms to re-frame.<a id="sec-3-2"></a>
