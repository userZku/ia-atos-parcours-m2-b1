# Choix techniques du pipeline

Ce document explicite les choix de pretraitement du projet M2-B1, avec leurs avantages, limites et alternatives.

## 1. Objectif

Le pipeline est concu pour standardiser la preparation des donnees de credit avant modelisation:

- comportement deterministe
- pas de valeurs manquantes en sortie
- schema de sortie stable (utile pour les tests contractuels)
- transformation rejouable a l'identique sur de nouveaux lots

## 2. Pourquoi garder toutes les variables (y compris sensibles)

### Decision

A ce stade, les variables sensibles disponibles dans les donnees (`personal_status_sex`, `foreign_worker`) sont conservees dans le pipeline.

### Justification

- Le brief demande un audit ethique quantifie (DI/rule des 4/5), ce qui necessite de mesurer les ecarts entre groupes.
- Conserver ces variables permet un monitoring de biais plus robuste en phase d'analyse.
- Retirer ces colonnes trop tot masque le signal et peut empecher de detecter une discrimination indirecte.

### Limite importante

Conserver une variable sensible dans le pipeline de preparation ne veut pas dire qu'elle doit etre utilisee automatiquement dans un modele de decision final en production.

### Position recommandee

- En phase audit/formation: conserver pour mesurer.
- En phase production: decision explicite avec DPO/juridique (garder, exclure ou restreindre l'usage), avec trace documentaire.

## 3. Choix d'encodage par type de variable

## 3.1 Variables numeriques

Colonnes concernees: `duration_months`, `credit_amount`, `installment_rate_pct_income`, `residence_since_years`, `age`, `n_existing_credits`, `n_people_liable`.

Transformation:

- imputation mediane
- standardisation (`StandardScaler`)

Raison:

- la mediane est robuste aux valeurs extremes
- la standardisation aligne les echelles et facilite les modeles sensibles a la magnitude

## 3.2 Variables ordinales

Colonnes concernees: `savings_account`, `employment_since`, `customer_segment`.

Transformation:

- imputation modalite la plus frequente
- `OrdinalEncoder` avec ordre explicite des categories

Raison:

- ces variables portent une progression metier interpretable
- un encodage ordinal preserve cette relation d'ordre
- plus parcimonieux qu'un one-hot quand l'ordre existe

Point d'attention:

- l'ordre de `customer_segment` est une hypothese metier (`basic < plus < premium < private`) et doit etre revalide si la definition business evolue

## 3.3 Variables categorielle nominales

Colonnes concernees: `checking_account_status`, `credit_history`, `purpose`, `personal_status_sex`, `other_debtors`, `property`, `other_installment_plans`, `housing`, `job`, `telephone`, `foreign_worker`.

Transformation:

- imputation modalite la plus frequente
- `OneHotEncoder(handle_unknown="ignore")`

Raison:

- pas d'ordre naturel entre modalites
- one-hot evite d'introduire une distance artificielle entre categories
- `handle_unknown="ignore"` protege le pipeline face a des categories nouvelles en inference

## 4. Pourquoi pas de target encoding a ce stade

Le target encoding peut etre performant, mais il est volontairement evite ici:

- risque de fuite d'information si mal encadre
- forte sensibilite au protocole de validation
- moins pedagogique pour un pipeline de base auditable

Le couple `OrdinalEncoder` (si ordre) + `OneHotEncoder` (sinon) est plus transparent et simple a verifier.

## 5. Gestion des manquants

Strategie appliquee:

- numerique -> mediane
- ordinal/nominal -> modalite la plus frequente

Raison:

- strategie simple, stable et robuste pour un premier niveau d'industrialisation
- garantit un pipeline sans NaN en sortie
- compatible avec les tests contractuels

## 6. Compromis et risques connus

- **Risque ethique**: `customer_segment` peut agir comme proxy socio-economique.
- **Risque de schema**: `personal_status_sex` melange deux dimensions (genre + statut civil).
- **Risque de simplification**: imputation modale peut lisser des sous-populations minoritaires.

Ces points sont suivis dans l'audit et doivent rester dans la boucle de gouvernance.

## 7. Alternatives envisageables

- exclure certaines variables sensibles en production apres validation DPO
- encodage frequenciel ou regularise pour categories tres cardinales
- imputation conditionnelle (par segment) au lieu d'une imputation globale
- calibration/contraintes de fairness en phase modelisation (M7/M8)
