(ns exec-practice.governor-test
  (:require [clojure.test :refer [deftest is testing]]
            [exec-practice.store :as store]
            [exec-practice.governor :as governor]))

(defn- fresh-store []
  (let [st (store/mem-store)]
    (store/register-engagement! st {:engagement-id "eng-1" :client-id "client-1"
                                      :scope "ops-advisory" :spending-limit 5000})
    st))

(deftest proceeds-on-clean-decision
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :decision :engagement-id "eng-1" :category :hiring
                   :safety-class :low :effect :propose :confidence 0.9}]
    (is (= :proceed (:decision (governor/assess env proposal))))))

(deftest holds-on-unregistered-engagement
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :decision :engagement-id "no-such-engagement" :category :hiring
                   :safety-class :low :effect :propose :confidence 0.9}
        result (governor/assess env proposal)]
    (is (= :hold (:decision result)))
    (is (some #(= :no-engagement (:rule %)) (:violations result)))))

(deftest holds-on-no-actuation-violation
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :decision :engagement-id "eng-1" :category :hiring
                   :safety-class :low :effect :direct-write :confidence 0.9}
        result (governor/assess env proposal)]
    (is (= :hold (:decision result)))
    (is (some #(= :no-actuation (:rule %)) (:violations result)))))

(deftest proceeds-on-spending-approval-under-limit
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :spending-approval :engagement-id "eng-1" :amount 2000
                   :safety-class :low :effect :propose :confidence 0.9}]
    (is (= :proceed (:decision (governor/assess env proposal))))))

(deftest holds-on-over-limit-spending-approval-without-high-safety-class
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :spending-approval :engagement-id "eng-1" :amount 8000
                   :safety-class :medium :effect :propose :confidence 0.9}
        result (governor/assess env proposal)]
    (is (= :hold (:decision result)))
    (is (some #(= :large-spending-safety (:rule %)) (:violations result)))))

(deftest human-approval-on-over-limit-spending-approval-with-high-safety-class
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :spending-approval :engagement-id "eng-1" :amount 8000
                   :safety-class :high :effect :propose :confidence 0.9}]
    (is (= :human-approval (:decision (governor/assess env proposal))))))

(deftest human-approval-on-low-confidence
  (let [st (fresh-store)
        env (governor/env-for-store st)
        proposal {:kind :decision :engagement-id "eng-1" :category :contract
                   :safety-class :none :effect :propose :confidence 0.2}
        result (governor/assess env proposal)]
    (is (= :human-approval (:decision result)))
    (is (= :low-confidence (:reason result)))))

(deftest store-records-append-only
  (let [st (fresh-store)]
    (store/record-decision! st {:decision-id "d1" :engagement-id "eng-1" :category :hiring})
    (store/record-spending-approval! st {:approval-id "a1" :engagement-id "eng-1" :amount 1000})
    (is (= 1 (count (store/decisions-of st "eng-1"))))
    (is (= 1 (count (store/spending-approvals-of st "eng-1"))))))

(deftest a-proposal-without-confidence-does-not-proceed
  (testing "確信度を言っていない提案は、確信していると言っていないので auto-proceed
            させない。この既定は 2026-07-30 まで 1.0 で、:confidence を持たない提案が
            :proceed していた（ADR-2607309100）。fleet の boolean 方言 346 件はすべて
            0.0 既定で、うち isco-5419 はそれを明示的にテストしている。"
    (let [st (fresh-store)
          env (governor/env-for-store st)
          proposal {:kind :decision :engagement-id "eng-1" :category :hiring :safety-class :low :effect :propose}
          result (governor/assess env proposal)]
      (is (= 0.0 (:confidence result))
          "欠落した :confidence は 0.0 であって 1.0 ではない")
      (is (not= :proceed (:decision result))
          "確信度不明の提案が自動で通ってはならない"))))
