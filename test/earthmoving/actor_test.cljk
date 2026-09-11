(ns earthmoving.actor-test
  (:require [clojure.test :refer [deftest is testing]]
            [earthmoving.actor :as actor]
            [earthmoving.store :as store]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-client! st {:client-id "client-1" :name "Kobo Earthworks"})
    (store/register-site! st {:site-id "S-1" :client-id "client-1"
                              :name "lot-14-foundation"
                              :max-cleared-depth-m 2.0
                              :cleared-zones #{"zone-a"}})
    st))

(deftest commits-an-in-depth-in-zone-excavation
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:client-id "client-1" :op :approve-excavation :stake :low
                 :site-id "S-1" :depth-m 1.5 :zone "zone-a"}
        result (actor/run-request! graph request {} "thread-1")]
    (is (= :done (:status result)))
    (is (some? (get-in result [:state :record])))
    (is (= 1 (count (store/records-of st "client-1"))))))

(deftest holds-an-unlocated-zone-excavation
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:client-id "client-1" :op :approve-excavation :stake :low
                 :site-id "S-1" :depth-m 1.5 :zone "zone-unsurveyed"}
        result (actor/run-request! graph request {} "thread-2")]
    (is (= :hold (:disposition (:state result))))
    (is (empty? (store/records-of st "client-1")))))

(deftest interrupts-then-approves-occupied-zone-entry-on-human-approval
  (let [st (fresh-store)
        graph (actor/build-graph {:store st})
        request {:client-id "client-1" :op :approve-occupied-zone-entry :stake :low
                 :site-id "S-1"}
        interrupted (actor/run-request! graph request {} "thread-3")]
    (is (= :interrupted (:status interrupted)))
    (is (empty? (store/records-of st "client-1")))
    (let [resumed (actor/approve! graph "thread-3")]
      (is (= :done (:status resumed)))
      (is (= 1 (count (store/records-of st "client-1")))))))
