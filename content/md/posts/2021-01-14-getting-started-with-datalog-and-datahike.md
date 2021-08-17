{:title "Getting Started With Datalog and Datahike"
 :layout :post
 :tags  ["Clojure" "Datalog" "Datahike"]}


# Table of Contents

1.  [Add dependencies and create a database](#org10812a7)
2.  [Add Example Data](#orgda6af90)
3.  [Basic Queries](#org6e549e6)
    1.  [Basic query with explicit properties.](#org26f23ea)
    2.  [Pull all properties](#org4e1a75e)
    3.  [Add a predicate function](#orgd96072d)
    4.  [Add a second predicate](#orga8338e5)
4.  [Transactions](#org255f136)
5.  [History](#orgcd9ca35)



<a id="org926352f"></a>

-   What is datalog <https://en.wikipedia.org/wiki/Datalog>.
-   Learn datalog  <https://learndatalogtoday.org>.
-   Database implementations:
    -   <https://www.datomic.com/>
    -   <https://github.com/tonsky/datascript>
    -   <https://github.com/replikativ/datahike>


<a id="org10812a7"></a>

## Add dependencies and create a database

    :dependencies [[org.clojure/clojure "1.10.1"]
                   [io.replikativ/datahike "0.3.2"]]

Import Datahike API as d and from export-db for DB introspection and create basic configuration for an in-memory database.

    (ns datahike-test.core
      (:require [datahike.api :as d]
                [datahike.migrate :refer [export-db]])
      (:gen-class))

    (def cfg {:schema-flexibility :read ;; allow write without explicit schema
              :store {:backend :mem}})  ;; use in-memory database

    (d/create-database cfg)

    (def conn (d/connect cfg))


<a id="orgda6af90"></a>

## Add Example Data


    (d/transact conn {:tx-data [{:db/id 4
                                 :job "Clojure Fullstack"
                                 :position :fullstack-developer
                                 :languages #{:javascript :clojure}}
                                {:db/id -1
                                 :job "Clojure Backend Developer"
                                 :position :backend-developer
                                 :languages #{:clojure}}
                                {:db/id -1
                                 :job "Javascript Fullstack"
                                 :position :fullstack-developer
                                 :languages #{:javascript}}]})

Create DB dump for Datoms to a file for introspection

    (export-db @conn "./eavt-dump")

    #datahike/Datom [4 :job "Clojure Fullstack" 536870914 true]
    #datahike/Datom [4 :languages #{:javascript :clojure} 536870914 true]
    #datahike/Datom [4 :position :fullstack-developer 536870914 true]

    #datahike/Datom [5 :job "Clojure Backend Developer" 536870915 true]
    #datahike/Datom [5 :languages #{:clojure} 536870915 true]
    #datahike/Datom [5 :position :backend-developer 536870915 true]

    #datahike/Datom [6 :job "Javascript Fullstack" 536870916 true]
    #datahike/Datom [6 :languages #{:javascript} 536870916 true]
    #datahike/Datom [6 :position :fullstack-developer 536870916 true]


<a id="org6e549e6"></a>

## Basic Queries


<a id="org26f23ea"></a>

### Basic query with explicit properties.

    (d/q '[:find ?e ?title ?langs
           :where
           [?e :position ?title]
           [?e :languages ?langs]]
         @conn)

    #{[5 :backend-developer #{:clojure}]
      [6 :fullstack-developer #{:javascript}]
      [4 :fullstack-developer #{:javascript :clojure}]}


<a id="org4e1a75e"></a>

### Pull all properties

Get all the jobs for clojure fullstack. Use the `(pull? ?e [*])` to
pull all fields of entity `?e` and use the ellipsis `...` to pull a set of
entities instead just of the first one.

    (d/q '[:find [(pull ?e [*]) ...]
           :where
           [?e :position ?title]
           [?e :languages ?langs]]
         @conn)

    [{:db/id 4,
      :job "Clojure Fullstack",
      :languages #{:javascript :clojure},
      :position :fullstack-developer}
     {:db/id 6,
      :job "Javascript Fullstack",
      :languages #{:javascript},
      :position :fullstack-developer}
     {:db/id 5,
      :job "Clojure Backend Developer",
      :languages #{:clojure},
      :position :backend-developer}]


<a id="orgd96072d"></a>

### Add a predicate function

Filter clojure jobs.

    (d/q '[:find [(pull ?e [*]) ...]
           :in $ [?l]
           :where
           [?e :position ?title]
           [?e :languages ?langs]
           [(contains? ?langs ?l)]]
         @conn
         [:clojure])

    [{:db/id 4,
      :job "Clojure Fullstack",
      :languages #{:javascript :clojure},
      :position :fullstack-developer}
     {:db/id 5,
      :job "Clojure Backend Developer",
      :languages #{:clojure},
      :position :backend-developer}]


<a id="orga8338e5"></a>

### Add a second predicate

Filter only the clojure backend jobs.


    (d/q '[:find [(pull ?e [*]) ...]
           :in $ [?l ?p]
           :where
           [?e :position ?title]
           [?e :languages ?langs]
           [?e :position ?pos]
           [(contains? ?langs ?l)]
           [(= ?pos ?p)]]
         @conn
         [:clojure :backend-developer])

    [{:db/id 5,
      :job "Clojure Backend Developer",
      :languages #{:clojure},
      :position :backend-developer}]


<a id="org255f136"></a>

## TODO Transactions

-   transaction introspection


<a id="orgcd9ca35"></a>

## TODO History

-   Query history
