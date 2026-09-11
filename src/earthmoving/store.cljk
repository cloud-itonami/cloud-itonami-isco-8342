(ns earthmoving.store
  "SSoT for the ISCO-08 8342 independent earthmoving operations actor
  (itonami actor pattern, ADR-2607011000 / CLAUDE.md Actors section;
  README's 'Robotics premise' — a survey and grade-checking robot
  performs site surveying and grade-verification under this
  advisor/governor pair, which never dispatches hardware itself).
  Modeled on cloud-itonami-isco-4311's bookkeeping.store.

  Domain:

    client — a registered organization (:client-id, :name)
    site   — a registered work site {:site-id :client-id :name
             :max-cleared-depth-m number :cleared-zones #{zone-str}}.
             `:max-cleared-depth-m` is the registered ceiling a
             proposed excavation's depth must not exceed — digging
             past the utility-cleared depth is a strike risk, not a
             scheduling choice; `:cleared-zones` is the registered
             set a proposed excavation's zone must be a member of —
             excavation is only permitted in surveyed, cleared zones,
             unlocated ground is not fair game.
    record — a committed operating record (approved excavation) —
             written ONLY via commit-record!.
    ledger — append-only audit trail, commit or hold."
  )

(defprotocol Store
  (client [s client-id])
  (site [s site-id])
  (records-of [s client-id])
  (ledger [s])
  (register-client! [s client])
  (register-site! [s st])
  (commit-record! [s record])
  (append-ledger! [s fact]))

(defrecord MemStore [a]
  Store
  (client [_ client-id] (get-in @a [:clients client-id]))
  (site [_ site-id] (get-in @a [:sites site-id]))
  (records-of [_ client-id] (filter #(= client-id (:client-id %)) (:records @a)))
  (ledger [_] (:ledger @a))
  (register-client! [s client]
    (swap! a assoc-in [:clients (:client-id client)] client) s)
  (register-site! [s st]
    (swap! a assoc-in [:sites (:site-id st)] st) s)
  (commit-record! [s record]
    (swap! a update :records (fnil conj []) record) s)
  (append-ledger! [s fact]
    (swap! a update :ledger (fnil conj []) fact) s))

(defn mem-store
  ([] (mem-store {}))
  ([seed] (->MemStore (atom (merge {:clients {} :sites {} :records [] :ledger []}
                                   seed)))))
