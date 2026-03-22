# Rapport de modélisation — Gestion de Centres de Congrès

**Université de Rennes — ISTIC | Module BMO | Contrôle Continu**

---

## Contexte

Ce projet modélise un système de gestion de centres de congrès permettant à un gestionnaire de gérer des réservations de salles, du matériel et des prestations, ainsi que les disponibilités et les tarifs. La modélisation suit une approche UML complète, du cahier des charges jusqu'à la génération d'une application web via **BESSER WME**.

---

## Outils utilisés

| Diagramme | Outil |
|---|---|
| Diagrammes de cas d'utilisations | draw.io |
| Diagramme de classes | BESSER WME |
| Diagramme d'objets | draw.io |
| Diagramme d'états | draw.io |
| Diagrammes de séquence | draw.io / StarUML |

---

## Diagramme de classes

Le modèle comprend **10 classes** et **3 énumérations**.

**Classes principales :**

| Classe | Rôle |
|---|---|
| `Gestionnaire` | Acteur central, orchestre toutes les opérations |
| `CentreCongres` | Regroupe les éléments réservables |
| `Element` | Salle ou espace (amphi, salle de réunion…) |
| `Reservation` | Réservation d'un élément pour un événement |
| `Evenement` | Manifestation liée à une réservation |
| `Supplement` *(abstract)* | Complément optionnel (matériel ou prestation) |
| `Materiel` | Équipement réservable (vidéoprojecteur…) |
| `Prestation` | Service optionnel (pause café, lunch…) |
| `Tarif` | Prix d'un élément selon la saison |
| `Indisponibilite` | Période de fermeture d'un élément |

NB: Nous voulions que la classe Supplement soit une interface , cependant, Besser ne possede pas cela.

**Énumérations :** `ReservationStatut` (EN_ATTENTE, CONFIRME, ANNULEE), `JourSemaine`, `Saison` (HAUTE, MOYENNE, BASSE).

### Difficultés rencontrées avec BESSER WME

- **Types de retour liste non supportés** : BESSER n'accepte pas les types de retour de la forme `List<Element>` ou `Element[]`. Les méthodes retournant une collection ont dû être simplifiées en retournant le type de base (`Element`, `Reservation`…).
- **Diagramme d'objets limité à un seul champ attribut par objet** : l'éditeur d'objets de BESSER ne permet de renseigner qu'un seul attribut par instance. Il n'a donc pas été possible de détailler correctement les objets. Les valeurs complètes sont documentées dans le diagramme d'objets réalisé séparément sur draw.io
- **Le nom des relations sur le diagramme de classe n'est pas visible**. Cependant, il est bien présent dans BESSER en configurant l'affichage. Le problème : lorsqu'on active cette option, la mise en forme du diagramme est dégradée.

---

## Diagramme d'objets

Trois scénarios illustrent le modèle avec des valeurs concrètes :

**Scénario 1 — Réservation en attente de paiement**
- `centre1 : CentreCongres` → `salle1 : Element` (Grand Amphi, 300 places)
- `event1 : Evenement` (Congrès IA 2026, 200 participants)
- `res1 : Reservation` (statut = EN_ATTENTE, limite = 2026-03-25)

**Scénario 2 — Réservation confirmée avec suppléments**
- `salle2 : Element` (Salle Pasteur, 80 places) + `event2 : Evenement` (Séminaire RH)
- `res2 : Reservation` (statut = CONFIRME)
- `videoProj : Materiel` (50€/unité) + `pauseCafe : Prestation` (8€/pers.)

**Scénario 3 — Indisponibilité d'un élément**
- `salle1 : Element` liée à `ind1 : Indisponibilite` (travaux du 01/06 au 18/06/2026)

---

## Diagramme d'états — classe `Reservation`

Le cycle de vie d'une réservation suit exactement les 3 valeurs de l'énumération `ReservationStatut` :


Points clés :
- La **modification** est une transition réflexive sur l'état CONFIRME (l'état ne change pas).
- Une réservation ANNULEE est **conservée en base** pour les statistiques ; la suppression physique est une action distincte.
- Le **paiement est externe** à la plateforme : la confirmation arrive de l'extérieur.

---

## Diagrammes de séquence

Les cas d'utilisation ont été modélisés, respectant les normes UML 2.x (fragments `alt`, `opt`, messages de retour en pointillés, lifelines nommées `instance : Classe`).

| Cas d'utilisation | Points notables |
|---|---|
| `CreationReservation` | `alt` disponible/indisponible + `opt` ajout supplément |
| `ConfirmationReservation` | `alt` succès/échec — paiement externe signalé par note UML |
| `ModifierReservation` | `alt` dispo / indispo / modification impossible (événement commencé) |
| `AnnulerReservation` | `alt` annulable / événement commencé / déjà annulée |
| `SupprimerReservation` | Guard `statut=ANNULEE`, notation `«destroy»` sur la lifeline |
| `DefinirIndisponibiliteElement` | Vérification de conflit avec les réservations existantes avant création |
| `AjouterSupplement` | `alt` Materiel (`estDisponible()`) / Prestation (`calculerCout()`) + recalcul `calculerCoutTotal()` |
| `GestionElementsCentre` | Suppression et ajout d'un élement dans un centre |
| `ConsultationStatistiques` | Génération des statistiques avec CentreCongres(`getStats()`) |
| `GestionTarifElement` | Modification du tarif d'un élément |



---

## Application générée

L'application web a été générée via `Generate → Web Application` dans BESSER WME et déployée avec Docker. Elle expose une interface permettant de créer et visualiser des instances de chaque classe du modèle.

Les instructions de lancement sont disponibles dans [README_BESSER.md](./README_BESSER.md).
