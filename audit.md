# Audit Banque Eckmühl — Pipeline scoring crédit conso

> Document à rendre au DPO Klaus Eichmann. **Max 2 pages.**
> Public cible : DPO non-technicien — vocabulaire métier, pas de jargon
> scikit-learn.

## 1. Verdict qualité

1. **Valeurs extrêmes sur `credit_amount` (7,2 % selon la règle IQR)**.
  Impact métier : une petite partie des dossiers porte des montants très
  supérieurs au reste de la population. Action mise en place : scaling
  standard sur les variables numériques pour limiter les effets d'échelle.

2. **Complément `customer_segment` incomplet (3,7 % de manquants)** dans
  `german_credit_supplement.csv`. Action mise en place : imputation par la
  modalité la plus fréquente dans le flux de préparation.

3. **Variable `personal_status_sex` composite** (genre + statut civil).
  Problème qualité de schéma : une seule colonne agrège deux dimensions
  métier différentes, ce qui réduit la lisibilité des analyses.

4. **Déséquilibre de la cible** : `good_credit` 70 % vs `bad_credit` 30 %.
  Risque : un suivi de performance global peut masquer les erreurs sur les
  dossiers à risque (`bad_credit`).

## 2. Verdict éthique

1. **Alerte DI sur `foreign_worker`** : DI = **1,2877** (hors intervalle
  [0,8 ; 1,25]). Signal : l'écart de taux `good_credit` entre groupes est
  au-dessus du seuil de vigilance (règle des 4/5).

2. **Signal proche du seuil sur `personal_status_sex`** : DI = **0,8179**.
  Le ratio reste dans l'intervalle, mais proche de 0,8. Interprétation
  limitée car la variable est composite.

3. **Risque proxy socio-économique sur `customer_segment`** (`basic`/
  `plus`/`premium`/`private`). Même sans variable de revenu explicite,
  ce segment peut capturer une information de richesse et produire un
  traitement indirectement discriminant.

## 3. Recommandations

> Liste courte d'actions concrètes pour Eckmühl (3-5 items).

1. **Collecte de données** : séparer `personal_status_sex` en deux champs
  distincts (genre, statut civil) pour permettre un contrôle de biais fiable.
2. **Gouvernance éthique** : valider avec le DPO l'usage de `customer_segment`
  en scoring crédit et documenter explicitement les usages interdits.
3. **Monitoring** : suivre à chaque version les DI par groupe sensible avec
  seuils d'alerte [0,8 ; 1,25].
4. **Performance métier** : reporter systématiquement les métriques par classe
  (`good_credit`/`bad_credit`) pour éviter les conclusions trompeuses dues au
  déséquilibre 70/30.
5. **Qualité de collecte** : fiabiliser le flux du complément segment
  commercial pour supprimer les manquants en amont.

## 4. Limites de cet audit

> Ce que tu n'as **pas** fait, et qu'il faudrait faire plus tard.

- Pas de mitigation des biais à ce stade (cf. modules M7-M8 du parcours).
- Pas de test de robustesse sur dataset d'évaluation séparé (déjà discuté
  avec Karim).
- Pas d'analyse causale : les signaux observés sont des corrélations, pas des
  preuves de causalité.

---

*Audit produit par Théo, 16/06/2026, dans le cadre du brief M2-B1 ATOS.*