# Datasheet — German Credit (version Eckmühl, clean)

> **Modèle Gebru et al. (2018), version simplifiée 7 sections, 1 page max.**
> À transmettre avec le dataset Parquet livré à Eckmühl.
> Public cible : DPO + équipe métier — lisible par un non-data-scientist.

## 1. Motivation

> Pourquoi ce dataset existe-t-il ? Qui l'a créé et pour quel objectif initial ?

- Ce dataset sert a produire un premier niveau d'aide a l'analyse du risque
	credit chez Eckmuhl.
- La base source est le jeu German Credit (1000 dossiers), enrichi localement
	par un complement `customer_segment` pour tester l'adaptation du pipeline.
- Objectif operationnel: standardiser la preparation des donnees avant
	modelisation (qualite, reproductibilite, auditabilite).

## 2. Composition

> Combien d'observations, quelles colonnes, quels types, distribution de la
> cible, **variables sensibles signalées explicitement**.

| Aspect | Valeur |
|---|---|
| Nombre de lignes | 1000 |
| Nombre de colonnes | 22 (dataset clean livré) |
| Cible | `credit_risk` : `good_credit` / `bad_credit` |
| Distribution cible | `good_credit` 70.0 % / `bad_credit` 30.0 % |
| Variables sensibles | `age`, `personal_status_sex` (composite), `foreign_worker` |
| Variable a risque proxy | `customer_segment` (niveau commercial) |
| Valeurs manquantes (clean) | 0 (apres preprocessing) |

**Schéma des colonnes** (types + modalités pour les catégorielles) :

| Colonne | Type | Modalités / range | Note |
|---|---|---|---|
| `duration_months` | numerique | 4 - 72 | Duree du credit |
| `credit_amount` | numerique | 250 - 18424 | Montant, queue a droite |
| `age` | numerique | 19 - 75 | Variable sensible potentielle |
| `savings_account` | categorielle ordonnee | `<100`, `100-500`, `500-1000`, `>=1000`, `unknown` | Ordre metier explicite |
| `employment_since` | categorielle ordonnee | `unemployed` a `>=7 years` | Ordre metier explicite |
| `customer_segment` | categorielle ordonnee | `basic`, `plus`, `premium`, `private` | Risque proxy socio-eco |
| `personal_status_sex` | categorielle nominale | 4 modalites | Variable composite (genre + statut civil) |
| `foreign_worker` | categorielle nominale | `yes` / `no` | Variable sensible potentielle |
| `credit_risk` | cible binaire | `good_credit` / `bad_credit` | Cible de prediction |

Resume des familles de variables (hors cible):
- 7 numeriques
- 3 categorielle ordonnees
- 11 categorielle nominales

## 3. Processus de collecte

> Connu / inconnu ? Période de collecte ? Quelles personnes sont
> représentées ? Quel biais de sélection probable ?

- La collecte historique detaillee (periode, protocole exact, representativite)
	n'est pas entierement documentee dans ce brief.
- Le complement `customer_segment` est fourni separement et joint par position
	(meme ordre de lignes, 1000/1000).
- Biais probables: biais temporel (donnees anciennes), biais geographique
	(contexte original non France), biais socio-economique.

## 4. Préprocessing appliqué (TOI)

> Ce que tu as fait dans ton pipeline. Concis, listé.

- Imputation des manquants:
	- numeriques -> mediane
	- categorielles (ordinales + nominales) -> modalite la plus frequente
- Encodage:
	- ordinales -> `OrdinalEncoder` (ordre defini explicitement)
	- nominales -> `OneHotEncoder(handle_unknown="ignore")`
- Normalisation des numeriques -> `StandardScaler`
- Variables exclues: aucune parmi les 21 features metier disponibles a ce
	stade (objectif: garder la capacite de diagnostic qualite/ethique).
- Sortie pipeline: 54 colonnes transformees (schema contractuel teste).

## 5. Usages prévus / à éviter

> Pour quoi ce dataset doit-il être utilisé (et pour quoi il ne **doit pas**
> l'être).

**Usages prévus** :
- exploration et comparaison de profils de dossiers de credit
- entrainement/validation de prototypes de scoring en environnement controle
- audit qualite et audit ethique (suivi de DI par sous-groupes)

**Usages à éviter** :
- refus de credit 100 % automatise sans revue humaine
- usage hors contexte credit sans re-validation metier/juridique
- decisions individuelles basees directement sur variables sensibles
	(ou proxies) sans validation DPO

## 6. Distribution

> À qui ce dataset est-il transmis ? Sous quelle forme ?

- Transmis a : equipe data/risque Eckmuhl + DPO (perimetre projet)
- Format : Parquet compresse (fichier `data/german_credit_clean.parquet`)
- Volume indicatif : ~28.0 KB (vs CSV brut ~207.6 KB, ratio ~7.4x)
- Conditions : usage interne projet, pas de republication externe sans revue
	conformite.

## 7. Maintenance

> Qui maintient ce dataset ? Comment signaler un problème ? Quelle version ?

- Mainteneur·euse : Théo
- Version : v1.0.0 - 2026-06-16
- Contact issue : ouvrir une issue GitHub sur le repository du projet
- Politique de mise a jour : toute evolution de schema doit mettre a jour
	`src/preprocess.py`, `contract_test.py`, et cette datasheet.

---

*Datasheet produite par Theo, 2026-06-16, dans le cadre du brief M2-B1 ATOS.*