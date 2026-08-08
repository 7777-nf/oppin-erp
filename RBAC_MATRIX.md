# RBAC_MATRIX.md — CHARLE / OPPIN

**Statut : Référence officielle.** Ce document est la source de vérité pour tous les contrôles d'accès de CHARLE. Les règles Firestore, et tout code applicatif futur, doivent implémenter cette matrice — jamais l'inverse. Toute évolution des permissions passe d'abord par une mise à jour de ce document, validée par le CEO, avant toute modification des règles.

Origine : matrice validée dans *CHARLE — Architecture Sécurité & Exploitation, Phase 2.5* (section 3), reprise ici sans modification comme référence exécutable du Sprint 2.

## Rôles

| Rôle | Définition |
|---|---|
| Super Admin | Administration technique du système (comptes, configuration, secrets) — usage réservé au responsable technique (Ulrich), jamais utilisé pour piloter le business. |
| CEO | Accès complet à toutes les données de toutes les filiales, seul habilité à valider investissements et décisions RH critiques, seul à administrer les rôles des autres utilisateurs. |
| Directeur | Responsable d'une filiale (rôle prévu pour la croissance future de TROPRAPIDE/BATE) : accès complet à sa filiale, lecture consolidée limitée sur le reste du groupe. |
| Commercial | Gestion du CRM, des devis et des contrats de sa filiale ; aucun accès à la trésorerie ni à la comptabilité. |
| Technicien | Rôle actuel de Kodjo : création de devis uniquement, lecture du stock nécessaire à son intervention. |
| SAV | Gestion des interventions et de la maintenance ; lecture des contrats clients concernés ; aucun accès financier. |
| Comptable | Écritures comptables, factures, obligations fiscales ; lecture de la trésorerie ; aucun accès au CRM ou aux devis commerciaux. |
| Magasin | Gestion du stock et des inventaires, lecture des fournisseurs et des achats ; aucun accès financier ou commercial. |
| Lecture seule | Accès en lecture à l'ensemble des tableaux de bord, aucune écriture possible nulle part — destiné à un investisseur, un expert-comptable externe ou un consultant. |

**Affectation actuelle (08/2026)** : seuls **CEO** (Fabrice N'Goran) et **Technicien** (Kodjo) ont un compte réel. Les 7 autres rôles existent dans le modèle et les règles, prêts à être attribués sans nouveau développement.

### Valeur technique du champ `role`

Table de correspondance entre le rôle métier ci-dessus et la valeur exacte stockée dans `utilisateurs/{uid}.role` — c'est cette valeur que les règles Firestore et le code lisent. Toute création de compte doit utiliser exactement cette valeur.

| Rôle | Valeur technique (`role`) | Statut |
|---|---|---|
| Super Admin | `superadmin` | Réservé (non attribué) |
| CEO | `ceo` | **Actif** — Fabrice N'Goran |
| Directeur | `directeur` | Réservé (non attribué) |
| Commercial | `commercial` | Réservé (non attribué) |
| Technicien | `tech` | **Actif** — Kodjo |
| SAV | `sav` | Réservé (non attribué) |
| Comptable | `comptable` | Réservé (non attribué) |
| Magasin | `magasin` | Réservé (non attribué) |
| Lecture seule | `readonly` | Réservé (non attribué) |

## Matrice des permissions par domaine

**Statut : figée, version définitive du modèle d'autorisation de CHARLE (validée par le CEO le 08/08/2026).** Toute évolution future passe par une mise à jour explicite de ce document avant toute modification des règles Firestore.

Légende : `—` Aucun · `L` Lecture · `E` Écriture · `V` Validation (décision finale/seuils) · `A` Administration (configuration du domaine lui-même).

| Domaine | Super Admin | CEO | Directeur | Commercial | Technicien | SAV | Comptable | Magasin | Lecture seule |
|---|---|---|---|---|---|---|---|---|---|
| Trésorerie | A | V | L | — | — | — | L | — | L |
| CRM / Commercial | A | V | E | E | — | — | — | — | L |
| Devis | A | V | E | E | E | — | — | — | L |
| Facturation | A | V | L | L | — | — | E | — | L |
| Recouvrement | A | V | L | L | — | — | E | — | L |
| Comptabilité | A | V | L | — | — | — | E | — | L |
| Fiscalité | A | V | L | — | — | — | E | — | L |
| Investissement | A | V | L | — | — | — | L | — | L |
| **Juridique & Documents** | A | V | L | L¹ | — | — | L | — | — |

¹ Pour Commercial, la lecture sera restreinte aux documents de sa propre filiale une fois le modèle multi-filiale actif (dépend de la restructuration Firestore, hors périmètre actuel — voir Limite technique). En attendant, Commercial a `L` sur l'ensemble du domaine.

Cette matrice reprend directement les règles déjà validées dans la Constitution et le Business Operating Model : investissement reste réservé à la Validation du CEO (« V »), quel que soit le rôle technique accordé par ailleurs.

### Domaines réservés, non implémentés (hors périmètre Sprint 2.2)

| Domaine | Statut |
|---|---|
| Stock / Achats | Aucun champ Firestore ne représente ce domaine aujourd'hui. Des collections racine orphelines (`stock`, `stock-entrees`, `stock-sorties`) existent mais ne sont reliées à aucune logique applicative — confirmées inertes. À réintroduire quand un modèle de données Stock/Achats existera. |
| Maintenance / SAV | Retiré du Sprint 2.2 sur décision du CEO. Ni `contracts` (analyse IA de documents contractuels, tous types confondus — reclassé en Juridique & Documents) ni la collection orpheline `contrats-sav` (jamais référencée dans le code applicatif) ne représentent le suivi opérationnel des contrats de maintenance INOVON. À réintroduire quand un véritable modèle de données Maintenance/SAV existera. |
| RH (ressources critiques) | Aucun champ Firestore ne représente ce domaine aujourd'hui. |
| Paramètres / rôles | Non géré via `charle/**` — collection `utilisateurs` séparée, déjà verrouillée en écriture pour tous (y compris CEO) depuis le Sprint 1A ; modification de rôle uniquement via la console Firebase. |

## Table de correspondance domaine → champs Firestore réels

Établie par audit direct du code de production (`index.html`) et de la structure réelle du document `charle/oppin-ceo`, pas par hypothèse.

| Domaine | Champ(s) Firestore | Rôles pouvant écrire (E/V/A) |
|---|---|---|
| Trésorerie | `transactions`, `cashSnapshots`, `expenses` | CEO, Super Admin |
| CRM / Commercial | `crmLeads`, `inovonPipeline` | CEO, Directeur, Commercial, Super Admin |
| Devis | `devisHistory` | CEO, Directeur, Commercial, Technicien, Super Admin |
| Facturation | `invoices` | CEO, Comptable, Super Admin |
| Recouvrement | `receivables` | CEO, Comptable, Super Admin |
| Comptabilité | `pnlTransactions` | CEO, Comptable, Super Admin |
| Fiscalité | `fiscalEvents` | CEO, Comptable, Super Admin |
| Investissement | `investHistory` | CEO, Super Admin |
| Juridique & Documents | `contracts`, `documents` | CEO, Super Admin |

`pnlTransactions` est rattaché à Comptabilité et non à Trésorerie : le code (`addPNLTransaction()`, module "Agent C — P&L par Business Unit") l'utilise pour construire un compte de résultat par filiale, une fonction comptable, distincte du suivi de trésorerie brut (`transactions`). Décision définitive du 08/08/2026.

`trDeliveries` (livraisons TROPRAPIDE) n'est pas encore mappé à un domaine de la matrice — n'existe pas encore dans le document de production, non couvert par une règle de domaine spécifique. Seul CEO/Super Admin peut y écrire tant qu'il n'est pas explicitement rattaché à un domaine.

## Champs transverses

### Domaine « Système CHARLE »

Regroupe `briefingHistory`, `chat`, ainsi que `paramètres IA` et `mémoire IA` **(réservés — ces deux éléments n'existent pas encore comme champs dans CHARLE ; aucun champ fictif n'a été créé, à cadrer lors de leur implémentation réelle)**.

Écriture : autorisée à tout rôle valide non-Lecture-seule. Justification : CHARLE ne dispose d'aucun composant serveur (Cloud Function ou backend dédié) capable d'écrire ces champs de façon distincte de l'utilisateur connecté — l'écriture est aujourd'hui toujours réalisée sous l'identité de l'utilisateur actif, via la logique applicative (pas une saisie manuelle). Introduire une distinction « Système » artificielle sans support technique réel serait une fausse garantie. Cette équivalence sera revue si CHARLE se dote un jour de Cloud Functions ou d'un backend dédié.

Lecture : autorisée à tout rôle valide (inchangé depuis le Sprint 2.1).

### `alerts`

Reste rattaché conceptuellement à ses domaines métier d'origine (trésorerie, créances, risques, investissements, etc.), **pas** au domaine « Système CHARLE ». Écriture : même traitement pragmatique que ci-dessus (tout rôle valide non-Lecture-seule, pas de distinction système/utilisateur). Lecture : voir limite technique ci-dessous — non filtrable par domaine au niveau des règles Firestore.

## Limite technique connue

### Lecture par domaine et par élément — non applicable au niveau des règles Firestore

Les données métier vivent dans un document Firestore unique (`charle/oppin-ceo`). Deux conséquences, aucune levée par le Sprint 2.2 :

1. **Lecture par domaine** : Firestore ne permet pas de restreindre la lecture à l'intérieur d'un même document — un rôle autorisé à lire une partie du document peut techniquement lire l'ensemble. La lecture reste donc accordée à tout utilisateur authentifié disposant d'un rôle valide, plutôt que filtrée domaine par domaine comme le prévoit la colonne `L` de la matrice.
2. **Filtrage par contenu à l'intérieur d'un tableau** (`alerts` par domaine d'origine, `chat` par utilisateur) : Firestore ne peut pas non plus filtrer le contenu d'un tableau élément par élément — un rôle autorisé à lire le document reçoit `alerts` et `chat` **en entier**. **Décision officielle du CEO (08/08/2026) : les règles Firestore protègent l'accès au document dans son ensemble ; le filtrage fin des alertes par domaine et du chat par utilisateur est assuré uniquement par l'application (couche d'affichage), ce n'est pas une garantie de sécurité au niveau Firestore.** Cette limitation est documentée et assumée pour cette version, plutôt que de créer une fausse impression de sécurité. Elle sera levée par une future restructuration (sortir `alerts` et `chat` du document partagé), explicitement hors périmètre du Sprint 2.2.

### Écriture par domaine — résolue au Sprint 2.2 sans modifier `pushToFirestore()`

`pushToFirestore()` continue d'envoyer l'intégralité de l'objet `DB` à chaque sauvegarde, **sans aucune modification de code applicatif**. Les règles Firestore utilisent `request.resource.data.diff(resource.data).affectedKeys()` : Firestore calcule côté serveur, à partir des valeurs avant/après, quels champs ont réellement changé — même si le payload envoyé contient tous les champs, seuls ceux dont la valeur diffère apparaissent comme modifiés. Feasibility prouvée par démonstration sur émulateur avec la structure réelle du document (voir rapport Sprint 2.2). Option retenue après étude comparative (Option A : règles seules) — la modification de `pushToFirestore()` (Option B, envoi différentiel) et la passerelle serveur (Option C, Cloud Functions) ont été explicitement écartées comme inutilement complexes pour le besoin actuel.

**Séquencement retenu par le CEO** :

- **Sprint 2 (base)** : accès `charle/**` réservé aux comptes ayant un rôle assigné et reconnu.
- **Sprint 2.1** : reconnaissance de rôle, restriction réelle pour Lecture seule (lecture oui, écriture jamais).
- **Sprint 2.2** : restriction d'écriture par domaine métier, via `diff().affectedKeys()`, sans modification de `pushToFirestore()`. Lecture inchangée depuis le Sprint 2.1 (limite documentée ci-dessus).

Le champ `businessUnit` est ajouté depuis le Sprint 2 sur les nouveaux enregistrements pour préparer l'évolution multi-filiale, sans migration ni régression.

## Journal des modifications

| Version | Date | Modification |
|---|---|---|
| 1.0 | 2026-08-08 | Création — reprise de la matrice Phase 2.5, ajout de la note de limite technique Sprint 2 |
| 1.1 | 2026-08-08 | Ajout table de correspondance rôle → valeur technique `role` ; séquencement Sprint 2 / 2.1 / 2.2 validé par le CEO |
| 2.0 | 2026-08-08 | **Version figée du modèle d'autorisation.** Ajout du domaine Juridique & Documents ; retrait de Maintenance/SAV et Stock/Achats et RH (non implémentés) ; `pnlTransactions` rattaché définitivement à Comptabilité ; table de correspondance domaine → champs Firestore réels ; domaine transverse Système CHARLE défini ; limite de filtrage par élément (`alerts`, `chat`) documentée officiellement ; confirmation que l'écriture par domaine est réalisée via `diff().affectedKeys()` sans modification de `pushToFirestore()` (Option A) |
