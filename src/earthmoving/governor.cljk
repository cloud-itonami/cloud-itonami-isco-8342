(ns earthmoving.governor
  "EarthmovingGovernor — the independent safety/traceability layer
  named in this repository's README/business-model.md, gating the
  robot-dispensed physical work (site surveying, grade-verification)
  an advisor may propose. The governor never dispatches hardware
  itself. Modeled on cloud-itonami-isco-4311's bookkeeping.governor.
  Excavation twist: a proposed excavation's depth is arithmetic
  comparison against the registered utility-cleared ceiling — digging
  past it is a strike risk, not a scheduling choice — and the
  proposed zone is either a member of the registered cleared-zones
  set or it is not — unlocated ground is not fair game.

  HARD invariants (:hard? true, ALWAYS :hold, never overridable):
    1. client provenance — the organization must be registered.
    2. no-actuation      — proposal :effect must be :propose (the
                           governor never dispatches hardware; it only
                           gates what the robot may execute).
    3. site basis           — an excavation approval must cite a
                           REGISTERED site belonging to this client.
    4. cleared-depth ceiling — the proposed excavation depth must not
                           exceed the site's registered
                           :max-cleared-depth-m (digging past it is a
                           strike risk, not a scheduling choice).
    5. cleared-zone membership — the proposed excavation zone must be
                           a member of the site's registered
                           :cleared-zones set (unlocated ground is
                           not fair game).
  ESCALATION invariants (:escalate? true, ALWAYS human sign-off per
  business-model.md's Trust Controls — these are :high/
  :safety-critical regardless of confidence):
    6. :op :approve-unlocated-utility-excavation (no excavation near
                           unlocated utilities without the governor
                           gate).
    7. :op :approve-occupied-zone-entry (occupied-work-zone entry
                           requires human sign-off).
    8. low confidence (< `confidence-floor`)."
  (:require [earthmoving.store :as store]))

(def confidence-floor 0.6)

(def ^:private always-escalate-ops #{:approve-unlocated-utility-excavation
                                     :approve-occupied-zone-entry})

(defn- hard-violations [{:keys [request proposal]} client-record st]
  (let [{:keys [op depth-m zone]} proposal
        excavate? (= :approve-excavation op)]
    (cond-> []
      (nil? client-record)
      (conj {:rule :no-client :detail "未登録 client"})

      (not= :propose (:effect proposal))
      (conj {:rule :no-actuation :detail "effect は :propose のみ許可（governor はハードウェアを直接起動しない）"})

      (and excavate? (nil? st))
      (conj {:rule :unknown-site :detail "未登録 site への掘削承認は不可"})

      (and excavate? st (not= (:client-id st) (:client-id request)))
      (conj {:rule :site-wrong-client :detail "site が別 client のもの"})

      (and excavate? st (number? depth-m) (> depth-m (:max-cleared-depth-m st)))
      (conj {:rule :depth-exceeds-cleared-ceiling
             :detail (str "掘削深度 " depth-m "m > 登録済み地下埋設物クリア上限 "
                          (:max-cleared-depth-m st) "m（クリア深度超過はストライクリスクであってスケジュールの都合ではない）")})

      (and excavate? st zone (not (contains? (:cleared-zones st) zone)))
      (conj {:rule :unlocated-zone
             :detail (str "区画 " zone " は登録済み測量済み区画集合の外（未測量地は対象外）")}))))

(defn check
  "Assess a proposal against `request`/`context`/`proposal` and a
  `store` implementing `earthmoving.store/Store`. Pure — never mutates
  the store, never dispatches the robot."
  [request context proposal store]
  (let [client-record (store/client store (:client-id request))
        st (some->> (:site-id proposal) (store/site store))
        hard (hard-violations {:request request :proposal proposal}
                              client-record st)
        hard? (boolean (seq hard))
        conf (or (:confidence proposal) 0.0)
        low? (< conf confidence-floor)
        always-risky? (contains? always-escalate-ops (:op proposal))]
    {:ok? (and (not hard?) (not low?) (not always-risky?))
     :violations hard
     :confidence conf
     :hard? hard?
     :escalate? (and (not hard?) (or low? always-risky?))}))
