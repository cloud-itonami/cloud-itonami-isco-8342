(ns earthmoving.advisor
  "EarthmovingAdvisor — the advisor named in this repository's
  README, proposing a site operation (approve an excavation, approve
  unlocated-utility excavation, approve occupied-zone entry) from a
  site plan, utility-locate report and grading specification.
  Swappable mock/llm; the advisor ONLY proposes —
  `earthmoving.governor` checks the cleared-depth ceiling and
  cleared-zone membership independently and always escalates
  unlocated-utility/occupied-zone decisions. Modeled on
  cloud-itonami-isco-4311's advisor.

  A proposal: {:op :approve-excavation|:approve-unlocated-utility-excavation|:approve-occupied-zone-entry
               :effect :propose :site-id str :depth-m number
               :zone str :stake kw :confidence n :rationale str}"
  (:require #?(:clj [clojure.edn :as edn] :cljs [cljs.reader :as edn])))

(defprotocol Advisor
  (-advise [advisor store request] "request -> proposal map"))

(defn- infer [_store {:keys [op stake site-id depth-m zone] :as request}]
  {:op op
   :effect :propose
   :site-id site-id
   :depth-m depth-m
   :zone zone
   :stake (or stake :low)
   :confidence (case (or stake :low) :high 0.7 :medium 0.85 :low 0.95)
   :rationale (str "proposed " (name op) " for client " (:client-id request))})

(defn mock-advisor []
  (reify Advisor
    (-advise [_ store request] (infer store request))))

(def ^:private system-prompt
  "You are an earthmoving-operations advisor. Given a request,
   propose an :op, the :site-id, :depth-m and :zone, an honest
   :confidence and a :stake. Never call an over-depth excavation or an
   unlocated-zone excavation conforming — the governor checks both
   against the registered site record. Unlocated-utility and
   occupied-zone decisions always require human sign-off regardless
   of confidence.")

(defn- parse-proposal [content]
  (try
    (let [p (edn/read-string content)]
      (if (map? p)
        (assoc p :effect :propose)
        {:op :unknown :effect :propose :confidence 0.0 :stake :high
         :rationale "unparseable LLM response"}))
    (catch #?(:clj Exception :cljs js/Error) _
      {:op :unknown :effect :propose :confidence 0.0 :stake :high
       :rationale "LLM response parse failure"})))

(defn llm-advisor
  [chat-model model-generate-fn gen-opts]
  (reify Advisor
    (-advise [_ _store request]
      (let [msgs [{:role :system :content system-prompt}
                  {:role :user :content (str "operation request: " (pr-str request))}]
            resp (model-generate-fn chat-model msgs gen-opts)]
        (parse-proposal (:content resp))))))
