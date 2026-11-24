# Stratégies de fetch et de cascade — AutoLoc

Toutes les associations sont en **LAZY** : les données liées ne sont chargées que lorsqu'on y accède, ce qui évite de charger des chaînes d'objets inutiles. Le défaut JPA est EAGER pour @ManyToOne et @OneToOne, c'est pourquoi LAZY est précisé explicitement.

| Association | Mapping | Côté propriétaire (clé étrangère) | Fetch | Cascade | Justification |
|---|---|---|---|---|---|
| Contrat → Paiement | @OneToMany / @ManyToOne | Paiement (`contrat_id_contrat`) | LAZY | ALL + orphanRemoval | Un paiement n'existe pas sans son contrat : il est enregistré et supprimé avec lui. |
| Agence → Véhicule | @OneToMany / @ManyToOne | Vehicule (`agence_id_agence`) | LAZY | Aucun | Les véhicules survivent à la suppression d'une agence. |
| Agence → Employé | @OneToMany / @ManyToOne | Employe (`agence_id_agence`) | LAZY | Aucun | Un employé a une existence propre, indépendante de l'agence. |
| Véhicule ↔ Équipement | @ManyToMany | Vehicule (table `vehicule_equipement`) | LAZY | Aucun | Les équipements sont partagés entre plusieurs véhicules : supprimer l'un ne doit pas supprimer l'autre. Un Set évite les doublons. |
| Client → Réservation | @OneToMany / @ManyToOne | Reservation (`client_id_client`) | LAZY | PERSIST | Enregistrer un nouveau client enregistre ses réservations, mais supprimer un client ne supprime pas l'historique. |
| Réservation → Véhicule | @ManyToOne | Reservation (`vehicule_id_vehicule`) | LAZY | Aucun | Un véhicule est réservé plusieurs fois et existe indépendamment des réservations. |
| Réservation ↔ Contrat | @OneToOne | Contrat (`reservation_id_reservation`) | LAZY | ALL (côté Réservation) | Le contrat découle de la réservation et n'a pas de vie propre sans elle. |
| Véhicule → Maintenance | @OneToMany / @ManyToOne | Maintenance (`vehicule_id_vehicule`) | LAZY | PERSIST | Les maintenances sont créées avec le véhicule, mais l'historique est conservé. |

## Remarques

- `mappedBy` est utilisé sur tous les côtés inverses pour éviter des tables de jointure inutiles.
- Lombok `@Data` n'est pas utilisé : seuls `@Getter` et `@Setter` évitent les boucles infinies dans `toString`, `equals` et `hashCode` sur les associations bidirectionnelles.
- Quand des données liées sont nécessaires hors transaction, il faudra utiliser une requête avec jointure explicite ou un DTO, pour éviter une `LazyInitializationException`.