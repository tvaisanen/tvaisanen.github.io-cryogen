{:title "Auth0 SPA login flow with Shadow CLJS"
 :layout :post
 :tags  ["Clojure" "ClojureScript" "JS Interop" "Shadow-CLJS" "Auth0"]}


# Table of Contents


1.  [Import dependencies](#orgb83cd7b)
2.  [Create login button component](#org8b1e848)
3.  [Create Auth0 Client](#org46a5e39)
4.  [Login With Redirect](#orgd80d997)
5.  [Handle Redirect or Initialize token silently](#org68e7609)
6.  [Everything Put Together](#org51c9e9b)



<a id="orgebe299f"></a>

# Initial setup

Create a new Clojurescript application to start with.

    npx create-cljs-app cljs-auth0-spa

Install the dependencies.

-   reagent
-   auth0-spa-js

Auth0-spa-js from npm. Clojurescript packages are downloaded automatically
on the startup if not yet downloaded, when they are listed in the dependencies
of `=shadow-cljs.edn`.

    npm i @auth0/auth0-spa-js

    {:dependencies [[reagent "0.8.1"]]}

The the development server is started by running `=npm start`.
This will run the shadow-cljs cli that takes care of installing the clojurescript
dependencies.


<a id="orgb83cd7b"></a>

## Import dependencies

Reagent as clojuresciprt deps packages.
NPM package `auth0/auth0-spa-js` can be imported by the package name.
Read more about the npm package imports from [Shadow CLJS documentation](https://shadow-cljs.github.io/docs/UsersGuide.html#_using_npm_packages).

    (ns app.home
     (:require [reagent.core :as r]
               [app.config :as c]
               [clojure.string :as str]
               ["@auth0/auth0-spa-js" :as auth0]))

<a id="org46a5e39"></a>

## Create Auth0 Client

Get the credentials from the Auth0 dashboard.

    (defonce auth0-client
      (auth0/Auth0Client.
       (clj->js {:client_id ""
                 :domain ""
                 :redirect_uri""
                 })))


<a id="orgd80d997"></a>

## Login With Redirect

    (defn login [] (.loginWithRedirect auth0-client))

    (defn login-button []
        [:button
          {:on-click login} "Login"])



<a id="org68e7609"></a>

## Handle Redirect or Initialize token silently

After successfull redirect the the id token call `auth0client.handleRedirectCallback`.

    (defn handle-auth-redirect []
      (->
       (.handleRedirectCallback auth0client)
       (.then load-profile)))

    (defn load-silently []
      (pr "load silently")
      (-> (.getTokenSilently auth0client)
          (.then load-profile)
          (.catch #(pr %))))


         (defn handle-redirect?
           "
           Handle auth0 redirect?

           Should handle redirect if state and code query params.
           "
           []
           (let [params (url-search-params)]
             (and
              (.has params "state")
              (.has params "code"))))

         (defn init-session []
           (if (should-handle-redirect?)
               (handle-redirect)
               (init-session)))

         (defn on-load-window [](init-session)
           (.addEventListener js/window "load" on-load-window)


<a id="org51c9e9b"></a>

## Everything Put Together


    (ns app.core
      "This namespace contains your application and is the entrypoint for 'yarn start'."
      (:require [reagent.core :as r]
                [app.home :as home]
                [app.config :as c]
                [clojure.string :as str]
                ["@auth0/auth0-spa-js" :as auth0]))

    (enable-console-print!)

    (defonce auth0client
      (auth0/Auth0Client.
      (clj->js c/auth0)))

    (defn url-search-params
      "Parse URLSearchParams from window location."
      []
      (-> js/window.location.href
          (str/split "?")
          (get 1)
          (js/URLSearchParams.)))

    (defn handle-redirect?
      "
      Handle auth0 redirect?

      Should handle redirect if state and code query params.
      "
      []
      (let [params (url-search-params)]
        (and
        (.has params "state")
        (.has params "code"))))

    (def raw_token (atom nil))

    (defn auth-action-to-take
      "Decide whether to handle a redirect or try to initialize session silently."
      []
      (pr "What auth action to take?")
      (if (handle-redirect?)
        :handle-redirect
        :load-silently))

    (def profile (r/atom {}))
    (def errors  (r/atom {}))

    (defn set-error [e] (reset! errors e))

    (defn id-token-claims-to-user [claims]
      (let [claims-map (js->clj claims :keywordize-keys true)]
        ;; save the claims to
        (reset! profile claims-map)))

    (defn load-profile []
      (->
        (.getIdTokenClaims auth0client)
        (.then id-token-claims-to-user)
        (.catch set-error)
        (.finally (pr "load-profile: finally"))))

    (defn handle-auth-redirect []
      (->
        (.handleRedirectCallback auth0client)
        (.then load-profile)))

    (defn load-silently []
      (pr "load silently")
      (-> (.getTokenSilently auth0client)
          (.then load-profile)
          (.catch #(pr %))))

    (defn on-window-load []
      (pr "on window load")
      (case (auth-action-to-take)
        :handle-redirect  (handle-auth-redirect)
        :load-silently  (load-silently)
        (pr "no action")))

    (.addEventListener js/window "load" on-window-load)

    (defn on-login [] (.loginWithRedirect auth0client))
    (defn on-logout [] (.logout auth0client))

    (defn login-button []
      [:button
      {:on-click on-login} "Login"])

    (defn logout-button []
      [:button
      {:on-click on-logout} "Logout"])

    (defn view []
      [:<>
      [:h1 "Auth0 SPA CLJS"]
      (when (not @profile) [login-button])
      (when @profile [logout-button])
      [:pre (.stringify js/JSON (clj->js c/auth0) nil 2)]
      [:pre (.stringify js/JSON (clj->js @errors) nil 2)]
      [:pre (.stringify js/JSON (clj->js @profile) nil 2)]])


    (defn ^:dev/after-load render
      "Render the toplevel component for this app."
      []
      (r/render [home/view] (.getElementById js/document "app")))


    (defn ^:export main
      "Run application startup logic."
      []
      (render))
