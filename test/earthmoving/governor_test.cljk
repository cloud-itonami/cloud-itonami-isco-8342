(ns earthmoving.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [earthmoving.store :as store]
            [earthmoving.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-client! st {:client-id "client-1" :name "Kobo Earthworks"})
    (store/register-site! st {:site-id "S-1" :client-id "client-1"
                              :name "lot-14-foundation"
                              :max-cleared-depth-m 2.0
                              :cleared-zones #{"zone-a" "zone-b"}})
    st))

(defn- excavate [depth zone]
  {:op :approve-excavation :effect :propose :site-id "S-1"
   :depth-m depth :zone zone :confidence 0.9 :stake :low})

(def ^:private req {:client-id "client-1"})

(deftest ok-within-cleared-depth-and-zone
  (let [st (fresh-store)
        v (governor/check req {} (excavate 1.5 "zone-a") st)]
    (is (:ok? v))))

(deftest ok-at-exact-depth-ceiling
  (testing "depth exactly at the cleared ceiling is within margin"
    (let [st (fresh-store)
          v (governor/check req {} (excavate 2.0 "zone-a") st)]
      (is (:ok? v)))))

(deftest hard-on-depth-exceeds-cleared-ceiling
  (testing "digging past the cleared depth is a strike risk, not a scheduling choice"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (excavate 4.0 "zone-a") :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :depth-exceeds-cleared-ceiling (:rule %)) (:violations v))))))

(deftest hard-on-unlocated-zone
  (testing "unlocated ground is not fair game"
    (let [st (fresh-store)
          v (governor/check req {} (assoc (excavate 1.5 "zone-unsurveyed") :confidence 0.99) st)]
      (is (:hard? v))
      (is (some #(= :unlocated-zone (:rule %)) (:violations v))))))

(deftest hard-on-unknown-site
  (let [st (fresh-store)
        v (governor/check req {} (assoc (excavate 1.5 "zone-a") :site-id "S-ghost") st)]
    (is (:hard? v))
    (is (some #(= :unknown-site (:rule %)) (:violations v)))))

(deftest hard-on-foreign-site
  (let [st (fresh-store)]
    (store/register-client! st {:client-id "client-2" :name "Other"})
    (let [v (governor/check {:client-id "client-2"} {} (excavate 1.5 "zone-a") st)]
      (is (:hard? v))
      (is (some #(= :site-wrong-client (:rule %)) (:violations v))))))

(deftest hard-on-unregistered-client
  (let [st (fresh-store)
        v (governor/check {:client-id "nobody"} {} (excavate 1.5 "zone-a") st)]
    (is (:hard? v))
    (is (some #(= :no-client (:rule %)) (:violations v)))))

(deftest hard-on-no-actuation-violation
  (let [st (fresh-store)
        v (governor/check req {} (assoc (excavate 1.5 "zone-a") :effect :direct-write) st)]
    (is (:hard? v))
    (is (some #(= :no-actuation (:rule %)) (:violations v)))))

(deftest always-escalates-unlocated-utility-excavation-even-at-high-confidence
  (testing "no excavation near unlocated utilities without the governor gate"
    (let [st (fresh-store)
          v (governor/check req {} {:op :approve-unlocated-utility-excavation :effect :propose
                                    :site-id "S-1" :confidence 0.99 :stake :low} st)]
      (is (not (:hard? v)))
      (is (:escalate? v)))))

(deftest always-escalates-occupied-zone-entry-even-at-high-confidence
  (testing "occupied-work-zone entry requires human sign-off"
    (let [st (fresh-store)
          v (governor/check req {} {:op :approve-occupied-zone-entry :effect :propose
                                    :site-id "S-1" :confidence 0.99 :stake :low} st)]
      (is (not (:hard? v)))
      (is (:escalate? v)))))

(deftest escalates-low-confidence
  (let [st (fresh-store)
        v (governor/check req {} (assoc (excavate 1.5 "zone-a") :confidence 0.3) st)]
    (is (not (:hard? v)))
    (is (:escalate? v))))
