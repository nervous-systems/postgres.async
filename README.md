clj-postgres-async
==================

Asynchronous PostgreSQL Clojure library

## Usage

```clojure
(ns clj-postgres-async.core
  (:require [clj-postgres-async.core :refer :all]
            [clojure.core.async :refer [go <!!]]))


(def db (open-db {:hostname "localhost"
                  :port 5432
                  :database "postgres"
                  :username "postgres"
                  :password "postgres"
                  :pool-size 20}))

(<!! (<insert! db {:table "products"} {:name "screwdriver" :price 15}))
; [{:updated 1, :rows ()} nil]

(<!! (<insert! db {:table "products" :returning "id"} {:name "hammer" :price 5}))
; [{:updated 1, :rows ({:id 1001})} nil]

(<!! (<query! db ["select name, price from products"]))
; [({:id 1000, :name "screwdriver", :price 15} {:id 1001, :name "hammer", :price 10) nil]

(<!! (<query! db ["select * from foobar"]))
; [nil #<SqlException com.github.pgasync.SqlException: ERROR: SQLSTATE=42P01, MESSAGE=relation "foobar" does not exist>

(<!! (<update! db {:table "users" :where ["id=$1" 1001}} {:price 6}))
; [{:updated 1, :rows ()} nil]

(go (dosql [tx (<begin! db)
            id (<insert! tx {:table products :returning "id"} {:name "saw"})
            _  (<insert! tx {:table promotions} {:product_id (-> id (:rows) (first) (:id))})
            rs (<query!  tx ["select * from promotions order by date limit 5"])
            _  (<commit! tx)]
         {:now-promoting rs}))

```

