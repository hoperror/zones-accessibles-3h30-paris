# Couches produites

Le notebook écrit l'ensemble des couches de travail dans `resultats/resultats.gpkg` (non versionné), et les couches
finales en WGS84 (EPSG:4326) dans `resultats/geojson/` et `resultats/storymap.gpkg`.

Ces couches sont dérivées de données sous licence ODbL (horaires SNCF, OpenStreetMap via openrouteservice) et sont
publiées sous la même licence : Open Database License (ODbL) 1.0.

| Couche | Géométrie | Entités | Champs principaux |
|---|---|---|---|
| `marche_anneaux` | polygones | 4 | `seuil_min`, `aire_km2`, `classe` |
| `velo_anneaux` | polygones | 4 | `seuil_min`, `aire_km2`, `classe` |
| `voiture_anneaux` | polygones | 4 | `seuil_min`, `aire_km2`, `classe` |
| `train_villes` | points | 312 | `ville`, `population`, `duree_min`, `stop_name` (gare la plus rapide), `parent_station_o` (gare de départ), `produit`, `classe`, `taille` |
| `avion_villes` | points | 165 | `iata`, `origine` (CDG ou ORY), `ville`, `pays`, `dist_km`, `duree_min`, `statut` (relevé ou modélisé), `limite`, `classe` |
| `depart` | point | 1 | `nom` |

Couches de travail supplémentaires dans `resultats.gpkg` : `grille_voiture` (durées voiture par point de grille),
`voiture_isochrones`, `marche_isochrones` et `velo_isochrones` (zones emboîtées), `comparaison_villes`
(durées train, voiture et avion des villes du train, mode le plus rapide).

Remarques de lecture :
- `seuil_min` désigne la borne haute de l'anneau ; `classe` donne l'intervalle (« 60–120 min »).
- Pour l'avion, `ville` est la commune où se trouve l'aéroport selon OurAirports (par exemple Spata-Artemida pour Athènes).
- `limite = true` signale une durée modélisée à ±15 minutes du budget.
