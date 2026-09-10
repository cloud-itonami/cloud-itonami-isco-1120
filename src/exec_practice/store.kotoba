(ns exec-practice.store
  "SSoT for the ISCO-08 1120 independent small-business-executive
  sole-proprietor actor, behind a `Store` protocol so the backend is a
  swap (MemStore default ‖ a real Datomic/kotoba-server backend, per the
  itonami actor pattern).

  Domain = independent small-business executive practice:

    engagement          — a client engagement (engagementId, clientId,
                          scope, spendingLimit)
    decision            — a decision event under an engagement
                          (decisionId, engagementId, category
                          #{:hiring :spending :contract})
    spending-approval   — a spending-approval event under an engagement
                          (approvalId, engagementId, amount)

  The append-only records are the operating ledger: a decision or
  spending-approval must reference a registered engagement, and these
  records are never mutated in place, only appended.")

(defprotocol Store
  (engagement [st engagement-id])
  (decisions-of [st engagement-id])
  (spending-approvals-of [st engagement-id])
  (register-engagement! [st engagement])
  (record-decision! [st decision])
  (record-spending-approval! [st spending-approval]))

(defrecord MemStore [state]
  Store
  (engagement [_ engagement-id]
    (get-in @state [:engagements engagement-id]))
  (decisions-of [_ engagement-id]
    (filter #(= engagement-id (:engagement-id %)) (:decisions @state)))
  (spending-approvals-of [_ engagement-id]
    (filter #(= engagement-id (:engagement-id %)) (:spending-approvals @state)))
  (register-engagement! [_ engagement]
    (swap! state assoc-in [:engagements (:engagement-id engagement)] engagement))
  (record-decision! [_ decision]
    (swap! state update :decisions (fnil conj []) decision))
  (record-spending-approval! [_ spending-approval]
    (swap! state update :spending-approvals (fnil conj []) spending-approval)))

(defn mem-store
  ([] (mem-store {}))
  ([seed]
   (->MemStore (atom (merge {:engagements {} :decisions [] :spending-approvals []} seed)))))
