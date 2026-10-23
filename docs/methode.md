# Note méthodologique

Ce document décrit les choix, les traitements, les contrôles et les limites du projet. Le code correspondant se trouve
dans [`notebooks/traitement.ipynb`](../notebooks/traitement.ipynb).

## 1. Question et cadrage

**Question** : depuis l'Île-de-France, qu'est-il possible d'atteindre en 3h30 passées à bord d'un mode de transport ?

Le raisonnement en **budget-temps** (une durée fixe « dépensée » à se déplacer) s'inscrit dans la géographie du temps
développée par Torsten Hägerstrand. Pour comparer cinq modes, une règle commune est nécessaire :

| Choix | Valeur | Justification |
|---|---|---|
| Temps compté | temps à bord uniquement | traduction directe de la question ; règle identique pour tous les modes |
| Budget | 210 minutes | durée de l'attente à l'origine du projet |
| Jour | mardi 6 octobre 2026 | jour ouvré ordinaire, hors vacances scolaires, avant le changement de service de décembre |
| Trajet retenu | le plus rapide de la journée | « atteignable » s'entend comme possible au moins une fois dans la journée |
| Destinations | hors Île-de-France (train, voiture, avion) | l'Île-de-France est le point de départ ; le GTFS SNCF n'y couvre pas le réseau Transilien et RER |

**Conséquence assumée** : le temps à bord avantage l'avion, puisque l'accès à l'aéroport, l'enregistrement et la sûreté
(souvent plus de deux heures) ne sont pas comptés. En temps porte-à-porte, le train repasserait devant l'avion pour
Lyon, Marseille, Nantes ou Bordeaux.

## 2. Train

**Données** : jeu « Horaires SNCF » (GTFS, TGV, Intercités, TER opérés par SNCF Voyageurs), flux couvrant le
26/09/2026 au 28/02/2027.

**Origine** : liste explicite de 12 zones d'arrêt (`StopArea`), correspondant aux 7 terminus parisiens (Montparnasse
compte deux zones) et aux gares TGV d'interconnexion de Roissy-CDG 2, Marne-la-Vallée–Chessy et du pôle de Massy
(Massy TGV et Massy-Palaiseau). Une liste d'identifiants a été préférée à un filtre par nom, qui manquait la gare de
Roissy (« Aéroport Charles de Gaulle 2 TGV ») et attrapait des homonymes (Montauban Prax Paris, Chessy dans le Rhône).

**Traitement**

1. Circulations actives le jour de référence (`calendar_dates.txt`, `exception_type = 1`), lignes de bus exclues.
2. **Exclusion des circulations en car.** Des lignes codées comme ferroviaires (`route_type = 2`) comportent des
   circulations assurées par car. Le mode a donc été lu au niveau de la circulation, à partir du produit inscrit dans
   l'identifiant du point d'arrêt (« Car TER », « Car à réservation »). 961 circulations ont été retirées, sans qu'aucune
   circulation mixte train-car ne soit concernée.
3. Pour chaque train, croisement des montées possibles en gare d'origine (`pickup_type ≠ 1`) avec les descentes
   possibles en aval (`drop_off_type ≠ 1`, `stop_sequence` supérieur).
4. Durée = arrivée − départ, calculée en secondes pour gérer les horaires au-delà de minuit (`25:30:00`).
5. Durée minimale de la journée par gare, puis filtre à 210 minutes : 334 gares.
6. Rattachement aux communes ADMIN EXPRESS COG par jointure spatiale (Lambert-93), exclusion des 18 gares
   franciliennes, agrégation par commune (Lille-Europe et Lille-Flandres deviennent Lille) : **312 villes**.
7. Les 15 gares étrangères (Belgique, Luxembourg, Allemagne, Suisse) reçoivent un nom d'usage et une population
   saisis dans `data/villes_etrangeres.csv`.

**Contrôles** : confrontation à des durées commerciales connues (Lille 55 min, Strasbourg 1h45, Bordeaux 1h57,
Marseille 3h03) ; vérification des villes attendues juste au-delà du budget avant filtrage ; égalité des effectifs
avant et après la jointure spatiale ; absence de gare française hors commune.

## 3. Voiture

**Contrainte** : l'API publique d'openrouteservice limite les isochrones voiture à 60 minutes.

**Contournement** : grille et matrice de temps.

1. Grille régulière au pas de 10 km dans un disque de 450 km autour de la gare du Nord, en ETRS89-LAEA (EPSG:3035).
2. Masque terrestre par les contours de pays GISCO ; exclusion du Royaume-Uni, de Jersey et de Guernesey,
   inaccessibles sans traversée maritime : **4 746 points**.
3. Temps de trajet depuis la gare du Nord vers chaque point par l'endpoint *matrix* (profil `driving-car`), par lots de 1 000.
4. Retrait des îles d'Yeu et Belle-Île, que l'endpoint *matrix* rattache par ferry sans possibilité d'exclusion.
5. Imputation des 5 points non routables par la médiane des voisins dans un rayon de 15 km (tous au-delà de 3h30, sans effet sur le résultat).
6. Chaque point devient une cellule carrée de 10 km ; les cellules sous 60, 120, 180 et 210 minutes sont fusionnées
   puis découpées par le trait de côte.

**Contrôles** : test préalable sur des trajets connus (Lille, Lyon, Bruxelles) ; aucun point de la couronne extérieure
du disque sous le budget, donc isochrone non tronquée ; superficies cohérentes avec le nombre de cellules.

**Résultat** : 6 600 km² à 1h, 49 991 km² à 2h, 138 474 km² à 3h, **192 809 km² à 3h30**.

**Limites** : trafic fluide (pas de congestion) ; vitesses du profil openrouteservice plus prudentes que celles des
calculateurs commerciaux ; contour à la résolution de 10 km ; quelques enclaves dues au réseau, conservées
volontairement car elles traduisent une réalité (une cellule proche d'un échangeur est plus accessible que sa voisine).

## 4. Avion

**Contrainte** : aucune base ouverte ne fournit d'horaires de vol exhaustifs.

1. **Aéroports candidats** : OurAirports, aéroports moyens et grands à service régulier, à moins de 2 500 km de
   Roissy ou d'Orly (514 aéroports). Le fichier est lu avec `keep_default_na=False` : le code « NA » (Namibie) serait
   sinon interprété comme une valeur manquante.
2. **Liaisons directes** : tableaux « Airlines and destinations » des pages Wikipédia de Roissy-CDG et d'Orly, copie
   datée du 8 octobre 2026. Vols charters exclus, vols saisonniers conservés (le 6 octobre appartient à la saison IATA
   d'été), lignes ouvertes après ou fermées avant le jour de référence retirées.
3. **Rapprochement** des noms Wikipédia et OurAirports par mot entier, accents et casse neutralisés, complété par une
   table manuelle de 11 correspondances (noms fusionnés, orthographes divergentes, Istanbul). **275 liaisons vers 184 aéroports.**
4. **Modèle de durée** calibré par moindres carrés sur six vols réels (Google Flights) :
   **durée = 38 min + distance orthodromique / 758 km/h**. Les résidus atteignent ±15 minutes aux longues distances.
5. Les durées relevées remplacent le modèle quand elles existent (Athènes, Istanbul). Les destinations modélisées entre
   195 et 225 minutes sont marquées `limite = vrai`.

**Résultat** : **165 destinations**, dont 8 incertaines (Chișinău, Skiathos, Varna, Kalamata, Bourgas, Marrakech,
Ouarzazate, Essaouira).

**Limites** : durée bloc estimée et non relevée ; liste des liaisons dépendante de Wikipédia ; liaisons d'hiver
possibles parmi les destinations saisonnières (contrôle effectué sur les principales stations de montagne).

## 5. Marche et vélo

Isochrones openrouteservice (profils `foot-walking` et `cycling-regular`) à 60, 120, 180 et 210 minutes, autorisées
par l'API jusqu'à 20 h à pied et 5 h à vélo. Contrairement à la voiture, les contours sont produits directement par
l'algorithme d'isochrones et non par une grille.

| Mode | 1h | 3h30 | Rapport |
|---|---|---|---|
| Marche | 67 km² | 822 km² | 12,3 |
| Vélo | 786 km² | 9 505 km² | 12,1 |
| Voiture | 6 600 km² | 192 809 km² | 29,2 |

À vitesse constante, la surface croît comme le carré du temps (rapport attendu : 3,5² ≈ 12,25). La marche et le vélo
suivent cette loi ; la voiture s'en écarte nettement, signature d'un réseau hiérarchisé. À vélo, la zone de 3h30
contient 191 communes hors Île-de-France, toutes dans l'Oise (point représentatif de chaque commune).

## 6. Comparaison des modes

Pour les 312 villes du train, la durée voiture est celle de la cellule de grille la plus proche (7,1 km au plus), la
durée avion celle de l'aéroport le plus proche (30 km au plus). Le train est le plus rapide pour 283 villes, l'avion
pour 19, la voiture pour 10. Les durées voiture manquantes concernent des villes hors du disque de 450 km, donc
nettement au-delà de 3h30.

## 7. Représentation

- Isochrones converties en **anneaux** disjoints pour éviter la superposition des aplats semi-transparents.
- Classes de durée identiques pour tous les modes (0–60, 60–120, 120–180, 180–210 min) ; une teinte séquentielle par mode.
- Points du train dimensionnés selon la population (moins de 20 000, 20 000 à 100 000, plus de 100 000 habitants).
- Pas de carte de chaleur pour le train et l'avion : elle représenterait la densité des gares et suggérerait une
  accessibilité continue entre elles, contraire au caractère discontinu de ces réseaux.

## 8. Versions des données

| Source | Version ou date |
|---|---|
| GTFS Horaires SNCF | flux téléchargé fin septembre 2026 (période du 26/09/2026 au 28/02/2027) |
| ADMIN EXPRESS COG | version 4.0 (GeoPackage, Lambert-93), populations légales millésimées 2023 |
| GISCO pays | CNTR_RG_03M_2020_3035 |
| OurAirports | téléchargement d'octobre 2026 |
| Wikipédia | pages consultées le 8 octobre 2026 |
| openrouteservice | calculs réalisés en octobre 2026 |
